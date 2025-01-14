iOS Quickstart
==============

`HelloWorld <https://github.com/pytorch/ios-demo-app/tree/master/HelloWorld>`__ for IOS is a simple image classification application that
demonstrates how to use PyTorch C++ libraries on iOS. The code is
written in Swift and uses Objective-C as a bridge.

Model Preparation
~~~~~~~~~~~~~~~~~

Let’s start with model preparation. If you are familiar with PyTorch,
you probably should already know how to train and save your model. In
case you don’t, we are going to use a pre-trained image classification
model, `MobileNet
v2 <https://pytorch.org/hub/pytorch_vision_mobilenet_v2/>`__, which is
already packaged in
`TorchVision <https://pytorch.org/docs/stable/vision/index.html>`__.
To install it, run the command below.

.. code:: shell

   pip install torchvision

.. Note::
   We highly recommend following the `Pytorch Github
   page <https://github.com/pytorch/pytorch>`__ to set up the PyTorch
   development environment on your local machine.

Once we have TorchVision installed successfully, let’s navigate to the
HelloWorld folder and run ``trace_model.py``. The script contains the
code of tracing and saving a `TorchScript
model <https://github.com/pytorch/ios-demo-app/blob/master/HelloWorld/trace_model.py>`__
that can be run on mobile devices.

.. code:: shell

   python trace_model.py

If everything works well, we should have our model, ``model.pt``,
generated in the ``HelloWorld`` folder. Now copy the model file to our
application folder ``HelloWorld/model``.

.. Note::
   For more details about TorchScript, see the `TorchScript documentation <https://pytorch.org/docs/stable/jit.html>`__.

Install LibTorch via Cocoapods
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The PyTorch C++ library is available in
`Cocoapods <https://cocoapods.org/>`__, to integrate it to our project,
simply run

.. code:: ruby

   pod install

Now it’s time to open the ``HelloWorld.xcworkspace`` in XCode, select an
iOS simulator and launch it (cmd + R). If everything works well, we
should see a wolf picture on the simulator screen along with the
prediction result.

Code Walkthrough
~~~~~~~~~~~~~~~~

In this part, we are going to walk through the code step by step.

Image Loading
^^^^^^^^^^^^^

Let’s begin with image loading.

.. code:: swift

   let image = UIImage(named: "image.jpg")!
   imageView.image = image
   let resizedImage = image.resized(to: CGSize(width: 224, height: 224))
   guard var pixelBuffer = resizedImage.normalized() else {
       return
   }

We first load the image from our bundle and resize it to 224x224. Then
we call this ``normalized()`` category method to normalized the pixel
buffer. Let’s take a closer look at the code below.

.. code:: swift

   var normalizedBuffer: [Float32] = [Float32](repeating: 0, count: w * h * 3)
   // normalize the pixel buffer
   // see https://pytorch.org/hub/pytorch_vision_resnet/ for more detail
   for i in 0 ..< w * h {
       normalizedBuffer[i]             = (Float32(rawBytes[i * 4 + 0]) / 255.0 - 0.485) / 0.229 // R
       normalizedBuffer[w * h + i]     = (Float32(rawBytes[i * 4 + 1]) / 255.0 - 0.456) / 0.224 // G
       normalizedBuffer[w * h * 2 + i] = (Float32(rawBytes[i * 4 + 2]) / 255.0 - 0.406) / 0.225 // B
   }

The code might look weird at first glance, but it’ll make sense once we
understand our model. The input data is a 3-channel RGB image of shape
(3 x H x W), where H and W are expected to be at least 224. The image
has to be loaded in to a range of ``[0, 1]`` and then normalized using
``mean = [0.485, 0.456, 0.406]`` and ``std = [0.229, 0.224, 0.225]``.

TorchScript Module
^^^^^^^^^^^^^^^^^^

Now that we have preprocessed our input data and we have a pre-trained
TorchScript model, the next step is to use them to run prediction. To
do that, we’ll first load our model into the application.

.. code:: swift

   private lazy var module: TorchModule = {
       if let filePath = Bundle.main.path(forResource: "model", ofType: "pt"),
           let module = TorchModule(fileAtPath: filePath) {
           return module
       } else {
           fatalError("Can't find the model file!")
       }
   }()

Note that the ``TorchModule`` Class is an Objective-C wrapper of
``torch::jit::mobile::Module``.

.. code:: cpp

   torch::jit::mobile::Module module = torch::jit::_load_for_mobile(filePath.UTF8String);

Since Swift can not talk to C++ directly, we have to either use an
Objective-C class as a bridge, or create a C wrapper for the C++
library. For demo purpose, we’re going to wrap everything in this
Objective-C class.

Run Inference
^^^^^^^^^^^^^

Now it’s time to run inference and get the results.

.. code:: swift

   guard let outputs = module.predict(image: UnsafeMutableRawPointer(&pixelBuffer)) else {
       return
   }

Again, the ``predict`` method is just an Objective-C wrapper. Under the
hood, it calls the C++ ``forward`` function. Let’s take a look at how
it’s implemented.

.. code:: cpp

   at::Tensor tensor = torch::from_blob(imageBuffer, {1, 3, 224, 224}, at::kFloat);
   torch::autograd::AutoGradMode guard(false);
   auto outputTensor = _impl.forward({tensor}).toTensor();
   float* floatBuffer = outputTensor.data_ptr<float>();

The C++ function ``torch::from_blob`` will create an input tensor from
the pixel buffer. Note that the shape of the tensor is ``{1,3,224,224}``
which represents ``NxCxWxH`` as we discussed in the above section.

.. code:: cpp

   torch::autograd::AutoGradMode guard(false);
   at::AutoNonVariableTypeMode non_var_type_mode(true);

The above two lines tells the PyTorch engine to do inference only. This
is because by default, PyTorch has built-in support for doing
auto-differentiation, which is also known as
`autograd <https://pytorch.org/docs/stable/notes/autograd.html>`__.
Since we don’t do training on mobile, we can just disable the autograd
mode.

Finally, we can call this ``forward`` function to get the output tensor
and convert it to a ``float`` buffer.

.. code:: cpp

   auto outputTensor = _impl.forward({tensor}).toTensor();
   float* floatBuffer = outputTensor.data_ptr<float>();

Collect Results
~~~~~~~~~~~~~~~

The output tensor is a one-dimensional float array of shape 1x1000,
where each value represents the confidence that a label is predicted
from the image. The code below sorts the array and retrieves the top
three results.

.. code:: swift

   let zippedResults = zip(labels.indices, outputs)
   let sortedResults = zippedResults.sorted { $0.1.floatValue > $1.1.floatValue }.prefix(3)

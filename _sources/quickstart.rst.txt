Android Quickstart
==================

`HelloWorld <https://github.com/pytorch/android-demo-app/tree/master/HelloWorldApp>`__
is a simple image classification application that demonstrates how to
use PyTorch Android API. This application runs TorchScript serialized
TorchVision pretrained resnet18 model on static image which is packaged
inside the app as android asset.

1. Model Preparation
^^^^^^^^^^^^^^^^^^^^

Let’s start with model preparation. If you are familiar with PyTorch,
you probably should already know how to train and save your model. In
case you don’t, we are going to use a pre-trained image classification
model
(`MobileNetV2 <https://pytorch.org/hub/pytorch_vision_mobilenet_v2/>`__).
To install it, run the command below:

::

   pip install torchvision

To serialize the model you can use python
`script <https://github.com/pytorch/android-demo-app/blob/master/HelloWorldApp/trace_model.py>`__
in the root folder of HelloWorld app:

::

   import torch
   import torchvision
   from torch.utils.mobile_optimizer import optimize_for_mobile

   model = torchvision.models.mobilenet_v2(pretrained=True)
   model.eval()
   example = torch.rand(1, 3, 224, 224)
   traced_script_module = torch.jit.trace(model, example)
   traced_script_module_optimized = optimize_for_mobile(traced_script_module)
   traced_script_module_optimized._save_for_lite_interpreter("app/src/main/assets/model.ptl")

If everything works well, we should have our model - ``model.ptl``
generated in the assets folder of android application. That will be
packaged inside android application as ``asset`` and can be used on the
device.

More details about TorchScript you can find in `tutorials on
pytorch.org <https://pytorch.org/docs/stable/jit.html>`__

2. Cloning from github
^^^^^^^^^^^^^^^^^^^^^^

::

   git clone https://github.com/pytorch/android-demo-app.git
   cd HelloWorldApp

If `Android
SDK <https://developer.android.com/studio/index.html#command-tools>`__
and `Android NDK <https://developer.android.com/ndk/downloads>`__ are
already installed you can install this application to the connected
android device or emulator with:

::

   ./gradlew installDebug

We recommend you to open this project in `Android Studio
3.5.1+ <https://developer.android.com/studio>`__. At the moment PyTorch
Android and demo applications use `android gradle plugin of version
3.5.0 <https://developer.android.com/studio/releases/gradle-plugin#3-5-0>`__,
which is supported only by Android Studio version 3.5.1 and higher.
Using Android Studio you will be able to install Android NDK and Android
SDK with Android Studio UI.

3. Gradle dependencies
^^^^^^^^^^^^^^^^^^^^^^

Pytorch android is added to the HelloWorld as `gradle
dependencies <https://github.com/pytorch/android-demo-app/blob/master/HelloWorldApp/app/build.gradle#L28-L29>`__
in build.gradle:

::

   repositories {
       jcenter()
   }

   dependencies {
       implementation 'org.pytorch:pytorch_android_lite:1.9.0'
       implementation 'org.pytorch:pytorch_android_torchvision:1.9.0'
   }

Where ``org.pytorch:pytorch_android`` is the main dependency with
PyTorch Android API, including libtorch native library for all 4 android
abis (armeabi-v7a, arm64-v8a, x86, x86_64). Further in this doc you can
find how to rebuild it only for specific list of android abis.

``org.pytorch:pytorch_android_torchvision`` - additional library with
utility functions for converting ``android.media.Image`` and
``android.graphics.Bitmap`` to tensors.

4. Reading image from Android Asset
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

All the logic happens in
```org.pytorch.helloworld.MainActivity`` <https://github.com/pytorch/android-demo-app/blob/master/HelloWorldApp/app/src/main/java/org/pytorch/helloworld/MainActivity.java#L31-L69>`__.
As a first step we read ``image.jpg`` to ``android.graphics.Bitmap``
using the standard Android API.

::

   Bitmap bitmap = BitmapFactory.decodeStream(getAssets().open("image.jpg"));

5. Loading Mobile Module
^^^^^^^^^^^^^^^^^^^^^^^^

::

   Module module = Module.load(assetFilePath(this, "model.ptl"));

``org.pytorch.Module`` represents ``torch::jit::mobile::Module`` that
can be loaded with ``load`` method specifying file path to the
serialized to file model.

6. Preparing Input
^^^^^^^^^^^^^^^^^^

::

   Tensor inputTensor = TensorImageUtils.bitmapToFloat32Tensor(bitmap,
       TensorImageUtils.TORCHVISION_NORM_MEAN_RGB, TensorImageUtils.TORCHVISION_NORM_STD_RGB);

``org.pytorch.torchvision.TensorImageUtils`` is part of
``org.pytorch:pytorch_android_torchvision`` library. The
``TensorImageUtils#bitmapToFloat32Tensor`` method creates tensors in the
`torchvision
format <https://pytorch.org/docs/stable/torchvision/models.html>`__
using ``android.graphics.Bitmap`` as a source.

   All pre-trained models expect input images normalized in the same
   way, i.e. mini-batches of 3-channel RGB images of shape (3 x H x W),
   where H and W are expected to be at least 224. The images have to be
   loaded in to a range of ``[0, 1]`` and then normalized using
   ``mean = [0.485, 0.456, 0.406]`` and ``std = [0.229, 0.224, 0.225]``

``inputTensor``\ ’s shape is ``1x3xHxW``, where ``H`` and ``W`` are
bitmap height and width appropriately.

7. Run Inference
^^^^^^^^^^^^^^^^

::

   Tensor outputTensor = module.forward(IValue.from(inputTensor)).toTensor();
   float[] scores = outputTensor.getDataAsFloatArray();

``org.pytorch.Module.forward`` method runs loaded module’s ``forward``
method and gets result as ``org.pytorch.Tensor`` outputTensor with shape
``1x1000``.

8. Processing results
^^^^^^^^^^^^^^^^^^^^^

Its content is retrieved using
``org.pytorch.Tensor.getDataAsFloatArray()`` method that returns java
array of floats with scores for every image net class.

After that we just find index with maximum score and retrieve predicted
class name from ``ImageNetClasses.IMAGENET_CLASSES`` array that contains
all ImageNet classes.

::

   float maxScore = -Float.MAX_VALUE;
   int maxScoreIdx = -1;
   for (int i = 0; i < scores.length; i++) {
     if (scores[i] > maxScore) {
       maxScore = scores[i];
       maxScoreIdx = i;
     }
   }
   String className = ImageNetClasses.IMAGENET_CLASSES[maxScoreIdx];
   
  
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
model - `MobileNet
v2 <https://pytorch.org/hub/pytorch_vision_mobilenet_v2/>`__, which is
already packaged in
`TorchVision <https://pytorch.org/docs/stable/torchvision/index.html>`__.
To install it, run the command below.

   We highly recommend following the `Pytorch Github
   page <https://github.com/pytorch/pytorch>`__ to set up the Python
   development environment on your local machine.

.. code:: shell

   pip install torchvision

Once we have TorchVision installed successfully, let’s navigate to the
HelloWorld folder and run ``trace_model.py``. The script contains the
code of tracing and saving a `torchscript
model <https://pytorch.org/tutorials/beginner/Intro_to_TorchScript_tutorial.html>`__
that can be run on mobile devices.

.. code:: shell

   python trace_model.py

If everything works well, we should have our model - ``model.pt``
generated in the ``HelloWorld`` folder. Now copy the model file to our
application folder ``HelloWorld/model``.

   To find out more details about TorchScript, please visit `tutorials
   on
   pytorch.org <https://pytorch.org/tutorials/advanced/cpp_export.html>`__

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
TorchScript model, the next step is to use them to run predication. To
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

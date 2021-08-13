
PyTorch Mobile
==============

There is a growing need to execute ML models on edge devices to reduce
latency, preserve privacy, and enable new interactive use cases.

The PyTorch Mobile runtime beta release allows you to seamlessly go from
training a model to deploying it, while staying entirely within the
PyTorch ecosystem. It provides an end-to-end workflow that simplifies
the research to production environment for mobile devices. In addition,
it paves the way for privacy-preserving features via federated learning
techniques.

PyTorch Mobile is in beta stage right now, and is already in wide scale
production use. It will soon be available as a stable release once the
APIs are locked down.

Key features
------------

-  Available for `iOS <%7B%7Bsite.baseurl%7D%7D/mobile/ios>`__,
   `Android <%7B%7Bsite.baseurl%7D%7D/mobile/android>`__ and Linux
-  Provides APIs that cover common preprocessing and integration tasks
   needed for incorporating ML in mobile applications
-  Support for tracing and scripting via TorchScript IR
-  Support for XNNPACK floating point kernel libraries for Arm CPUs
-  Integration of QNNPACK for 8-bit quantized kernels. Includes support
   for per-channel quantization, dynamic quantization and more
-  Provides an `efficient mobile interpreter in Android and
   iOS <https://pytorch.org/tutorials/recipes/mobile_interpreter.html>`__.
   Also supports build level optimization and selective compilation
   depending on the operators needed for user applications (i.e., the
   final binary size of the app is determined by the actual operators
   the app needs).
-  Streamline model optimization via optimize_for_mobile
-  Support for hardware backends like GPU, DSP, and NPU will be
   available soon in Beta

Prototypes
----------

We have launched the following features in prototype, available in the
PyTorch nightly releases, and would love to get your feedback on the
`PyTorch forums <https://discuss.pytorch.org/c/mobile/18>`__:

-  GPU support on `iOS via
   Metal <https://pytorch.org/tutorials/prototype/ios_gpu_workflow.html>`__
-  GPU support on `Android via
   Vulkan <https://pytorch.org/tutorials/prototype/vulkan_workflow.html>`__
-  DSP and NPU support on Android via `Google
   NNAPI <https://pytorch.org/tutorials/prototype/nnapi_mobilenetv2.html>`__

Deployment workflow
-------------------

A typical workflow from training to mobile deployment with the optional
model optimization steps is outlined in the following figure.

Examples to get you started
---------------------------

-  `PyTorch Mobile Runtime for
   iOS <https://www.youtube.com/watch?v=amTepUIR93k>`__
-  `PyTorch Mobile Runtime for
   Android <https://www.youtube.com/watch?v=5Lxuu16_28o>`__
-  `PyTorch Mobile Recipes in
   Tutorials <https://pytorch.org/tutorials/recipes/ptmobile_recipes_summary.html>`__
-  `Image Segmentation DeepLabV3 on
   iOS <https://pytorch.org/tutorials/beginner/deeplabv3_on_ios.html>`__
-  `Image Segmentation DeepLabV3 on
   Android <https://pytorch.org/tutorials/beginner/deeplabv3_on_android.html>`__
-  `D2Go Object Detection on
   iOS <https://github.com/pytorch/ios-demo-app/tree/master/D2Go>`__
-  `D2Go Object Detection on
   Android <https://github.com/pytorch/android-demo-app/tree/master/D2Go>`__
-  `PyTorchVideo on
   iOS <https://github.com/pytorch/ios-demo-app/tree/master/TorchVideo>`__
-  `PyTorchVideo on
   Android <https://github.com/pytorch/android-demo-app/tree/master/TorchVideo>`__
-  `Speech Recognition on
   iOS <https://github.com/pytorch/ios-demo-app/tree/master/SpeechRecognition>`__
-  `Speech Recognition on
   Android <https://github.com/pytorch/android-demo-app/tree/master/SpeechRecognition>`__
-  `Question Answering on
   iOS <https://github.com/pytorch/ios-demo-app/tree/master/QuestionAnswering>`__
-  `Question Answering on
   Android <https://github.com/pytorch/android-demo-app/tree/master/QuestionAnswering>`__

Demo apps
---------

Our new demo apps also include examples of image segmentation, object
detection, neural machine translation, question answering, and vision
transformers. They are available on both iOS and Android:

-  `iOS demo apps <https://github.com/pytorch/ios-demo-app>`__
-  `Android demo apps <https://github.com/pytorch/android-demo-app>`__

.. toctree::
   :maxdepth: 1
   :caption: Quickstarts
   
   quickstarts/androidquickstart
   quickstarts/iosquickstart


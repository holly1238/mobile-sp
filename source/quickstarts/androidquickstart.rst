Android Quickstart
==================

`HelloWorld`_ is a simple image classification application that
demonstrates how to use PyTorch Android API. This application runs
TorchScript serialized TorchVision pretrained resnet18 model on static
image which is packaged inside the app as android asset.

1. Model Preparation
^^^^^^^^^^^^^^^^^^^^

Let’s start with model preparation. If you are familiar with PyTorch,
you probably should already know how to train and save your model. In
case you don’t, we are going to use a pre-trained image classification
model (`MobileNetV2`_). To install it, run the command below:

::

   pip install torchvision

To serialize the model you can use python `script`_ in the root folder
of HelloWorld app:

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
pytorch.org`_

2. Cloning from github
^^^^^^^^^^^^^^^^^^^^^^

::

   git clone https://github.com/pytorch/android-demo-app.git
   cd HelloWorldApp

If `Android SDK`_ and `Android NDK`_ are already installed you can
install this application to the connected android device or emulator
with:

::

   ./gradlew installDebug

We recommend you to open this project in `Android Studio 3.5.1+`_. At
the moment PyTorch Android and demo applications use `android gradle
plugin of version 3.5.0`_, which is supported only by Android Studio
version 3.5.1 and higher. Using Android Studio you will be able to
install Android NDK and Android SDK with Android Studio UI.

3. Gradle dependencies
^^^^^^^^^^^^^^^^^^^^^^

Pytorch android is added to the HelloWorld as `gradle dependencies`_ in
build.gradle:

::

   repositories {
       jcenter()
   }

   dependencies {
       implementation 'org.pytorch:pytorch_android_lite:1.9.0'
       implementation 'org.pytorch:pytorch_android_torchvision:1.9.0'
   }

Where ``org.pytorch:pytorch_android`` is the main dependency with
PyTorch Android API

.. _HelloWorld: https://github.com/pytorch/android-demo-app/tree/master/HelloWorldApp
.. _MobileNetV2: https://pytorch.org/hub/pytorch_vision_mobilenet_v2/
.. _script: https://github.com/pytorch/android-demo-app/blob/master/HelloWorldApp/trace_model.py
.. _tutorials on pytorch.org: https://pytorch.org/docs/stable/jit.html
.. _Android SDK: https://developer.android.com/studio/index.html#command-tools
.. _Android NDK: https://developer.android.com/ndk/downloads
.. _Android Studio 3.5.1+: https://developer.android.com/studio
.. _android gradle plugin of version 3.5.0: https://developer.android.com/studio/releases/gradle-plugin#3-5-0
.. _gradle dependencies: https://github.com/pytorch/android-demo-app/blob/master/HelloWorldApp/app/build.gradle#L28-L29

Building PyTorch Android from Source
====================================

In some cases you might want to use a local build of PyTorch android.
For example, you may build custom LibTorch binary with another set of
operators or to make local changes, or try out the latest PyTorch code.

For this you can use ``./scripts/build_pytorch_android.sh`` script.

::

   git clone https://github.com/pytorch/pytorch.git
   cd pytorch
   sh ./scripts/build_pytorch_android.sh

The workflow contains the following steps:

1. Build libtorch for android for all 4 android abis (armeabi-v7a,
arm64-v8a, x86, x86_64)

2. Create symbolic links to the results of those builds:
``android/pytorch_android/src/main/jniLibs/${abi}`` to the directory
with output libraries
``android/pytorch_android/src/main/cpp/libtorch_include/${abi}`` to the
directory with headers. These directories are used to build
``libpytorch_jni.so`` library, as part of the
``pytorch_android-release.aar`` bundle, that will be loaded on android
device.

3. Run ``gradle`` in ``android/pytorch_android`` directory
with task ``assembleRelease``

Script requires that Android SDK, Android NDK, Java SDK, and gradle are
installed. They are specified as environment variables:

``ANDROID_HOME`` - path to `Android
SDK <https://developer.android.com/studio/command-line/sdkmanager.html>`__

``ANDROID_NDK`` - path to `Android
NDK <https://developer.android.com/studio/projects/install-ndk>`__

``GRADLE_HOME`` - path to `gradle <https://gradle.org/releases/>`__

``JAVA_HOME`` - path to `JAVA
JDK <https://www.oracle.com/java/technologies/javase-downloads.html#javasejdk>`__

After successful build, you should see the result as an aar file:

::

   $ find android -type f -name *aar
   android/pytorch_android/build/outputs/aar/pytorch_android-release.aar
   android/pytorch_android_torchvision/build/outputs/aar/pytorch_android_torchvision-release.aar

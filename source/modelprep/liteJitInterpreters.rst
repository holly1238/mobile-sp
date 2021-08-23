PyTorch Lite and Jit Interpreters
=================================

Android PyTorch Jit Interpreter
-------------------------------

PyTorch JIT interpreter is the default interpreter before 1.9 (a version
of our PyTorch interpreter that is not as size-efficient). It will still
be supported in 1.9, and can be used via ``build.gradle``:

::

   repositories {
       jcenter()
   }

   dependencies {
       implementation 'org.pytorch:pytorch_android:1.9.0'
       implementation 'org.pytorch:pytorch_android_torchvision:1.9.0'
   }

iOS PyTorch Jit Interpreter
---------------------------

PyTorch JIT interpreter is the default interpreter before 1.9 (a version
of our PyTorch interpreter that is not as size-efficient). It will still
be supported in 1.9, and can be used in CocoaPods:

::

   pod 'LibTorch', '~>1.9.0'

Building PyTorch iOS from Source
================================

To track the latest updates for iOS, you can build the PyTorch iOS
libraries from the source code.

::

   git clone --recursive https://github.com/pytorch/pytorch
   cd pytorch
   # if you are updating an existing checkout
   git submodule sync
   git submodule update --init --recursive

..

.. Note::
   Make sure you have ``cmake`` and Python installed correctly on your
   local machine. We recommend following the `Pytorch Github
   page <https://github.com/pytorch/pytorch>`__ to set up the Python
   development environment.

Build LibTorch-Lite for iOS Simulators
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Open the terminal and navigate to the PyTorch root directory. Run the
following command (if you already build LibTorch-Lite for iOS devices (see
below), run ``rm -rf build_ios`` first):

::

   BUILD_PYTORCH_MOBILE=1 IOS_PLATFORM=SIMULATOR ./scripts/build_ios.sh

After the build succeeds, all static libraries and header files will be
generated under ``build_ios/install``

Build LibTorch-Lite for arm64 Devices
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Open terminal and navigate to the PyTorch root directory. Run the
following command (if you already build LibTorch-Lite for iOS simulators (see above), run
``rm -rf build_ios`` first):

::

   BUILD_PYTORCH_MOBILE=1 IOS_ARCH=arm64 ./scripts/build_ios.sh

After the build succeeds, all static libraries and header files will be
generated under ``build_ios/install``

XCode Setup
~~~~~~~~~~~

Open your project in XCode, go to your project Target’s ``Build Phases``
- ``Link Binaries With Libraries``, click the + sign and add all the
library files located in ``build_ios/install/lib``. Navigate to the
project ``Build Settings``, set the value **Header Search Paths** to
``build_ios/install/include`` and **Library Search Paths** to
``build_ios/install/lib``.

In the build settings, search for **other linker flags**. Add a custom
linker flag below

::

   -all_load

To use the custom built libraries the project, replace
``#import <LibTorch-Lite/LibTorch-Lite.h>`` (in ``TorchModule.mm``) which is
needed when using LibTorch-Lite via Cocoapods with the code below:

::

   #include "ATen/ATen.h"
   #include "caffe2/core/timer.h"
   #include "caffe2/utils/string_utils.h"
   #include "torch/csrc/autograd/grad_mode.h"
   #include "torch/csrc/jit/mobile/import.h"
   #include "torch/csrc/jit/mobile/module.h"
   #include "torch/script.h"

Finally, disable bitcode for your target by selecting the Build
Settings, searching for **Enable Bitcode**, and set the value to **No**.

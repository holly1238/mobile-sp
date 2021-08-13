Build PyTorch iOS Libraries from Source
=======================================

To track the latest updates for iOS, you can build the PyTorch iOS
libraries from the source code.

::

   git clone --recursive https://github.com/pytorch/pytorch
   cd pytorch
   # if you are updating an existing checkout
   git submodule sync
   git submodule update --init --recursive

..

   Make sure you have ``cmake`` and Python installed correctly on your
   local machine. We recommend following the `Pytorch Github
   page <https://github.com/pytorch/pytorch>`__ to set up the Python
   development environment

Build LibTorch for iOS Simulators
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Open terminal and navigate to the PyTorch root directory. Run the
following command (if you already build LibTorch for iOS devices (see
below), run ``rm -rf build_ios`` first):

::

   BUILD_PYTORCH_MOBILE=1 IOS_PLATFORM=SIMULATOR ./scripts/build_ios.sh

After the build succeeds, all static libraries and header files will be
generated under ``build_ios/install``

Build LibTorch for arm64 Devices
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Open terminal and navigate to the PyTorch root directory. Run the
following command (if you already build LibTorch for iOS simulators, run
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
``#import <LibTorch/LibTorch.h>`` (in ``TorchModule.mm``) which is
needed when using LibTorch via Cocoapods with the code below:

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

Custom Build
------------

Starting from 1.4.0, PyTorch supports custom build. You can now build
the PyTorch library that only contains the operators needed by your
model. To do that, follow the steps below

1. Verify your PyTorch version is 1.4.0 or above. You can do that by
checking the value of ``torch.__version__``.

2. To dump the operators in your model, say ``MobileNetV2``, run the
following lines of Python code:

.. code:: python

   import torch, yaml
   model = torch.jit.load('MobileNetV2.pt')
   ops = torch.jit.export_opnames(model)
   with open('MobileNetV2.yaml', 'w') as output:
       yaml.dump(ops, output)

In the snippet above, you first need to load the ScriptModule. Then, use
``export_opnames`` to return a list of operator names of the
ScriptModule and its submodules. Lastly, save the result in a yaml file.

3. To run the iOS build script locally with the prepared yaml list of
operators, pass in the yaml file generate from the last step into the
environment variable ``SELECTED_OP_LIST``. Also in the arguments,
specify ``BUILD_PYTORCH_MOBILE=1`` as well as the platform/architechture
type. Take the arm64 build for example, the command should be:

::

   SELECTED_OP_LIST=MobileNetV2.yaml BUILD_PYTORCH_MOBILE=1 IOS_ARCH=arm64 ./scripts/build_ios.sh

4. After the build succeeds, you can integrate the result libraries to
your project by following the `XCode Setup <#xcode-setup>`__ section
above.

5. The last step is to add a single line of C++ code before running
``forward``. This is because by default JIT will do some optimizations
on operators (fusion for example), which might break the consistency
with the ops we dumped from the model.

.. code:: cpp

   torch::jit::GraphOptimizerEnabledGuard guard(false);

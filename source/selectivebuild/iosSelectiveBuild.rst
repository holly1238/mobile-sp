iOS Selective Build
===================

Starting from 1.4.0, to reduce the size of binaries, PyTorch supports selective builds. 
You can now build the PyTorch library that only contains the operators that are needed by your
model. To do that, follow the steps below:

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
your project by following the steps in `XCode Setup <#xcode-setup>`__.

5. The last step is to add a single line of C++ code before running
``forward``. This is because by default JIT (TorchScript) will do some optimizations
on operators (fusion for example), which might break the consistency
with the ops we dumped from the model.

.. code:: cpp

   torch::jit::GraphOptimizerEnabledGuard guard(false);

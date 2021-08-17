Andriod Selective Build
=======================

To reduce the size of binaries you can do custom build of PyTorch
Android with only set of operators required by your model. This includes
two steps: preparing the list of operators from your model, rebuilding
pytorch android with specified list.

1. Verify your PyTorch version is 1.4.0 or above. You can do that by
checking the value of ``torch.__version__``.

2. Preparation of the list of operators

List of operators of your serialized torchscript model can be prepared
in yaml format using python api function ``torch.jit.export_opnames()``.
To dump the operators in your model, say ``MobileNetV2``, run the
following lines of Python code:

::

   # Dump list of operators used by MobileNetV2:
   import torch, yaml
   model = torch.jit.load('MobileNetV2.pt')
   ops = torch.jit.export_opnames(model)
   with open('MobileNetV2.yaml', 'w') as output:
       yaml.dump(ops, output)

3. Building PyTorch Android with prepared operators list.

To build PyTorch Android with the prepared yaml list of operators,
specify it in the environment variable ``SELECTED_OP_LIST``. Also in the
arguments, specify which Android ABIs it should build; by default it
builds all 4 Android ABIs.

::

   # Build PyTorch Android library customized for MobileNetV2:
   SELECTED_OP_LIST=MobileNetV2.yaml scripts/build_pytorch_android.sh arm64-v8a

After successful build you can integrate the result aar files to your
android gradle project, following the steps from previous section of
this tutorial (Building PyTorch Android from Source).

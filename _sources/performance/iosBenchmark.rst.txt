iOS Benchmarking
================

The best way to benchmark (to check if optimizations helped your use case) - is to measure your particular use case that you want to optimize, as performance behavior can vary in different environments.

PyTorch distribution provides a way to benchmark naked binary that runs the model forward,
this approach can give more stable measurements rather than testing inside the application.

For iOS, we'll be using our `TestApp <https://github.com/pytorch/pytorch/tree/master/ios/TestApp>`_ as the benchmarking tool.

To begin with, let's apply the ``optimize_for_mobile`` method to our python script located at `TestApp/benchmark/trace_model.py <https://github.com/pytorch/pytorch/blob/master/ios/TestApp/benchmark/trace_model.py>`_. Simply modify the code as below.

::

  import torch
  import torchvision
  from torch.utils.mobile_optimizer import optimize_for_mobile

  model = torchvision.models.mobilenet_v2(pretrained=True)
  model.eval()
  example = torch.rand(1, 3, 224, 224)
  traced_script_module = torch.jit.trace(model, example)
  torchscript_model_optimized = optimize_for_mobile(traced_script_module)
  torch.jit.save(torchscript_model_optimized, "model.pt")

Now let's run ``python trace_model.py``. If everything works well, we should be able to generate our optimized model in the benchmark directory.

Next, we're going to build the PyTorch libraries from source.

::

  BUILD_PYTORCH_MOBILE=1 IOS_ARCH=arm64 ./scripts/build_ios.sh

Now that we have the optimized model and PyTorch ready, it's time to generate our XCode project and do benchmarking. To do that, we'll be using a ruby script - `setup.rb` which does the heavy lifting jobs of setting up the XCode project.

::

  ruby setup.rb

Now open the `TestApp.xcodeproj` and plug in your iPhone, you're ready to go. Below is an example result from iPhoneX

::

  TestApp[2121:722447] Main runs
  TestApp[2121:722447] Main run finished. Milliseconds per iter: 28.767
  TestApp[2121:722447] Iters per second: : 34.762
  TestApp[2121:722447] Done.

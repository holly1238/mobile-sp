Andriod Benchmarking
====================

The best way to benchmark (to check if optimizations helped your use case) - is to measure your particular use case that you want to optimize, as performance behavior can vary in different environments.

PyTorch distribution provides a way to benchmark naked binary that runs the model forward,
this approach can give more stable measurements rather than testing inside the application.

For this you first need to build benchmark binary:

::

    <from-your-root-pytorch-dir>
    rm -rf build_android
    BUILD_PYTORCH_MOBILE=1 ANDROID_ABI=arm64-v8a ./scripts/build_android.sh -DBUILD_BINARY=ON

You should have arm64 binary at: ``build_android/bin/speed_benchmark_torch``.
This binary takes ``--model=<path-to-model>``, ``--input_dim="1,3,224,224"`` as dimension information for the input and ``--input_type="float"`` as the type of the input as arguments.

Once you have your android device connected,
push speedbenchark_torch binary and your model to the phone:

::

  adb push <speedbenchmark-torch> /data/local/tmp
  adb push <path-to-scripted-model> /data/local/tmp


Now we are ready to benchmark your model:

::

  adb shell "/data/local/tmp/speed_benchmark_torch --model=/data/local/tmp/model.pt" --input_dims="1,3,224,224" --input_type="float"
  ----- output -----
  Starting benchmark.
  Running warmup runs.
  Main runs.
  Main run finished. Microseconds per iter: 121318. Iters per second: 8.24281

Using the PyTorch iOS Libraries
===============================

Using the Nightly PyTorch iOS Libraries in CocoaPods
----------------------------------------------------

If you want to try out the latest features added to PyTorch iOS, you can
use the ``LibTorch-Lite-Nightly`` pod in your ``Podfile``, it includes
the nightly built libraries:

::

   pod 'LibTorch-Lite-Nightly'

And then run ``pod install`` to add it to your project. If you wish to
update the nightly pod to the newer one, you can run ``pod update`` to
get the latest version. But be aware you may need to build the model
used on mobile in the latest PyTorch - using either the latest PyTorch
code or a quick nightly install with commands like
``pip install --pre torch torchvision -f https://download.pytorch.org/whl/nightly/cpu/torch_nightly.html``
- to avoid possible model version mismatch errors when running the model
on mobile.

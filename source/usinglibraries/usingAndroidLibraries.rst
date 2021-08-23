Using the PyTorch Android Libraries
===================================

Using the PyTorch Android Libraries Built from Source or Nightly
----------------------------------------------------------------

To use the PyTorch Android Libraries, first add the two aar files built in `Building PyTorch Android From Source <..//androidsourcebuild.html>`__, 
or downloaded from the nightly built PyTorch Android repos from
`here <https://oss.sonatype.org/#nexus-search;quick~pytorch_android>`__
and
`here <https://oss.sonatype.org/#nexus-search;quick~torchvision_android>`__,
to the Android project’s ``lib`` folder. Then, add in the project’s app
``build.gradle`` file:

::

   allprojects {
       repositories {
           flatDir {
               dirs 'libs'
           }
       }
   }

   dependencies {

       // if using the libraries built from source
       implementation(name:'pytorch_android-release', ext:'aar')
       implementation(name:'pytorch_android_torchvision-release', ext:'aar')

       // if using the nightly built libraries downloaded above, for example the 1.8.0-snapshot on Jan. 21, 2021
       // implementation(name:'pytorch_android-1.8.0-20210121.092759-172', ext:'aar')
       // implementation(name:'pytorch_android_torchvision-1.8.0-20210121.092817-173', ext:'aar')

       ...
       implementation 'com.android.support:appcompat-v7:28.0.0'
       implementation 'com.facebook.fbjni:fbjni-java-only:0.0.3'
   }

In addition, we have to add all transitive dependencies of our aars. As
``pytorch_android`` depends on
``com.android.support:appcompat-v7:28.0.0`` or
``androidx.appcompat:appcompat:1.2.0``, we need to one of them. (In the case
of using maven dependencies they are added automatically from
``pom.xml``).

Using the Nightly PyTorch Android Libraries
-------------------------------------------

Other than using the aar files built from source or downloaded from the
links in the previous section, you can also use the nightly built
Android PyTorch and TorchVision libraries by adding in your app
``build.gradle`` file the maven url and the nightly libraries
implementation as follows:

::

   repositories {
       maven {
           url "https://oss.sonatype.org/content/repositories/snapshots"
       }
   }

   dependencies {
       ...
       implementation 'org.pytorch:pytorch_android:1.8.0-SNAPSHOT'
       implementation 'org.pytorch:pytorch_android_torchvision:1.8.0-SNAPSHOT'
   }

This is the easiest way to try out the latest PyTorch code and the
Android libraries, if you do not need to make any local changes. But be
aware you may need to build the model used on mobile in the latest
PyTorch - using either the latest PyTorch code or a quick nightly
install with commands like
``pip install --pre torch torchvision -f https://download.pytorch.org/whl/nightly/cpu/torch_nightly.html``
- to avoid possible model version mismatch errors when running the model
on mobile.

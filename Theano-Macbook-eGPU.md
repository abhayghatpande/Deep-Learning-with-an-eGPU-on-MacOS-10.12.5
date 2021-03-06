# Get Theano (TensorFlow / Keras) running on a Macbook Pro with an eGPU
July 22 2017

By: [J. Ross Mitchell, PhD](mailto:joseph.ross.mitchell@gmail.com)

### Hardware Requirements (what I used):
1. a Macbook Pro 15-inch (MacBookPro13,3 Intel Core i7 @ 2.9 GHz 16 GB ram) running Mac OS 10.12.5 Sierra. Though this process should work on any Mac with Thunderbolt 3 / USB-C
1. a Thunderbolt 3 eGPU enclosure. Make sure it has the TI83 Thunderbolt 3 chipset. For more info see this site: [eGPU Buyers Guide](https://egpu.io/external-gpu-buyers-guide-2017/). Lots of good info there. I chose the [Mantiz Venus](https://mymantiz.com/collections/shop/products/mz-02-venus?variant=28876425155). It arrived 4 days after I ordered it (shipped from Singapore). The Mantiz website says the Venus requires a host running Windows 10. Don't worry - it works just fine with Mac OS 10.12.5 Sierra, after you install a kernel extension (more below). [The kernel extension should no longer be needed once Mac OS High Sierra is released](https://9to5mac.com/2017/06/07/hands-on-macos-high-sierra-native-egpu-support-shows-promise-video/). The Venus has a number of other nice features (read the review on the Buyers Guide), one of which is the ability to charge the Macbook Pro over the TB3 connection.
1. a recent NVIDIA GPU. I chose the [eVGA GTX 1080Ti](https://www.amazon.com/gp/product/B06Y11DFZ3/). This is based on the NVIDIA Pascal architecture. This card fits nicely inside the Mantiz Venus, with good air circulation around the GPU. The card draws 250 watts, well within the limits available for PCI cards installed in the Venus. So far, everything has been stable (4 days).

### Hardware Installation Tips
* The Mantiz Venus user's guide asks you to make sure the unit is unplugged, then to unscrew the "card holder". This is a small L-shaped metal bracket. However, this causes the card holder to fall down inside the Venus, and strike the motherboard! My suggestion is to first remove the perforated side panel from the Venus. You can just pull it off. There are soft plugs that hold it in place. Next, place your hand inside the unit, above the Venus motherboard, near the rear of the unit and _below_ the card holder so you can catch it. _Now_  unscrew the card holder!
* I recommend attaching the internal power cables to the GPU before you install it in the Venus PCI-e slot. It is tough to connect the power cables after the GPU is installed in the slot.
* Once you have the GPU installed and connected to power cables, you can connect the external power cord and plug it into the wall. If you turn the Venus on via its rear power switch, nothing will happen! This was a bit of a shock. Do not worry. Eventually I figured out that the Venus will not power on until after it is connected to a host via the TB3 / USB-C cable! Would be nice for Mantiz to mention this in the (very brief) user guide.

![eGPU + MacBook Pro Setup](https://user-images.githubusercontent.com/13511772/28494636-0dc371d6-6ee9-11e7-8632-703352874dd2.jpg)

![Mantiz Venus](https://user-images.githubusercontent.com/13511772/28494927-3ca7711a-6ef2-11e7-9657-cb336048fe3d.jpg)

### Software Requirements and Installation
1. [Anaconda Python](https://www.continuum.io) Download and install. Next, add conda-forge to the list of channels anaconda searches for packages. At the command line enter: "_conda config --append channels conda-forge_". This adds conda-forge to the search list for _all_ Anaconda environments. Now, create a new environment for Python 2.7 with the following command: "_conda create --name deeplearning python=2.7_". Activate the new environment by entering: _"source activate deeplearning"_
1. [Theano](http://deeplearning.net/software/theano/) (version 0.9.0 as of this writing). You can get this via conda. At the command line enter: _"conda install theano"_. Once _theano_ is installed, you need to add the following line to your _"~/.bash_profile"_:
    > export THEANO_FLAGS="device=gpu,floatX=float32"
1. _Optional and not tested:_ [Keras](https://keras.io) (which will also install TensorFlow). Enter this command: _"conda install keras"_
1. Xcode 8.2.1. Note: this is not the current release of Xcode. You _must_ use version 8.2.1. Newer versions of Xcode are incompatible with the deep learning environment we are creating. Xcode 8.2.1 may be obtained from [Apple's Developer Download page](https://developer.apple.com/download/more/). On that page, scroll down until you see **"See more downloads"**. Click on that phrase (it does not look like a link!) to access older versions of Xcode. Access to Apple's Dev website requires a (free) account. Just do it. 
    1. Note: We need (this version of) Xcode for its compiler. Theano needs _clang_ to compile functions (train / test / validate, for example) on the fly. If you already have a newer version of Xcode installed, no problem! Install v8.2.1 in its own directory, say _/Applications/Xcode8.2.1_. You can make v8.2.1 the "active" version by entering the following command: _"sudo xcode-select -s /Applications/Xcode8.2.1/Contents/Developer"_. You can switch back to your normal Xcode using: _"sudo xcode-select -s /Applications/Xcode/Contents/Developer"_.
1. A Mac OS kernel extension (kext) that recognizes the eGPU. Fortunately, installing this has been totally automated for you by [goalque](https://github.com/goalque/automate-eGPU). I recommend following the very nice instructions over at [9to5Mac.com](https://9to5mac.com/2017/04/11/hands-on-powering-the-macbook-pro-with-an-egpu-using-nvidias-new-pascal-drivers/). After you are done, you will have to reboot your Mac (at least once). Then the eGPU should show up in your _System Report_. 
    1. Note: after installing this kext, my Macbook Pro displays a very glitchy screen during its boot. The boot progress bar continues to advance on the glitchy screen, and eventually the Finder comes up looking just fine. As I mentioned above, this step should go away with Mac OS High Sierra.
1. NVIDIA CUDA v8. I recommend following [NVIDIA's nice instructions](http://docs.nvidia.com/cuda/cuda-installation-guide-mac-os-x). Pay attention to **Table 1** on that page: we are installing on Mac OS 10.12, and this requires Xcode 8.2. You can skip to Step 3 on that page - we have already fullfilled the Prerequisites. Remember to add the _"export"_ statements to your _"~/.bash_profile"_, and invoke those exports by entering: _"source ~/.bash_profile"_.
1. It is important to complete **Step 4. Verification** on that page. Make sure NVIDIA's compiler is installed and working. Enter: _"nvcc -V"_ in a Terminal window. Also, make sure you compile the four sample programs as listed on that page. The _"makefile"_ and source code files for these programs are located in _"/Developer/NVIDIA/CUDA-8.0/samples/"_. However, to build these programs you need to prepend each _"make"_ command with _"sudo"_. This is because the _"samples"_ directory is read-only. The output executables are installed in _"/Developer/NVIDIA/CUDA-8.0/samples/bin/x86_64/darwin/release/"_. Go there and run the _"deviceQuery"_ and _"bandwidthTest"_ programs. Output from those programs on my system is shown below.
1. [cuDNN v5.1 for CUDA 8.0](https://developer.nvidia.com/rdp/cudnn-download). You need a (free) NVIDIA developer account to access this library. cuDNN contains a number of CUDA algorithms to accelerate deep neural networks. You _must_ use cuDNN v5.1! Newer versions of cuDNN are currently incompatible. Pascal architecture GPUs require cuDNN v5 or newer.
    1. cuDNN is packaged as a .zip file. After unpacking it you will see a folder with two subfolders: _"lib"_ and _"include"_. Copy the files from the _"lib"_ subfolder to _"/Developer/NVIDIA/CUDA-8.0/lib"_. Copy the file in the _"include"_ subfolder to _"/Developer/NVIDIA/CUDA-8.0/include"_.

![deviceQuery output](https://user-images.githubusercontent.com/13511772/28495273-4ba03a3a-6efc-11e7-8059-3aef12a07bf7.png)

![bandwidthTest ouput](https://user-images.githubusercontent.com/13511772/28495276-4fa9565c-6efc-11e7-835a-ebbbf48b8a51.png)


### Test Theano!

Finally we are ready to test our environment! Activate your deep learning Python environment by entering: _"source activate deeplearning"_. Next, launch Python by entering: _"python"_. From the Python prompt enter: _"import theano"_. It should pause, then eventually describe which GPU it is using, and that cuDNN is available. Output on my system is shown below.

![theano](https://user-images.githubusercontent.com/13511772/28495497-76fb71b6-6f03-11e7-8c53-f420aae5969a.png)

Enter this simple program, from the [Theano homepage](http://deeplearning.net/software/theano/introduction.html), one line at a time (or copy / paste). You should get no complaints from Python after executing the _"assert"_ statement...

```python
import theano
from theano import tensor

# declare two symbolic floating-point scalars
a = tensor.dscalar()
b = tensor.dscalar()

# create a simple expression
c = a + b

# convert the expression into a callable object that takes (a,b)
# values as input and computes a value for c
f = theano.function([a,b], c)

# bind 1.5 to 'a', 2.5 to 'b', and evaluate 'c'
assert 4.0 == f(1.5, 2.5)
```

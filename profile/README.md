## Welcome to Motorola-SM6375-Devs organization 

This organization contains device specific & common sources for various SM6375 based Motorola devices.

Founded and maintained by [@AnandSuresh02](https://github.com/AnandSuresh02), feel free to contribute.

If you like my work, feel free to donate:

* UPI - anandzzz360@oksbi
* Buy me a Coffee - https://buymeacoffee.com/anandzzz36g
* PayPal - https://www.paypal.com/paypalme/anandsuresh02

## Devices supported

* Motorola Moto G34 5G - fogos
* Motorola Moto G82 5G - rhodep
* Motorola Moto G84 5G - bangkk

## Build Instructions

### Step 1: Setup build environment

The catch: Make sure you have atleast the following minimum specs:

* A 12-thread CPU.
* Minimum 16GB of installed RAM, and that too requires swap.
* A decent amount of disk space (250GB at least). Note that SSDs will build faster than HDDs.
* A decent internet connection to sync source.
* A Linux distro environment (Personally I recommend Ubuntu 24.04 LTS).
* Familiarity with basic shell commands, git and version control.

Lower specs than this could take longer time to build.

### Note:

You can still make use of RBE for a low spec device using this guide: https://fosson.top/aosp/remote-build-execution/

To continue setting up the build environment, follow the instructions:

```bash
# Enter Superuser.
sudo su

# Install JDK 
add-apt-repository ppa:openjdk-r/ppa

# Update all packages.
apt-get update

# Install necessary packages.
apt-get install git-core gnupg flex bison build-essential zip curl zlib1g-dev gcc-multilib g++-multilib libc6-dev-i386 lib32ncurses5-dev x11proto-core-dev libx11-dev lib32z1-dev libgl1-mesa-dev libxml2-utils xsltproc unzip fontconfig

exit

# Create a bin folder and set up using AkhilNarang Script.

mkdir ~/bin

PATH=~/bin:$PATH

cd ~/bin

curl http://commondatastorage.googleapis.com/git-repo-downloads/repo > ~/bin/repo

chmod a+x ~/bin/repo

git clone https://github.com/akhilnarang/scripts.git scripts

cd scripts

bash setup/android_build_env.sh
```

You can refer the AOSP method to do this as well.

https://source.android.com/setup/build/initializing

### Step 2: Sync ROM Source

Go to the ROM's manifest repo (android in case of LineageOS) and follow their instructions to sync the source. Use --depth=1 to shallow clone unless you're doing serious source tweaks and modifications.

Eg:
```
mkdir lineage

cd lineage

repo init -u https://github.com/LineageOS/android.git -b lineage-22.1 --git-lfs --depth=1

repo sync
```
If there occurs any error with checkouts during syncing, just do this once it's finished:
```
repo sync -j1 --fail-fast
```
### Step 3: Sync Device Sources

Sync the Device's device specific tree, common tree and kernel tree.

Eg:
```
git clone https://github.com/Motorola-SM6375-Devs/android_device_motorola_bangkk.git device/motorola/bangkk

git clone https://github.com/Motorola-SM6375-Devs/android_device_motorola_sm6375-common.git device/motorola/sm6375-common

git clone https://github.com/Motorola-SM6375-Devs/android_kernel_motorola_sm6375.git kernel/motorola/sm6375
```
You'll have to clone hardware/motorola manually since we build some HALs based off of it.
```
git clone https://github.com/LineageOS/android_hardware_motorola.git hardware/motorola
```
After this you can choose to prepare the vendor tree using the extract-files.sh script inside the device specific tree, given that you have cloned your device's stock firmware dump in your build environment.

To prepare this, first clone the ROM dump to your build environment. For example, I'm cloning bangkk's dump to my build environment:
```
git clone https://dumps.tadiphone.dev/dumps/motorola/bangkk.git path/to/rom_dump
```
Then run the extract script after giving it proper permissions:
```
chmod +x device/motorola/bangkk/extract-files.py

./device/motorola/bangkk/extract-files.py path/to/rom_dump
```
Once that's finished, you can view your vendor tree at vendor/motorola/bangkk

Alternatively, you can just clone the already prepared and maintained vendor tree from [here](https://gitlab.com/Motorola-SM6375-Devs/)
```
git clone https://gitlab.com/Motorola-SM6375-Devs/proprietary_vendor_motorola_bangkk.git vendor/motorola/bangkk

git clone https://gitlab.com/Motorola-SM6375-Devs/proprietary_vendor_motorola_sm6375-common.git vendor/motorola/sm6375-common
```
### Note:

If you're building any other ROM than LineageOS, you might need to do a basic bring up for the specific ROM.

For example, If you're building PixelOS, you should modify the makefiles to adapt to PixelOS. To do that, rename the `lineage_bangkk.mk` inside device specific tree to `aosp_bangkk.mk`, and open the file and change lineage prefixes to aosp.

To find out what prefixes to use for different ROMs, you can refer their manifest or official devices org for simplicity.

Do the same with AndroidProducts.mk and BoardConfigCommon.mk (in common tree) as well. And you will face errors if you've missed any, or done anything wrong. Fix it accordingly.

### Step 4: Time to build

Assuming you've done everything right and reached this step, let's proceed.

First we need to include the environment setup.
```
. build/envsetup.sh
```
Now we need to lunch our target.
```
lunch lineage_bangkk-bp2a-userdebug
```
Lunch target can vary depending on ROMs too.

Eg: For PixelOS,
```
lunch aosp_bangkk-bp2a-userdebug
```
### Note:

The release variant "ap3a" can change with source updates such as security patch merges or QPR merges. Look that up accordingly in the ROM's source.

You can enable CCACHE to enable caching and speeding up the build. Here I'm using 50GB for CCACHE.
```
export USE_CCACHE=1 && ccache -M 50G
```
And finally make the ROM!
```
mka bacon -j12
```
You can set different `-j` values depending on how many threads you want to use.

Happy building!!!

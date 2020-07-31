# thinkpad_trackpoint_linuxkernel

# NOTE: The patch below is now part of the mainline for all modern linux distributions. You should no longer need to patch the Lenovo trackpoint drives.


How to compile the Linux kernel using the trackpoint drivers for the new thinkpad series on Ubuntu. This has been tested on ubuntu 14.04 on a Thinkpad T450s. YMMV

DISCLAIMER: When updating your kernel, you might make your system unbootable, lose data, or end the universe. I claim no responsibility if you suffer any of these fates. Additionally, updating to a custom kernel will mean that using ```apt-get dist-upgrade``` will no longer work as expected (it'll wipe what is outlined below and put you back on an Ubuntu stock kernel). The Ubuntu kernel wiki is also useful: https://wiki.ubuntu.com/KernelTeam/GitKernelBuild

We'll be doing everything in the terminal. First fetch required packages to build the kernel:

```bash
sudo apt-get install build-essential kernel-package fakeroot libncurses5-dev
```

Pick what version of the kernel you want to download from https://www.kernel.org/ . For this I chose 3.18.9. Use wget to fetch it and extract.

```bash
wget https://www.kernel.org/pub/linux/kernel/v3.x/linux-3.18.9.tar.xz
tar -xvf linux-3.18.9.tar.xz
cd linux-3.18.9
```

Additionally, you'll have to grab the patch for the trackpoint drivers:

```bash
curl https://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/patch/?id=b314acaccd7e0d55314d96be4a33b5f50d0b3344 > trackpoint.patch
patch -Np1 < trackpoint.patch

```

When running the patch command, you should see a bunch of messages about the synaptics.c file being updated. I get a warning saying "2 out of 2 hunks ignored" at the end, but the drivers still worked for me. Next, we're going to use the configuration for your current kernel compile when compiling the new one:

```bash
cp /boot/config-`uname -r` .config
yes '' | make oldconfig
make clean
```

Compile the kernel (this takes a long time -- the command below will use 2 threads to make it faster, but it'll still take on the order of an hour or two). You can change the LOCALVERSION argument if you want -- it will append the string "-trackpoint" to your linux kernel uname so it's clear you're using a custom kernel compiled to make the new trackpoint work:

```bash
make -j 2 deb-pkg LOCALVERSION=-trackpoint
```

Now that it's compiled, install it:

```bash
cd ..
sudo dpkg -i linux-image-*.deb
sudo dpkg -i linux-headers-*.deb
sudo reboot

```



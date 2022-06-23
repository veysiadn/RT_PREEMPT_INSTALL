# RT_PREEMPT_INSTALL
  This repository contains installation of RT_PREEMPT patch on Ubuntu 20.04.4 LTS ; kernel 5.9.1. If you want to install with different kernel, installation steps are same, you just have to download your desired kernel sources from [Linux Kernel Sources](https://mirrors.edge.kernel.org/pub/linux/kernel/) and [RT_PREEMPT Patch Sources](https://mirrors.edge.kernel.org/pub/linux/kernel/projects/rt/). Hope this repository will save time for you.
#### Note that, if you want better real-time performance you will need to change several BIOS settings. It is recommended to disable management related settings. Mostly this settings scales your CPU frequency based on the load. However to keep constant periodicity it is important to have stable frequency. Please change BIOS settings after doing a brief search about the settings, do not follow advices blindly. Each hardware has different BIOS settings, therefore, I can not know all possible settings. Keep in mind that if you have latency in your implementation, it is a good starting point to tweak BIOS settings.

##### Safe Boot - Disable
##### Hyper-Threading - Disable
##### System Management Mode - Disable
##### Virtualization - Disable
##### Power Management Related Settings - Disable


## Before starting to build, make sure that Safe Boot option is disabled in your BIOS settings and run commands below to get required libraries for building/installation.
```
sudo apt-get update
```   
```
sudo apt-get install git build-essential automake autoconf libtool pkg-config cmake linux-source bc kmod cpio flex -y
```
```
sudo apt-get install intltool autoconf-archive libpcre3-dev libglib2.0-dev libgtk-3-dev libxml2-utils zstd dwarves -y
```
```
sudo apt-get install libnuma-dev libssl-dev libtool libncurses5 libncurses5-dev autogen libudev-dev libelf-dev stress -y
```   
```
sudo apt-get install kernel-package fakeroot zlib1g-dev bin86 g++ bison cpufrequtils -y
```
## RT_PREEMPT patch Installation 
#### You can download kernel version from                   : [Linux Kernel Sources](https://mirrors.edge.kernel.org/pub/linux/kernel/) 
#### You can download RT_Preempt version with same kernel   : [RT_PREEMPT Patch Sources](https://mirrors.edge.kernel.org/pub/linux/kernel/projects/rt/)
#### Create source folder where you will keep all build sources. It is important to keep it tidy.
     mkdir sources
     cd sources
#### Download the kernel source and download corresponding RT patch. Note that patch has same version 5.9.1 as kernel.
     wget https://mirrors.edge.kernel.org/pub/linux/kernel/v5.x/linux-5.9.1.tar.xz
     wget https://mirrors.edge.kernel.org/pub/linux/kernel/projects/rt/5.9/patch-5.9.1-rt20.patch.xz
     xz -cd linux-5.9.1.tar.xz | tar xvf -
     cd linux-5.9.1
     xzcat ../patch-5.9.1-rt20.patch.xz | patch -p1
     sudo mv ../linux-5.9.1 /usr/src/ -f
     cd /usr/src/linux-5.9.1
     
#### Now it is time for kernel configuration. Every kernel configuration parameter might have effect on real-time performance, you must read kernel configuration parameters and what they means. A good resource is [Kernel Config Documentation](https://www.kernelconfig.io/index.html). You can search any configuration settings and learn about the dependencies.

     sudo make  menuconfig

## In menu that will show up, we select;

    Processor type and features -> Preemption Model -> Fully Preemptible Kernel (RT).
 
 Alternatively you can edit .config file, which includes all settings that you see in menuconfig window.
<!--  When measuring system latency all kernel debug options should be turned off. They require much overhead and distort the measurement result. Examples for those debug mechanism are:

DEBUG_PREEMPT

Lock Debugging (spinlocks, mutexes, etc. . . )

DEBUG_OBJECTS

… -->
<!-- 
Some of those debugging mechanisms (like lock debugging) produce a randomized overhead in a range of some micro seconds to several milliseconds depending on the kernel configuration as well as on the compile options (DEBUG_PREEMPT has a low overhead compared to Lock Debugging or DEBUG_OBJECTS).

However, in the first run of a real-time capable Linux kernel it might be advisable to use those debugging mechanisms. This helps to locate fundamental problems.
For more details : [Wiki-RT-Linux](https://wiki.linuxfoundation.org/realtime/documentation/howto/applications/preemptrt_setup/)  (RT-Linux-Wiki)
 -->
#### Note in your .config file.

    CONFIG_SYSTEM_TRUSTED_KEYS=""
#### part above should be empty, otherwise it will give an error during make process.
#### To check that;
      sudo nano .config
  and find the part with CONFIG_SYSTEM_TRUSTED_KEYS and make sure that it's empty like above. 
#### ----------------------------------------------------------------------------------------
#### OPTIONAL : You don't have to apply settings below. It will work fine without it as well.
For additional kernel configurations check [My-Xenomai-Installation](https://github.com/veysiadn/xenomai-install) (Configurations For Realtime). Just ignore ACPI settings and Xenomai related configurations and apply all other configurations, for better real-time performance. Note that only Fully Preemptible Kernel option is enough, but if you want better performance you can try those options as well.

     CONFIG_PREEMPT_RT_FULL

    CONFIG_CPU_FREQ=n

    CONFIG_CPU_IDLE=n

    CONFIG_NO_HZ_FULL=y

    CONFIG_RCU_NOCB_CPU=y
 #### ---------------------------------------------------------------------------------------
## Now we are ready for kernel compilation.
     sudo -s
     make -j4
     make && make modules && make modules_install && make install
     reboot
##### Just make sure there is no error during make and install process. If you face any error during make process, just read the error carefully, probably the solution will be in the explanation of the error. After reboot, if you see GRUB screen, select Advanced Options for Ubuntu, and select compiled RT kernel version to start.

## After reboot to make sure about installation check kernel version. 
     uname -v
### If you see newer kernel version than the 5.9.1 it means that you have to change grub menu settings.
 ```sh
     sudo nano /etc/default/grub     
 ```
 ### Change settings like in below : 
 ```
    GRUB_TIMEOUT_STYLE = menu
    GRUB_TIMEOUT = 10
 ```
 #### Save and exit, and then, 
 ```
  sudo update-grub
  sudo reboot
 ``` 
 #### After reboot, if you see GRUB screen, select Advanced Options for Ubuntu, and select compiled RT kernel version to start
 ## Then to make sure about installation, check kernel version. 
     uname -v
 ### -----------------------------------------------------------------------
 ### If your system doesn't start after building, check this thread [Compressing initramfs](https://stackoverflow.com/questions/51669724/install-rt-linux-patch-for-ubuntu) and apply steps below. 
  Restart your computer start with non-rt kernel. Open your terminal:
  ### Step 1 - Strip the kernel modules
  
    cd /lib/modules/5.9.1-rt20
    sudo find . -name *.ko -exec strip --strip-unneeded {} +

  ### Step 2 - Change the initramfs compression
  
    sudo nano /etc/initramfs-tools/initramfs.conf
  
  find COMPRESS option and change it to xz, after change it should be like below:
  
    COMPRESS=xz

  save and exit (CTRL+X and Y and Enter).
  ### Step 3 - Update initramfs
  
    sudo update-initramfs -u -k 5.9.1-rt20
    sudo update-grub2
 ### --------------------------------------------------------------------------------

# Kernel 

Building kernel manually for bananapi to enable Wifi.


#### Check kernel config: 


    zcat /proc/config.gz | grep -i <config>

#### Build kernel: 

Download from git:

    git clone https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git
or download tarball file from https://www.kernel.org/ unpack and copy the current kernel config

    cd linux/
    zcat /proc/config.gz >.config
    
change config:    
    
    make menuconfig
 To enter search mode type "/" and enter seach config.
 Change config to y and exit.

confirm changes: 

    grep -i ac100 .config
Expected results: 

    CONFIG_MFD_AC100=y
    CONFIG_RTC_DRV_AC100=y

### Build

Time is only for statistic purpose. it takes around 50-90min
     
     time make -j9
     make install
     make modules_install
     make dtbs_install
     update-initramfs -u -k <kernel-version>
     

After installation go boot dir and check if the symlinks are correct. If not create manual symlinks for the following files:

    zImage
    dtb
    

# Troubleshooting!

 wrong zImage: copy zImage manually 
   
    cp ~/linux-5.4.40/arch/arm/boot/zImage /boot/vmlinuz-5.4.40
    
symlink zImage in /boot

    ln -s /boot/vmlinuz-5.4.40 /boot/zImage

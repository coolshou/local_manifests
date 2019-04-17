## How to compile Android 7.1 for Pine A64 LTS
0. Requirement
  ```
    HDD space: 110G
    openjdk 8: apt install openjdk-8-jdk-headless
    mcopy: apt install mtools
    pigz: sudo apt install pigz
  ```

1. get repo tool from https://storage.googleapis.com/git-repo-downloads/repo
  ```
  mkdir ~/bin
  exportPATH=~/bin:$PATH
  curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
  chmod a+x ~/bin/repo
  ```
2. Create a new directory:
  ```
  mkdir android
  cd android
  ```

3. Initialize manifests:
  ```
  repo init -u https://android.googlesource.com/platform/manifest -b android-7.1.2_r11 --depth=1
  git clone https://github.com/coolshou/pine64_local_manifests -b nougat-7.1 .repo/local_manifests
  ```

4. Checkout sources:
  ```
  repo sync -c
  ```

5.  reduce jack server number to avoid out of memory issue (change setting need to restart jack server)
  ```
    > export JACK_SERVER_VM_ARGUMENTS="-Dfile.encoding=UTF-8 -XX:+TieredCompilation -Xmx4096m"
    > jack-admin kill-server
    # edit ~/.jack-server/config.properties
    # jack.server.max-service=1
    > jack-admin start-server
  ```
5.1 multi user jack server (option)
    # edit ~/.jack-settings & ~/.jack-server/config.properties
    # service port & admin port to same value

6. Bug fix: Remove duplicated WrappedAvoidBadWifiTracker class
  ```
    #edit /frameworks/base/services/tests/servicestests/src/com/android/server/ConnectivityServiceTest.java:623:
    #          Duplicate nested type WrappedAvoidBadWifiTracker
  ```

7. avoid flex error
  ```
export LC_ALL=C
  ```
  
8. Setup device for compile:
  ```
  source build/envsetup.sh
  # tulip_chihpd-eng: use for normal Android build with Launcher
  # tulip_chiphd_atv-eng: use for Android TV build with Leanback Launcher
  lunch tulip_chiphd-eng
  ```

9. Compile sources:
  ```
  make  -j$(nproc --all)
  ```

10. Create SD card image: (vendor/ayufan-pine64/pine64-common/vendorsetup.sh)
  ```
  sdcard_image pine64_android_7_1.img.gz sopine
  ```

11. Write image to SD card with etcher.

## disk format
  ```
 /dev/block/mmcblk0p1            43008   143359   100352   49M  6 FAT16                  /bootloader& kernel, ramdisk, recovery
 /dev/block/mmcblk0p2           143360  4337663  4194304    2G 83 Linux              /system
 /dev/block/mmcblk0p3         4337664  5910527  1572864  768M 83 Linux           /cache
 /dev/block/mmcblk0p4         5910528 62333951 56423424 26.9G 83 Linux           /data
  ```

## UART

The successful connection is made on the EXP connector:

[FTDI TTL-232R-RPi](http://www.digikey.com/product-search/en?keywords=TTL-232R-RPi) 3-wire cable

- Connect the Black GND wire to Pin 6 or 9 of the EXP connector.
- Connect the Yellow RX wire to Pin 7 of the EXP connector.
- Connect the Orange TX wire to Pin 8 of the EXP connector.

## Changes

I did backport minimal amount of Allwinner changes to make it run.
You can see a commit history.

## Author

Kamil Trzciński (ayufan@ayufan.eu)

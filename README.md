## How to compile Android 7.1 for Pine A64
0. Requirement
    HDD space: 110G
    openjdk 8: apt install openjdk-8-jdk-headless
    mcopy: apt install mtools

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

5.  reduce jack server number to avoid out of memory issue
    # edit ~/.jack-server/config.properties
    # jack.server.max-service=1

6. Bug fix: Remove duplicated WrappedAvoidBadWifiTracker class
    #edit /frameworks/base/services/tests/servicestests/src/
    #          com/android/server/ConnectivityServiceTest.java:623:
    #          Duplicate nested type WrappedAvoidBadWifiTracker

7. Setup device for compile:
  ```
  source build/envsetup.sh
  # tulip_chihpd-eng: use for normal Android build with Launcher
  # tulip_chiphd_atv-eng: use for Android TV build with Leanback Launcher
  lunch tulip_chiphd-eng
  ```

8. avoid flex error
  ```
export LC_ALL=C
  ```

9. Compile sources:
  ```
  make  -j$(nproc --all)
  ```

10. Create SD card image: (TODO: why disk full error?? vendor/ayufan-pine64/pine64-common/vendorsetup.sh)
  ```
  sdcard_image pine64_android_7_1.img.gz
  ```

11. Write image to SD card with Rasplex Installer (this is multiplatform tool):
  https://github.com/RasPlex/rasplex-installer/releases or use `DD`.

## disk format
/dev/block/mmcblk0p1            43008   143359   100352   49M  6 FAT16                  /bootloader
/dev/block/mmcblk0p2           143360  4337663  4194304    2G 83 Linux              /system
/dev/block/mmcblk0p3         4337664  5910527  1572864  768M 83 Linux           /cache
/dev/block/mmcblk0p4         5910528 62333951 56423424 26.9G 83 Linux           /data

## Changes

I did backport minimal amount of Allwinner changes to make it run.
You can see a commit history.

## Author

Kamil Trzciński (ayufan@ayufan.eu)

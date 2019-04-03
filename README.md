## How to compile Android 7.1 for Pine A64
0. Requirement
    HDD space: 110G
    openjdk: openjdk-8-jdk-headless

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

10. Create SD card image:
  ```
  sdcard_image pine64_android_7_1.img.gz
  ```

11. Write image to SD card with Rasplex Installer (this is multiplatform tool):
  https://github.com/RasPlex/rasplex-installer/releases or use `DD`.

## Changes

I did backport minimal amount of Allwinner changes to make it run.
You can see a commit history.

## Author

Kamil Trzciński (ayufan@ayufan.eu)

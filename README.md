Research for [Pixel 4a Battery Performance Program](https://wiki.rossmanngroup.com/wiki/Pixel_4a_Battery_Performance_Program)

## Summary

Google recently updated the Pixel 4a. The new update (`TQ3A.230805.001.S2`) drastically reduces battery life.

I tried to do some research to determine what was done but only made little progress.

However, someone who works closely with the Linux kernel did an analysis: https://social.treehouse.systems/@marcan/113914172433692339

#### What was changed?

According to https://social.treehouse.systems/@marcan/113914172433692339 :

Certain batteries seem to have been identified by Google as being defective. These batteries:

- Have had capacity reduced by half (from 3080 mAh to 1539 mAh)
- Have had charging voltage reduced from 4.44 volts to 3.95 volts
- Are now marked as "dead" by the kernel

#### Determine if battery is impacted by the new update

1. Install adb and follow the instructions to enable adb debugging on your phone: https://developer.android.com/tools/adb

2. Run this command:

   ```
   adb shell cat /sys/class/power_supply/battery/serial_number
   ```

If your serial number contains this string, it will be impacted by the battery performance update:

```
8230020501
```

If your serial number contains this string, your battery is unaffected:

```
8230015901
```

## My research

#### Build information

Latest build with changes to battery:

- `android-13.0.0_r84`

  ```
  ro.build.description=sunfish-user 13 TQ3A.230805.001.S2 12655424 release-keys
  ro.vendor.build.date=Thu Nov 14 10:07:56 UTC 2024
  ro.vendor.build.svn=66
  ```

Previous build:

- `android-13.0.0_r83`

  ```
  ro.build.description=sunfish-user 13 TQ3A.230805.001.S1 10786265 release-keys
  ro.vendor.build.date=Sat Sep  9 13:24:08 UTC 2023
  ro.vendor.build.svn=65
  ```

#### Latest binary kernel

```
commit f0e5311ad616d4c3c7a7d4580d330bb33a958cd4
Author: Jenny Ho <hsiufangho@google.com>
Date:   Tue Nov 12 15:42:03 2024 +0800

    sunfish kernel prebuilt

    Update device tree setting
    http://pa/q/topic:%22s5_chg_profile%22

    Bug: 376505184
    Signed-off-by: Jenny Ho <hsiufangho@google.com>
    (cherry picked from https://googleplex-android-review.googlesource.com/q/commit:a6920838f65a9eeb29ba45c490cd771a0951ef62)
    Merged-In: I19d6385960e495e800d9a165b0813d27b0399462
    Change-Id: I19d6385960e495e800d9a165b0813d27b0399462
```

(https://android.googlesource.com/device/google/sunfish-kernel/+/f0e5311ad616d4c3c7a7d4580d330bb33a958cd4)

- Date of change corresponds very closely with build date of latest image (`Thu Nov 14 10:07:56 UTC 2024`)
- What is `http://pa`? Some kind of internal Google link?
- `s5_chg_profile`: Charge profile? What is S5?
  - Getting my battery serial number with `adb shell cat /sys/class/power_supply/battery/serial_number`, characters 26-27 are `S5`. Is this how problematic batteries are being identified?
- `Bug: 376505184`: Is this it?: https://issuetracker.google.com/issues/376505184
- Link not publicly accessible: https://googleplex-android-review.googlesource.com/q/commit:a6920838f65a9eeb29ba45c490cd771a0951ef62

#### Latest kernel source

```
commit 6ff6ddc33f7db8b0e858c38b082dbc1fcc351071 (HEAD -> android-msm-sunfish-4.14-android13-qpr3, tag: android-13.0.0_r0.130, tag: android-13.0.0_r0.110, tag: android-13.0.0_r0.101, m/android-msm-sunfish-4.14-android13-qpr3, aosp/android-msm-sunfish-4.14-android13-qpr3)
Author: JohnnLee <johnnlee@google.com>
Date:   Thu Apr 27 14:52:27 2023 +0800

    arm64: configs: sunfish: Remove CONFIG_BT_LE and CONFIG_BT_BREDR

    Bug: 255028881
    Change-Id: Ia2b207e58da4b3f3129bf79d47eb89e632ed28d4
    Signed-off-by: JohnnLee <johnnlee@google.com>
```

(https://android.googlesource.com/kernel/msm/+/refs/heads/android-msm-sunfish-4.14-android13-qpr3)

#### Comparing changes to binary kernel

1. Setup

   ```
   git checkout android-13.0.0_r83
   lz4 -d Image.lz4 Image-s1
   git checkout android-13.0.0_r84
   lz4 -d Image.lz4 Image-s2
   ```

1. Diff

   ```
   $ diff <(strings Image-s1 | egrep -i 'battery|charger|chg.profile' | sort) <(strings Image-s2 | egrep -i 'battery|charger|chg.profile' | sort) | egrep '^<|^>'
   > 3google_battery: cannot register power supply notifer, ret=%d
   > 3google_charger: Cannot register power supply notifer, ret=%d
   > 6google_battery: time to full not available
   > 6google_battery: update debug_chg_profile:%d -> %d
   > 6google_charger: MSC_RESET: charge full in unexpected soc.
   > chg_profile_switch
   > google_debug_chg_profile
   > google,enable-switch-chg-profile
   ```

   👉 It seems these strings have been added to the s2 image but aren't in the kernel source

#### Other kernel differences

```
$ strings Image-s1 | grep entry.S
/buildbot/src/partner-android/t-dev-msm-pixel-4.14-tm-qpr3/private/msm-google/arch/arm64/kernel/entry.S

$ strings Image-s2 | grep entry.S
/usr/local/google/home/hsiufangho/android-msm-pixel-4.14-tm-qpr3/private/msm-google/arch/arm64/kernel/entry.S
```

```
$ strings Image-s1 | grep "Linux version 4"
Linux version 4.14.302-g6ff6ddc33f7d-ab10092322 (android-build@abfarm-2004-4012) (Android (7284624, based on r416183b) clang version 12.0.5 (https://android.googlesource.com/toolchain/llvm-project c935d99d7cf2016289302412d708641d52d2f7ee), LLD 12.0.5 (/buildbot/src/android/llvm-toolchain/out/llvm-project/lld c935d99d7cf2016289302412d708641d52d2f7ee)) #1 SMP PREEMPT Tue May 9 09:35:06 UTC 2023

$ strings Image-s2 | grep "Linux version 4"
Linux version 4.14.302-g92e0d94b6cba (hsiufangho@hsiufanghocloud0.c.googlers.com) (Android (7284624, based on r416183b) clang version 12.0.5 (https://android.googlesource.com/toolchain/llvm-project c935d99d7cf2016289302412d708641d52d2f7ee), LLD 12.0.5 (/buildbot/src/android/llvm-toolchain/out/llvm-project/lld c935d99d7cf2016289302412d708641d52d2f7ee)) #1 repo:t-dev-msm-pixel-4.14-tm-qpr3 SMP PREEMPT Tue Nov 12 11:1
```

## Other notes

#### Get Android source for a Google device

https://support.google.com/pixelphone/answer/13202895

ⓘ Generic instructions for future reference

1. Look up latest build for the device here: https://source.android.com/docs/setup/reference/build-numbers

   e.g. for Pixel 4a it's `android-13.0.0_r84`

1. Follow steps here to download source: https://source.android.com/docs/setup/download

## Dump of more research

See [docs/research.md](docs/research.md)

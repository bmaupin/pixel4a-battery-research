Research for [Pixel 4a Battery Performance Program](https://wiki.rossmanngroup.com/wiki/Pixel_4a_Battery_Performance_Program)

See [docs/research.md](docs/research.md) for more notes

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

## High-level notes

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

- What is `http://pa`? Some kind of internal Google link?
- `chg_profile`: Charge profile?
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
   $ diff <(strings Image-s1 | egrep -i 'battery|chg_profile' | sort) <(strings Image-s2 | egrep -i 'battery|chg_profile' | sort) | egre
   p '^<|^>'
   > 3google_battery: cannot register power supply notifer, ret=%d
   > 6google_battery: time to full not available
   > 6google_battery: update debug_chg_profile:%d -> %d
   > chg_profile_switch
   > google_debug_chg_profile
   ```

   👉 It seems these strings have been added to the s2 image but aren't in the kernel source

## Other notes

#### Get Android source for a Google device

https://support.google.com/pixelphone/answer/13202895

ⓘ Generic instructions for future reference

1. Look up latest build for the device here: https://source.android.com/docs/setup/reference/build-numbers

   e.g. for Pixel 4a it's `android-13.0.0_r84`

1. Follow steps here to download source: https://source.android.com/docs/setup/download

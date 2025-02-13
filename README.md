Research for [Pixel 4a Battery Performance Program](https://wiki.rossmanngroup.com/wiki/Pixel_4a_Battery_Performance_Program)

## Summary

Google recently updated the Pixel 4a. The new update (`TQ3A.230805.001.S2`) drastically reduces battery life but only for some devices.

#### What was changed?

👉 Google has now posted [the source code of the updates](https://android.googlesource.com/kernel/msm/+log/refs/heads/android-msm-sunfish-4.14-android13-qpr3).

[According to Google](https://support.google.com/pixelphone/answer/15701861):

> the software update reduces available battery capacity and impacts charging performance

Analysis by [androidauthority.com](https://www.androidauthority.com/pixel-4a-battery-update-explained-3522417/):

> - The Google Pixel 4a battery update does even more than we thought, reducing capacity by around 44% and halving the maximum charging speed on affected devices.
> - The update also disables features like Adaptive Charging and charging time ETAs.
> - Google made the update an Emergency Maintenance Release (EMR), which means it’s been trying to release it as quickly as possible.

Analysis by [Hector Martin](https://social.treehouse.systems/@marcan/113914172433692339):

> they lowered the max charge voltage from 4.44 V to 3.95 V

> Only the "LSN" profile got the downgrade. ... LSN is probably Lishen [en.lishen.com.cn], which are the manufacturers of the actual cells.

> Battery capacity goes down from 3080mAh to 1539mAh!

#### How do I know if I'm impacted?

1. Install adb and follow the instructions to enable adb debugging on your phone: https://developer.android.com/tools/adb

2. Run this command:

   ```
   adb shell cat /sys/class/power_supply/battery/serial_number
   ```

If your serial number contains this string, it is an LSN battery and will be negatively impacted by the battery performance update:

```
8230020501
```

If your serial number contains this string, your battery is ATL and mostly unaffected:

```
8230015901
```

According to [androidauthority.com](https://www.androidauthority.com/pixel-4a-battery-update-explained-3522417/), this is the only impact to ATL batteries:

> Google also added a check for ATL cells — if they exceed 800 cycles, a health issue will be reported ... but it won’t trigger the same mitigations.

#### What should I do if I'm impacted

Since Google believes there is enough of a problem with bad batteries to severely limit battery life, the best option is to take advantage of Google's offer to replace the battery for free if you're able to: https://support.google.com/pixelphone?p=pixel4a_battery_help

⚠️ Any other action to bypass the battery update is at your own risk. Although Google has remained silent as to the possible risk, if it has taken such drastic measures then it means the battery could catch fire or explode. If you're willing to accept the risk:

Alternatively or as a temporary measure, you can block the battery update: https://www.reddit.com/r/Pixel4a/s/ghnkFVDUX9

A longer-term solution if you're unable to replace the battery or while you're waiting for battery replacement is to install custom firmware. Lots of options are available, but LineageOS is one of the more popular options. This will also allow you implement other mitigations such as limiting the battery charge to 80%. More information here: https://news.ycombinator.com/item?id=42869207

You can also use a device such as a [Chargie](https://chargie.org/) to limit the battery charge.

#### More information

- [Google's support article](https://support.google.com/pixelphone/answer/15701861)
- [Article at Louis Rossmann's wiki](https://wiki.rossmanngroup.com/wiki/Pixel_4a_Battery_Performance_Program)
- [Technical analysis from Hector Martin](https://social.treehouse.systems/@marcan/113914172433692339)

## My research

Before Hector Martin posted [his analysis](https://social.treehouse.systems/@marcan/113914172433692339), I attempted to do my own. I didn't make much progress, but I've left my research below.

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

#### More info on Hector Martin's analysis

After reading the analysis at https://social.treehouse.systems/@marcan/113914172433692339 I was curious how he came up with it.

He confirmed my finding that the kernel was changed whereas the source has not been changed.

What I found particularly interesting is he said:

> The base device tree is unchanged, but the overlays change

Looking into this some more, the overlays appear to be here in the source:

- Affected battery: https://android.googlesource.com/kernel/msm/+/refs/heads/android-msm-sunfish-4.14-android13-qpr3/arch/arm64/boot/dts/google/batterydata-qcom-s5-SWD-LSN.dtsi
- Unaffected battery: https://android.googlesource.com/kernel/msm/+/refs/heads/android-msm-sunfish-4.14-android13-qpr3/arch/arm64/boot/dts/google/batterydata-qcom-s5-SWD-ATL.dtsi

Because the source hasn't been updated, we can't see changes to them.

However, apparently those .dtsi overlay files are used to create a dtbo.img file, and we can see that the latest kernel binaries contain such a file: https://android.googlesource.com/device/google/sunfish-kernel/+/f0e5311ad616d4c3c7a7d4580d330bb33a958cd4%5E%21/#F6

To extract the files:

```
# Get mkdtboimg.py
git clone https://android.googlesource.com/platform/system/libufdt
cd libufdt/utils

# Use mkdtboimg.py to dump device tree offsets, e.g.
python3 mkdtboimg.py dump path/to/dtbo.img | egrep "entry|size|offset"

# Use these offsets to extract device tree files, e.g.
git checkout android-13.0.0_r83
mkdir -p dtb-s1
cd dtb-s1

offsets=(288 251399 501509 751619 1001781 1251957 1501947 1753058)
sizes=(251111 250110 250110 250162 250176 249990 251111 249990)

for i in "${!offsets[@]}"; do
    dd if=../dtbo.img of=dtbo_$i.dtb bs=1 skip="${offsets[$i]}" count="${sizes[$i]}"
done

# Convert DTBO to DTS
for dtb in dtbo_*.dtb; do
    dtc -I dtb -O dts "$dtb" -o "${dtb%.dtb}.dts"
done

git checkout android-13.0.0_r84
mkdir dtb-s2
cd dtb-s2
offsets=(288 250864 501492 753069 1003645 1255222 1505864 1756320)
sizes=(250576 250628 251577 250576 251577 250642 250456 250456)

for i in "${!offsets[@]}"; do
    dd if=../dtbo.img of=dtbo_$i.dtb bs=1 skip="${offsets[$i]}" count="${sizes[$i]}"
done

# Convert DTBO to DTS
for dtb in dtbo_*.dtb; do
    dtc -I dtb -O dts "$dtb" -o "${dtb%.dtb}.dts"
done
```

Now you can see the changes to the charging voltage that Hector showed:

```
$ diff -u dtb-s1/dtbo_0.dts dtb-s2/dtbo_0.dts | grep -A 4 4255777_Google_S5_SWD_LSN_3080mAH_PM7150
                                qcom,4255777_Google_S5_SWD_LSN_3080mAH_PM7150 {
-                                       qcom,max-voltage-uv = <0x43e6d0>;
-                                       qcom,fg-cc-cv-threshold-uv = <0x43bfc0>;
+                                       qcom,max-voltage-uv = <0x3c45b0>;
+                                       qcom,fg-cc-cv-threshold-uv = <0x3c45b0>;
```

Max voltage has been reduced from `0x43e6d0` (4450000) to `0x3c45b0` (3950000)

Examining the S2 dts file, you can see a new charging profile was added (`google_debug_chg_profile`):

```
$ grep -B 1 chg-battery-capacity dtb-s2/dtbo_0.dts
                                google,chg-battery-default-capacity = <0xc08>;
                                google,chg-battery-capacity = <0xc08>;
--
                                google_debug_chg_profile {
                                        google,chg-battery-capacity = <0x604>;
```

Capacity has been reduced from `0xc08` (3080) to `0x604` (1540)

#### Other observations

I was able to see the following differences from a device with the new update to a device without it, both with affected batteries:

- `/sys/class/power_supply/battery/chg_profile_switch` appears only after the update, assumingly related to the new charging profile (more below)
- `/sys/class/power_supply/battery/health` returns `Good` before the update and `Dead` after the update
- `/sys/class/power_supply/battery/soh` ([state of health](https://en.wikipedia.org/wiki/State_of_health)) changes from `1` to `0`

## Other notes

#### Get Android source for a Google device

https://support.google.com/pixelphone/answer/13202895

ⓘ Generic instructions for future reference

1. Look up latest build for the device here: https://source.android.com/docs/setup/reference/build-numbers

   e.g. for Pixel 4a it's `android-13.0.0_r84`

1. Follow steps here to download source: https://source.android.com/docs/setup/download

## Dump of more research

See [docs/research.md](docs/research.md)

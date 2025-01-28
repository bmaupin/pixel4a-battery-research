Research for [Pixel 4a Battery Performance Program](https://wiki.rossmanngroup.com/wiki/Pixel_4a_Battery_Performance_Program)

See [docs/research.md](docs/research.md) for more notes

#### Build information

Latest build with changes to battery:

```
ro.build.description=sunfish-user 13 TQ3A.230805.001.S2 12655424 release-keys
ro.vendor.build.date=Thu Nov 14 10:07:56 UTC 2024
ro.vendor.build.svn=66
```

Previous build:

```
ro.build.description=sunfish-user 13 TQ3A.230805.001.S1 10786265 release-keys
ro.vendor.build.date=Sat Sep  9 13:24:08 UTC 2023
ro.vendor.build.svn=65
```

#### Relevant change?

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

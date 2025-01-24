Research for [Pixel 4a Battery Performance Program](https://wiki.rossmanngroup.com/wiki/Pixel_4a_Battery_Performance_Program)

#### Research changes made in latest update (TQ3A.230805.001.S2)

1. Download images from https://developers.google.com/android/images

   - 13.0.0 (TQ3A.230805.001.S1, Nov 2023)
   - 13.0.0 (TQ3A.230805.001.S2, Jan 2025)

1. Unzip packages

1. Unzip image-sunfish packages inside them and move contents to same directory

1. Do a diff to see which files are the same

   ```
   $ diff -rsq sunfish-tq3a.230805.001.s1 sunfish-tq3a.230805.001.s2 | grep identical
   Files sunfish-tq3a.230805.001.s1/android-info.txt and sunfish-tq3a.230805.001.s2/android-info.txt are identical
   Files sunfish-tq3a.230805.001.s1/bootloader-sunfish-s5-0.5-10252351.img and sunfish-tq3a.230805.001.s2/bootloader-sunfish-s5-0.5-10252351.img are identical
   Files sunfish-tq3a.230805.001.s1/flash-base.sh and sunfish-tq3a.230805.001.s2/flash-base.sh are identical
   Files sunfish-tq3a.230805.001.s1/radio-sunfish-g7150-00112-230505-b-10075601.img and sunfish-tq3a.230805.001.s2/radio-sunfish-g7150-00112-230505-b-10075601.img are identical
   Files sunfish-tq3a.230805.001.s1/super_empty.img and sunfish-tq3a.230805.001.s2/super_empty.img are identical
   ```

1. Remove identical files

   ```
   for filename in $(diff -rsq sunfish-tq3a.230805.001.s1 sunfish-tq3a.230805.001.s2 | grep identical | awk '{ print $2}' | cut -d / -f 2); do rm sunfish-tq3a.230805.001.s*/${filename}; done
   ```

1. Remove flashall.bat; only difference is version number

   ```
   $ diff sunfish-tq3a.230805.001.s1/flash-all.bat sunfish-tq3a.230805.001.s2/flash-all.bat
   23c23
   < fastboot -w update image-sunfish-tq3a.230805.001.s1.zip
   ---
   > fastboot -w update image-sunfish-tq3a.230805.001.s2.zip

   $ rm sunfish-tq3a.230805.001.s*/flash-all.bat
   # Same for flash-all.sh
   $ rm sunfish-tq3a.230805.001.s*/flash-all.sh
   ```

1. Get file types of remaining (`.img`) files

   ```
   $ file *
   boot.img:          Android bootimg, kernel (0x8000), ramdisk (0x1000000), page size: 4096, cmdline (console=ttyMSM0,115200n8 androidboot.console=ttyMSM0 printk.devkmsg=on    msm_rtb.filter=0x237 ehci-hcd.park=3 service_locator.ena)
   dtbo.img:          data
   product.img:       Linux rev 1.0 ext2 filesystem data, UUID=51f47b67-03cc-5c75-b82b-be4fe46bf51a, volume name "product" (extents) (large files) (huge files)
   system_ext.img:    Linux rev 1.0 ext2 filesystem data, UUID=3f03299a-4175-53fd-9041-1f6ac7a6973b, volume name "system_ext" (extents) (large files) (huge files)
   system.img:        Linux rev 1.0 ext2 filesystem data, UUID=696629ba-ba9e-55d6-9890-51c386547520 (extents) (large files) (huge files)
   system_other.img:  Linux rev 1.0 ext2 filesystem data, UUID=346dcb6d-2e5a-5168-bcc1-7e535dd632f8, volume name "system" (extents) (large files) (huge files)
   vbmeta.img:        data
   vbmeta_system.img: data
   vendor.img:        Linux rev 1.0 ext2 filesystem data, UUID=42a88ff6-b674-5c60-a509-3205fda18b6c, volume name "vendor" (extents) (large files) (huge files)
   ```

1. Start with smallest image first

   ```
   $ ls -lSrh
   total 4.3G
   -rw-r--r-- 1 user user 4.0K Jan  1  2009 vbmeta_system.img
   -rw-r--r-- 1 user user 8.0K Jan  1  2009 vbmeta.img
   -rw-r--r-- 1 user user 8.0M Jan  1  2009 dtbo.img
   -rw-r--r-- 1 user user  26M Jan  1  2009 system_other.img
   -rw-r--r-- 1 user user  64M Jan  1  2009 boot.img
   -rw-r--r-- 1 user user 312M Jan  1  2009 system_ext.img
   -rw-r--r-- 1 user user 618M Jan  1  2009 vendor.img
   -rw-r--r-- 1 user user 826M Jan  1  2009 system.img
   -rw-r--r-- 1 user user 2.5G Jan  1  2009 product.img
   ```

1. Apparently vbmeta files are for AVB; we can ignore them for now. From ChatGPT:

   > The vbmeta_system.img file is not a regular disk image file like ext4 or f2fs. It is part of the Android Verified Boot (AVB) system and contains metadata for verifying the integrity of partitions.

1. I think we can ignore dtbo.img too

   > The dtbo.img file is the Device Tree Blob Overlay (DTBO) image, which is used in Android devices to provide device-specific configuration for the kernel at boot time. It works in conjunction with the Device Tree Blob (DTB) and allows the device's kernel to load additional or modified hardware-specific configurations without modifying the main device tree.

1. Let's compare system_other.img now

   1. Mount the images

      ```
      sudo mkdir /mnt/system_other-s1
      sudo mkdir /mnt/system_other-s2
      sudo mount -o loop,ro sunfish-tq3a.230805.001.s1/system_other.img /mnt/system_other-s1
      sudo mount -o loop,ro sunfish-tq3a.230805.001.s2/system_other.img /mnt/system_other-s2
      ```

      👉 `ro` (read-only) is important because otherwise it will fail to mount:

      ```
      $ sudo mount sunfish-tq3a.230805.001.s1/system_other.img /mnt/system_other-s1/
      mount: /mnt/system_other-s1: wrong fs type, bad option, bad superblock on /dev/loop19, missing codepage or helper program, or other error.
              dmesg(1) may have more information after failed mount system call.
      ```

   1. Compare the images; no difference:

      ```
      $ sudo diff -r /mnt/system_other-s1 /mnt/system_other-s2
      ```

   1. Cleanup

      ```
      sudo umount /mnt/system_other-s1
      sudo umount /mnt/system_other-s2
      sudo rmdir /mnt/system_other-s1
      sudo rmdir /mnt/system_other-s2
      ```

1. Repeat for other images

   - product.img

     ```
     $ sudo diff -rq /mnt/product-s1 /mnt/product-s2 2>/dev/null
     Files /mnt/product-s1/app/CalculatorGooglePrebuilt/CalculatorGooglePrebuilt.apk and /mnt/product-s2/app/CalculatorGooglePrebuilt/CalculatorGooglePrebuilt.apk differ
     Files /mnt/product-s1/app/CalendarGooglePrebuilt/CalendarGooglePrebuilt.apk and /mnt/product-s2/app/CalendarGooglePrebuilt/CalendarGooglePrebuilt.apk differ
     Files /mnt/product-s1/app/Drive/Drive.apk and /mnt/product-s2/app/Drive/Drive.apk differ
     Files /mnt/product-s1/app/GoogleCamera/GoogleCamera.apk and /mnt/product-s2/app/GoogleCamera/GoogleCamera.apk differ
     Files /mnt/product-s1/app/GoogleContacts/GoogleContacts.apk and /mnt/product-s2/app/GoogleContacts/GoogleContacts.apk differ
     Files /mnt/product-s1/app/GoogleTTS/GoogleTTS.apk and /mnt/product-s2/app/GoogleTTS/GoogleTTS.apk differ
     Files /mnt/product-s1/app/LatinIMEGooglePrebuilt/LatinIMEGooglePrebuilt.apk and /mnt/product-s2/app/LatinIMEGooglePrebuilt/LatinIMEGooglePrebuilt.apk differ
     Files /mnt/product-s1/app/Maps/Maps.apk and /mnt/product-s2/app/Maps/Maps.apk differ
     Files /mnt/product-s1/app/MarkupGoogle/MarkupGoogle.apk and /mnt/product-s2/app/MarkupGoogle/MarkupGoogle.apk differ
     Files /mnt/product-s1/app/ModuleMetadataGoogle/ModuleMetadataGoogle.apk and /mnt/product-s2/app/ModuleMetadataGoogle/ModuleMetadataGoogle.apk differ
     Files /mnt/product-s1/app/NexusWallpapersStubPrebuilt2020_midyear/NexusWallpapersStubPrebuilt2020_midyear.apk and /mnt/product-s2/app/NexusWallpapersStubPrebuilt2020_midyear/NexusWallpapersStubPrebuilt2020_midyear.apk differ
     Files /mnt/product-s1/app/NgaResources/NgaResources.apk and /mnt/product-s2/app/NgaResources/NgaResources.apk differ
     Files /mnt/product-s1/app/Photos/Photos.apk and /mnt/product-s2/app/Photos/Photos.apk differ
     Files /mnt/product-s1/app/PixelCameraServicesSunfish/PixelCameraServicesSunfish.apk and /mnt/product-s2/app/PixelCameraServicesSunfish/PixelCameraServicesSunfish.apk differ
     Files /mnt/product-s1/app/PixelThemesStub/PixelThemesStub.apk and /mnt/product-s2/app/PixelThemesStub/PixelThemesStub.apk differ
     Files /mnt/product-s1/app/PlayAutoInstallConfig/PlayAutoInstallConfig.apk and /mnt/product-s2/app/PlayAutoInstallConfig/PlayAutoInstallConfig.apk differ
     Files /mnt/product-s1/app/PrebuiltDeskClockGoogle/PrebuiltDeskClockGoogle.apk and /mnt/product-s2/app/PrebuiltDeskClockGoogle/PrebuiltDeskClockGoogle.apk differ
     Files /mnt/product-s1/app/PrebuiltGmail/PrebuiltGmail.apk and /mnt/product-s2/app/PrebuiltGmail/PrebuiltGmail.apk differ
     Files /mnt/product-s1/app/PrebuiltGoogleAdservicesTvp/PrebuiltGoogleAdservicesTvp.apk and /mnt/product-s2/app/PrebuiltGoogleAdservicesTvp/PrebuiltGoogleAdservicesTvp.apk differ
     Files /mnt/product-s1/app/PrebuiltGoogleTelemetryTvp/PrebuiltGoogleTelemetryTvp.apk and /mnt/product-s2/app/PrebuiltGoogleTelemetryTvp/PrebuiltGoogleTelemetryTvp.apk differ
     Files /mnt/product-s1/app/SafetyRegulatoryInfo/SafetyRegulatoryInfo.apk and /mnt/product-s2/app/SafetyRegulatoryInfo/SafetyRegulatoryInfo.apk differ
     Files /mnt/product-s1/app/SoundAmplifierPrebuilt/SoundAmplifierPrebuilt.apk and /mnt/product-s2/app/SoundAmplifierPrebuilt/SoundAmplifierPrebuilt.apk differ
     Files /mnt/product-s1/app/SoundPickerPrebuilt/SoundPickerPrebuilt.apk and /mnt/product-s2/app/SoundPickerPrebuilt/SoundPickerPrebuilt.apk differ
     Files /mnt/product-s1/app/talkback/talkback.apk and /mnt/product-s2/app/talkback/talkback.apk differ
     Files /mnt/product-s1/app/Tycho/Tycho.apk and /mnt/product-s2/app/Tycho/Tycho.apk differ
     Files /mnt/product-s1/app/uimremoteclient/uimremoteclient.apk and /mnt/product-s2/app/uimremoteclient/uimremoteclient.apk differ
     Files /mnt/product-s1/app/uimremoteserver/uimremoteserver.apk and /mnt/product-s2/app/uimremoteserver/uimremoteserver.apk differ
     Files /mnt/product-s1/app/Videos/Videos.apk and /mnt/product-s2/app/Videos/Videos.apk differ
     Files /mnt/product-s1/app/WallpaperEmojiPrebuilt/WallpaperEmojiPrebuilt.apk and /mnt/product-s2/app/WallpaperEmojiPrebuilt/WallpaperEmojiPrebuilt.apk differ
     Files /mnt/product-s1/app/WallpapersBReel2020a/WallpapersBReel2020a.apk and /mnt/product-s2/app/WallpapersBReel2020a/WallpapersBReel2020a.apk differ
     Files /mnt/product-s1/app/xdivert/xdivert.apk and /mnt/product-s2/app/xdivert/xdivert.apk differ
     Files /mnt/product-s1/app/YouTube/YouTube.apk and /mnt/product-s2/app/YouTube/YouTube.apk differ
     Files /mnt/product-s1/app/YouTubeMusicPrebuilt/YouTubeMusicPrebuilt.apk and /mnt/product-s2/app/YouTubeMusicPrebuilt/YouTubeMusicPrebuilt.apk differ
     Files /mnt/product-s1/etc/build.prop and /mnt/product-s2/etc/build.prop differ
     Files /mnt/product-s1/etc/NOTICE.xml.gz and /mnt/product-s2/etc/NOTICE.xml.gz differ
     Files /mnt/product-s1/overlay/BuiltInPrintService__auto_generated_rro_product.apk and /mnt/product-s2/overlay/BuiltInPrintService__auto_generated_rro_product.apk differ
     Files /mnt/product-s1/overlay/CaptivePortalLoginOverlay/CaptivePortalLoginOverlay.apk and /mnt/product-s2/overlay/CaptivePortalLoginOverlay/CaptivePortalLoginOverlay.apk differ
     Files /mnt/product-s1/overlay/CellBroadcastServiceOverlay/CellBroadcastServiceOverlay.apk and /mnt/product-s2/overlay/CellBroadcastServiceOverlay/CellBroadcastServiceOverlay.apk differ
     Files /mnt/product-s1/overlay/ContactsProvider__auto_generated_rro_product.apk and /mnt/product-s2/overlay/ContactsProvider__auto_generated_rro_product.apk differ
     Files /mnt/product-s1/overlay/DisplayCutoutAvoidAppsInCutout/AvoidAppsInCutoutOverlay.apk and /mnt/product-s2/overlay/DisplayCutoutAvoidAppsInCutout/AvoidAppsInCutoutOverlay.apk differ
     Files /mnt/product-s1/overlay/DisplayCutoutEmulationCorner/DisplayCutoutEmulationCornerOverlay.apk and /mnt/product-s2/overlay/DisplayCutoutEmulationCorner/DisplayCutoutEmulationCornerOverlay.apk differ
     Files /mnt/product-s1/overlay/DisplayCutoutEmulationDouble/DisplayCutoutEmulationDoubleOverlay.apk and /mnt/product-s2/overlay/DisplayCutoutEmulationDouble/DisplayCutoutEmulationDoubleOverlay.apk differ
     Files /mnt/product-s1/overlay/DisplayCutoutEmulationHole/DisplayCutoutEmulationHoleOverlay.apk and /mnt/product-s2/overlay/DisplayCutoutEmulationHole/DisplayCutoutEmulationHoleOverlay.apk differ
     Files /mnt/product-s1/overlay/DisplayCutoutEmulationTall/DisplayCutoutEmulationTallOverlay.apk and /mnt/product-s2/overlay/DisplayCutoutEmulationTall/DisplayCutoutEmulationTallOverlay.apk differ
     Files /mnt/product-s1/overlay/DisplayCutoutEmulationWaterfall/DisplayCutoutEmulationWaterfallOverlay.apk and /mnt/product-s2/overlay/DisplayCutoutEmulationWaterfall/DisplayCutoutEmulationWaterfallOverlay.apk differ
     Files /mnt/product-s1/overlay/DisplayCutoutNoCutout/NoCutoutOverlay.apk and /mnt/product-s2/overlay/DisplayCutoutNoCutout/NoCutoutOverlay.apk differ
     Files /mnt/product-s1/overlay/DMService__auto_generated_rro_product.apk and /mnt/product-s2/overlay/DMService__auto_generated_rro_product.apk differ
     Files /mnt/product-s1/overlay/Flipendo__auto_generated_rro_product.apk and /mnt/product-s2/overlay/Flipendo__auto_generated_rro_product.apk differ
     Files /mnt/product-s1/overlay/FontNotoSerifSource/FontNotoSerifSourceOverlay.apk and /mnt/product-s2/overlay/FontNotoSerifSource/FontNotoSerifSourceOverlay.apk differ
     Files /mnt/product-s1/overlay/framework-res__auto_generated_rro_product.apk and /mnt/product-s2/overlay/framework-res__auto_generated_rro_product.apk differ
     Files /mnt/product-s1/overlay/GoogleConfigOverlay.apk and /mnt/product-s2/overlay/GoogleConfigOverlay.apk differ
     Files /mnt/product-s1/overlay/GooglePermissionControllerOverlay.apk and /mnt/product-s2/overlay/GooglePermissionControllerOverlay.apk differ
     Files /mnt/product-s1/overlay/GoogleWebViewOverlay.apk and /mnt/product-s2/overlay/GoogleWebViewOverlay.apk differ
     Files /mnt/product-s1/overlay/ManagedProvisioningPixelOverlay.apk and /mnt/product-s2/overlay/ManagedProvisioningPixelOverlay.apk differ
     Files /mnt/product-s1/overlay/MediaProviderOverlay/MediaProviderOverlay.apk and /mnt/product-s2/overlay/MediaProviderOverlay/MediaProviderOverlay.apk differ
     Files /mnt/product-s1/overlay/NavigationBarMode3Button/NavigationBarMode3ButtonOverlay.apk and /mnt/product-s2/overlay/NavigationBarMode3Button/NavigationBarMode3ButtonOverlay.apk differ
     Files /mnt/product-s1/overlay/NavigationBarModeGestural/NavigationBarModeGesturalOverlay.apk and /mnt/product-s2/overlay/NavigationBarModeGestural/NavigationBarModeGesturalOverlay.apk differ
     Files /mnt/product-s1/overlay/NavigationBarModeGesturalExtraWideBack/NavigationBarModeGesturalOverlayExtraWideBack.apk and /mnt/product-s2/overlay/NavigationBarModeGesturalExtraWideBack/NavigationBarModeGesturalOverlayExtraWideBack.apk differ
     Files /mnt/product-s1/overlay/NavigationBarModeGesturalNarrowBack/NavigationBarModeGesturalOverlayNarrowBack.apk and /mnt/product-s2/overlay/NavigationBarModeGesturalNarrowBack/NavigationBarModeGesturalOverlayNarrowBack.apk differ
     Files /mnt/product-s1/overlay/NavigationBarModeGesturalWideBack/NavigationBarModeGesturalOverlayWideBack.apk and /mnt/product-s2/overlay/NavigationBarModeGesturalWideBack/NavigationBarModeGesturalOverlayWideBack.apk differ
     Files /mnt/product-s1/overlay/NetworkStackOverlay.apk and /mnt/product-s2/overlay/NetworkStackOverlay.apk differ
     Files /mnt/product-s1/overlay/NfcNci__auto_generated_rro_product.apk and /mnt/product-s2/overlay/NfcNci__auto_generated_rro_product.apk differ
     Files /mnt/product-s1/overlay/PixelConfigOverlay2018.apk and /mnt/product-s2/overlay/PixelConfigOverlay2018.apk differ
     Files /mnt/product-s1/overlay/PixelConfigOverlay2019.apk and /mnt/product-s2/overlay/PixelConfigOverlay2019.apk differ
     Files /mnt/product-s1/overlay/PixelConfigOverlay2019Midyear.apk and /mnt/product-s2/overlay/PixelConfigOverlay2019Midyear.apk differ
     Files /mnt/product-s1/overlay/PixelConfigOverlayCommon.apk and /mnt/product-s2/overlay/PixelConfigOverlayCommon.apk differ
     Files /mnt/product-s1/overlay/PixelConfigOverlaySunfish.apk and /mnt/product-s2/overlay/PixelConfigOverlaySunfish.apk differ
     Files /mnt/product-s1/overlay/PixelConnectivityOverlay2020_midyear.apk and /mnt/product-s2/overlay/PixelConnectivityOverlay2020_midyear.apk differ
     Files /mnt/product-s1/overlay/PixelDocumentsUIGoogleOverlay/PixelDocumentsUIGoogleOverlay.apk and /mnt/product-s2/overlay/PixelDocumentsUIGoogleOverlay/PixelDocumentsUIGoogleOverlay.apk differ
     Files /mnt/product-s1/overlay/PixelSetupWizard__auto_generated_rro_product.apk and /mnt/product-s2/overlay/PixelSetupWizard__auto_generated_rro_product.apk differ
     Files /mnt/product-s1/overlay/PixelSetupWizardOverlay2019.apk and /mnt/product-s2/overlay/PixelSetupWizardOverlay2019.apk differ
     Files /mnt/product-s1/overlay/PixelSetupWizardOverlay.apk and /mnt/product-s2/overlay/PixelSetupWizardOverlay.apk differ
     Files /mnt/product-s1/overlay/PixelTetheringOverlay.apk and /mnt/product-s2/overlay/PixelTetheringOverlay.apk differ
     Files /mnt/product-s1/overlay/SafetyRegulatoryInfo__auto_generated_rro_product.apk and /mnt/product-s2/overlay/SafetyRegulatoryInfo__auto_generated_rro_product.apk differ
     Files /mnt/product-s1/overlay/SettingsGoogle__auto_generated_rro_product.apk and /mnt/product-s2/overlay/SettingsGoogle__auto_generated_rro_product.apk differ
     Files /mnt/product-s1/overlay/SettingsGoogleOverlaySunfish.apk and /mnt/product-s2/overlay/SettingsGoogleOverlaySunfish.apk differ
     Files /mnt/product-s1/overlay/SettingsOverlayG025J.apk and /mnt/product-s2/overlay/SettingsOverlayG025J.apk differ
     Files /mnt/product-s1/overlay/SettingsOverlayG025M.apk and /mnt/product-s2/overlay/SettingsOverlayG025M.apk differ
     Files /mnt/product-s1/overlay/SettingsOverlayG025N.apk and /mnt/product-s2/overlay/SettingsOverlayG025N.apk differ
     Files /mnt/product-s1/overlay/SettingsProvider__auto_generated_rro_product.apk and /mnt/product-s2/overlay/SettingsProvider__auto_generated_rro_product.apk differ
     Files /mnt/product-s1/overlay/SimAppDialog__auto_generated_rro_product.apk and /mnt/product-s2/overlay/SimAppDialog__auto_generated_rro_product.apk differ
     Files /mnt/product-s1/overlay/StorageManagerGoogle__auto_generated_rro_product.apk and /mnt/product-s2/overlay/StorageManagerGoogle__auto_generated_rro_product.apk differ
     Files /mnt/product-s1/overlay/SystemUIGoogle__auto_generated_rro_product.apk and /mnt/product-s2/overlay/SystemUIGoogle__auto_generated_rro_product.apk differ
     Files /mnt/product-s1/overlay/SystemUIGXOverlay.apk and /mnt/product-s2/overlay/SystemUIGXOverlay.apk differ
     Files /mnt/product-s1/overlay/Telecom__auto_generated_rro_product.apk and /mnt/product-s2/overlay/Telecom__auto_generated_rro_product.apk differ
     Files /mnt/product-s1/overlay/TelephonyProvider__auto_generated_rro_product.apk and /mnt/product-s2/overlay/TelephonyProvider__auto_generated_rro_product.apk differ
     Files /mnt/product-s1/overlay/TeleService__auto_generated_rro_product.apk and /mnt/product-s2/overlay/TeleService__auto_generated_rro_product.apk differ
     Files /mnt/product-s1/overlay/Traceur__auto_generated_rro_product.apk and /mnt/product-s2/overlay/Traceur__auto_generated_rro_product.apk differ
     Files /mnt/product-s1/overlay/WifiOverlay/PixelWifiOverlay2020_midyear.apk and /mnt/product-s2/overlay/WifiOverlay/PixelWifiOverlay2020_midyear.apk differ
     Files /mnt/product-s1/priv-app/AmbientSensePrebuilt/AmbientSensePrebuilt.apk and /mnt/product-s2/priv-app/AmbientSensePrebuilt/AmbientSensePrebuilt.apk differ
     Files /mnt/product-s1/priv-app/AmbientStreaming/AmbientStreaming.apk and /mnt/product-s2/priv-app/AmbientStreaming/AmbientStreaming.apk differ
     Files /mnt/product-s1/priv-app/CarrierLocation/CarrierLocation.apk and /mnt/product-s2/priv-app/CarrierLocation/CarrierLocation.apk differ
     Files /mnt/product-s1/priv-app/CarrierMetrics/CarrierMetrics.apk and /mnt/product-s2/priv-app/CarrierMetrics/CarrierMetrics.apk differ
     Files /mnt/product-s1/priv-app/CarrierServices/CarrierServices.apk and /mnt/product-s2/priv-app/CarrierServices/CarrierServices.apk differ
     Files /mnt/product-s1/priv-app/CarrierSettings/CarrierSettings.apk and /mnt/product-s2/priv-app/CarrierSettings/CarrierSettings.apk differ
     Files /mnt/product-s1/priv-app/CarrierWifi/CarrierWifi.apk and /mnt/product-s2/priv-app/CarrierWifi/CarrierWifi.apk differ
     Files /mnt/product-s1/priv-app/ConfigUpdater/ConfigUpdater.apk and /mnt/product-s2/priv-app/ConfigUpdater/ConfigUpdater.apk differ
     Files /mnt/product-s1/priv-app/ConnMO/ConnMO.apk and /mnt/product-s2/priv-app/ConnMO/ConnMO.apk differ
     Files /mnt/product-s1/priv-app/DCMO/DCMO.apk and /mnt/product-s2/priv-app/DCMO/DCMO.apk differ
     Files /mnt/product-s1/priv-app/DeviceIntelligenceNetworkPrebuilt/DeviceIntelligenceNetworkPrebuilt.apk and /mnt/product-s2/priv-app/DeviceIntelligenceNetworkPrebuilt/DeviceIntelligenceNetworkPrebuilt.apk differ
     Files /mnt/product-s1/priv-app/DevicePersonalizationPrebuiltPixel4/DevicePersonalizationPrebuiltPixel4.apk and /mnt/product-s2/priv-app/DevicePersonalizationPrebuiltPixel4/DevicePersonalizationPrebuiltPixel4.apk differ
     Files /mnt/product-s1/priv-app/DiagMon/DiagMon.apk and /mnt/product-s2/priv-app/DiagMon/DiagMon.apk differ
     Files /mnt/product-s1/priv-app/DiagnosticsToolPrebuilt/DiagnosticsToolPrebuilt.apk and /mnt/product-s2/priv-app/DiagnosticsToolPrebuilt/DiagnosticsToolPrebuilt.apk differ
     Files /mnt/product-s1/priv-app/DMService/DMService.apk and /mnt/product-s2/priv-app/DMService/DMService.apk differ
     Files /mnt/product-s1/priv-app/EuiccGoogle/EuiccGoogle.apk and /mnt/product-s2/priv-app/EuiccGoogle/EuiccGoogle.apk differ
     Files /mnt/product-s1/priv-app/FilesPrebuilt/FilesPrebuilt.apk and /mnt/product-s2/priv-app/FilesPrebuilt/FilesPrebuilt.apk differ
     Files /mnt/product-s1/priv-app/GCS/GCS.apk and /mnt/product-s2/priv-app/GCS/GCS.apk differ
     Files /mnt/product-s1/priv-app/GoogleDialer/GoogleDialer.apk and /mnt/product-s2/priv-app/GoogleDialer/GoogleDialer.apk differ
     Files /mnt/product-s1/priv-app/GoogleOneTimeInitializer/GoogleOneTimeInitializer.apk and /mnt/product-s2/priv-app/GoogleOneTimeInitializer/GoogleOneTimeInitializer.apk differ
     Files /mnt/product-s1/priv-app/GoogleRestorePrebuilt/GoogleRestorePrebuilt.apk and /mnt/product-s2/priv-app/GoogleRestorePrebuilt/GoogleRestorePrebuilt.apk differ
     Files /mnt/product-s1/priv-app/HardwareInfo/HardwareInfo.apk and /mnt/product-s2/priv-app/HardwareInfo/HardwareInfo.apk differ
     Files /mnt/product-s1/priv-app/HelpRtcPrebuilt/HelpRtcPrebuilt.apk and /mnt/product-s2/priv-app/HelpRtcPrebuilt/HelpRtcPrebuilt.apk differ
     Files /mnt/product-s1/priv-app/HotwordEnrollmentOKGoogleRT5514P/HotwordEnrollmentOKGoogleRT5514P.apk and /mnt/product-s2/priv-app/HotwordEnrollmentOKGoogleRT5514P/HotwordEnrollmentOKGoogleRT5514P.apk differ
     Files /mnt/product-s1/priv-app/HotwordEnrollmentXGoogleRT5514P/HotwordEnrollmentXGoogleRT5514P.apk and /mnt/product-s2/priv-app/HotwordEnrollmentXGoogleRT5514P/HotwordEnrollmentXGoogleRT5514P.apk differ
     Files /mnt/product-s1/priv-app/ImsServiceEntitlement/ImsServiceEntitlement.apk and /mnt/product-s2/priv-app/ImsServiceEntitlement/ImsServiceEntitlement.apk differ
     Files /mnt/product-s1/priv-app/KidsSupervisionStub/KidsSupervisionStub.apk and /mnt/product-s2/priv-app/KidsSupervisionStub/KidsSupervisionStub.apk differ
     Files /mnt/product-s1/priv-app/MaestroPrebuilt/MaestroPrebuilt.apk and /mnt/product-s2/priv-app/MaestroPrebuilt/MaestroPrebuilt.apk differ
     Files /mnt/product-s1/priv-app/OdadPrebuilt/OdadPrebuilt.apk and /mnt/product-s2/priv-app/OdadPrebuilt/OdadPrebuilt.apk differ
     Files /mnt/product-s1/priv-app/OemDmTrigger/OemDmTrigger.apk and /mnt/product-s2/priv-app/OemDmTrigger/OemDmTrigger.apk differ
     Files /mnt/product-s1/priv-app/PartnerSetupPrebuilt/PartnerSetupPrebuilt.apk and /mnt/product-s2/priv-app/PartnerSetupPrebuilt/PartnerSetupPrebuilt.apk differ
     Files /mnt/product-s1/priv-app/Phonesky/Phonesky.apk and /mnt/product-s2/priv-app/Phonesky/Phonesky.apk differ
     Files /mnt/product-s1/priv-app/PixelLiveWallpaperPrebuilt/PixelLiveWallpaperPrebuilt.apk and /mnt/product-s2/priv-app/PixelLiveWallpaperPrebuilt/PixelLiveWallpaperPrebuilt.apk differ
     Files /mnt/product-s1/priv-app/PrebuiltBugle/PrebuiltBugle.apk and /mnt/product-s2/priv-app/PrebuiltBugle/PrebuiltBugle.apk differ
     Files /mnt/product-s1/priv-app/PrebuiltGmsCore/app_chimera/m/PrebuiltGmsCoreSc_AdsDynamite.apk and /mnt/product-s2/priv-app/PrebuiltGmsCore/app_chimera/m/PrebuiltGmsCoreSc_AdsDynamite.apk differ
     Files /mnt/product-s1/priv-app/PrebuiltGmsCore/app_chimera/m/PrebuiltGmsCoreSc_CronetDynamite.apk and /mnt/product-s2/priv-app/PrebuiltGmsCore/app_chimera/m/PrebuiltGmsCoreSc_CronetDynamite.apk differ
     Files /mnt/product-s1/priv-app/PrebuiltGmsCore/app_chimera/m/PrebuiltGmsCoreSc_DynamiteLoader.apk and /mnt/product-s2/priv-app/PrebuiltGmsCore/app_chimera/m/PrebuiltGmsCoreSc_DynamiteLoader.apk differ
     Files /mnt/product-s1/priv-app/PrebuiltGmsCore/app_chimera/m/PrebuiltGmsCoreSc_DynamiteModulesA.apk and /mnt/product-s2/priv-app/PrebuiltGmsCore/app_chimera/m/PrebuiltGmsCoreSc_DynamiteModulesA.apk differ
     Files /mnt/product-s1/priv-app/PrebuiltGmsCore/app_chimera/m/PrebuiltGmsCoreSc_DynamiteModulesC.apk and /mnt/product-s2/priv-app/PrebuiltGmsCore/app_chimera/m/PrebuiltGmsCoreSc_DynamiteModulesC.apk differ
     Files /mnt/product-s1/priv-app/PrebuiltGmsCore/app_chimera/m/PrebuiltGmsCoreSc_GoogleCertificates.apk and /mnt/product-s2/priv-app/PrebuiltGmsCore/app_chimera/m/PrebuiltGmsCoreSc_GoogleCertificates.apk differ
     Files /mnt/product-s1/priv-app/PrebuiltGmsCore/app_chimera/m/PrebuiltGmsCoreSc_MapsDynamite.apk and /mnt/product-s2/priv-app/PrebuiltGmsCore/app_chimera/m/PrebuiltGmsCoreSc_MapsDynamite.apk differ
     Files /mnt/product-s1/priv-app/PrebuiltGmsCore/app_chimera/m/PrebuiltGmsCoreSc_MeasurementDynamite.apk and /mnt/product-s2/priv-app/PrebuiltGmsCore/app_chimera/m/PrebuiltGmsCoreSc_MeasurementDynamite.apk differ
     Files /mnt/product-s1/priv-app/PrebuiltGmsCore/PrebuiltGmsCoreSc.apk and /mnt/product-s2/priv-app/PrebuiltGmsCore/PrebuiltGmsCoreSc.apk differ
     Files /mnt/product-s1/priv-app/RecorderPrebuilt/RecorderPrebuilt.apk and /mnt/product-s2/priv-app/RecorderPrebuilt/RecorderPrebuilt.apk differ
     Files /mnt/product-s1/priv-app/SafetyHubPrebuilt/SafetyHubPrebuilt.apk and /mnt/product-s2/priv-app/SafetyHubPrebuilt/SafetyHubPrebuilt.apk differ
     Files /mnt/product-s1/priv-app/SCONE/SCONE.apk and /mnt/product-s2/priv-app/SCONE/SCONE.apk differ
     Files /mnt/product-s1/priv-app/ScribePrebuilt/ScribePrebuilt.apk and /mnt/product-s2/priv-app/ScribePrebuilt/ScribePrebuilt.apk differ
     Files /mnt/product-s1/priv-app/SecurityHubPrebuilt/SecurityHubPrebuilt.apk and /mnt/product-s2/priv-app/SecurityHubPrebuilt/SecurityHubPrebuilt.apk differ
     Files /mnt/product-s1/priv-app/SettingsIntelligenceGooglePrebuilt/SettingsIntelligenceGooglePrebuilt.apk and /mnt/product-s2/priv-app/SettingsIntelligenceGooglePrebuilt/SettingsIntelligenceGooglePrebuilt.apk differ
     Files /mnt/product-s1/priv-app/SetupWizardPrebuilt/SetupWizardPrebuilt.apk and /mnt/product-s2/priv-app/SetupWizardPrebuilt/SetupWizardPrebuilt.apk differ
     Files /mnt/product-s1/priv-app/SprintDM/SprintDM.apk and /mnt/product-s2/priv-app/SprintDM/SprintDM.apk differ
     Files /mnt/product-s1/priv-app/SSRestartDetector/SSRestartDetector.apk and /mnt/product-s2/priv-app/SSRestartDetector/SSRestartDetector.apk differ
     Files /mnt/product-s1/priv-app/TetheringEntitlement/TetheringEntitlement.apk and /mnt/product-s2/priv-app/TetheringEntitlement/TetheringEntitlement.apk differ
     Files /mnt/product-s1/priv-app/TipsPrebuilt/TipsPrebuilt.apk and /mnt/product-s2/priv-app/TipsPrebuilt/TipsPrebuilt.apk differ
     Files /mnt/product-s1/priv-app/TurboPrebuilt/TurboPrebuilt.apk and /mnt/product-s2/priv-app/TurboPrebuilt/TurboPrebuilt.apk differ
     Files /mnt/product-s1/priv-app/USCCDM/USCCDM.apk and /mnt/product-s2/priv-app/USCCDM/USCCDM.apk differ
     Files /mnt/product-s1/priv-app/Velvet/Velvet.apk and /mnt/product-s2/priv-app/Velvet/Velvet.apk differ
     Files /mnt/product-s1/priv-app/WellbeingPrebuilt/WellbeingPrebuilt.apk and /mnt/product-s2/priv-app/WellbeingPrebuilt/WellbeingPrebuilt.apk differ
     Files /mnt/product-s1/priv-app/WfcActivation/WfcActivation.apk and /mnt/product-s2/priv-app/WfcActivation/WfcActivation.apk differ
     ```

   - system.img

     ```
     $ sudo diff -rq /mnt/system-s1 /mnt/system-s2 2>/dev/null
     Files /mnt/system-s1/system/apex/com.android.i18n.apex and /mnt/system-s2/system/apex/com.android.i18n.apex differ
     Files /mnt/system-s1/system/apex/com.android.runtime.apex and /mnt/system-s2/system/apex/com.android.runtime.apex differ
     Files /mnt/system-s1/system/apex/com.android.vndk.current.apex and /mnt/system-s2/system/apex/com.android.vndk.current.apex differ
     Files /mnt/system-s1/system/apex/com.google.android.adbd.apex and /mnt/system-s2/system/apex/com.google.android.adbd.apex differ
     Files /mnt/system-s1/system/apex/com.google.android.adservices.apex and /mnt/system-s2/system/apex/com.google.android.adservices.apex differ
     Files /mnt/system-s1/system/apex/com.google.android.appsearch.apex and /mnt/system-s2/system/apex/com.google.android.appsearch.apex differ
     Files /mnt/system-s1/system/apex/com.google.android.art.apex and /mnt/system-s2/system/apex/com.google.android.art.apex differ
     Files /mnt/system-s1/system/apex/com.google.android.btservices.apex and /mnt/system-s2/system/apex/com.google.android.btservices.apex differ
     Files /mnt/system-s1/system/apex/com.google.android.cellbroadcast.apex and /mnt/system-s2/system/apex/com.google.android.cellbroadcast.apex differ
     Files /mnt/system-s1/system/apex/com.google.android.conscrypt.apex and /mnt/system-s2/system/apex/com.google.android.conscrypt.apex differ
     Files /mnt/system-s1/system/apex/com.google.android.extservices.apex and /mnt/system-s2/system/apex/com.google.android.extservices.apex differ
     Files /mnt/system-s1/system/apex/com.google.android.ipsec.apex and /mnt/system-s2/system/apex/com.google.android.ipsec.apex differ
     Files /mnt/system-s1/system/apex/com.google.android.media.apex and /mnt/system-s2/system/apex/com.google.android.media.apex differ
     Files /mnt/system-s1/system/apex/com.google.android.mediaprovider.apex and /mnt/system-s2/system/apex/com.google.android.mediaprovider.apex differ
     Files /mnt/system-s1/system/apex/com.google.android.media.swcodec.apex and /mnt/system-s2/system/apex/com.google.android.media.swcodec.apex differ
     Files /mnt/system-s1/system/apex/com.google.android.neuralnetworks.apex and /mnt/system-s2/system/apex/com.google.android.neuralnetworks.apex differ
     Files /mnt/system-s1/system/apex/com.google.android.ondevicepersonalization.apex and /mnt/system-s2/system/apex/com.google.android.ondevicepersonalization.apex differ
     Files /mnt/system-s1/system/apex/com.google.android.os.statsd.apex and /mnt/system-s2/system/apex/com.google.android.os.statsd.apex differ
     Files /mnt/system-s1/system/apex/com.google.android.permission.apex and /mnt/system-s2/system/apex/com.google.android.permission.apex differ
     Files /mnt/system-s1/system/apex/com.google.android.resolv.apex and /mnt/system-s2/system/apex/com.google.android.resolv.apex differ
     Files /mnt/system-s1/system/apex/com.google.android.scheduling.apex and /mnt/system-s2/system/apex/com.google.android.scheduling.apex differ
     Files /mnt/system-s1/system/apex/com.google.android.sdkext.apex and /mnt/system-s2/system/apex/com.google.android.sdkext.apex differ
     Files /mnt/system-s1/system/apex/com.google.android.tethering.apex and /mnt/system-s2/system/apex/com.google.android.tethering.apex differ
     Files /mnt/system-s1/system/apex/com.google.android.tzdata4.apex and /mnt/system-s2/system/apex/com.google.android.tzdata4.apex differ
     Files /mnt/system-s1/system/apex/com.google.android.uwb.apex and /mnt/system-s2/system/apex/com.google.android.uwb.apex differ
     Files /mnt/system-s1/system/apex/com.google.android.wifi.apex and /mnt/system-s2/system/apex/com.google.android.wifi.apex differ
     Files /mnt/system-s1/system/apex/com.google.mainline.primary.libs.apex and /mnt/system-s2/system/apex/com.google.mainline.primary.libs.apex differ
     Files /mnt/system-s1/system/app/BasicDreams/BasicDreams.apk and /mnt/system-s2/system/app/BasicDreams/BasicDreams.apk differ
     Files /mnt/system-s1/system/app/BluetoothMidiService/BluetoothMidiService.apk and /mnt/system-s2/system/app/BluetoothMidiService/BluetoothMidiService.apk differ
     Files /mnt/system-s1/system/app/BookmarkProvider/BookmarkProvider.apk and /mnt/system-s2/system/app/BookmarkProvider/BookmarkProvider.apk differ
     Files /mnt/system-s1/system/app/CameraExtensionsProxy/CameraExtensionsProxy.apk and /mnt/system-s2/system/app/CameraExtensionsProxy/CameraExtensionsProxy.apk differ
     Files /mnt/system-s1/system/app/CaptivePortalLoginGoogle/CaptivePortalLoginGoogle.apk and /mnt/system-s2/system/app/CaptivePortalLoginGoogle/CaptivePortalLoginGoogle.apk differ
     Files /mnt/system-s1/system/app/CarrierDefaultApp/CarrierDefaultApp.apk and /mnt/system-s2/system/app/CarrierDefaultApp/CarrierDefaultApp.apk differ
     Files /mnt/system-s1/system/app/CertInstaller/CertInstaller.apk and /mnt/system-s2/system/app/CertInstaller/CertInstaller.apk differ
     Files /mnt/system-s1/system/app/CompanionDeviceManager/CompanionDeviceManager.apk and /mnt/system-s2/system/app/CompanionDeviceManager/CompanionDeviceManager.apk differ
     Files /mnt/system-s1/system/app/EasterEgg/EasterEgg.apk and /mnt/system-s2/system/app/EasterEgg/EasterEgg.apk differ
     Files /mnt/system-s1/system/app/GoogleBluetoothLegacyMigration/GoogleBluetoothLegacyMigration.apk and /mnt/system-s2/system/app/GoogleBluetoothLegacyMigration/GoogleBluetoothLegacyMigration.apk differ
     Files /mnt/system-s1/system/app/GoogleExtShared/GoogleExtShared.apk and /mnt/system-s2/system/app/GoogleExtShared/GoogleExtShared.apk differ
     Files /mnt/system-s1/system/app/GooglePrintRecommendationService/GooglePrintRecommendationService.apk and /mnt/system-s2/system/app/GooglePrintRecommendationService/GooglePrintRecommendationService.apk differ
     Files /mnt/system-s1/system/app/HTMLViewer/HTMLViewer.apk and /mnt/system-s2/system/app/HTMLViewer/HTMLViewer.apk differ
     Files /mnt/system-s1/system/app/KeyChain/KeyChain.apk and /mnt/system-s2/system/app/KeyChain/KeyChain.apk differ
     Files /mnt/system-s1/system/app/NfcNci/NfcNci.apk and /mnt/system-s2/system/app/NfcNci/NfcNci.apk differ
     Files /mnt/system-s1/system/app/PacProcessor/PacProcessor.apk and /mnt/system-s2/system/app/PacProcessor/PacProcessor.apk differ
     Files /mnt/system-s1/system/app/PartnerBookmarksProvider/PartnerBookmarksProvider.apk and /mnt/system-s2/system/app/PartnerBookmarksProvider/PartnerBookmarksProvider.apk differ
     Files /mnt/system-s1/system/app/PrintSpooler/PrintSpooler.apk and /mnt/system-s2/system/app/PrintSpooler/PrintSpooler.apk differ
     Files /mnt/system-s1/system/app/SecureElement/SecureElement.apk and /mnt/system-s2/system/app/SecureElement/SecureElement.apk differ
     Files /mnt/system-s1/system/app/SimAppDialog/SimAppDialog.apk and /mnt/system-s2/system/app/SimAppDialog/SimAppDialog.apk differ
     Files /mnt/system-s1/system/app/Stk/Stk.apk and /mnt/system-s2/system/app/Stk/Stk.apk differ
     Files /mnt/system-s1/system/app/Traceur/Traceur.apk and /mnt/system-s2/system/app/Traceur/Traceur.apk differ
     Files /mnt/system-s1/system/app/WallpaperBackup/WallpaperBackup.apk and /mnt/system-s2/system/app/WallpaperBackup/WallpaperBackup.apk differ
     Files /mnt/system-s1/system/build.prop and /mnt/system-s2/system/build.prop differ
     Files /mnt/system-s1/system/etc/bpf/gpu_mem.o and /mnt/system-s2/system/etc/bpf/gpu_mem.o differ
     Files /mnt/system-s1/system/etc/bpf/time_in_state.o and /mnt/system-s2/system/etc/bpf/time_in_state.o differ
     Files /mnt/system-s1/system/etc/NOTICE.xml.gz and /mnt/system-s2/system/etc/NOTICE.xml.gz differ
     Files /mnt/system-s1/system/framework/framework-res.apk and /mnt/system-s2/system/framework/framework-res.apk differ
     Files /mnt/system-s1/system/priv-app/BackupRestoreConfirmation/BackupRestoreConfirmation.apk and /mnt/system-s2/system/priv-app/BackupRestoreConfirmation/BackupRestoreConfirmation.apk differ
     Files /mnt/system-s1/system/priv-app/BlockedNumberProvider/BlockedNumberProvider.apk and /mnt/system-s2/system/priv-app/BlockedNumberProvider/BlockedNumberProvider.apk differ
     Files /mnt/system-s1/system/priv-app/BuiltInPrintService/BuiltInPrintService.apk and /mnt/system-s2/system/priv-app/BuiltInPrintService/BuiltInPrintService.apk differ
     Files /mnt/system-s1/system/priv-app/CalendarProvider/CalendarProvider.apk and /mnt/system-s2/system/priv-app/CalendarProvider/CalendarProvider.apk differ
     Files /mnt/system-s1/system/priv-app/CallLogBackup/CallLogBackup.apk and /mnt/system-s2/system/priv-app/CallLogBackup/CallLogBackup.apk differ
     Files /mnt/system-s1/system/priv-app/CellBroadcastLegacyApp/CellBroadcastLegacyApp.apk and /mnt/system-s2/system/priv-app/CellBroadcastLegacyApp/CellBroadcastLegacyApp.apk differ
     Files /mnt/system-s1/system/priv-app/ContactsProvider/ContactsProvider.apk and /mnt/system-s2/system/priv-app/ContactsProvider/ContactsProvider.apk differ
     Files /mnt/system-s1/system/priv-app/DocumentsUIGoogle/DocumentsUIGoogle.apk and /mnt/system-s2/system/priv-app/DocumentsUIGoogle/DocumentsUIGoogle.apk differ
     Files /mnt/system-s1/system/priv-app/DownloadProvider/DownloadProvider.apk and /mnt/system-s2/system/priv-app/DownloadProvider/DownloadProvider.apk differ
     Files /mnt/system-s1/system/priv-app/DownloadProviderUi/DownloadProviderUi.apk and /mnt/system-s2/system/priv-app/DownloadProviderUi/DownloadProviderUi.apk differ
     Files /mnt/system-s1/system/priv-app/DynamicSystemInstallationService/DynamicSystemInstallationService.apk and /mnt/system-s2/system/priv-app/DynamicSystemInstallationService/DynamicSystemInstallationService.apk differ
     Files /mnt/system-s1/system/priv-app/ExternalStorageProvider/ExternalStorageProvider.apk and /mnt/system-s2/system/priv-app/ExternalStorageProvider/ExternalStorageProvider.apk differ
     Files /mnt/system-s1/system/priv-app/FusedLocation/FusedLocation.apk and /mnt/system-s2/system/priv-app/FusedLocation/FusedLocation.apk differ
     Files /mnt/system-s1/system/priv-app/GooglePackageInstaller/GooglePackageInstaller.apk and /mnt/system-s2/system/priv-app/GooglePackageInstaller/GooglePackageInstaller.apk differ
     Files /mnt/system-s1/system/priv-app/InputDevices/InputDevices.apk and /mnt/system-s2/system/priv-app/InputDevices/InputDevices.apk differ
     Files /mnt/system-s1/system/priv-app/IntentResolver/IntentResolver.apk and /mnt/system-s2/system/priv-app/IntentResolver/IntentResolver.apk differ
     Files /mnt/system-s1/system/priv-app/LiveWallpapersPicker/LiveWallpapersPicker.apk and /mnt/system-s2/system/priv-app/LiveWallpapersPicker/LiveWallpapersPicker.apk differ
     Files /mnt/system-s1/system/priv-app/LocalTransport/LocalTransport.apk and /mnt/system-s2/system/priv-app/LocalTransport/LocalTransport.apk differ
     Files /mnt/system-s1/system/priv-app/ManagedProvisioning/ManagedProvisioning.apk and /mnt/system-s2/system/priv-app/ManagedProvisioning/ManagedProvisioning.apk differ
     Files /mnt/system-s1/system/priv-app/MediaProviderLegacy/MediaProviderLegacy.apk and /mnt/system-s2/system/priv-app/MediaProviderLegacy/MediaProviderLegacy.apk differ
     Files /mnt/system-s1/system/priv-app/MmsService/MmsService.apk and /mnt/system-s2/system/priv-app/MmsService/MmsService.apk differ
     Files /mnt/system-s1/system/priv-app/MtpService/MtpService.apk and /mnt/system-s2/system/priv-app/MtpService/MtpService.apk differ
     Files /mnt/system-s1/system/priv-app/MusicFX/MusicFX.apk and /mnt/system-s2/system/priv-app/MusicFX/MusicFX.apk differ
     Files /mnt/system-s1/system/priv-app/NetworkStackGoogle/NetworkStackGoogle.apk and /mnt/system-s2/system/priv-app/NetworkStackGoogle/NetworkStackGoogle.apk differ
     Files /mnt/system-s1/system/priv-app/ONS/ONS.apk and /mnt/system-s2/system/priv-app/ONS/ONS.apk differ
     Files /mnt/system-s1/system/priv-app/ProxyHandler/ProxyHandler.apk and /mnt/system-s2/system/priv-app/ProxyHandler/ProxyHandler.apk differ
     Files /mnt/system-s1/system/priv-app/SettingsProvider/SettingsProvider.apk and /mnt/system-s2/system/priv-app/SettingsProvider/SettingsProvider.apk differ
     Files /mnt/system-s1/system/priv-app/SharedStorageBackup/SharedStorageBackup.apk and /mnt/system-s2/system/priv-app/SharedStorageBackup/SharedStorageBackup.apk differ
     Files /mnt/system-s1/system/priv-app/Shell/Shell.apk and /mnt/system-s2/system/priv-app/Shell/Shell.apk differ
     Files /mnt/system-s1/system/priv-app/SoundPicker/SoundPicker.apk and /mnt/system-s2/system/priv-app/SoundPicker/SoundPicker.apk differ
     Files /mnt/system-s1/system/priv-app/TagGoogle/TagGoogle.apk and /mnt/system-s2/system/priv-app/TagGoogle/TagGoogle.apk differ
     Files /mnt/system-s1/system/priv-app/Telecom/Telecom.apk and /mnt/system-s2/system/priv-app/Telecom/Telecom.apk differ
     Files /mnt/system-s1/system/priv-app/TelephonyProvider/TelephonyProvider.apk and /mnt/system-s2/system/priv-app/TelephonyProvider/TelephonyProvider.apk differ
     Files /mnt/system-s1/system/priv-app/TeleService/TeleService.apk and /mnt/system-s2/system/priv-app/TeleService/TeleService.apk differ
     Files /mnt/system-s1/system/priv-app/UserDictionaryProvider/UserDictionaryProvider.apk and /mnt/system-s2/system/priv-app/UserDictionaryProvider/UserDictionaryProvider.apk differ
     Files /mnt/system-s1/system/priv-app/VpnDialogs/VpnDialogs.apk and /mnt/system-s2/system/priv-app/VpnDialogs/VpnDialogs.apk differ
     Files /mnt/system-s1/system/system_dlkm/etc/build.prop and /mnt/system-s2/system/system_dlkm/etc/build.prop differ
     ```

   - system_ext.img

     ```
     $ sudo diff -rq /mnt/system_ext-s1 /mnt/system_ext-s2 2>/dev/null
     Files /mnt/system_ext-s1/app/atfwd/atfwd.apk and /mnt/system_ext-s2/app/atfwd/atfwd.apk differ
     Files /mnt/system_ext-s1/app/com.qualcomm.qti.services.secureui/com.qualcomm.qti.services.secureui.apk and /mnt/system_ext-s2/app/com.qualcomm.qti.services.secureui/com.qualcomm.qti.services.secureui.apk differ
     Files /mnt/system_ext-s1/app/datastatusnotification/datastatusnotification.apk and /mnt/system_ext-s2/app/datastatusnotification/datastatusnotification.apk differ
     Files /mnt/system_ext-s1/app/EmergencyInfoGoogleNoUi/EmergencyInfoGoogleNoUi.apk and /mnt/system_ext-s2/app/EmergencyInfoGoogleNoUi/EmergencyInfoGoogleNoUi.apk differ
     Files /mnt/system_ext-s1/app/Flipendo/Flipendo.apk and /mnt/system_ext-s2/app/Flipendo/Flipendo.apk differ
     Files /mnt/system_ext-s1/app/PresencePolling/PresencePolling.apk and /mnt/system_ext-s2/app/PresencePolling/PresencePolling.apk differ
     Files /mnt/system_ext-s1/app/QtiTelephonyService/QtiTelephonyService.apk and /mnt/system_ext-s2/app/QtiTelephonyService/QtiTelephonyService.apk differ
     Files /mnt/system_ext-s1/app/RcsService/RcsService.apk and /mnt/system_ext-s2/app/RcsService/RcsService.apk differ
     Files /mnt/system_ext-s1/app/uceShimService/uceShimService.apk and /mnt/system_ext-s2/app/uceShimService/uceShimService.apk differ
     Files /mnt/system_ext-s1/etc/build.prop and /mnt/system_ext-s2/etc/build.prop differ
     Files /mnt/system_ext-s1/etc/NOTICE.xml.gz and /mnt/system_ext-s2/etc/NOTICE.xml.gz differ
     Files /mnt/system_ext-s1/priv-app/CarrierSetup/CarrierSetup.apk and /mnt/system_ext-s2/priv-app/CarrierSetup/CarrierSetup.apk differ
     Files /mnt/system_ext-s1/priv-app/EuiccGoogleOverlay/EuiccGoogleOverlay.apk and /mnt/system_ext-s2/priv-app/EuiccGoogleOverlay/EuiccGoogleOverlay.apk differ
     Files /mnt/system_ext-s1/priv-app/EuiccSupportPixel/EuiccSupportPixel.apk and /mnt/system_ext-s2/priv-app/EuiccSupportPixel/EuiccSupportPixel.apk differ
     Files /mnt/system_ext-s1/priv-app/EuiccSupportPixelPermissions/EuiccSupportPixelPermissions.apk and /mnt/system_ext-s2/priv-app/EuiccSupportPixelPermissions/EuiccSupportPixelPermissions.apk differ
     Files /mnt/system_ext-s1/priv-app/GoogleFeedback/GoogleFeedback.apk and /mnt/system_ext-s2/priv-app/GoogleFeedback/GoogleFeedback.apk differ
     Files /mnt/system_ext-s1/priv-app/GoogleServicesFramework/GoogleServicesFramework.apk and /mnt/system_ext-s2/priv-app/GoogleServicesFramework/GoogleServicesFramework.apk differ
     Files /mnt/system_ext-s1/priv-app/grilservice/grilservice.apk and /mnt/system_ext-s2/priv-app/grilservice/grilservice.apk differ
     Files /mnt/system_ext-s1/priv-app/HbmSVManager/HbmSVManager.apk and /mnt/system_ext-s2/priv-app/HbmSVManager/HbmSVManager.apk differ
     Files /mnt/system_ext-s1/priv-app/ims/ims.apk and /mnt/system_ext-s2/priv-app/ims/ims.apk differ
     Files /mnt/system_ext-s1/priv-app/NexusLauncherRelease/NexusLauncherRelease.apk and /mnt/system_ext-s2/priv-app/NexusLauncherRelease/NexusLauncherRelease.apk differ
     Files /mnt/system_ext-s1/priv-app/PixelNfc/PixelNfc.apk and /mnt/system_ext-s2/priv-app/PixelNfc/PixelNfc.apk differ
     Files /mnt/system_ext-s1/priv-app/PixelSetupWizard/PixelSetupWizard.apk and /mnt/system_ext-s2/priv-app/PixelSetupWizard/PixelSetupWizard.apk differ
     Files /mnt/system_ext-s1/priv-app/qcrilmsgtunnel/qcrilmsgtunnel.apk and /mnt/system_ext-s2/priv-app/qcrilmsgtunnel/qcrilmsgtunnel.apk differ
     Files /mnt/system_ext-s1/priv-app/QuickAccessWallet/QuickAccessWallet.apk and /mnt/system_ext-s2/priv-app/QuickAccessWallet/QuickAccessWallet.apk differ
     Files /mnt/system_ext-s1/priv-app/RilConfigService/RilConfigService.apk and /mnt/system_ext-s2/priv-app/RilConfigService/RilConfigService.apk differ
     Files /mnt/system_ext-s1/priv-app/SettingsGoogle/oat/arm64/SettingsGoogle.art and /mnt/system_ext-s2/priv-app/SettingsGoogle/oat/arm64/SettingsGoogle.art differ
     Files /mnt/system_ext-s1/priv-app/SettingsGoogle/oat/arm64/SettingsGoogle.odex and /mnt/system_ext-s2/priv-app/SettingsGoogle/oat/arm64/SettingsGoogle.odex differ
     Files /mnt/system_ext-s1/priv-app/SettingsGoogle/oat/arm64/SettingsGoogle.vdex and /mnt/system_ext-s2/priv-app/SettingsGoogle/oat/arm64/SettingsGoogle.vdex differ
     Files /mnt/system_ext-s1/priv-app/SettingsGoogle/SettingsGoogle.apk and /mnt/system_ext-s2/priv-app/SettingsGoogle/SettingsGoogle.apk differ
     Files /mnt/system_ext-s1/priv-app/StorageManagerGoogle/StorageManagerGoogle.apk and /mnt/system_ext-s2/priv-app/StorageManagerGoogle/StorageManagerGoogle.apk differ
     Files /mnt/system_ext-s1/priv-app/SystemUIGoogle/oat/arm64/SystemUIGoogle.odex and /mnt/system_ext-s2/priv-app/SystemUIGoogle/oat/arm64/SystemUIGoogle.odex differ
     Files /mnt/system_ext-s1/priv-app/SystemUIGoogle/oat/arm64/SystemUIGoogle.vdex and /mnt/system_ext-s2/priv-app/SystemUIGoogle/oat/arm64/SystemUIGoogle.vdex differ
     Files /mnt/system_ext-s1/priv-app/SystemUIGoogle/SystemUIGoogle.apk and /mnt/system_ext-s2/priv-app/SystemUIGoogle/SystemUIGoogle.apk differ
     Files /mnt/system_ext-s1/priv-app/TurboAdapter/TurboAdapter.apk and /mnt/system_ext-s2/priv-app/TurboAdapter/TurboAdapter.apk differ
     Files /mnt/system_ext-s1/priv-app/UvExposureReporter/UvExposureReporter.apk and /mnt/system_ext-s2/priv-app/UvExposureReporter/UvExposureReporter.apk differ
     Files /mnt/system_ext-s1/priv-app/WallpaperPickerGoogleRelease/WallpaperPickerGoogleRelease.apk and /mnt/system_ext-s2/priv-app/WallpaperPickerGoogleRelease/WallpaperPickerGoogleRelease.apk differ
     ```

   - vendor.img

     ```
     $ sudo diff -rq /mnt/vendor-s1 /mnt/vendor-s2 2>/dev/null
     Files /mnt/vendor-s1/apex/com.android.vibrator.sunfish.apex and /mnt/vendor-s2/apex/com.android.vibrator.sunfish.apex differ
     Files /mnt/vendor-s1/app/CACertService/CACertService.apk and /mnt/vendor-s2/app/CACertService/CACertService.apk differ
     Files /mnt/vendor-s1/app/CneApp/CneApp.apk and /mnt/vendor-s2/app/CneApp/CneApp.apk differ
     Files /mnt/vendor-s1/app/gpu_profiling_vulkan_layer/gpu_profiling_vulkan_layer.apk and /mnt/vendor-s2/app/gpu_profiling_vulkan_layer/gpu_profiling_vulkan_layer.apk differ
     Files /mnt/vendor-s1/app/IWlanService/IWlanService.apk and /mnt/vendor-s2/app/IWlanService/IWlanService.apk differ
     Files /mnt/vendor-s1/app/TimeService/TimeService.apk and /mnt/vendor-s2/app/TimeService/TimeService.apk differ
     Files /mnt/vendor-s1/build.prop and /mnt/vendor-s2/build.prop differ
     Files /mnt/vendor-s1/etc/NOTICE.xml.gz and /mnt/vendor-s2/etc/NOTICE.xml.gz differ
     Files /mnt/vendor-s1/lib/libril-qc-ltedirectdisc.so and /mnt/vendor-s2/lib/libril-qc-ltedirectdisc.so differ
     Files /mnt/vendor-s1/lib/libril-qc-radioconfig.so and /mnt/vendor-s2/lib/libril-qc-radioconfig.so differ
     Files /mnt/vendor-s1/lib/modules/adsp_loader_dlkm.ko and /mnt/vendor-s2/lib/modules/adsp_loader_dlkm.ko differ
     Files /mnt/vendor-s1/lib/modules/apr_dlkm.ko and /mnt/vendor-s2/lib/modules/apr_dlkm.ko differ
     Files /mnt/vendor-s1/lib/modules/bolero_cdc_dlkm.ko and /mnt/vendor-s2/lib/modules/bolero_cdc_dlkm.ko differ
     Files /mnt/vendor-s1/lib/modules/br_netfilter.ko and /mnt/vendor-s2/lib/modules/br_netfilter.ko differ
     Files /mnt/vendor-s1/lib/modules/drv2624.ko and /mnt/vendor-s2/lib/modules/drv2624.ko differ
     Files /mnt/vendor-s1/lib/modules/ftm5.ko and /mnt/vendor-s2/lib/modules/ftm5.ko differ
     Files /mnt/vendor-s1/lib/modules/gspca_main.ko and /mnt/vendor-s2/lib/modules/gspca_main.ko differ
     Files /mnt/vendor-s1/lib/modules/hdmi_dlkm.ko and /mnt/vendor-s2/lib/modules/hdmi_dlkm.ko differ
     Files /mnt/vendor-s1/lib/modules/heatmap.ko and /mnt/vendor-s2/lib/modules/heatmap.ko differ
     Files /mnt/vendor-s1/lib/modules/incrementalfs.ko and /mnt/vendor-s2/lib/modules/incrementalfs.ko differ
     Files /mnt/vendor-s1/lib/modules/lcd.ko and /mnt/vendor-s2/lib/modules/lcd.ko differ
     Files /mnt/vendor-s1/lib/modules/llcc_perfmon.ko and /mnt/vendor-s2/lib/modules/llcc_perfmon.ko differ
     Files /mnt/vendor-s1/lib/modules/machine_dlkm.ko and /mnt/vendor-s2/lib/modules/machine_dlkm.ko differ
     Files /mnt/vendor-s1/lib/modules/mbhc_dlkm.ko and /mnt/vendor-s2/lib/modules/mbhc_dlkm.ko differ
     Files /mnt/vendor-s1/lib/modules/modules.alias and /mnt/vendor-s2/lib/modules/modules.alias differ
     Files /mnt/vendor-s1/lib/modules/mpq-adapter.ko and /mnt/vendor-s2/lib/modules/mpq-adapter.ko differ
     Files /mnt/vendor-s1/lib/modules/mpq-dmx-hw-plugin.ko and /mnt/vendor-s2/lib/modules/mpq-dmx-hw-plugin.ko differ
     Files /mnt/vendor-s1/lib/modules/msm_11ad_proxy.ko and /mnt/vendor-s2/lib/modules/msm_11ad_proxy.ko differ
     Files /mnt/vendor-s1/lib/modules/msm-geni-ir.ko and /mnt/vendor-s2/lib/modules/msm-geni-ir.ko differ
     Files /mnt/vendor-s1/lib/modules/native_dlkm.ko and /mnt/vendor-s2/lib/modules/native_dlkm.ko differ
     Files /mnt/vendor-s1/lib/modules/pinctrl_lpi_dlkm.ko and /mnt/vendor-s2/lib/modules/pinctrl_lpi_dlkm.ko differ
     Files /mnt/vendor-s1/lib/modules/pinctrl_wcd_dlkm.ko and /mnt/vendor-s2/lib/modules/pinctrl_wcd_dlkm.ko differ
     Files /mnt/vendor-s1/lib/modules/platform_dlkm.ko and /mnt/vendor-s2/lib/modules/platform_dlkm.ko differ
     Files /mnt/vendor-s1/lib/modules/q6_dlkm.ko and /mnt/vendor-s2/lib/modules/q6_dlkm.ko differ
     Files /mnt/vendor-s1/lib/modules/q6_notifier_dlkm.ko and /mnt/vendor-s2/lib/modules/q6_notifier_dlkm.ko differ
     Files /mnt/vendor-s1/lib/modules/q6_pdr_dlkm.ko and /mnt/vendor-s2/lib/modules/q6_pdr_dlkm.ko differ
     Files /mnt/vendor-s1/lib/modules/rdbg.ko and /mnt/vendor-s2/lib/modules/rdbg.ko differ
     Files /mnt/vendor-s1/lib/modules/rx_macro_dlkm.ko and /mnt/vendor-s2/lib/modules/rx_macro_dlkm.ko differ
     Files /mnt/vendor-s1/lib/modules/snd_event_dlkm.ko and /mnt/vendor-s2/lib/modules/snd_event_dlkm.ko differ
     Files /mnt/vendor-s1/lib/modules/stub_dlkm.ko and /mnt/vendor-s2/lib/modules/stub_dlkm.ko differ
     Files /mnt/vendor-s1/lib/modules/swr_ctrl_dlkm.ko and /mnt/vendor-s2/lib/modules/swr_ctrl_dlkm.ko differ
     Files /mnt/vendor-s1/lib/modules/swr_dlkm.ko and /mnt/vendor-s2/lib/modules/swr_dlkm.ko differ
     Files /mnt/vendor-s1/lib/modules/tx_macro_dlkm.ko and /mnt/vendor-s2/lib/modules/tx_macro_dlkm.ko differ
     Files /mnt/vendor-s1/lib/modules/usf_dlkm.ko and /mnt/vendor-s2/lib/modules/usf_dlkm.ko differ
     Files /mnt/vendor-s1/lib/modules/va_macro_dlkm.ko and /mnt/vendor-s2/lib/modules/va_macro_dlkm.ko differ
     Files /mnt/vendor-s1/lib/modules/wcd934x_dlkm.ko and /mnt/vendor-s2/lib/modules/wcd934x_dlkm.ko differ
     Files /mnt/vendor-s1/lib/modules/wcd937x_dlkm.ko and /mnt/vendor-s2/lib/modules/wcd937x_dlkm.ko differ
     Files /mnt/vendor-s1/lib/modules/wcd937x_slave_dlkm.ko and /mnt/vendor-s2/lib/modules/wcd937x_slave_dlkm.ko differ
     Files /mnt/vendor-s1/lib/modules/wcd9xxx_dlkm.ko and /mnt/vendor-s2/lib/modules/wcd9xxx_dlkm.ko differ
     Files /mnt/vendor-s1/lib/modules/wcd_core_dlkm.ko and /mnt/vendor-s2/lib/modules/wcd_core_dlkm.ko differ
     Files /mnt/vendor-s1/lib/modules/wcd_spi_dlkm.ko and /mnt/vendor-s2/lib/modules/wcd_spi_dlkm.ko differ
     Files /mnt/vendor-s1/lib/modules/wglink_dlkm.ko and /mnt/vendor-s2/lib/modules/wglink_dlkm.ko differ
     Files /mnt/vendor-s1/lib/modules/wlan.ko and /mnt/vendor-s2/lib/modules/wlan.ko differ
     Files /mnt/vendor-s1/lib/modules/wsa881x_dlkm.ko and /mnt/vendor-s2/lib/modules/wsa881x_dlkm.ko differ
     Files /mnt/vendor-s1/lib/modules/wsa_macro_dlkm.ko and /mnt/vendor-s2/lib/modules/wsa_macro_dlkm.ko differ
     Files /mnt/vendor-s1/lib64/libril-qc-ltedirectdisc.so and /mnt/vendor-s2/lib64/libril-qc-ltedirectdisc.so differ
     Files /mnt/vendor-s1/lib64/libril-qc-radioconfig.so and /mnt/vendor-s2/lib64/libril-qc-radioconfig.so differ
     Files /mnt/vendor-s1/odm/etc/build.prop and /mnt/vendor-s2/odm/etc/build.prop differ
     Files /mnt/vendor-s1/odm_dlkm/etc/build.prop and /mnt/vendor-s2/odm_dlkm/etc/build.prop differ
     Files /mnt/vendor-s1/overlay/Flipendo__auto_generated_rro_vendor.apk and /mnt/vendor-s2/overlay/Flipendo__auto_generated_rro_vendor.apk differ
     Files /mnt/vendor-s1/overlay/framework-res__auto_generated_rro_vendor.apk and /mnt/vendor-s2/overlay/framework-res__auto_generated_rro_vendor.apk differ
     Files /mnt/vendor-s1/overlay/HbmSVManager__auto_generated_rro_vendor.apk and /mnt/vendor-s2/overlay/HbmSVManager__auto_generated_rro_vendor.apk differ
     Files /mnt/vendor-s1/overlay/NfcNci__auto_generated_rro_vendor.apk and /mnt/vendor-s2/overlay/NfcNci__auto_generated_rro_vendor.apk differ
     Files /mnt/vendor-s1/overlay/SettingsGoogle__auto_generated_rro_vendor.apk and /mnt/vendor-s2/overlay/SettingsGoogle__auto_generated_rro_vendor.apk differ
     Files /mnt/vendor-s1/overlay/StorageManagerGoogle__auto_generated_rro_vendor.apk and /mnt/vendor-s2/overlay/StorageManagerGoogle__auto_generated_rro_vendor.apk differ
     Files /mnt/vendor-s1/overlay/SystemUIGoogle__auto_generated_rro_vendor.apk and /mnt/vendor-s2/overlay/SystemUIGoogle__auto_generated_rro_vendor.apk differ
     Files /mnt/vendor-s1/overlay/TeleService__auto_generated_rro_vendor.apk and /mnt/vendor-s2/overlay/TeleService__auto_generated_rro_vendor.apk differ
     Files /mnt/vendor-s1/overlay/Traceur__auto_generated_rro_vendor.apk and /mnt/vendor-s2/overlay/Traceur__auto_generated_rro_vendor.apk differ
     Files /mnt/vendor-s1/vendor_dlkm/etc/build.prop and /mnt/vendor-s2/vendor_dlkm/etc/build.prop differ
     ```

1. Diff boot.img

   1. Extract images

      ```
      sudo apt install mkbootimg
      unpack_bootimg --boot_img sunfish-tq3a.230805.001.s1/boot.img --out sunfish-tq3a.230805.001.s1/boot
      unpack_bootimg --boot_img sunfish-tq3a.230805.001.s2/boot.img --out sunfish-tq3a.230805.001.s2/boot
      ```

   1. Compare

      ```
      $ diff -rq sunfish-tq3a.230805.001.s1/boot sunfish-tq3a.230805.001.s2/boot
      Files sunfish-tq3a.230805.001.s1/boot/kernel and sunfish-tq3a.230805.001.s2/boot/kernel differ
      Files sunfish-tq3a.230805.001.s1/boot/ramdisk and sunfish-tq3a.230805.001.s2/boot/ramdisk differ
      ```

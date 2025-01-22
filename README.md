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

1.

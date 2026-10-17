# Xiaomi Camera for Vermeer

Device-side integration from Vermeer-Development, based on commit
`957fd0e8d44f8d3887a98052a90232f64cd9b54f` on `lineage-24.0`.

The companion blob tree is `vendor/xiaomi/vermeer-miuicamera`, maintained at
`mint1246/proprietary_vendor_xiaomi_vermeer-miuicamera` on `lineage-24.0`.
Its initial contents come from upstream `lineage-23.2`, commit
`2c33895e0527c55517eb5cbcdb4a698c90c3ae96`, extracted from
`OS3.0.301.0.WNKMIXM`. The camera APK is `com.android.camera` version
`6.4.000380.0` (`640003800`), targeting SDK 35.

## Build integration

The Vermeer device tree includes this tree's `BoardConfig.mk` and inherits
`device.mk`. Keep both camera repositories in the local manifest. The blob
tree's `vendorsetup.sh` reassembles the split APK when `build/envsetup.sh` is
sourced; the assembled APK is generated and ignored by Git.

This tree supplies the camera properties, native-library allowlists,
privileged permission configuration, hidden API allowlist, libgui shim, and
vendor SELinux policy. The companion tree supplies the patched APK and four
system native libraries. Existing copyright and license notices are retained.

On Android 17, this platform-signed SDK-35 app uses `platform_app_36`.
The inherited `platform_app_all` rules therefore cover both app domain
variants, including the camera HAL's Binder callbacks. The policy also
includes MiSys service lookup, algorithm-library labels, and camera property
access from the upstream October fixes.

The stale upstream grant to `vendor_panel_info_prop` is omitted: that type
was removed upstream and is not defined or assigned to any property in this
device configuration. The existing Qualcomm `vendor_display_prop` grant is
retained.

## Validation and device testing

```sh
source build/envsetup.sh
lunch lineage_vermeer-cp2a-userdebug
m -j4 MiuiCamera libgui_shim_miuicamera selinux_policy sepolicy_tests
```

A successful build does not establish working capture on the phone. After
flashing, test with SELinux enforcing: app launch, rear and front capture,
HDR/night/portrait modes, video recording and playback, and repeated captures.
If capture stalls, collect the camera logs and SELinux denials together and
check the `com.android.camera` process domain and HAL-to-app callbacks.

Validated on 2026-10-17 for `lineage_vermeer-cp2a-userdebug`: the camera APK,
libgui shim, SELinux policy, and policy tests built successfully. APK signature
verification passed. All four prebuilt libraries passed AOSP's SONAME,
dependency, and undefined-symbol checks using the new shim and available
system/APEX dependencies. Installed file-context lookups gave the expected
`same_process_hal_file` label to the tested camera algorithm libraries.
The generated policy retains Binder calls in both directions between the
camera HAL and `platform_app_all`, which includes `platform_app_36`.
Full ROM and on-device capture tests remain pending.

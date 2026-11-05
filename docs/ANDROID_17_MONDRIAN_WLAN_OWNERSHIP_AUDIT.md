# PE-Reborn Android 17 — Qualcomm WLAN ownership audit (mondrian)

**2026-11-05 | P2: hardware/qcom-caf/wlan | Historical source preserved; do not enable duplicate WLAN tree.**

## Provenance and real consumers

- Archived PE `seventeen`: `bb7e8bf629e9e97d554784b7bca86fc4706e073a`, Qualcomm `qcwcn`, `cld80211-lib`, `wcnss-service` and old `Android.mk` pathmap `project-path-for(qcom-wlan)`; the history is principally 2018–2023.
- Official AOSP 17 r1 already provides `platform/hardware/qcom/wlan` at the selected `hardware/qcom/wlan` path in our [AOSP17 reference manifest](https://github.com/PixelExperience-Reborn/manifest/blob/port/a17-aosp17-manifest-20261102/aosp17_reference.xml); official r1 commit `b467db6eb6e31cc160a2600d0c8cb4e20c52f3fa` (tree `20b83951680bc43da6f697a354b271367ecb59c0`) includes `legacy`, `wcn6740`, `wcn7760`, `Android.bp`, `Android.mk`.
- Cross-check upstream `LineageOS/android_hardware_qcom_wlan` `lineage-24.0` at `4110d72de3eaee4c1091ba17af8b52449acb62a6`: modern `Android.bp` implements `wifihal_qcom_defaults`, sources the `libwifi-hal-qcom` and `libcld80211` modules through fully qualified paths such as `//hardware/qcom/wlan/legacy:...`. It is **not** a drop-in source tree for `hardware/qcom-caf/wlan` without resolving those imports.
- Actual `LF52406/mondrian_android_device_xiaomi_sm8450-common` `lineage-24.0` references: `BoardConfigCommon.mk` sets `BOARD_WLAN_DEVICE := qcwcn`, NL80211 supplicant/hostapd and `qca_cld3` driver; `common.mk` selects `android.hardware.wifi-service`, `libwifi-hal-qcom` and copies `qca6490/qca6750` Wi-Fi configurations.
- `BOARD_WLAN_CHIP` is **not declared** in the inspected `BoardConfigCommon.mk`; the final value may be supplied by another product/board file. Do not infer a verified wcn6740/wcn7760 runtime target just from the chipset name.

## Old PE delta that merits review, not automatic cherry-pick

The historical commit [`bb7e8bf`](https://github.com/PixelExperience-Reborn/hardware_qcom-caf_wlan/commit/bb7e8bf629e9e97d554784b7bca86fc4706e073a) changed `qcwcn/wifi_hal/llstats.cpp`: when `QCA_WLAN_VENDOR_ATTR_LL_STATS_IFACE_INFO_TS_DUTY_CYCLE` is absent, it sets `time_slicing_duty_cycle_percent=100` rather than rejecting the link-layer stats; it also limits verbose `LOG_NDEBUG=0` to `eng` builds. The modern `legacy/qcwcn/wifi_hal/llstats.cpp` differs in structure and did not show the same attribute handling in the inspected revision. Evaluate the current link-layer stats contract and netlink/vendor attributes **before** adapting any behavior. This old patch cannot safely be applied verbatim to the Android17 path.

## Integration decision

1. **Keep official AOSP17 `hardware/qcom/wlan` as the sole current source owner** in the platform-only staged manifests. Do not also activate the archived `hardware/qcom-caf/wlan` checkout or duplicate `libwifi-hal-qcom`, `libcld80211`, `lib_driver_cmd_qcwcn` or related namespaces.
2. Review exact modern AOSP17/LineageOS module variants against the new independent PE-Reborn device graph, vendor WCN configuration and kernel WLAN UAPI; prefer a targeted, reviewed AOSP17-compatible patch only if an actual regression is established.
3. Verify `BOARD_WLAN_CHIP`, the service/driver stack, module naming and Soong visibility, firmware paths, permissions and SELinux enforcing.
4. Full build tests are pending: Soong `libwifi-hal-qcom`, `android.hardware.wifi-service`, and interface HAL, VINTF, Wi-Fi STA/AP/P2P/NAN, WPA3/PMF, roaming, link-layer stats and RTT on a physical mondrian.
5. Historical PE source remains on `seventeen` unmodified. No Wi-Fi module has been compiled, no device has been tested and no assumption of Android17 product readiness is made.

**Decision:** source ownership and one historical behavior change reviewed; P2 archive **not enabled**; device Wi-Fi integration and selective porting gates remain OPEN. Next row: `hardware/qcom/audio`.

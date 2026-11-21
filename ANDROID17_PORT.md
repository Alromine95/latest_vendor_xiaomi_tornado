# proprietary_vendor_xiaomi_tornado — Android 17 (CP2A / LineageOS 24.0)

Branch: `lineage-23.2` → `lineage-24.0`.

## Changed

Four interface libraries were removed from `Android.bp` and from
`proprietary/`, because `hardware/mediatek` on 24.0 builds them from source and
the prebuilt copies duplicate the same modules and the same installed files:

| Removed prebuilt | Now provided by |
|---|---|
| `vendor/lib64/vendor.mediatek.hardware.bluetooth.audio-V1-ndk.so` | `hardware/mediatek/interfaces/hardware/bluetooth/audio/aidl` (AIDL, V1..V6) |
| `vendor/lib64/vendor.mediatek.hardware.bluetooth.audio@2.1.so` | `hardware/mediatek/interfaces/hardware/bluetooth/audio/2.1` (HIDL) |
| `vendor/lib64/vendor.mediatek.hardware.bluetooth.audio@2.2.so` | `hardware/mediatek/interfaces/hardware/bluetooth/audio/2.2` (HIDL) |
| `vendor/lib64/vendor.mediatek.hardware.netdagent@1.0.so` | `hardware/mediatek/interfaces/hardware/netdagent/1.0` (HIDL) |

Blobs that link them keep working — the source modules are
`system_ext_specific` interfaces, installed to `/system_ext/lib64` with the
usual `/vendor/lib64` symlinks:

* `audio.primary.mt6835.so` → `bluetooth.audio@2.1`, `@2.2`
* `libbluetooth_audio_session_aidl_mediatek.so` → `bluetooth.audio@2.1`, `@2.2`
* `libmtk-ril.so`, `libmtkmipc-ril.so` → `netdagent@1.0`

The `PRODUCT_PACKAGES` entries for those four modules were deliberately left in
`tornado-vendor.mk`; they now resolve to the source-built modules, so the
installed file set is unchanged.

## Unchanged, on purpose

* **`libmnl_mtk.so` / `libformatter_mtk.so`** — already renamed (with patched
  sonames) relative to the stock ROM, which is exactly what Android 17 needs
  because AOSP 17 has its own `libmnl` / `libformatter` modules. Do not revert
  these to `libmnl.so` / `libformatter.so`.
* **`libion_mtk.so`, `libion_ulit.so`, `libformatter_mtk.so`, camera/media
  blobs** — they carry `DT_NEEDED: libion.so`. Android 17 removed the old ION
  library, so the device tree now selects the legacy implementation
  (`$(call soong_config_set_bool,libion,legacy_impl,true)`) and adds
  `device/lineage/sepolicy/libion/sepolicy.mk`. 54 blobs in this tree depend on
  that.
* **`Android.bp` module types** (`cc_prebuilt_library_shared`,
  `cc_prebuilt_binary`, `install_symlink`, `prebuilt_etc_xml`, `dex_import`) —
  all still valid in 17.
* **`BoardConfigVendor.mk`** — empty, nothing to change.

## Regenerating

`extract-files.sh` in `device/xiaomi/tornado` now produces the same four-lib
exclusion and the same `_mtk` renames, so a re-extraction from the stock ROM
should reproduce this tree. If you extract from a newer ROM, diff against this
state before pushing.

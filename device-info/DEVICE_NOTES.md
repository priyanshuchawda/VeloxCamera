# Initial device notes

Captured over ADB on 2026-10-06. Raw command output is retained alongside this note.

## Identity and software

- Manufacturer/model properties: Xiaomi / `2406ERN9CI`
- Device/product properties: `breeze` / `breeze_in`
- Build fingerprint: `Redmi/breeze_in/breeze:16/BP2A.250605.031.A3/OS3.0.303.0.WNUINXM:user/release-keys`
- Android 16, API level 36; build `OS3.0.303.0.WNUINXM`
- SoC properties are inconsistent: `ro.soc.model=SM4450`, while `ro.boot.hardware=qcom`, and the vendor camera service exposes extensive Qualcomm CamX/QCamera metadata. Treat exact SoC identity as unresolved pending a more authoritative check.
- ABI: `arm64-v8a`; EGL/Vulkan properties report Adreno.

## Camera service

The camera service reports five camera devices. `media-camera.txt` contains the full capability dump, including per-camera facing/orientation, stream configurations, FPS ranges, sensor metadata, and vendor tags. Use that dump as raw evidence; parse per-camera sections before designing around particular modes.

## Power and system

Battery, display, memory, storage, thermal and system service snapshots are preserved in their correspondingly named files. These are point-in-time readings and may change with charging, workload, firmware, or temperature.

## Privacy

The raw `getprop.txt` and service dumps may contain device-specific identifiers or configuration details. Review them before sharing this public repository more widely; the ADB serial was omitted from the notes, but the remaining raw camera dump should be reviewed for device specific metadata before further redistribution.

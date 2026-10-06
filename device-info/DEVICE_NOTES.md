# Initial device notes

Captured over ADB on 2026-10-06. Raw command output is retained alongside this note.

## Identity and software

- Manufacturer/model properties: Xiaomi / `2406ERN9CI`
- Device/product properties: `breeze` / `breeze_in`
- Build fingerprint: `Redmi/breeze_in/breeze:16/BP2A.250605.031.A3/OS3.0.303.0.WNUINXM:user/release-keys`
- Android 16, API level 36; build `OS3.0.303.0.WNUINXM`
- `ro.soc.model=SM4450`, corresponding to the Snapdragon 4 Gen 2 AE in this Redmi 13 5G; `ro.boot.hardware=qcom` and the camera service exposes Qualcomm CamX/QCamera metadata.
- ABI: `arm64-v8a`; EGL/Vulkan properties report Adreno.

## Camera service

The camera service reports five camera devices, including front and back logical/auxiliary entries. The default back logical camera advertises a binned 4000x3000 array and maximum-resolution 12000x9000 metadata; this is not proof that a third-party app can capture full-resolution JPEGs at every mode. `media-camera.txt` contains per-camera stream configurations, FPS ranges, sensor metadata, and vendor tags. Query and test supported combinations at runtime.

## Power and system

Battery, display, memory, storage, thermal and system service snapshots are preserved in their correspondingly named files. These are point-in-time readings and may change with charging, workload, firmware, or temperature.

## Privacy

The earlier raw `getprop.txt` and live sensor dump were removed before publication because they contained sensitive identifiers or transient readings. Remaining camera and system dumps are capability evidence, but should still be reviewed before redistribution beyond this project.

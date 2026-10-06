# Redmi 13 5G camera capability audit

Captured from the connected handset over ADB on 2026-10-06. Camera IDs are represented by their enumeration order here rather than copied into conclusions. The full Camera2 static metadata dump is in `media-camera.txt`; the event history containing other apps' package names was removed before commit.

## Device and camera modules

- Device: Xiaomi Redmi 13 5G India, model `2406ERN9CI`, codename `breeze`, Android 16 / API 36, HyperOS `OS3.0.303.0.WNUINXM`.
- Platform: Qualcomm Snapdragon 4 Gen 2 AE (`SM4450`); GPU reports Adreno 613; ABI `arm64-v8a`.
- Vendor camera properties identify the configured main module as `n19_ofilm_s5khm6_i` (Samsung HM6 family), macro as `n19_ofilm_gc02m1b_i` (2 MP class), and front as `n19_ofilm_ov13b10_i` (13 MP class). These are firmware module labels, not a guarantee of every app-visible path.
- A live API 36 `CameraManager.getCameraIdList()` probe reports **two app-visible cameras: one rear and one front**. `getPhysicalCameraIds()` is empty for both and `getConcurrentCameraIds()` is empty, so the macro cannot be selected through the stock public camera API and front/rear simultaneous capture is not supported. The low-level `dumpsys media.camera` HAL reports five legacy/internal enumerations; those are not five app-selectable cameras. Use `CameraSelector.DEFAULT_BACK_CAMERA` and `CameraSelector.DEFAULT_FRONT_CAMERA` in the app.

## Camera2 capabilities from vendor static metadata

- Main rear entry: `FULL` hardware level; BACKWARD_COMPATIBLE, RAW, YUV/private reprocessing, manual sensor, manual post-processing, burst, sensor settings, constrained high speed.
- Main standard pixel array and top advertised regular JPEG/RAW output: **4000x3000 (12 MP)**. JPEG, RAW_SENSOR, YUV_420_888, YV12 and PRIVATE outputs are listed at this size.
- Front standard pixel array/top JPEG output: **4208x3120 (~13.1 MP)**; RAW_SENSOR is listed at this size too.
- Macro rear entry: **1600x1200 (1.9 MP)** maximum, consistent with a 2 MP macro module.
- The main rear metadata has `pixelArraySizeMaximumResolution=12000x9000`, but the live Camera2 `getHighResolutionOutputSizes()` probe returns an empty list for JPEG and RAW; the metadata maximum-resolution stream table only lists up to **1920x1080** for JPEG/YUV/YV12/PRIVATE. The main sensor does not advertise a 12000x9000 JPEG/RAW output to ordinary apps. Xiaomi's stock camera may have a proprietary capture path; the third-party high-resolution-blob property is `false`. A 108 MP third-party mode is unavailable through the standard API observed here.
- Main rear regular AE target ranges include 15–24, 15–30, and 30 fps; front includes 10–15, 15–24, and 10–30/30 fps. Rear manual exposure range is approximately 94.7 μs–507 ms with ISO 50–3200; front is 42 μs–342 ms with ISO 50–853. Rear minimum focus distance is 9.09 diopters (about 11 cm); front reports 0 diopters (fixed focus). These are metadata limits, not guarantees for every session.
- Live `StreamConfigurationMap` confirms high-speed sizes 1280x720, 720x480 and 640x480; all report a 30–120 fps range and an exact 120 fps range. This is a Camera2 constrained-high-speed path, not proof that CameraX Recorder or Xiaomi's standard profile can save it. Xiaomi `media_profiles_ravelin.xml` exposes regular 1080p at 30 fps; official specs list rear/front video up to 1080p30. Treat **1080p30** as the stable baseline. A 720p120 experimental path needs a Camera2 constrained-high-speed session and sustained recording validation.
- Main and front standard metadata report video stabilization modes `[0, 0]` and optical stabilization `[0]` (off-only/no OIS through these keys). Xiaomi may apply proprietary stabilization in its own camera app; a third-party app should not claim EIS/OIS until a recorded sample proves it.
- Main camera max digital zoom reports 10x in Camera2 metadata; Xiaomi advertises 3x in-sensor zoom. This is crop/digital zoom behavior, not a telephoto lens. Start the UI at 1x and expose smooth zoom only to the range validated for live video and stills.
- Main has flash; device advertises camera autofocus, flash and FULL camera hardware via package features.
- Main also advertises RAW_SENSOR and manual sensor capability. This provides a path for optional DNG/manual controls, but exposure/focus ranges and actual DNG output need an app-side capture test. RAW is not automatically higher-quality for normal users: it lacks the OEM processed JPEG's finished tone/noise pipeline and costs storage/processing.
- Reprocessing and burst capabilities exist in metadata. CameraX ZSL support should be queried at runtime; `camera.disable_zsl_mode=1` suggests firmware policy may disable it even though private reprocessing is declared.

## Platform, power, and graphics

- Display snapshot: 1080x2460, density 440 dpi. The display can refresh up to 120 Hz per Xiaomi, but preview should use adaptive refresh where available; avoid forcing 120 Hz during long recording.
- Vulkan compute is present (Vulkan level 1; reported driver version property `4198400`) and EGL/Vulkan report Adreno. The stock Xiaomi camera ships GPU shader resources. This supports optional GPU post-effects, but does not prove a custom effect will improve captures; the camera ISP and hardware video encoder are the first-choice image/video pipeline.
- Vendor codec config includes Qualcomm AVC and HEVC encoders, up to 4096x2176 size limits and high encoder performance points (including 3840x2160@60). This is encoder capacity only: camera stream paths and vendor recording profiles constrain what can actually be recorded. Do not infer 4K capture support from codec XML.
- Battery snapshot during audit: 59%, charging over USB, battery temperature 35.0 C. Thermal service reported status 0 (none) at that instant. This is a single snapshot, not a battery-life or sustained-thermal benchmark.
- Accelerometer, gyroscope, compass, light and proximity sensors are present. Use rotation-vector/gravity data only while the camera UI is foregrounded for horizon/level tools. Do not continuously process sensor streams in background.
- Side fingerprint reader is not a raw touch sensor API for third-party apps. Don't build the shutter gesture around fingerprint contacts.

## Audit experiment status

- Read-only ADB collection completed: Camera2 metadata, display, sensor/thermal/battery snapshots, installed stock camera package/version, Xiaomi camera module properties, vendor codec and recording-profile configs, and feature flags.
- A minimal framework `CameraManager`/`StreamConfigurationMap`/Camera2-extension/`MediaCodecList` probe was compiled against Android API 36, installed, and executed. It confirmed two app-visible cameras, empty physical-camera/concurrent-camera/extension lists, empty high-resolution JPEG/RAW lists, and the reported modes above. The first install failed because the APK omitted `uses-sdk`; after fixing that, a packaging error nested `classes.dex` instead of placing it at the APK root. Both were fixed; the probe installed successfully. The diagnostic app has since been uninstalled. No install-verification settings were changed.
- No photo/video files were captured; only the probe app camera permission was granted and then removed by uninstalling the app. No camera/system settings were changed. Remaining experiments: JPEG EXIF and image-quality comparison, DNG capture, 1080p30 recordings (front/rear), constrained 720p120 sustained recording, focus/exposure/zoom behavior, camera-switch latency, heat/frame drops, and battery drain.

## Architecture implications

1. Kotlin native app. Use CameraX `ProcessCameraProvider` for lifecycle-safe preview/still/video and Camera2 interop for feature enumeration.
2. Ship default processed 12 MP rear and ~13 MP front photos first; use highest supported JPEG size, not the sensor's 108 MP marketing count. Offer RAW/manual mode only after confirming a useful capture workflow.
3. Use 1080p30 as video baseline, hardware AVC/HEVC encoder selected by supported quality profiles and file compatibility. Add 720p120 only if a constrained high-speed session records and plays correctly over a sustained test.
4. Implement double-tap-to-switch on the preview with a debounce window, no trigger during pinch zoom/control interaction, haptic/visual feedback, and a visible accessible switch button.
5. Do not run permanent frame analysis. Use ISP/encoder. Add optional GPU/ML effects only if a measured still/video comparison justifies heat and battery cost.
6. Observe thermal status and suggest reduced workload; never silently change quality mid-recording. Test sustained recording with repeatable temperature and dropped-frame measurements.

## Primary references

- [Xiaomi Redmi 13 5G official specifications](https://www.mi.com/in/product/redmi-13-5g/specs/)
- [Android CameraX architecture and Camera2 interoperability](https://developer.android.com/media/camera/camerax/architecture)
- [CameraX resolution and stream configuration](https://developer.android.com/media/camera/camerax/configuration)
- [CameraX OEM extensions](https://developer.android.com/media/camera/camerax/extensions-api)
- [Android constrained high-speed camera sessions](https://developer.android.com/reference/android/hardware/camera2/CameraConstrainedHighSpeedCaptureSession)
- [Android thermal-aware camera guidance](https://developer.android.com/agents/skills/camera/camerax/references/thermals)

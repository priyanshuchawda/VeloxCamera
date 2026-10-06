# VeloxCamera: product and technical direction

## Product goal

Make a fast, dependable camera for the Redmi 13 5G that produces the best photos and videos its stock camera HAL and sensor pipeline expose. Optimize for image quality, capture reliability, responsiveness, thermals, and battery use. Do not imply that software can recover detail, stabilization, frame rates, or sensor access that the hardware/firmware does not provide.

## Verified device constraints

- Target: Redmi 13 5G India, model `2406ERN9CI`, codename `breeze`.
- Connected software snapshot: Android 16 / API 36, HyperOS build `OS3.0.303.0.WNUINXM`.
- Platform: Snapdragon 4 Gen 2 AE (`SM4450`) with Adreno GPU.
- Rear: marketed 108 MP main camera plus 2 MP macro camera. Front: 13 MP.
- Xiaomi's India specifications list rear/front video modes through 1080p30, not 4K. The application must enumerate encoder and CameraX/Camera2 output options rather than promise higher resolution or frame rates.
- Xiaomi advertises 3x in-sensor zoom. Verify available zoom/crop behavior; don't present it as a separate telephoto lens.
- Camera service reports five public IDs. Main rear regular JPEG/RAW output tops at 4000x3000; 12000x9000 maximum-resolution pixel-array metadata does not correspond to an advertised 108 MP JPEG/RAW stream. See [`device-info/CAMERA_CAPABILITY_AUDIT.md`](../device-info/CAMERA_CAPABILITY_AUDIT.md) for findings and app-side capture tests still needed.
- The phone has a side fingerprint reader. Ordinary app APIs provide biometric authentication, not raw fingerprint sensor touch events suitable for a shutter gesture.

## Product principles

1. Take a good default photo quickly. Use the OEM camera extension only when runtime capability checks show a supported mode and real output improvement.
2. Offer clear controls: Photo, Video, front/rear switch, zoom, exposure, flash/torch, and a small quality/settings panel.
3. Add the requested double-tap preview gesture to switch front/rear cameras. Debounce taps, avoid conflicting with pinch zoom, provide haptic/visual feedback, and keep a visible switch button for accessibility.
4. Make quality and battery choices explicit: **Balanced** defaults, **Max photo** for higher-resolution stills and longer processing, and **Efficient video** for reduced recording load/file size. Expose only combinations confirmed by the device.
5. Keep voice capture opt-in and user-started, e.g. tap a mic button then say “take photo” or “start recording.” Prefer Android on-device speech recognition when available; never listen continuously in the background.
6. Respect thermal limits: observe Android thermal status and reduce optional analysis/effects first; if severe, offer a lower video preset and explain it before changing settings.
7. Keep the preview pipeline lean. Let camera ISP/JPEG/encoder do their work; do not add a permanent full-resolution CPU/GPU frame-processing loop without a measured benefit.

## Proposed implementation

- **Language/UI:** Kotlin, native Android UI (Compose is suitable for controls; use a low-overhead `PreviewView`/Surface for live camera preview).
- **Camera:** CameraX `ProcessCameraProvider` to bind preview, still, and video use cases together. Use Camera2 interop for capability inspection or controls CameraX does not expose. Fall back gracefully when an extension or use-case combination is unavailable.
- **Still image:** Default to processed high-quality still capture. Query maximum-resolution support; expose 108 MP mode only if the app can actually request and save an image near the expected dimensions, and test quality, latency, storage, and thermal cost. Check supported OEM HDR/night extensions on-device before surfacing them.
- **Video:** Enumerate supported resolutions/FPS/encoder combinations. Begin with 1080p30; compare stabilization on/off, bitrate, dropped frames, heat, and file size in sustained recordings. Avoid advertising 4K/60 fps unless the connected handset reports and records them reliably.
- **Gesture:** Detect a deliberate double tap on the preview surface, excluding controls and active pinch; camera switching rebinds the selected front/back camera and restores a valid zoom.
- **Optional feature:** Voice shutter through Android `SpeechRecognizer.createOnDeviceSpeechRecognizer` after explicit user action and permission. Availability/languages vary by installed recognition service.
- **Power:** No background camera/sensor activity. Stop analysis on lifecycle pause. Avoid forcing high display refresh. Thermal listener can suggest lower cost modes while recording.
- **Fingerprint:** Do not design shutter around direct fingerprint contacts. Could consider biometric prompt only for an optional protected-gallery feature, separate from the camera capture path.

## Staged work

1. **Capability audit:** Build a tiny Kotlin probe that lists CameraX camera selectors, Camera2 IDs/characteristics, output sizes by format, FPS ranges, stabilization, flash, zoom, extensions, concurrent camera support, and video encoder profiles. Avoid publishing per-device identifiers.
2. **MVP:** Native preview, still capture to MediaStore, video record/stop to MediaStore, permissions/lifecycle handling, visible camera switch, and double-tap switch. Verify both cameras and saved outputs on the phone.
3. **Quality presets:** Add only capabilities confirmed in stage 1; compare representative image/video output, not just API flags.
4. **Power/thermal:** Measure a baseline recording and test graceful quality suggestions under increasing thermal status. Make no battery-life claims without repeatable measurements.
5. **Optional interactions:** User-started voice shutter and any gallery features after core reliability.

## Research sources

- [Xiaomi Redmi 13 5G official specifications](https://www.mi.com/in/product/redmi-13-5g/specs/): SoC, sensors, battery and advertised video modes.
- [CameraX architecture](https://developer.android.com/media/camera/camerax/architecture): lifecycle-managed use cases and Camera2 interoperability.
- [CameraX configuration](https://developer.android.com/media/camera/camerax/configuration): supported resolution selection and concurrent camera capability.
- [CameraX extensions](https://developer.android.com/media/camera/camerax/extensions-api): runtime availability of OEM HDR, Night, Bokeh and other photo extensions.
- [Android thermal-aware camera guidance](https://developer.android.com/agents/skills/camera/camerax/references/thermals): monitor thermal status and adapt camera workload.
- [Android SpeechRecognizer API](https://developer.android.com/reference/android/speech/SpeechRecognizer): on-device recognition availability and lifecycle constraints.
- [Android sensor guidance](https://developer.android.com/develop/sensors-and-location/sensors/sensors_overview): use sensors only while foregrounded and unregister listeners when paused.

# VeloxCamera

An experimental Android camera app focused on making the most of the connected Xiaomi phone's camera hardware, with careful attention to image quality, responsiveness, thermals, and battery use.

## Device research

This app targets the Redmi 13 5G (`2406ERN9CI`, codename `breeze`). It has a Snapdragon 4 Gen 2 AE platform, Adreno GPU, 108 MP main camera, and a 13 MP front camera. Xiaomi lists video recording up to 1080p at 30 fps. See [Xiaomi's specifications](https://www.mi.com/in/product/redmi-13-5g/specs/) and the initial ADB captures in [`device-info/`](device-info/). The camera service lists five logical/auxiliary camera devices; the app must identify and verify usable lenses at runtime rather than assume every device ID is independently selectable.

## Direction

Build a native Android app in Kotlin. Use CameraX `ProcessCameraProvider` for lifecycle-managed preview, still capture, video recording, zoom, and tap-to-focus; use Camera2 interop to inspect stream and sensor capabilities exposed by the firmware. Keep a simple photo/video control surface, with a screen double tap as a configurable front/rear switch gesture.

Quality modes should be honest and measured: balanced preview plus maximum processed still output; optional full-resolution still capture only when the camera advertises it; and video presets only for advertised combinations (expected up to 1080p30). Avoid permanent GPU/CPU image analysis. Add user-invoked voice shutter only if on-device speech recognition is available. Monitor thermal state during recording and degrade gracefully. Treat the side fingerprint reader as a hardware authentication/input device, not as a sensor that exposes raw touch events to ordinary apps.

See [`docs/PRODUCT_AND_TECHNICAL_DIRECTION.md`](docs/PRODUCT_AND_TECHNICAL_DIRECTION.md) for product scope, research sources, and staged implementation plan.

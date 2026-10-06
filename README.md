# VeloxCamera

An experimental Android camera app focused on making the most of the connected Xiaomi phone's camera hardware, with careful attention to image quality, responsiveness, thermals, and battery use.

## Device research

Initial ADB capability and system captures are in [`device-info/`](device-info/). The device currently reports conflicting identity properties; see [`device-info/DEVICE_NOTES.md`](device-info/DEVICE_NOTES.md) before treating any model or chipset identification as settled.

## Direction

Start with native Android in Kotlin, using CameraX where its controls and capture paths are sufficient, and Camera2 for capabilities that need direct access to advertised camera metadata. Validate capture behavior on this device before adding image processing or hardware-specific tuning.

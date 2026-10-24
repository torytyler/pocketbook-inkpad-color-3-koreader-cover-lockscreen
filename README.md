# PocketBook InkPad Color 3 — KOReader Cover Lock Screen

Use the cover of the book you are reading in KOReader as the PocketBook's
native lock-screen image — in full color.

Tested on **PocketBook InkPad Color 3 (PB743K3)** with firmware 6.x and
KOReader v2026.10.1.

## What it does

- Writes the current KOReader cover to PocketBook's native lock background:
  `/mnt/ext1/system/resources/Line/taskmgr_lock_background.bmp`
- Applies InkView's color adjustment before saving, so the image is not
  washed out on the Color 3.
- Updates whenever KOReader opens a book.
- Keeps the existing PocketBook boot/power-off cover behavior.

This does not modify the PocketBook firmware. It only writes image files and
extends KOReader's existing cover-image plugin.

## Requirements

- PocketBook InkPad Color 3 (PB743K3)
- KOReader with the standard `coverimage` plugin
- USB access to the device

## Setup

1. Connect the PocketBook by USB.
2. Create this folder and an empty placeholder file:

   ```text
   system/resources/Line/taskmgr_lock_background.bmp
   ```

3. Apply the patch from this repository to your KOReader source tree:

   ```sh
   cd koreader
   patch -p1 < /path/to/coverimage-pocketbook-lockscreen.patch
   ```

   Then install the patched `coverimage` plugin on the device, or copy the
   edited `plugins/coverimage.koplugin/main.lua` into your existing install.

4. In KOReader, enable cover saving:

   **Menu → Screen → Cover image**

   - Save cover image: on
   - Image path: `system/logo/offlogo/cover.bmp`
   - Format: BMP
   - Background: black
   - Stretch limit: ~20%
   - Grayscale: off on color devices

5. Restart KOReader and open a book. Lock the device to check the cover.

## Why the color adjustment matters

On color PocketBooks, KOReader normally runs framebuffer updates through
InkView's `adjustAreaDefault()`, which boosts saturation and applies dithering.
The lock image is displayed by PocketBook itself, so it does not get that
processing automatically. Without it, the exact same cover looks noticeably
dull on the lock screen.

The patch applies the same adjustment to the cover buffer before writing the
lock image:

```lua
local inkview = require("ffi/inkview")
inkview.adjustAreaDefault(
    ffi.cast("unsigned char *", image.data),
    tonumber(image.stride),
    image:getWidth(),
    image:getHeight()
)
```

Monochrome PocketBooks should keep the lock image grayscale.

## Notes

- PocketBook may cache the lock image. If it does not refresh after changing
  books, reboot once.
- This was tested on the InkPad Color 3 only. Other models using the same
  `/mnt/ext1/system/resources/Line/taskmgr_lock_background.bmp` path may work,
  but are untested.

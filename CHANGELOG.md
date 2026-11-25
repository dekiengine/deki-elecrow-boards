# Changelog

Notable changes to `deki-elecrow-boards`. Engine and editor changes are in the
[engine changelog](https://github.com/dekiengine/deki-engine/blob/master/CHANGELOG.md).

A package's `minEngine` names the engine version it needs. Before 1.0 a
breaking change bumps the minor across the editor, the engine and every
package together, so a package with no changes of its own is still released
alongside one that has them.

## Unreleased

### Added
- `elecrow_crowpanel_advanced_9_p4`: the CrowPanel Advanced 9" ESP32-P4, with
  its MIPI-DSI display (EK79007, 1024x600) and GT911 touch. Runs on the
  board: display, touch, and Tiny Farm at 44.2 fps with three frame buffers
  and a 256 KB L2 cache (16.6 fps when it first ran). The backlight is driven
  with PWM at 30 kHz (98%), as Elecrow's firmware drives it; held high, the
  panel's brightness flickered.
- The board draws in strips in internal RAM, copied into the frame buffer by
  DMA (`drawInStrips`): 27 to 42 fps on the starter scene, 56.4 with
  deki-esp32-integration's larger scaled cache.

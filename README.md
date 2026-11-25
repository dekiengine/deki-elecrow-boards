# Deki Elecrow Boards

Build targets for Elecrow boards. Install the package and the boards appear in the Build panel's target list; choosing one copies it into your project with a boot scene that brings the display and touch up.

| Target | Board | Display | Touch |
| --- | --- | --- | --- |
| `elecrow_crowpanel_advanced_9_p4` | CrowPanel Advanced 9" ESP32-P4 | 1024x600 EK79007, MIPI-DSI (2 lanes), through deki-display | GT911 through deki-display |

**Status: experimental. The display, backlight and touch run on the board; the SD card, audio and WiFi are not set up yet.** Not in the Package Manager yet: it needs the next Deki release. Pins, timing and the panel's start-up come from Elecrow's own ESP-IDF examples (listed under the board). If something is wrong on your board, fix it in your project's copy and please open an issue.

[Deki](https://dekiengine.com) is a modular C++ engine for embedded systems and desktop, with a visual editor. Every capability is a package, and a board is one too: this package has no code, only a description of each board and the scene that boots it.

## Install

Package Manager in the Deki Editor, or `DekiEditor --packages-add deki-elecrow-boards <project>`. It brings `deki-esp32-integration`, `deki-display`, `deki-input`, `deki-i2c` and `deki-gpio` with it.

Then pick the board in the Build panel, or:

```
DekiEditor --tool set_active_platform --args '{"platform_id":"elecrow_crowpanel_advanced_9_p4"}' <project>
```

## The copy is yours

Choosing a board copies `platforms/<id>/` from this package into your project. Edit it there: the platform in the Build panel's editor, the boot scene like any scene.

When this package updates a board, the Build panel shows **update available**. **Update** takes the package's changes and keeps yours. **Restore defaults** goes back to the package's version. The command line has `update_platform` and `restore_platform`.

## CrowPanel Advanced 9" (ESP32-P4)

An ESP32-P4 at 360 MHz with 32 MB of PSRAM and 16 MB of flash, built with ESP-IDF for chip revisions before 3.0 (`CONFIG_ESP32P4_SELECTS_REV_LESS_V3`), as Elecrow's own firmware is. Its L2 cache is 256 KB with 128-byte lines (ESP-IDF's default is 128 KB with 64-byte lines): frames are drawn in PSRAM, and the larger cache took Tiny Farm from 16.6 to 18.3 fps, for 128 KB of internal RAM. A board with a revision 3 chip needs that line taken out of its copy's `sdkconfig` list, and can then run at 400 MHz.

Boot order, as the boot scene lists it:

1. LDO channel 3 at 2.5 V (`ESP32LdoSetup`): the supply of the P4's MIPI-DSI PHY.
2. LDO channel 4 at 3.3 V: the panel's rail, as Elecrow's examples turn it on.
3. Display: deki-display's `DsiPanelComponent`. EK79007, 1024x600, two DSI lanes at 900 Mbps, a 51 MHz pixel clock (HSYNC 70, HBP 160, HFP 160, VSYNC 10, VBP 23, VFP 12: 55.9 Hz). Its start-up is the one Espressif's EK79007 driver sends: a software reset (the panel's reset pin is not wired), two lanes, the module's own registers 80h to 86h, then sleep out. Three frame buffers in PSRAM, 1.2 MB each: the game draws into one the panel is not reading, and never waits for a refresh (with two, each frame waited about half of one). Frames are drawn in strips in internal RAM, each copied into the frame buffer by DMA while the next is drawn (`drawInStrips`): the starter scene's full-screen gradient went from 27 to 42 fps, and to 56.4 (the panel's refresh) with deki-esp32-integration's scaled cache keeping a whole-screen copy. The strips' height is the project's Rendering setting; 20 rows draws as fast here as the default 40 and keeps 83 KB more internal RAM for the game. Backlight on GPIO 31, driven with PWM at 30 kHz at 98%, as Elecrow's firmware drives it. Held high instead, the panel's brightness flickers, most at its edges.
4. I2C bus: port 0, SDA 45, SCL 46, 400 kHz.
5. Touch: deki-display's `TouchPanelComponent`. GT911 at 0x5D (0x14 as a fallback), interrupt on GPIO 42, reset on GPIO 40, 1024x600 as the panel.

Set up by this package: display, backlight, touch.

Not set up yet: the SD card (SDMMC), audio (I2S amplifier and PDM microphone), WiFi (the board's ESP32-C6, which needs Espressif's `esp_hosted`), the wireless module slot, the battery.

The serial console is UART0, through the board's CH340 USB-UART bridge; firmware is flashed through it.

Sources: Elecrow's [CrowPanel Advanced 9" ESP32-P4 repository](https://github.com/Elecrow-RD/CrowPanel-Advanced-9inch-ESP32-P4-HMI-AI-Display-1024x600-IPS-Touch-Screen) (`example/V1.2/idf-code`, lessons 05 and 07), Espressif's `esp_lcd_ek79007` driver, and the [board's wiki page](https://elecrow.com/wiki/CrowPanel_Advanced_9inch_ESP32-P4_HMI_AI_Display_1024x600_IPS.html).

## Adding a board

One folder per board:

```
platforms/<id>/<id>.json    the platform
platforms/<id>/boot.scene   what brings its hardware up
```

Prefix the id with `elecrow_`. Two installed packages offering the same id is an error, not a choice the editor makes for you.

## Community

Questions, ideas and what you made with Deki: [the Deki Discord](https://dekiengine.com/discord/).

## License

Apache 2.0. See [LICENSE](LICENSE).

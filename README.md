# GivEnergy Gen1 Battery Emulator

An RS485 emulator for the GivEnergy Gen1 low-voltage BMS, built to keep a
Gen1 Hybrid inverter running after its original battery pack died (red
fault LED, inverter idle, consumption reverting to grid). Runs on a
LilyGO T-CAN485 (ESP32) and answers the inverter's BMS polls as if a real
battery pack were connected.

## Hardware

- Board: LilyGO T-CAN485 (ESP32)
- BMS link: Serial1 @ 9600 8N1, RS485, wired to the inverter's BMS port
  - RX: GPIO21
  - TX: GPIO22
  - GPIO16 / GPIO17 / GPIO19 held HIGH
- USB console: Serial @ 115200

## What it does

- Answers FC03 holding-register polls (HR0-27), including the inverter's
  short poll `01 03 00 11 00 09` (HR17-25).
- Answers the three non-standard GivEnergy FC04 telemetry blocks (device 1
  "present", devices 2-5 "absent"), including the confirmed 38-byte Block 2
  capacity/SOC layout (4-byte big-endian `uint32` capacity fields, 0.01 Ah
  units; direct 0-100 SOC byte).
- Serves a web diagnostics/mapping-lab page over WiFi (RS485 activity, last
  RX/TX frames, decoded fields, live counters) and supports OTA updates.

See [open-giv/bms-analysis](https://github.com/open-giv/bms-analysis) for
the community protocol reference this project builds on and cross-checks
against (see [issue #41](https://github.com/open-giv/bms-analysis/issues/41)
for the connection between the two).

## Setup

1. Copy `include/secrets.h.example` to `include/secrets.h` and fill in your
   real WiFi SSID/password. `secrets.h` is gitignored and never committed -
   do not put real credentials anywhere else in the source.
2. Open the project in PlatformIO (VS Code) and build/upload
   `src/main_full_listener.cpp` (see `platformio.ini`).
3. Wire the board to the inverter's BMS RS485 port per the pinout above.

## Status

Experimental / personal project, not a packaged library. Confirmed working
against a real Gen1 Hybrid inverter: HR25 (current-limit field) reaches the
inverter and permits charging, with the inverter actively polling the
emulator over RS485. Several status-field bits (FC04 Block 2 offsets
27/29/31/33) are exposed as raw settable values but their bit-level meaning
isn't fully established yet. Values are volatile and reset to safe defaults
on reboot.

**Safety note:** keep inverter charge/discharge disabled while experimenting
with register values, and use conservative limits for functional tests.

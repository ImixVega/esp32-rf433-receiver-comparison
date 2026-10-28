# ESP32 RF433 receiver comparison (ESPHome)

**Four 433.92 MHz ASK/OOK receivers, one ESP32, one remote control, real field tests up to 110 m.**

This small project compares four simple DATA-output 433 MHz receiver setups in ESPHome:

- **AK-R231101** — original black wire-coil antenna
- **RX470C-V01 + spring** — long bare helical antenna
- **RX470C-V01 + SMA** — 433 MHz SMA whip via 50 Ω coax, originally supplied with a CC1101 433 MHz module
- **MX-RM-5V** — tested at **3.3 V**; a dedicated spring antenna was added before the second series

[**Polish documentation / Dokumentacja po polsku**](README_PL.md)

The motivation: an Ethernet-connected **KinCony KC868-A8** at an outdoor gate has few conveniently accessible GPIOs. A digital DATA-output receiver can use one GPIO without adding SPI wiring or another Wi-Fi ESP32. **CC1101 was not measured head-to-head.** The receiver test is independent of the proposed gate/lighting installation.

## Key result

In a second series toward the street, with the receivers remaining **indoors** (in an electronics-filled home office), both RX470C versions decoded **25/25 intended button presses at 110 m**. At 40 m in a different location on that route they decoded only **14/25**. This is not a certified or guaranteed range; buildings, antenna orientation and multipath affect results strongly.

The remote used in the experiment was an **8-button, 433 MHz handheld remote with a telescopic antenna**. Results with a small key-fob or another transmitter may differ considerably.

## First series — 50 intended presses per location

| Distance | AK-R231101 | RX470C SMA | RX470C spring | MX-RM-5V |
|---|---:|---:|---:|---:|
| 1 m, desk | 50/50 (100%) | 50/50 (100%) | 50/50 (100%) | 12/50 (24%) |
| **10 m, two walls** ⚠️ | 50/50 (100%) | 23/50 (46%) | 26/50 (52%) | 0/50 |
| 28 m | 50/50 (100%) | 32/50 (64%) | 38/50 (76%) | 0/50 |
| 57 m | 41/50 (82%) | 43/50 (86%) | 37/50 (74%) | 0/50 |
| 75 m | 0/50 | 8/50 (16%) | 30/50 (60%) | 0/50 |

⚠️ **The 10 m comparison is NOT controlled:** AK's antenna was vertical while both RX470C antennas were horizontal. Do not use that row to rank receiver performance. At other outdoor positions the antennas were upright. The one-metre desk antenna orientation was not fully documented. See [first-series raw counts](results/range_tests.csv).

## Second series — 25 intended presses per location, opposite direction toward the street

| Distance | AK-R231101 | RX470C SMA | RX470C spring | MX-RM-5V |
|---|---:|---:|---:|---:|
| 1 m | 25/25 (100%) | 25/25 (100%) | 22/25 (88%) | 0/25 |
| 10 m | 25/25 (100%) | 25/25 (100%) | 24/25 (96%) | 0/25 |
| 40 m | 19/25 (76%) | 14/25 (56%) | 14/25 (56%) | 0/25 |
| 60 m | 25/25 (100%) | 23/25 (92%) | 22/25 (88%) | 0/25 |
| 90 m | 16/25 (64%) | 22/25 (88%) | 24/25 (96%) | 0/25 |
| 110 m | 22/25 (88%) | **25/25 (100%)** | **25/25 (100%)** | 0/25 |

For the second series, MX-RM-5V received a dedicated soldered spring antenna, but **still ran at 3.3 V**. Many MX-RM-5V boards are rated for 5 V; therefore, the zero readings **must not be interpreted as a fair comparison with a correctly powered MX-RM-5V**. Its DATA line must be level-shifted or verified before connecting it to a 3.3 V ESP32 input.

See [second-series raw counts](results/range_tests_series2.csv) and [test limitations](results/notes.md). Never pool the two series into one uniform distance-response curve: directions, sites, press counts and MX antenna conditions changed.

## Hardware and connections

Classic **ESP32-WROOM-32** or **ESP32-WROOM-32U**, running ESPHome with ESP-IDF:

| Receiver | DATA on ESP32 |
|---|---|
| RX470C-V01 with spring | GPIO25 |
| RX470C-V01 with SMA | GPIO32 |
| AK-R231101 | GPIO26 |
| MX-RM-5V | GPIO27 |

Share a common **GND**. Supply **VCC according to each actual module's specifications**. Pin order varies by board; check each PCB's GND / DATA / VCC markings rather than assuming mechanically identical headers have an identical pinout. **Never apply a 5 V DATA output directly to ESP32 GPIO.** The published test uses one independent `remote_receiver` per RF module.

## Run the ESPHome test

1. Copy [`esphome/esp32-rf433-benchmark.yaml`](esphome/esp32-rf433-benchmark.yaml) to your ESPHome directory.
2. Configure your own Wi-Fi and ESPHome API/OTA credentials using [`secrets.example.yaml`](esphome/secrets.example.yaml). Never commit actual `secrets.yaml`.
3. Set your own **test** RCSwitch protocol-1 24-bit code at the top of the YAML, as matching decimal and binary strings. A harmless placeholder is committed; **the user's real remote codes are not published**.
4. Upload to ESP32; add the node to Home Assistant. Set "planned presses" to 25, 50 or another number, and reset counters before each location.
5. Press the same button at a known interval, then compare **recognized presses**, **matching frames** and **calculated reception percentage**.

**Counts are estimates, not RF RSSI.** Repeated frames are grouped using an 800 ms off delay. A sustained press, bursts or interference can cause overcounts or undercounts. The trial used a permissive **50% pulse tolerance**; preserve that to reproduce the original test or tune with care for your devices. Last-code diagnostics may report noise or unrelated RF codes.

The public YAML is an adapted, sanitized example based on working tests. It has not been fully recompiled and hardware-validated after removing private settings.

## ESPHome versus Home Assistant (optional application, not part of the benchmark)

ESPHome **decodes the received RF pulses on ESP32**. Home Assistant does not directly demodulate or decode the radio signal. A separate production receiver can publish recognized codes as `esphome.rf433_received` events, and HA can use those events in automations. When **multiple ESP devices** hear the same press, deduplication in HA avoids triggering one light/gate automation more than once.

**This repository contains only the four-channel comparison firmware.** It does **not** contain the user's real home-control automations, access codes, gate-control firmware, or a production access-control implementation. Fixed-code 433 MHz remotes can be replayed; do not rely on one unverified RF code as the sole security check for an access gate.

## Scope and conclusions

- Successful reception by RX470C suggests a practical **low-GPIO alternative for testing** at the Ethernet-connected KC868-A8 gate, inside its plastic enclosure; perform a final on-site check before using it in daily operation.
- A plastic antenna sleeve alone is not evidence of poorer performance. Coil geometry, antenna matching, polarization, nearby electronics, and path matter.
- The home-office bench had four receivers beside computers, screens and other electronics. This was **not** a controlled EMI test: the exact effect of local interference was not measured.
- **CC1101 was not included in these field measurements**. Reusing an antenna from a CC1101 kit does not compare receiver chip performance.

Licensed under [MIT](LICENSE). Independent repeat tests, supply-voltage measurements and corrections are welcome.

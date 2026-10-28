# Receiver connections / Połączenia

Each module supplies a **digital DATA** output to a separate GPIO of a classic ESP32-WROOM-32 or ESP32-WROOM-32U:

| Module | Signal | ESP32 pin |
|---|---|---|
| RX470C-V01 + spring | DATA | GPIO25 |
| RX470C-V01 + SMA | DATA | GPIO32 |
| AK-R231101 | DATA | GPIO26 |
| MX-RM-5V | DATA | GPIO27 |

All modules share a common **GND**. Power each module from the voltage required by its own datasheet/PCB. All DATA lines into ESP32 must be **3.3 V compatible**; if a module is run on 5 V, verify the DATA high level with a meter/oscilloscope or use proper level shifting. **Do not supply 5 V directly to ESP32 GPIO.**

Identical-looking four-pin boards are not guaranteed to share the same header order. Confirm silkscreen markings for each module before wiring.

The MX-RM-5V test used **3.3 V** throughout. The later addition of a dedicated antenna did not change its measured zero-detection field results; supply-voltage mismatch remains a likely possible explanation.

## Antennas

- Spring antennas: positioned upright in most field trials. The **10 m measurement in series 1 is not comparable** because RX470C antennas were horizontal while AK was upright.
- RX470C SMA: 433 MHz antenna from a CC1101 433 MHz kit, attached by a **50 Ω coaxial cable**; center conductor to ANT, shield to a confirmed ground. Use a short RF lead.
- Do not infer frequency solely from SMA connector type; only an antenna known to be matched for 433 MHz was used here.

## Safety for optional gate use

The code in this repository is for measuring reception **only**. It does not implement a gate actuator, access control, RF transmitter or Home Assistant automations. Fixed-code remotes are susceptible to replay attacks and should not be used alone to authenticate opening a gate.

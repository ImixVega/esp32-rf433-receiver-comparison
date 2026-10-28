# Test conditions and limitations / Warunki i ograniczenia

- **Date:** October 27–28, 2026, in and around a residential building in Poland.
- **Receivers:** four DATA-output ASK/OOK modules connected simultaneously to one ESP32 on a solderless breadboard in a home office. The office contained multiple computers, monitors and other electronics, but no independent measurement of electromagnetic interference was made.
- **Transmitter:** handheld **8-button 433 MHz remote with extendable telescopic antenna**. Radio power, true carrier frequency, antenna deployment length and pilot orientation were not measured or held constant.
- **Firmware:** ESPHome remote_receiver, RCSwitch protocol 1, pulse tolerance 50%, filtered repeat press estimates using delayed_off 800 ms; intended press interval about 1.5–2 seconds.
- **No RSSI:** all reported percentages are decoded-press counts divided by intended presses, not measured RF signal strength.
- **First test:** locations 1/10/28/57/75 m, each 50 intended presses. At 10 m the AK antenna was upright but RX470C antennas horizontal. **Do not use the 10 m observation to rank hardware.**
- **Second test:** locations 1/10/40/60/90/110 m, each 25 intended presses. The transmitter was moved in a **different direction**, toward the street. The receiver setup remained indoors.
- **MX-RM-5V:** ran on 3.3 V in both series. A dedicated soldered helical antenna was added before series 2, without fixing the poor observed results. Many variants expect 5 V, so under-voltage is a plausible confounder. Do not generalize MX-RM-5V performance from these numbers.
- **RX470C antennas:** one bare helical spring, the other an SMA 433 MHz antenna and coaxial cable taken from a CC1101 433 MHz antenna kit. The plastic sleeve by itself is not evidence of worse RF performance.
- **Electronics near the receivers:** laptop/PCs, screens, keyboard and wiring; their actual contribution to interference was not isolated.
- **Multipath:** distance does not uniquely predict received quality. At 110 m the RX470Cs performed better than at 40 m in the second series.
- **Experimental limitations:** no systematic swap of receiver board positions; no repeated independent trials at every point; no control for remote antenna orientation or battery voltage.
- **Do not compare with CC1101 directly:** CC1101 itself was not run in the field trials. Only an antenna originating from its kit was reused.
- **RF gate safety:** do not use easily replayable fixed RF codes as the only access control; HA deduplication helps prevent repeated actions but is not authentication or RF anti-replay.
- **Public privacy:** no real remote codes, internal LAN addressing, Wi-Fi keys or automation secrets are committed. Any public screenshots should have decoded RF codes masked.

## Meaning of each sensor

1. **Presses**: estimated distinct accepted activations of the chosen test code (not guaranteed one-to-one with physical clicks).
2. **Matching frames**: all matching, correctly decoded RCSwitch frames; one physical press typically repeats packets.
3. **Reception percentage**: estimated presses divided by the user's manually configured intended count. Values above 100% mean overcount or wrong denominator.
4. **Last decoded code**: any nonzero code the decoder reported. Noise may appear here even when matching-frame count stays zero.

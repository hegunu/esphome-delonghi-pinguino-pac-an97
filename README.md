# DeLonghi Pinguino (PAC AN97) – ESPHome IR control

Control a DeLonghi Pinguino portable air conditioner (**PAC AN97**) from Home Assistant using a
[Seeed XIAO Smart IR Mate](https://www.seeedstudio.com/) (ESP32-C3) running ESPHome.

It generates the IR frame for any combination of mode, fan speed and temperature (no recorded
raw codes per button), and exposes a real **climate** entity driven by a room temperature sensor.

> Tested only on my own unit. Not affiliated with DeLonghi.

## Features
- Temperature 18–32 °C, fan Auto / 1/3 / 2/3 / 3/3
- Modes: Cool, Dehumidify (dry), Fan only, Comfort (house icon, probably "Real Feel"), Off
- ESPHome `thermostat` climate entity with hysteresis and compressor protection
  (3 min min. run / 3 min min. off for cooling)
- Plain select/number entities (`AC Läge`, `AC Fläkt`, `AC Temperatur`) for manual control

## Setup
1. Flash the XIAO IR Mate with `delonghi-pinguino.yaml` (adjust name, secrets, and the
   `sensor.YOUR_ROOM_TEMPERATURE_SENSOR` entity id).
2. Needs `wifi_ssid`, `wifi_password` and `ap_password` in `secrets.yaml`.
3. In Home Assistant use the new `AC Termostat` climate entity.
   Mode mapping in the thermostat card: Cool = Kyla, Dry = dehumidify, Fan only = fan,
   **Auto = Comfort**, Off = off.

Note: the thermostat only starts cooling when the room is above target + 0.5 °C.

## IR protocol (PAC AN97 remote)
38 kHz, 40 bits, MSB first, the whole state is sent in every frame:

`D2 | b1 | b2 | b3 | 00`

| Byte | Meaning |
|------|---------|
| `D2` | constant |
| `b1` | bits 7..3 = temperature − 12 (bit7 = LSB … bit3 = MSB), bits 1..0 = fan: auto `00`, 1/3 `10`, 2/3 `01`, 3/3 `11` |
| `b2` | mode: cool `80`, dry `C0`, fan only `A0`, comfort `88`, off `00` |
| `b3` | `30` in my latest captures (`70` earlier); no visible effect found |
| last | `00` |

Timing: header ≈ 9150 µs mark / 4575 µs space, bit mark ≈ 520 µs, "1" space ≈ 1690 µs,
"0" space ≈ 590 µs, then a stop mark and ≈ 10 ms gap.

Verified against ~35 frames captured from the original remote.

## Disclaimer
Use at your own risk. Short cycling can damage compressors – keep the minimum run/off times.

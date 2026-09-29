![picokit-02-traffic-light](https://raw.githubusercontent.com/mytechnotalent/picokit-02-traffic-light/main/picokit-02-traffic-light.png)

<br>

## FREE Reverse Engineering Self-Study Course [HERE](https://github.com/mytechnotalent/reverse-engineering)
## FREE Embedded Hacking Course [HERE](https://github.com/mytechnotalent/Embedded-Hacking)

<br>

# PICOKIT-02 TRAFFIC LIGHT

### Traffic Light Phases and Authenticated Heartbeat
#### Lesson 2 of the Picokit Series

<br>

***
**LEGAL DISCLAIMER:**
The information, tools, and code provided in this repository and course are strictly for educational, research, and defensive purposes only.

You are explicitly prohibited from using any materials contained herein to access, test, modify, or exploit any device, network, or system that you do not own 100% or for which you do not have explicit, documented, and legally binding authorization to interact with.

By using this repository and course, you acknowledge and agree that:

1. Any illegal, unauthorized, or malicious use of this information is solely your responsibility.
2. The author(s) and contributor(s) of this repository and course shall not be held liable for any damages, legal repercussions, criminal charges, or unauthorized actions resulting from the use, misuse, or abuse of the contents herein.
3. You will comply with all applicable local, state, national, and international laws regarding cybersecurity and computer fraud.

**IF YOU DO NOT AGREE WITH THESE TERMS, DO NOT USE THIS REPOSITORY AND COURSE.**
***

<br>
<br>

## Overview

The second Picokit lesson. The node runs a real traffic light sequence on the
red, green, and yellow lamps, holding red for three seconds, green for three
seconds, and yellow for one second, and it blinks the onboard LED once per
phase. Every five seconds it transmits an authenticated heartbeat over LoRa
to a Python gateway that logs and displays it. It reuses the standard node
shape introduced in lesson one and adds a multi-duration phase state machine.

<br>

## What it teaches

- A multi-duration phase state machine: red 3 s, green 3 s, yellow 1 s.
- Reading the elapsed phase from a monotonic clock and scheduling the next
  transition.
- Sealing a tiny JSON body with Argon2id and XChaCha20-Poly1305 and sending it
  with an AT+SEND over the RYLR998.
- The gateway side: receive, authenticate, reject, log, and display.

<br>

## Hardware

| Peripheral | Pico 2 pin | Role |
| --- | --- | --- |
| Red / Yellow / Green | GP16 / GP18 / GP17 | the signal |
| Onboard LED | GP25 | heartbeat, one blink per phase |
| RYLR998 | GP8 TX / GP9 RX | LoRa heartbeat |
| Debug Probe | SWCLK/SWDIO/GND, GP0/GP1 | SWD and the console |

<br>

## How it works

The node runs `monitor_step` in a loop. It enters the red phase, then green,
then yellow, holding each lamp for its own duration and blinking GP25 once per
phase, and every 5 seconds it seals `{"n":2,"s":<seq>,"p":<phase>}` with the
field key and sends it over LoRa. The gateway authenticates each frame and
only then parses it.

<br>

## Build and flash

```bash
cd firmware
cmake -S . -B build -G Ninja -DPICO_BOARD=pico2 -DPICO_PLATFORM=rp2350-arm-s
cmake --build build
openocd -f interface/cmsis-dap.cfg -f target/rp2350.cfg \
  -c "program build/picokit_02_traffic_light.elf verify reset exit"
```

<br>

## Watch the node

Open the console at 115200 and reset:

```text
BOOT
I2C scan:
  no devices
=== PICOKIT-02 TRAFFIC LIGHT // PHASES + AUTHENTICATED HEARTBEAT ===
PHASE 0 RED seq=0
PHASE 1 GREEN seq=0
PHASE 2 YELLOW seq=0
RX from 0x0001, N bytes
```

<br>

## The gateway

```bash
cd gateway
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
python3 listen.py --port /dev/cu.usbserial-A50285BI --hub 0001 --network 18 --db gateway.db
```

It prints `OK node=2 rssi=...` per authenticated heartbeat. The terminal
dashboard `python3 tui.py --db gateway.db` and the web dashboard
`python3 web/app.py --db gateway.db` show the same rows.

<br>

## Verify

```bash
python3 .opencode/skill/embedded-c-standard/audit_c_standard.py
python3 .opencode/skill/embedded-python-standard/audit_python_standard.py
python3 .opencode/skill/iot-readme-standard/validate_readme.py
python3 .opencode/skill/iot-banner-standard/validate_banner.py
python3 scripts/run_tests.py
python3 scripts/check_coverage.py
```

<br>

# Next
[picokit-03-led-pwm-breathing](https://github.com/mytechnotalent/picokit-03-led-pwm-breathing)

<br>

# License
[MIT License](https://github.com/mytechnotalent/picokit-02-traffic-light/blob/main/LICENSE)

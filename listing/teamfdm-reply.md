# TeamFDM reply — thread #6534 "BTT SFS 2.0 with Leviathan mainboard"

> À poster en réponse au thread (compte user). Ton : partage d'une alternative,
> humble. La réponse est AUTOSUFFISANTE (tout le how-to inline) ; le repo n'est
> qu'un pointeur final.
> https://www.teamfdm.com/forums/topic/6534-btt-sfs-20-with-leviathan-mainboard/

---

Nice find with PC0/PC1 + the neopixel 5V — glad it works.

For anyone landing here who'd rather not deal with board-specific pins and rails
at all, here's another route that works on any printer: give the sensor **its own
MCU**. A Waveshare **RP2040-Zero** (~$3) flashed as a plain Klipper USB MCU reads
both SFS signals, and the mainboard isn't involved at all — one USB cable to the
Pi.

**1. Flash the pico** — stock Klipper, nothing custom:

```
make menuconfig   # Micro-controller: Raspberry Pi RP2040 · Communication: USB
make
```

Hold BOOT, plug USB → the pico mounts as an `RPI-RP2` drive → copy
`out/klipper.uf2` onto it. Done. (Optional: put Katapult under it so future
reflashes don't need the button.)

**2. Wire the SFS to the pico** — 4 wires:

| SFS wire | → pico |
|---|---|
| RED (VCC) | **3V3** |
| BLACK (GND) | GND |
| GREEN (motion) | GP0 |
| BLUE (switch) | GP1 |

Power it from the pico's **3.3 V** pin: the SFS is rated 3.3–5 V per the BTT
manual (p.5 — the "5V" in BTT's wiring diagrams is just because their example
boards are 5V-tolerant STM32s), and the RP2040 itself is *not* 5V-tolerant, so
3.3 V keeps every signal safe whatever the output topology is. This also
sidesteps the diode-drop / sagging-rail questions entirely — the pico's 3V3
comes regulated from USB.

**3. Klipper config:**

```ini
[mcu sfs]
serial: /dev/serial/by-id/usb-Klipper_rp2040_XXXX-if00

[filament_motion_sensor sfs_motion]
switch_pin: ^sfs:gpio0        # motion encoder — clog/slip detection
detection_length: 2.88        # SFS pulse pitch; raise if false triggers
extruder: extruder
pause_on_runout: False        # observe first in Mainsail, arm later

[filament_switch_sensor sfs_switch]
switch_pin: ^sfs:gpio1        # runout switch
pause_on_runout: False
```

The `^` pull-ups are mandatory (both SFS outputs are switch-to-ground). Bring it
up with `pause_on_runout: False`, watch the two sensors in Mainsail while you
poke filament in and out, then arm the pause once you trust it. One gotcha if
you already have other RP2040s (Eddy, PIS…): they all share the same USB
VID:PID, so identify the right `/dev/serial/by-id/` entry by unplugging the pico
and seeing which one disappears.

I also designed a one-piece printed receptacle that holds the SFS + pico
together and mounts on a 2020 extrusion — that plus the full write-up
(flashing with Katapult, bring-up, assembly) is here if useful:
GitHub `Nitrooxyde/BTT-SFS-2.0-RP2040-Zero-Klipper` · Printables model 1781579.
Free/open (MIT + CC-BY), my own project.

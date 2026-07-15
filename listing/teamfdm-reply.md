# TeamFDM reply — thread #6534 "BTT SFS 2.0 with Leviathan mainboard"

> À poster en réponse au thread (compte user). Ton : partage d'une alternative,
> humble, sans critiquer la carte ni prétendre avoir eu les mêmes soucis.
> https://www.teamfdm.com/forums/topic/6534-btt-sfs-20-with-leviathan-mainboard/

---

Nice find with PC0/PC1 + the neopixel 5V — glad it works.

If someone lands here and would rather skip the board-specific wiring entirely,
another route is to give the sensor **its own MCU**: a Waveshare **RP2040-Zero**
(~$3) flashed as a standard **Klipper USB MCU**, reading both SFS signals
directly:

```ini
[mcu sfs]
serial: /dev/serial/by-id/usb-Klipper_rp2040_XXXX-if00

[filament_motion_sensor sfs_motion]
switch_pin: ^sfs:gpio0
detection_length: 2.88
extruder: extruder

[filament_switch_sensor sfs_switch]
switch_pin: ^sfs:gpio1
```

- No mainboard pins or rails involved — one USB cable to the Pi.
- The SFS runs from the pico's 3.3 V (it's rated 3.3–5 V per the BTT manual,
  p.5). ⚠️ If you copy this, do power it at 3.3 V — the RP2040 itself is *not*
  5V-tolerant.
- Both outputs are switch-to-ground, so the `^` internal pull-ups do the rest.

I put together a one-piece printed receptacle that holds the SFS + pico and
mounts on a 2020 extrusion, with the wiring/flashing/config written up:

- GitHub (docs + config + STL): https://github.com/Nitrooxyde/BTT-SFS-2.0-RP2040-Zero-Klipper
- Printables: https://www.printables.com/model/1781579-btt-sfs-20-rp2040-zero-receptacle

Full disclosure: my own project, free/open (MIT + CC-BY). Hope it's useful to
someone.

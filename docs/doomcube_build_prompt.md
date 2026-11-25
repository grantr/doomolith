### Project Context: 300mm Voron Doomcube Electronics & Klipper Build

I am building a 300mm Voron Doomcube with split electronics bays (AC + PSUs in the bottom bay; compute, MCU, and drivers in the top bay). Please use the finalized hardware and wiring baseline below to help me finish the remaining `[TBD]` details and generate my Klipper/Moonraker configs:

#### 1. Finalized Hardware & Power Architecture
* **Host & Main MCU (Top Bay):** Raspberry Pi 4B + **FYSETC Spider V3.0 H7** (or BTT Octopus Pro V1.1 H723). USB `5V` jumper removed (or USB `+5V` wire isolated) to prevent back-powering when the 24V PSU is off.
* **X/Y Motion (36V DC):** 2WD Monolith gantry with **2× LDO-42STH48-2504AC** steppers (`2.5A` peak / `~1.77A` RMS) driven by **2× external BTT TMC5160T Plus** modules fed directly from a **Mean Well LRS-200-36** (`36V / 212W`, fused at `10A` in the bottom bay).
* **Z & Purger Motion (24V DC):** 4× independent Z steppers (`Z0–Z3`) + 1× **NEMA 11 Goose Belt Purger** (`M7` slot) on onboard TMC2209s, fed from a **Mean Well LRS-350-24** (`24V / 350.4W`, fused at `15A` in the bottom bay). All DC `V-` grounds (`5V`, `24V`, `36V`) are tied together.
* **CAN Bus Network (10 Nodes @ 1M Baud):** Dedicated USB-to-CAN adapter (`txqueuelen 2048` on `can0`) connecting:
  * **1× BTT EBB36 Gen2** toolhead board driving a **MOONS' CSE14HRA2L4100** NEMA 14 pancake extruder (`1.0A` peak, `run_current: 0.600` at 24V)
  * **8× per-lane Emu MMU SLB** boards
  * **1× Stealthmax V2** chamber filter
* **AC Heating Loads (120V AC, 15A Slow-Blow Master IEC Inlet):**
  * **450W AC Bed Heater** (`3.75A`) on an Omron G3NA-210B SSR + `125°C–150°C` thermal cutoff.
  * **2× 250W AC Chamber Heaters** wired **in parallel** (`500W` total / `4.17A`) on one Omron G3NA-210B SSR, with **one separate `100°C–120°C` thermal cutoff** in series with each heater.
* **Always-On Standby Pi & Safety Interlock Contactor:**
  * **Mean Well HDR-30-5** (`5V` standby PSU, `3A–5A` inline fuse) keeps the Pi 4B running 24/7.
  * **Finder 22.32.0.024.1320** (`25A`, 2-pole NO, `24V DC` coil @ `~83 mA`) cuts both AC Live and Neutral to the `LRS-350-24`, `LRS-200-36`, Bed SSR, and Chamber SSR.
  * **Coil Control & Overrides:** `HDR-30-5 (+5V)` -> **Front-Skirt Latching Safety Interlock Switch** -> **Pololu U3V50F24** (`5V->24V` step-up) -> Finder `A1 (+)`. Finder `A2 (-)` is pulled to `Common V-` either by a bottom-bay **1-channel 3.3V optocoupler-isolated MOSFET module** (driven via a 700mm `22 AWG` twisted pair from Pi `GPIO17` + `GND`) or bypassed manually via an **internal top-bay Force-On maintenance switch**.

#### 2. Remaining `[TBD]` Tasks to Complete Next
1. Confirm pinout/mounting dimensions once the exact 3.3V optocoupler MOSFET board SKU is chosen (4-terminal vs. 6-terminal and whether a `3.3V VCC` logic pin is required).
2. Generate the `moonraker.conf` `[power printer]` configuration (`gpiochip0/gpio17` with `restart_klipper_when_powered: True`) and a `PRINT_END` auto-cooldown shutdown macro.
3. Generate the consolidated `printer.cfg` sections for the main MCU, external 5160 Plus X/Y, EBB36 Gen2 + `CSE14HRA2L4100`, NEMA 11 Goose Belt Purger, and dual-250W `[heater_generic chamber]`.

# Voron Doomcube Electronics, Power & Wiring Specification

## 1. System Architecture Overview

* **Frame & Bay Split:** 300mm build-area Doomcube with a split electronics layout:
  * **Bottom Electronics Bay:** AC mains inlet, standby 5V PSU, dual-pole safety contactor, 24V & 36V DC power supplies, Bed SSR, Chamber SSR, and source-side AC/DC fuses.
  * **Top Electronics Bay:** Raspberry Pi 4B host, main Klipper MCU, external X/Y stepper drivers, CAN bus interface/breakout, and internal maintenance override switch.
* **Motion & Extrusion Hardware:**
  * **X/Y Gantry:** 2WD Monolith gantry driven by **2× LDO-42STH48-2504AC** steppers (`2.5A` peak / `~1.77A` RMS, `1.4 mH` inductance) powered at **36V DC** via **2× external BTT TMC5160T Plus** drivers.
  * **Z Axis:** 4× independent Z steppers (`Z0–Z3`) driven at **24V DC** on the main MCU for synchronized Quad Gantry Leveling (`QUAD_GANTRY_LEVEL`).
  * **Extruder:** **MOONS' CSE14HRA2L4100** NEMA 14 pancake stepper (`1.0A` peak / `~0.707A` RMS, Class B 130°C) driven at **24V DC** via a **BIGTREETECH EBB36 Gen2** CAN toolhead board (`run_current: 0.600`).
  * **Purger:** **NEMA 11 stepper** in a Goose Belt Purger (`[manual_stepper goose_belt]`, `run_current: 0.400`), driven at **24V DC** from a spare onboard stepper slot (`M7`).
* **CAN Bus Network (10 Nodes @ 1,000,000 / 1M Baud):**
  * **1×** BTT EBB36 Gen2 (Toolhead)
  * **8×** Per-lane Emu MMU SLB boards
  * **1×** Stealthmax V2 chamber filter
  * *Host CAN Interface:* Dedicated USB-to-CAN adapter (e.g., **BTT U2C V2.1** or **Isik's Tech ToqueCAN** HAT) with `txqueuelen 2048` on `can0`.
* **AC Heating Loads (120V AC):**
  * **Bed Heater:** 450W silicone AC mat (`3.75A` @ 120V) switched by a dedicated zero-cross SSR + 125°C–150°C thermal cutoff.
  * **Chamber Heaters:** **2× 250W** AC heaters wired **in parallel** (`500W` total / `4.17A` @ 120V), controlled by one zero-cross SSR, with **one separate 100°C–120°C thermal cutoff** in series with each heater.

---

## 2. Bill of Materials (BOM) & Selected Components

### Bottom Electronics Bay (AC & Power Conversion)

| Subsystem | Selected Component | Key Specifications & Notes |
| :--- | :--- | :--- |
| **AC Inlet & Master Switch** | **15A 250V AC IEC C14 Inlet** (DPST rocker + fuse holder) | Master hard disconnect switching both AC Live and Neutral. |
| **Standby 5V PSU** | **Mean Well HDR-30-5** (DIN-rail) | Always-on 5V DC supply powering Raspberry Pi 4B and contactor control circuit. |
| **24V Main PSU** | **Mean Well LRS-350-24** (`350.4W` / `14.6A`) | Chassis: `215 × 115 × 30 mm` (M4 mounts). Powers main MCU, Z steppers, purger, and 10 CAN nodes. |
| **36V Motion PSU** | **Mean Well LRS-200-36** (`212W` / `5.9A`, fanless) | Chassis: `215 × 115 × 30 mm` (M4 mounts; identical to LRS-350-24). Dedicated to external 5160 Plus X/Y drivers. |
| **PSU DIN Brackets** | **4× Voron `lrs_psu_bracket_clip.stl`** (2 per PSU) | Printed in ABS/ASA; attach with **M4×6 mm** screws max to avoid internal PCB contact. |
| **Double-Pole Contactor** | **Finder 22.32.0.024.1320** (`25A`, 2NO, `24V AC/DC` coil) | Cuts both AC Live & Neutral to 24V PSU, 36V PSU, Bed SSR, and Chamber SSR. Coil draws `~83 mA` (`2W`). |
| **5V-to-24V Coil Booster** | **Pololu U3V50F24** (Fixed 24V step-up regulator) | Fed from always-on 5V via Skirt Safety Interlock; includes M2 mounting holes (`43.2 × 10.2 mm` pitch) for DIN carrier. |
| **Optocoupler Switch** | **1-Channel 3.3V Optocoupler MOSFET Module** | Low-side MOSFET switch (`XY-MOS` / `D4184` / `LR7843` style) mounted on a 35mm DIN-rail PCB carrier. |
| **Safety Interlock Switch** | **Latching Skirt Rocker Switch** | Mounted on front skirt; wired in series on `+5V` feeding Pololu `VIN` (hardware lockout). |
| **Heater SSRs** | **2× Omron G3NA-210B-UTU DC5-24** (`10A`) + heatsinks | 1× for 450W bed heater; 1× for parallel 2× 250W chamber heaters. |
| **DIN Distribution** | **Wago 221-415** DIN carriers or **Phoenix Contact PT 4** | Separate blocks for Switched AC Live, Switched AC Neutral, Unswitched Earth GND, and Common DC GND (`V-`). |

### Top Electronics Bay (Compute, Motion & CAN)

| Subsystem | Selected Component | Key Specifications & Notes |
| :--- | :--- | :--- |
| **Klipper Host** | **Raspberry Pi 4B** | Powered 24/7 by bottom-bay `HDR-30-5` via 18–20 AWG USB-C power feed. |
| **Main Klipper MCU** | **FYSETC Spider V3.0 H7** *(or BTT Octopus Pro V1.1 H723)* | Narrow `155.3 × 76.5 mm` footprint (Spider) leaves generous DIN space; 8 driver slots (`4× Z`, `2× X/Y adapters`, `1× NEMA 11`, `1× spare`). |
| **External X/Y Drivers** | **2× BTT TMC5160T Plus** | Powered at **36V DC** from `LRS-200-36`; keep StepStick-to-driver SPI ribbons under `15–20 cm`. |
| **Z & Purger Drivers** | **5× TMC2209** (onboard main MCU) | Powered at **24V DC** (`4×` for `Z0–Z3`, `1×` for `M7` Goose Belt Purger). |
| **CAN Interface** | **BTT U2C V2.1** *(or Isik's Tech ToqueCAN HAT)* | Runs 1M baud CAN network across toolhead, Stealthmax V2, and 8 Emu MMU SLBs. |
| **Maintenance Override** | **Internal Force-On Toggle/Rocker Switch** | Located inside upper electronics bay; pulls Finder `A2 (-)` directly to top-bay `GND (V-)` for offline servicing. |

---

## 3. Complete 7-Fuse Safety Map

| # | Circuit / Branch | Fuse Rating & Type | Exact Physical Location | Protection Scope |
| :- | :--- | :--- | :--- | :--- |
| **1** | **Main AC Inlet** | **15A Slow-Blow** (`5×20mm` ceramic, `T15AH250V`) | Integrated inside **15A IEC C14 inlet** on AC Live leg before any splits. | Protects `14 AWG` main AC feed (`~10.5A` max printer load) while tolerating PSU turn-on surge. |
| **2** | **24V Main DC Line** | **15A Fast-Blow** (ATO/ATC automotive blade) | Bottom bay, **within 10–15 cm of LRS-350-24 `V+`** before `16 AWG` trunk rises to top bay. | Protects `16 AWG` 24V trunk feeding MCU, Z/purger steppers, EBB36 Gen2, Stealthmax, and 8 Emu lanes. |
| **3** | **36V X/Y DC Line** | **10A Fast-Blow** (ATO/ATC automotive blade) | Bottom bay, **within 10–15 cm of LRS-200-36 `V+`** before `16–18 AWG` trunk rises to top bay. | Protects `16–18 AWG` 36V trunk feeding external TMC5160T Plus X/Y drivers. |
| **4** | **5V Standby Line** | **3A – 5A Fast-Blow** (inline ATO/ATC blade or glass) | Bottom bay, **immediately after `HDR-30-5` `+5V`**, before `18–20 AWG` run to top bay. | Protects always-on 5V feed to Raspberry Pi 4B and contactor coil booster. |
| **5** | **450W Bed Cutoff** | **125°C – 150°C Thermal Fuse** (`10A+` / `250V AC`) | Clamped directly to underside of **aluminum bed plate**, in series with bed heater AC Live. | Permanent hard failsafe if Bed SSR fails closed. |
| **6** | **Chamber Heater #1** | **100°C – 120°C Thermal Cutoff** (`10A+` / `250V AC`) | Mounted to **Chamber Heater #1 housing/heatsink**, in series with Heater #1's parallel Live leg. | Cuts power to Heater #1 independently if fan stalls or overheats. |
| **7** | **Chamber Heater #2** | **100°C – 120°C Thermal Cutoff** (`10A+` / `250V AC`) | Mounted to **Chamber Heater #2 housing/heatsink**, in series with Heater #2's parallel Live leg. | Cuts power to Heater #2 independently if fan stalls or overheats. |

*(Note: Crimp all thermal cutoffs with uninsulated butt splices and high-temp fiberglass sleeving—never solder them, as soldering heat can prematurely weaken or trip the thermal pellet.)*

---

## 4. End-to-End Wiring Diagram (Bottom Bay & Top Bay)

```text
====================================================================================================
                        1. BOTTOM BAY: MAIN AC INLET & STANDBY 5V POWER
====================================================================================================
[Wall 120V AC] ──► [15A IEC C14 Inlet + 15A Slow-Blow Fuse + Master DPST Switch]
                         │         │         │
                    (Earth GND) (AC Live) (AC Neutral)
                         │         │         │
                         │         ├─────────┼──► [Mean Well HDR-30-5 (Always-On 5V PSU)]
                         │         │         │           ├──► (+5V) ──► [3A–5A Inline Fuse] ──(5V Bus)
                         │         │         │           └──► (V-)  ──► [Common DC GND Bus]
                         │         │         │
                         │         ▼         ▼
                         │   ┌───────────────────────┐
                         │   │ Finder 22.32.0.024    │
                         │   │ (25A 2-Pole Contactor)│
                         │   │  • Terminal 1 ◄─ L In │
                         │   │  • Terminal 2 ─► L Out├──► [Switched AC Live Block] (14 AWG)
                         │   │  • Terminal 3 ◄─ N In │
                         │   │  • Terminal 4 ─► N Out├──► [Switched AC Neutral Block] (14 AWG)
                         │   └───┬───────────────┬───┘
                         │    (A1 +)          (A2 -)
                         │       ▲               │
================================═╪═══════════════╪==================================================
                        2. CONTACTOR COIL & DUAL-SWITCH INTERLOCK CIRCUIT
====================================================================================================
                         │       │               │
  [5V Bus (+5V)]         │       │               │
       │                 │       │               │
       ├──(18–20 AWG)────┼───────┼───────────────┼────────────────────────────────┐ (Up to Top Bay)
       │                 │       │               │                                ▼
       ▼                 │       │               │                       [Raspberry Pi 4B]
  [Skirt Safety          │       │               │                       (5V USB-C / GPIO)
   Interlock Switch]     │       │               │                         │            │
  (Latching Lockout)     │       │               │                    (GPIO17 Out)   (Pi GND)
       │                 │       │               │                         │            │
       ▼                 │       │               │           (700mm 22 AWG Twisted Pair down to Bottom Bay)
  ┌─────────────────┐    │       │               │                         ▼            ▼
  │ Pololu U3V50F24 │    │       │               │               ┌────────────────────────────────┐
  │ (5V -> 24V Step)│    │       │               │               │  3.3V Optocoupler MOSFET Board │
  │  • VIN  ◄── +5V │    │       │               │               │  (XY-MOS / D4184 / LR7843)     │
  │  • GND  ◄── V-  │    │       │               │               │   • SIG / IN+ ◄── Pi GPIO17    │
  │  • VOUT ──► +24V├────┼───────┘               │               │   • GND / IN- ◄── Pi GND       │
  └─────────────────┘    │  (Optional 1N4007:    │               │   • [TBD: 3.3V VCC if 6-pin]   │
                         │   Stripe on A1,       │               │   ──────────────────────────── │
                         │   Anode on A2)        │               │   • OUT- / LOAD- ◄── Finder A2 │
                         │                       ├───────────────┤   • DC- / GND ────► Common V-  │
                         │                       │               └────────────────────────────────┘
                         │                       │
                         │                       └──(22 AWG wire up to TOP BAY)──┐
                         │                                                       ▼
                         │                                         [Internal Force-On Switch]
                         │                                         (Top-Bay Maintenance Override)
                         │                                                       │
                         │                                                       ▼
                         │                                         [Top-Bay 24V PSU GND (V-)]
                         │
====================================================================================================
                        3. SWITCHED AC LOADS & TOP-BAY DC DISTRIBUTION (16 AWG)
====================================================================================================
                         │
  [Switched L / N Blocks]┼──┬──► [Mean Well LRS-350-24] ──► (+24V) ──► [15A Fuse] ──► (Top Bay 24V)
                         │  │          └──► (V- tied to Common DC GND Bus)
                         │  │
                         │  ├──► [Mean Well LRS-200-36] ──► (+36V) ──► [10A Fuse] ──► (Top Bay 36V)
                         │  │          └──► (V- tied to Common DC GND Bus)
                         │  │
                         │  ├──► [450W Bed Mat] ◄── [125–150°C Thermal Fuse] ◄── [Bed SSR (1/2)]
                         │  │     (MCU Bed Pin +/-) ───────────────────────────► [Bed SSR (3/4)]
                         │  │
                         │  └──► [Chamber Heater #1 (250W)] ◄── [100–120°C Fuse #1] ┐
                         │       [Chamber Heater #2 (250W)] ◄── [100–120°C Fuse #2] ┼─ [Chamber SSR (1/2)]
                         │        (MCU Chamber Heater Pin +/-) ─────────────────────► [Chamber SSR (3/4)]
                         │
  [Earth Ground] ────────┴──► [GND Dist. Block] ──► (Doomcube Frame, Bed Plate, Heater Housings, PSU Cases)
```

---

## 5. Critical Wiring & Isolation Rules

1. **Common DC Ground (`V-`) Tie:** Tie the `V-` terminals of `HDR-30-5` (5V), `LRS-350-24` (24V), and `LRS-200-36` (36V) together in the bottom bay. This ensures clean SPI signaling to the external 5160 Plus drivers and lets the top-bay Force-On switch short `Finder A2` directly to the top-bay `24V V-` bus.
2. **Prevent USB 5V Back-Powering:** Because the Pi 4B stays on 24/7 while the 24V PSU switches off, remove the `USB-5V` jumper on your main MCU (or cut/tape the red `+5V` wire in the Pi-to-MCU USB cable) so the Pi does not back-power the mainboard's 5V rail when the contactor is open.
3. **700mm Signal Wire Hygiene:** Twist the `GPIO17` and `Pi GND` pair (`22 AWG`) running from the top bay down to the bottom-bay optocoupler input, route away from AC/36V cables, and ensure a `10 kΩ` pull-down resistor is present across the optocoupler input so the contactor stays off during Pi boot.
4. **CAN Bus Termination & Linux Queue:** Run `1,000,000` (1M) baud across all 10 CAN nodes, place `120 Ω` terminators only at the two physical ends of each CAN trunk, and set `txqueuelen 2048` in `/etc/network/interfaces.d/can0`.

---

## 6. Remaining `[TBD]` Items & Next Steps

* **`[TBD]` 3.3V Optocoupler MOSFET Board SKU & Pinout:**
  * Confirm whether the chosen board is a **4-terminal** (`SIG`/`GND` in, `OUT-`/`DC-` out) or **6-terminal** (`DC+`/`LOAD+` pass-through) module, and whether a separate `3.3V VCC` logic pin is present (if so, feed from Pi `3V3` Pin 1—never 5V).
  * Measure exact PCB hole spacing (`28 × 12 mm` vs. `38 × 18 mm`) to print the matching 35mm DIN-rail carrier.
* **`[TBD]` Final Software & Config Generation:**
  * Generate `moonraker.conf` `[power printer]` block (`gpiochip0/gpio17`, `restart_klipper_when_powered: True`, `restart_delay: 3.0`).
  * Generate `PRINT_END` auto-cooldown shutdown macro (`M109 S50` / temperature check -> `POWER_OFF_PRINTER`).
  * Generate consolidated `printer.cfg` sections for main MCU, external 5160 Plus X/Y (`run_current: 1.500`), EBB36 Gen2 + `CSE14HRA2L4100` (`run_current: 0.600`), NEMA 11 Goose Belt Purger (`run_current: 0.400`), and dual-250W `[heater_generic chamber]`.

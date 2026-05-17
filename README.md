# TP4056 LiPo Battery Charger PCB

A compact, production-ready LiPo battery charger PCB with full battery protection,
designed from scratch in KiCad. USB-C powered with CC/CV charging and DW01A protection.

![3D Render](https://github.com/user-attachments/assets/84ee7c83-d014-4019-bee2-f47ba27c09f7)

## Features

- **Charger IC:** TP4056 — Constant Current / Constant Voltage (CC/CV) charging
- **Protection IC:** DW01A — overcharge, overdischarge, and short circuit protection
- **Protection Switch:** FS8205A — dual N-MOSFET protection switch
- **Input:** USB-C with 5.1kΩ CC resistors (proper power negotiation)
- **Charge Current:** 1A (set by R_prog = 1.2kΩ)
- **Indicators:** Red LED = Charging | Green LED = Charge Complete
- **Output:** BAT+/BAT- for LiPo connection | VCC/GND for load
- **Thermal:** EPAD connected to GND with thermal vias
- **Layers:** 2-layer PCB
- **DRC:** 0 errors, 0 warnings
- **Unrouted:** 0

## PCB Screenshots

| Schematic | PCB Layout | 3D Front | 3D Back |
|---|---|---|---|
| ![](https://github.com/user-attachments/assets/1730859f-a766-468c-b40f-dc39cecf2823) | ![](https://github.com/user-attachments/assets/158a3872-dd71-48d2-bd16-29860ed1e3d8) | ![](https://github.com/user-attachments/assets/551e2a03-241c-42d3-8736-789b19d3e22f) | ![](https://github.com/user-attachments/assets/ab2afb30-2db3-40df-bbe9-90f2dde1f0b3) |

## The Key Formula

Charge current is set by a single resistor:

```
I_charge = 1200 / R_prog (Ω)

R_prog = 1.2kΩ  →  I_charge = 1A     (this design)
R_prog = 2kΩ    →  I_charge = 600mA  (smaller batteries)
R_prog = 10kΩ   →  I_charge = 130mA  (tiny batteries)
```

Change one resistor. Change the charging speed.

## Schematic Blocks

### Block 1 — USB-C Input
```
USB-C VBUS ──► 10µF filter cap ──► VCC rail
USB-C CC1  ──► 5.1kΩ ──► GND
USB-C CC2  ──► 5.1kΩ ──► GND
USB-C SHIELD/GND ──► GND
```

### Block 2 — TP4056 Charger
| Pin | Name | Connection |
|---|---|---|
| 1 | GND | GND |
| 2 | PROG | 1.2kΩ → GND (sets 1A) |
| 3 | TE | GND (enable timer) |
| 4 | CE | VCC (enable charging) |
| 5 | BAT | Battery+ / DW01A VCC |
| 6 | STDBY | 1kΩ → Green LED → VCC |
| 7 | CHRG | 1kΩ → Red LED → VCC |
| 8 | VCC | 5V from USB-C |
| 9 | EPAD | GND + thermal vias |

### Block 3 — DW01A Protection
| Pin | Connection |
|---|---|
| VCC | BAT+ (TP4056 BAT pin) |
| GND | FS8205A Source (pin 2/5) |
| OD | FS8205A Gate 1 (pin 6) |
| OC | FS8205A Gate 2 (pin 1) |
| CS | FS8205A Source (pin 2/5) |

### Block 4 — FS8205A Dual MOSFET
| Pin | Connection |
|---|---|
| 1 (Gate 2) | DW01A OC |
| 2 (Source 2) | DW01A GND + DW01A CS |
| 3 (Drain 2) | B- Output (protected negative) |
| 4 (Drain 1) | Battery B- (LiPo negative) |
| 5 (Source 1) | Same net as Pin 2 |
| 6 (Gate 1) | DW01A OD |

## How Protection Works

```
CHARGING path:
USB-C → TP4056 → BAT+ → LiPo+
LiPo- → FS8205A D1→D2 → GND
OC MOSFET controls this path

DISCHARGING path:
LiPo+ → Load+
LiPo- → FS8205A D1→D2 → Load-
OD MOSFET controls this path

PROTECTION triggers:
Overcharge (>4.2V)   → DW01A cuts OC → blocks charging
Overdischarge (<2.4V) → DW01A cuts OD → blocks discharging
Short circuit         → CS sees spike → both MOSFETs cut off instantly
```

## Critical Design Rules Followed

| Rule | Implementation |
|---|---|
| CC pins on USB-C | 5.1kΩ pull-down on CC1 + CC2 |
| EPAD thermal dissipation | GND connection + 2×2 thermal via grid |
| BAT trace width | 1mm minimum (carries full charge current) |
| DW01A GND routing | Connected to FS8205A source, NOT main GND |
| Input filter cap | 10µF on VBUS |
| Output filter cap | 10µF on BAT output |

## What I Learned Designing This

- CC pins on USB-C are not optional — float them and the charger won't draw current
- DW01A GND does NOT connect to main GND — it connects to FS8205A source pins
- EPAD must connect to GND with thermal vias — TP4056 gets warm at 1A
- BAT traces need 1mm width minimum — they carry full charge current
- The entire protection circuit (DW01A + FS8205A) is what separates a safe charger from a fire hazard

## Files

| File/Folder | Description |
|---|---|
| [GARBER_tp4056.zip](https://github.com/user-attachments/files/27904022/GARBER_tp4056.zip) | Production-ready Gerber files |
| [Doc1.docx](https://github.com/user-attachments/files/27904139/Doc1.docx) | Full KiCad schematic export |
| [tp4056.csv](https://github.com/user-attachments/files/27904082/tp4056.csv) | Bill of materials |

## Bill of Materials

| Reference | Component | Value | Package |
|---|---|---|---|
| U1 | TP4056 | — | ESOP8 |
| U2 | DW01A | — | SOT-23-6 |
| Q1 | FS8205A | — | SOT-23-6 |
| J1 | USB-C Receptacle | PowerOnly_6P | — |
| J2 | Battery Connector | JST-PH 2pin | — |
| J3 | Output Header | Conn_01x02 | — |
| R1 | Resistor | 1.2kΩ (PROG) | 0402 |
| R2,R3 | Resistor | 5.1kΩ (CC) | 0402 |
| R4 | Resistor | 1.2kΩ | 0402 |
| R5,R6 | Resistor | 1kΩ (LED) | 0402 |
| R7,R8 | Resistor | 4.7kΩ | 0402 |
| R9 | Resistor | 1kΩ | 0402 |
| C1 | Capacitor | 10µF | 0603 |
| C2 | Capacitor | 0.1µF | 0402 |
| C3 | Capacitor | 10µF | 0603 |
| D1 | LED | Red (charging) | 0402 |
| D2 | LED | Green (complete) | 0402 |

## Fabrication Specs

- **Layers:** 2
- **Min trace:** 0.25mm (signal) / 1mm (power)
- **Via drill:** 0.8mm
- **Thermal vias:** 0.3mm drill under EPAD

## Part of #30DaysOfPCB Series

This is Day 6 of my daily PCB design challenge.
Follow along on [LinkedIn](https://www.linkedin.com/in/dharahaas-simhadri-b17b4b358?utm_source=share_via&utm_content=profile&utm_medium=member_android)
for daily PCB designs and what I learn from each one.

## Author

**Simhadri Dharahaas** — ECE Undergraduate | PCB Design & Embedded Systems
[LinkedIn](https://www.linkedin.com/in/dharahaas-simhadri-b17b4b358?utm_source=share_via&utm_content=profile&utm_medium=member_android) | [GitHub](https://github.com/dharahaas23)

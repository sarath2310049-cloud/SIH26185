HELMCOM — Zero-Profile Adaptive Communication System

Smart India Hackathon 2026 | PS 26185 | Team CRAZAC

Overview

HELMCOM is a proposed helmet-mounted conformal antenna system for tactical communications in urban Close-Quarter Battle (CQB) environments.

The system replaces a protruding whip antenna with low-profile antenna elements integrated into the helmet cover. Three spatially separated UHF antenna paths are monitored and the strongest path is selected automatically using an STM32-controlled RF switching subsystem. A separate L-band antenna is proposed for the video path.

Key Features

Three-element UHF spatial diversity — Left, Right and Rear antenna paths.

Adaptive antenna selection — STM32 compares signal-strength inputs and selects the best UHF path.

Dedicated L-band path — Separate crown-mounted antenna for the proposed video subsystem.

Conformal integration — Flexible antenna elements designed for helmet-cover integration.

EBG/AMC shielding concept — Intended to reduce back-radiation toward the operator and direct radiation outward.

Low-profile interconnect — Flexible coaxial routing to the radio/camera interface.

System Architecture

          HELMCOM HELMET
                │
       ┌────────┴────────┐
       │                 │
   UHF SPATIAL        L-BAND
    DIVERSITY           PATH
       │                 │
 ┌─────┼─────┐      Crown Antenna
 │     │     │           │
Left Right Rear          │
 │     │     │           │
 └─────┼─────┘           │
       │                 │
 Directional             │
 Couplers +              │
 AD8318 Detectors        │
       │                 │
       └──────┬──────────┘
              │
           STM32
              │
        SP3T RF Switch
              │
       Selected UHF Path
              │
        Tactical Radio

Antenna Design

UHF Meander Antenna — 433 MHz Target

The UHF element is a compact meandered radiator developed in CST Studio Suite.

Parameter

Value

Target frequency

433 MHz

Substrate

LCP

εr

2.9

tanδ

0.0025

Substrate thickness

0.5 mm

Board footprint

~76 × 76 mm

Meander rows

12

Segment length

38 mm

Row pitch

6 mm

Trace width

2 mm

Copper thickness

0.035 mm

Feed

50 Ω discrete edge port

Current UHF Simulation

The current flat CST model resonates at approximately 460 MHz, so it is not yet the final 433 MHz design.

S11 at resonance: ~−6.8 dB

Radiation efficiency at 433 MHz: ~92%

Directivity at 433 MHz: ~1.92 dBi

A head/foam loading study produced a resonance around 406 MHz, showing that loading can significantly shift the antenna resonance.

Status: UHF geometry and loading studies are still being tuned for the final conformal helmet implementation.

L-band Patch — 1.575 GHz Prototype

A separate rectangular microstrip patch was simulated in CST.

Parameter

Value

Frequency

1.575 GHz

Patch

55 × 69.1 mm

Substrate

LCP

εr

2.8

Thickness

0.535 mm

Board size

80 × 80 mm

Feed position

−13 mm

Feed width

3 mm

Flat free-space simulation results:

S11 ≈ −21 dB

Z11 ≈ 44 Ω

Directivity ≈ 9.44 dBi

Radiation efficiency ≈ −0.873 dB

Total efficiency ≈ −1.281 dB

The current CST model is a 1.575 GHz prototype. The final video-system operating frequency must be confirmed before finalizing the L-band antenna.

Adaptive UHF Switching

The UHF paths are monitored using RF power sampling:

UHF Antenna
     ↓
Directional Coupler
     ↓
AD8318 RF Detector
     ↓
Analog RSSI Voltage
     ↓
STM32 ADC
     ↓
Signal Comparison
     ↓
SP3T RF Switch
     ↓
Selected Antenna

The design uses receive-side sensing and is intended to apply hysteresis/debounce to prevent unnecessary switching. Switching is intended to be inhibited during active transmission.

Proteus Validation

The adaptive switching controller was validated at bench level using Proteus VSM and an STM32F103C8T6.

Potentiometers are used to emulate the analog outputs of the RF detectors.

Pin Mapping

Function

Component

STM32 Pin

Left RSSI

RV1

PA0 / ADC0

Right RSSI

RV2

PA1 / ADC1

Rear RSSI

RV3

PA2 / ADC2

Left path control

D1

PB0

Right path control

D2

PB1

Rear path control

D3

PB10

Test Results

Test

Strongest Path

Result

RV1 highest

Left

PASS

RV2 highest

Right

PASS

RV3 highest

Rear

PASS

The Proteus simulation demonstrates that the STM32 can compare the three analog inputs and activate only the corresponding antenna-selection output.

Note: This simulation validates the controller logic; it does not represent complete RF performance, detector characteristics, RF-switch losses, or real antenna link performance.

Repository Structure

HELMCOM/
│
├── README.md
│
├── CST/
│   ├── UHF_433MHz_Meander/
│   └── LBand_1.575GHz_Patch/
│
├── Proteus/
│   └── Adaptive_Switching/
│
├── Firmware/
│   └── STM32/
│
├── Documentation/
│   ├── UHF_CST.md
│   ├── LBAND_CST.md
│   └── ADAPTIVE_SWITCHING.md
│
└── Images/
    ├── CST/
    ├── Proteus/
    └── Architecture/

Current Status

Subsystem

Status

HELMCOM architecture

Defined

UHF CST model

Completed; further tuning required

UHF conformal/loading study

In progress

L-band flat CST model

Validated at 1.575 GHz

L-band curved model

In progress

STM32 adaptive logic

Validated in Proteus

Physical RF switch

Not yet integrated

Physical AD8318 sensing

Not yet integrated

Complete helmet prototype

Future work

End-to-end RF/link testing

Future work

Next Steps

Tune the UHF antenna for the actual radio operating frequency.

Finalize the helmet curvature and head/foam loading model.

Complete the conformal L-band simulation.

Fabricate and measure the antenna prototypes using a VNA.

Integrate real RF detectors and the SP3T switch.

Validate adaptive switching with real RF signals.

Test the complete helmet-mounted system against a conventional whip-antenna baseline.

Tools

CST Studio Suite 2026 — Electromagnetic antenna simulation

Proteus VSM — Embedded switching-system simulation

STM32F103C8T6 — Adaptive controller

Project Status

Research / Prototype Stage

HELMCOM currently combines electromagnetic antenna simulations with a bench-level embedded adaptive-switching demonstration. The next phase is integration of the simulated antenna concepts, real RF sensing, RF switching hardware and physical helmet-mounted prototypes.

Team CRAZAC
Smart India Hackathon 2026
Problem Statement 26185

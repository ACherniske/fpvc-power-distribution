# PCB Design Decisions & Rationale

> Power distribution board designed for the Fluid Powered Vehicle Challenge Senior Design project at Bucknell Unviersity. PCB interfaces with the Velocio Ace PLC to enable the powering and switching of essential bike systems.

---

## Contents

- [Copper Weight](#copper-weight)
- [Trace Width Safety Margins](#trace-width-safety-margins)
- [Ground Planes](#ground-planes)
- [Via Sizing & Parallelism](#via-sizing--parallelism)
- [2-Layer vs 4-Layer](#2-layer-vs-4-layer)

---

## Copper Weight

**Decision:** 2oz copper (70µm) throughout both layers.

2oz was chosen over the standard 1oz, providing roughly double the current carrying capacity per unit trace width. This allows narrower traces for the same current rating and is particularly important for the high current 24V input and clutch circuits, where 1oz copper would require impractically wide traces.

---
 
## Trace Width Safety Margins
 
**Decision:** ~50% margin added over IPC-2221 calculated minimums.
 
All trace widths were calculated using the IPC-2221 standard with the following assumptions:
 
| Parameter | Value |
|-----------|-------|
| Copper weight | 2oz |
| Max temperature rise | 20°C |
| Worst case ambient | 50°C |
| Max conductor temperature | 70°C |
 
The 50% margin accounts for manufacturing variation, hot spots at vias and corners, and transient currents from inductive switching.
 
| Net | IPC-2221 minimum | Routed width |
|-----|-----------------|--------------|
| Signal / gate drive | 0.1mm | 0.2mm |
| +5V rail | 0.2mm | 0.5mm |
| +12V solenoids (4A total) | 0.6mm | 1.0mm |
| +12V clutch (8A) | 1.8mm | 2.5mm |
| 24V input (15A) | 3.5mm | 4.0mm |

---
 
## Ground Planes
 
**Decision:** GND pours are used on both top and bottom layers.
 
Rather than routing individual signal returns, both copper layers include ground pours tied to the same GND net. This provides low-impedance return paths near active routing on each layer, minimises loop area, and improves current sharing for high-current returns (clutch and solenoid circuits) without requiring oversized dedicated return traces.
 
Stitching vias are used to tie the two GND pours together and prevent isolated copper islands. They are arranged in a 2.54mm grid for regular stitching in open copper regions.

---
 
## Via Sizing & Parallelism
 
**Decision:** Vias are sized by function, with multiple vias in parallel for power nets and a dedicated stitching-via setup for GND pours.
 
Rather than a single standard via size, three classes are used:
 
| Via type | Drill | Finished diameter | Parallelism |
|----------|-------|-------------------|-------------|
| Signal | 0.3mm | 0.7mm | Single |
| Power (5V, 12V) | 0.4mm | 0.8mm | 2–3 in parallel |
| High current (24V, clutch) | 0.6mm | 1.2mm | 4–6 in parallel |
| GND stitching | 0.3mm | 0.6mm | 2.54mm grid + additional vias at edges and return-layer transitions |
 
Parallel vias distribute current across multiple paths and provide redundancy against manufacturing defects in any single via.

For GND stitching, KiCad fill-area settings are 0.6mm via diameter, 0.3mm drill, 0.2mm clearance, and 2.54mm grid spacing.

---
 
## Decoupling Capacitor Placement
 
**Decision:** Two-stage decoupling per rail, with strict placement order.
 
Each power rail uses two capacitors placed in sequence along the power trace:
 
```
Connector → 100µF electrolytic → 100nF ceramic → IC power pin
```

- The **100nF ceramic** is placed as close as physically possible to the IC power pin to suppress high frequency noise. Its GND via is placed directly at the pad — not routed to a distant ground point.
- The **100µF electrolytic** is placed further back as a bulk charge reservoir for low frequency ripple.

---
 
## Fuse Strategy
 
**Decision:** SMD polyfuses for low current circuits, Mini blade fuses for high current circuits.
 
| Circuit | Fuse type | Reason |
|---------|-----------|--------|
| Solenoids (2A each) | SMD polyfuse, 2920 package | Self-resetting, appropriate for low current inductive loads |
| Clutch (7.5A) | Mini blade, PCB-mount holder | Polyfuses impractical at this current level |
| 24V input (15A) | Mini blade, PCB-mount holder | Polyfuses impractical at this current level |
 
At 7A and above, polyfuses have high hold resistance causing significant voltage drop, are difficult to source, and are expensive. Mini blade fuses are universally available at any automotive supplier, making field replacement straightforward without specialist parts.

---

## 2-Layer vs 4-Layer
 
**Decision:** 2-layer board.
 
The signal frequencies in this design are very low — PLC digital I/O, MOSFET switching for solenoids and a clutch, and DC power regulation. There are no high frequency switching converters, differential signal pairs, or RF circuits that would benefit from the additional ground plane isolation that 4-layer construction provides.
 
4-layer construction often increases PCB fabrication cost. If EMI issues are identified during testing, a 4-layer revision would be straightforward, but this is not anticipated given the low frequency nature of the design.
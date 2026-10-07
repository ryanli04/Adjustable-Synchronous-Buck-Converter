# Adjustable Synchronous Buck Converter

A personal project to learn power electronics and PCB design by building a synchronous buck converter in three phases: a fixed 5 V supply, then an adjustable output, then a display that shows live measurements.

Designed in KiCad 10 around the Analog Devices [LTC3624](https://www.analog.com/en/products/ltc3624.html), a 17 V, 2 A synchronous step-down regulator with both power MOSFETs built in.

## Current status

**Phase 1: layout complete, next step is simulation.**

| Phase | Goal | Status |
|---|---|---|
| 1 | Fixed 12 V to 5 V, 2 A converter | Schematic, placement and routing done |
| 2 | Adjustable output with a potentiometer | Not started |
| 3 | LCD showing voltage, current, power and temperature | Not started |

Phase 1 progress:

- [x] Component calculations (feedback divider, inductor, input and output capacitors)
- [x] Schematic, ERC clean
- [x] Footprints assigned and checked against datasheets
- [x] PCB placement and routing, DRC clean
- [ ] LTspice simulation
- [ ] Pre-order review
- [ ] Order boards and parts
- [ ] Assembly
- [ ] Bench testing

## Phase 1 design

### Specifications

| Parameter | Value |
|---|---|
| Input voltage | 9 to 15 V (12 V nominal) |
| Output voltage | 5.0 V |
| Output current | 2 A max (10 W) |
| Switching frequency | 1 MHz (fixed by the LTC3624) |
| Operating mode | Pulse-skipping (MODE/SYNC tied to GND) for lowest output ripple |

### Key design calculations

- **Feedback divider:** V<sub>OUT</sub> = 0.6 V × (1 + R1/R2). R1 = 73.2 kΩ (top) and R2 = 10 kΩ (bottom) give 4.99 V. The datasheet names these the other way round (R2 on top).
- **Inductor:** Sized for about 40% ripple at the worst case, maximum input voltage: L = V<sub>OUT</sub> / (f × ΔI) × (1 − V<sub>OUT</sub>/V<sub>IN(max)</sub>) = 4.17 µH, so 4.7 µH is used. Ripple is 0.71 A peak-to-peak at 15 V, so the normal peak current is about 2.35 A.
- **Inductor saturation:** The inductor must not saturate below the IC's peak current limit, 3.6 A max for the E-grade part. The Coilcraft XAL5030-472 is rated for 6.7 A.
- **Output ripple:** ΔV ≈ ΔI / (8 × f × C<sub>OUT</sub>), about 2 mV with 47 µF, before accounting for ceramic capacitance loss under DC voltage.
- **Input protection:** The LTC3624's absolute maximum input is 17 V, only 2 V above the 15 V maximum input. Hot-plugging a supply into ceramic capacitors can ring up to nearly twice the input voltage, so a 47 µF electrolytic (C1) damps the input.

### Bill of materials

| Ref | Value | Package | Part |
|---|---|---|---|
| U1 | LTC3624 buck regulator | DFN-8 3×3 mm, exposed pad | Analog Devices `LTC3624EDD#PBF` |
| L1 | 4.7 µH, Isat 6.7 A, DCR 36 mΩ | 5.3 × 5.5 mm | Coilcraft `XAL5030-472MEC` |
| C1 | 47 µF 25 V aluminum electrolytic | 6.3 × 7.7 mm SMD | |
| C2 | 10 µF 25 V X7R | 1206 | |
| C3 | 2.2 µF 10 V X7R (INTVCC, datasheet minimum 2.2 µF) | 0805 | |
| C4 | 47 µF 16 V X7R/X5R | 1210 | |
| R1 | 73.2 kΩ 1% | 0805 | |
| R2 | 10 kΩ 1% | 0805 | |
| R3 | 100 kΩ (PGOOD pull-up, optional) | 0805 | |
| J1, J2 | 2-position screw terminal, 5.08 mm | | Phoenix Contact `1715721` |

### PCB

- 2 layers, 41.8 × 18.3 mm, solid GND plane on the bottom layer
- Laid out following the LTC3624 datasheet's board layout checklist:
  - **C2 sits right against the VIN pins**, with its ground pad next to the exposed pad, so the switching loop (VIN, C2, GND) is as small as possible.
  - **L1 sits directly above the SW pin**, with C4 beside it.
  - **The feedback divider and INTVCC capacitor** are on the opposite side of U1 from SW and L1.
  - **Thermal vias** in the exposed pad connect it to the ground plane.
- Design rules: 0.15 mm minimum clearance (PCBWay standard), with 1 mm tracks for the power nets (VIN, +5V, GND, SW)
- U1 uses KiCad's built-in `DFN-8-1EP_3x3mm_P0.5mm_EP1.66x2.38mm` footprint, which matches the datasheet's recommended land pattern.
- J1 is the input (top screw +VIN, bottom screw GND). J2 is the output (top screw GND, bottom screw +5V), as marked on the silkscreen.

## Next steps

### 1. Simulation (LTspice)

Analog Devices provides an LTspice model of the LTC3624. Simulate before ordering to catch design mistakes cheaply:

- [ ] Startup at 9 V, 12 V and 15 V input: output reaches 5.0 V with no large overshoot (internal soft-start is about 1 ms)
- [ ] Steady-state output ripple and inductor ripple current at 2 A load, compared with the hand calculations
- [ ] Load step from 0.2 A to 2 A and back: check how far the output dips, and for ringing
- [ ] Input step from 9 V to 15 V
- [ ] Short-circuit or overload behavior: peak inductor current stays below L1's saturation rating
- [ ] Compare against a version with a feedforward capacitor (about 180 pF across R1) to see if it is worth adding

### 2. Pre-order review

- [ ] DRC clean with schematic parity checked
- [ ] Print the PCB at 1:1 scale and check real parts against the footprints
- [ ] Confirm all parts are in stock (Digi-Key, Mouser, LCSC, or via Octopart)
- [ ] Add `MPN` fields to all symbols for the assembly BOM
- [ ] Export Gerbers and drill files, BOM, and component placement (`.pos`) files
- [ ] Open the Gerbers in KiCad's Gerber viewer to check every layer looks right

### 3. Order and assemble

- Order bare boards and parts.
- U1 and L1 have pads underneath the part and need hot air or a hot plate with solder paste. Everything else can be soldered with an iron. Alternatively, have PCBWay assemble only U1 and L1.

### 4. Bench testing

Before applying power:

- [ ] Inspect U1 under magnification for solder bridges
- [ ] Check resistance from VIN, +5V and SW to GND with a multimeter: none should be a short

Bring-up, using a bench supply with the current limit set to about 100 mA:

- [ ] Power at 9 V with no load: INTVCC about 3.6 V, VOUT 5.0 V, PGOOD high
- [ ] Raise the current limit and step the load: 0 A, 0.5 A, 1 A, 2 A
- [ ] Repeat at 12 V and 15 V input

Measurements to record:

| Test | Target |
|---|---|
| Output voltage | 5.0 V ±2% (±1% reference plus 1% resistors) |
| Output ripple at 2 A | Within a few tens of mV. Measure at C4 with a short ground spring on the scope probe, not the long ground clip |
| Efficiency (P<sub>OUT</sub> / P<sub>IN</sub>) | Record at each input voltage and load |
| U1 and L1 temperature at 2 A | Datasheet example estimates about 50 °C junction at 25 °C ambient |
| Load step 0.2 A to 2 A | Output recovers without sustained ringing. If it rings, fit a feedforward capacitor |
| SW waveform | Clean switching at 1 MHz without excessive ringing |

## Future phases

### Phase 2: adjustable output (3 to 10 V)

Make the output adjustable by adding a potentiometer to the feedback divider. Things to resolve first:

- **Input voltage range:** A buck converter can only step down. A 10 V output needs well over 10 V at the input, so the 9 V minimum input can't reach the top of the range. Either raise the minimum input (for example, 12 V minimum) or lower the maximum output.
- **Feedback network:** A potentiometer in parallel with one resistor gives a non-linear adjustment. Using the potentiometer in series with a fixed resistor gives a more even range. Choose the arrangement so that if the potentiometer wiper loses contact, the output drops instead of rising.
- **Output capacitor voltage rating:** 10 V output needs at least 25 V-rated ceramics to keep useful capacitance under DC voltage.
- **Inductor:** Ripple is largest when V<sub>OUT</sub> is half of V<sub>IN</sub>. At 15 V in and 7.5 V out, 4.7 µH gives about 0.8 A ripple (40%), so the current inductor still works across the range.
- **Feedforward capacitor:** More important here because the divider ratio changes. Plan a footprint across the top resistor.
- Use the Phase 1 test results to decide what to change.

### Phase 3: monitoring display

Add a microcontroller and display showing output voltage, current, power and temperature:

- **Current and voltage sensing:** A current-sense IC with a shunt resistor, such as the INA219 or INA226 over I2C
- **Temperature:** NTC thermistor near U1 and L1
- **Display:** Small I2C LCD or OLED
- **Microcontroller power:** With an adjustable output, the microcontroller can't run from VOUT. It needs its own supply from VIN, for example a small LDO or second regulator.
- **PGOOD:** Connect to a microcontroller input to show a fault, with the pull-up going to the microcontroller's logic voltage

## Repository structure

```
Buck_Converter/
  Buck_ConverterV1/
    Buck_ConverterV1.kicad_pro   KiCad project (design rules, net classes)
    Buck_ConverterV1.kicad_sch   Schematic
    Buck_ConverterV1.kicad_pcb   PCB layout
    Buck_ConverterNew.kicad_sym  Project symbol library (rearranged LTC3624 symbol)
    footprints.pretty/           Vendor LTC3624 footprints (Ultra Librarian)
```

## References

- [LTC3624 datasheet](https://www.analog.com/media/en/technical-documentation/data-sheets/36242fd.pdf)
- [Coilcraft XAL50xx inductor datasheet](https://www.coilcraft.com/getmedia/49bc46c8-4b2c-45b9-9b6c-2eaa235ea698/xal50xx.pdf)
- [LTspice](https://www.analog.com/en/resources/design-tools-and-calculators/ltspice-simulator.html)

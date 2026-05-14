# Hardware Design Notes

## Power supply: LDO vs. buck converter

The TPS79733 (ultra-low-Iq 3.3 V LDO, <1 µA Iq) was chosen over a synchronous buck converter
(TPS62172, 500 mA, 2.5–7 V input). Two simulation files documenting this exploration were
removed from the repo after this note was written (recoverable from git history):

- `Simulations/slvm434a/` — PSpice transient startup simulation of TPS62172 (OrCAD binary)
- `Simulations/PowerFilter.asc` — LTSpice AC sweep of LC + RC output filter topologies
  (L swept 100 µH–2 mH, R = 27 Ω, C = 44 µF), evaluating whether buck switching noise
  could be adequately filtered for the analog signal chain

The LDO was chosen for two reasons:

1. **Analog noise sensitivity.** The two LTC2054 TIA op-amps driving the IR photodiodes are
   extremely sensitive to supply noise. Buck-converter switching ripple (and its harmonics)
   would couple directly into the transimpedance measurement. The PowerFilter simulations
   represent the investigation into whether filtering could solve this; the complexity and
   board area of a multi-stage LC filter were apparently not worth it compared to simply
   using an LDO.

2. **Low load current.** Total board current is well below 50 mA (ATtiny841 + VEML6075 +
   VEML6030 + ADS1115 + ADXL343 + 2× LTC2054). The efficiency loss of an LDO at this
   current level (≤85 mW dissipation from a 5 V supply) is negligible for a solar radiation
   sensor, which is not battery-constrained.

Even with the clean LDO supply, R3 (27 Ω) in the SmallBoard design forms an RC filter
between the digital 3V3 rail and the ADXL343's VS analog supply pin, providing an extra
layer of decoupling for the accelerometer's analog domain.

## Transimpedance amplifier for NIR photodiodes

`Simulations/PhotodiodeAmp.asc` and `Simulations/PhotodiodeAmp_Photoelectric.asc` simulate
the LTC2054 TIA circuit used for the IR photodiode channels (VEMD1060X01 and SD003-151-001).

- `PhotodiodeAmp.asc` — DC sweep of photodiode current (–1.59 µA), 1 MΩ feedback, 2 µF
  feedback capacitor, 100-run Monte Carlo for component tolerance analysis.
- `PhotodiodeAmp_Photoelectric.asc` — Full photodiode model (Cj = 10 pF, Rsh = 2 GΩ,
  Rs = 1 µΩ), transient analysis, Monte Carlo. R1 = 1 kΩ, R2 = 1 MΩ, R3 = 2 kΩ,
  C1 = 0.15 µF, C2 = 0.01 µF.

The 1 MΩ feedback resistor sets the transimpedance gain; output is a voltage proportional
to photodiode current, reported as volts (not W/m²). Irradiance calibration requires
comparison against a reference pyranometer.

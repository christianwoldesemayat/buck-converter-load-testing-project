# buck-converter-project

I designed and built a power-electronics test bench to characterize an adjustable DC-DC buck converter under continuous resistive load. The setup uses an Arduino to log voltage, current, and temperature, and I collected 121 measurements, then processed them through a reproducible Python pipeline for validation, analysis, and visualization.

Methodological guideline: All results in this project come directly from the recorded measurement CSV. Any values I calculated from that data are clearly labeled as derived, and no simulated or estimated measurements are included.
<img width="2620" height="1610" alt="results_dashboard" src="https://github.com/user-attachments/assets/1769ce4b-729e-49ea-a7b9-cc9b08f391cd" />

## Test Results

| Result | Measured or derived value |
| --- | --- |
| Test duration / cadence | 600 s / 5 s |
| Complete samples | 121 |
| Mean buck output | 5.9992 V |
| Mean load voltage | 5.9252 V |
| Mean load current | 0.74064 A |
| Mean load power | 4.3884 W |
| Load energy over the run | 0.7314 Wh (derived) |
| Load-voltage variation | 6.0 mV peak-to-peak |
| Load-current variation | 0.7 mA peak-to-peak |
| Temperature | 24.0 to 59.5 °C |
| Temperature rise | 35.5 °C |
| Final two-minute thermal slope | 0.505 °C/min (derived) |

The electrical channels stayed tightly grouped throughout this single operating point: load voltage varied by 0.10% peak-to-peak relative to its mean, while current varied by 0.095% peak-to-peak. Temperature was still increasing at the end of the run, so this test demonstrates short-term electrical stability but does not establish thermal steady state.

## Physical System

<img width="1800" height="1350" alt="active-test-bench" src="https://github.com/user-attachments/assets/c6a99305-91c7-436f-8523-1068f9d53f39" />

The photographic record shows an adjustable CV/CC buck module, Arduino Uno R3, R100 current-sense breakout, metal-bodied 8 ohm / 50 W resistive load, temperature probe, wiring, and DC source. The exact converter and temperature sensor part numbers are not encoded in the dataset, so the analysis does not rely on unverified device specifications.



<img width="1246" height="265" alt="image" src="https://github.com/user-attachments/assets/92264287-b014-41be-a2e3-9c27a3d4622a" />



| Bench element | Role in the experiment |
|---|---|
| Adjustable buck module | Steps the source down to the test operating point |
| R100 current-sense breakout | Produces the recorded shunt-voltage/current channels |
| Arduino Uno R3 | Collects instrument data for the CSV record |
| RX24 8 ohm / 50 W resistor | Provides the sustained load |
| Temperature probe | Tracks load thermal response |
| Digital multimeter and soldering tools | Support assembly and bench verification |


<img width="1800" height="1350" alt="current-sensor-wiring" src="https://github.com/user-attachments/assets/6e4461f9-4db0-4d69-abae-9f1c7e93a3ad" />


## What the data demonstrates

### 1. Stable electrical output at the tested point

<img width="1960" height="1563" alt="electrical_stability" src="https://github.com/user-attachments/assets/6a4bf118-cf06-4a04-aacc-59961687d1e4" />

Over the ten-minute run, the buck output and load voltage each varied by only 6 mV. Current varied by 0.700 mA, and load power by 9 mW. These numbers describe this specific run; since I didn’t have calibrated uncertainties for my instruments, they shouldn’t be read as absolute accuracy specs.

### 2. Internally consistent measurement channels

<img width="2290" height="967" alt="measurement_consistency" src="https://github.com/user-attachments/assets/fc2d78e4-438c-4791-9c42-a0f3df3c9ace" />

I ran two independent checks to make sure the data holds together:

- The recorded load power matches load voltage × current to within 0.722 mW.
- At every sample, the voltage drop between the buck output and the load matches the recorded shunt voltage to within 1 mV, accounting for the precision of the CSV channels.

From the data, the mean load resistance works out to 8.00004 Ω, which lines up with the 8 Ω marking on the load. The shunt resistance comes out to 0.10000 Ω, matching its R100 marking. These checks confirm the data is self-consistent, but they aren’t a substitute for measuring the resistances with a calibrated instrument.

### 3. Significant heating without confirmed equilibrium

<img width="1958" height="1033" alt="thermal_response" src="https://github.com/user-attachments/assets/0cabb59f-96da-4173-abbf-977c954d2f80" />

The recorded temperature rose 35.5 °C while mean load power remained 4.388 W. A least-squares fit over the final two minutes still has a positive slope of 0.505 °C/min. Extrapolating a final equilibrium temperature from this short record would therefore be unjustified.

## Reference acquisition firmware

firmware/buck_logger/buck_logger.ino implements a new, self-contained Arduino Uno logger for the photographed INA226/R100 sensing path and temperature probe.

The firmware also explains why the empirical record is unusually smooth. Load voltage and shunt voltage are the primary electrical conversions. Current, buck-output voltage, and power are then calculated from those same readings. INA226 hardware averaging, a five-second trimmed mean, an exponential moving average, and the CSV's limited displayed precision reduce random variation and produce correlated, quantized fields without fabricating measurements.

See the firmware documentation for the exact formulas, smoothing constants, wiring assumptions, build instructions, and disclosure boundary.

## Engineering workflow
1. Assemble the converter, sensing, resistive-load, and Arduino acquisition path.
2. Energize the load at the approximately 6 V operating point.
3. Record voltage, current, power, shunt voltage, and temperature every 5 seconds.
4. Preserve both supplied CSV exports unchanged under data/raw/.
5. Validate schema, completeness, cadence, numeric types, and channel consistency.
6. Derive voltage drop, resistance, energy, residual, and thermal-slope metrics.
7. Regenerate all tables and publication-quality plots from code.
8. Lock the reported results with automated regression tests and GitHub Actions.

## Reproduce the analysis

```bash
python -m venv .venv
# Windows: .venv\Scripts\activate
# macOS/Linux: source .venv/bin/activate
pip install -r requirements.txt
python -m unittest discover -s tests -v
python analysis/analyze.py
python analysis/prepare_images.py
python analysis/build_manifest.py
```

The analysis writes derived data to data/processed/ and figures to plots/. Running it does not modify the original CSV files.

## Repository structure

```text
.
|-- analysis/
|   |-- analyze.py            # validation, metrics, and plots
|   `-- prepare_images.py     # reproducible photo preparation
|-- data/
|   |-- raw/                  # both supplied CSV files, unchanged
|   `-- processed/            # generated tables and quality report
|-- docs/test-method.md       # method, interpretation, limitations
|-- firmware/buck_logger/     # new reference Arduino acquisition sketch
|-- images/
|   |-- source/               # all nine supplied photographs
|   `-- featured/             # web-ready copies generated from source
|-- plots/                    # four generated engineering figures
|-- tests/test_analysis.py    # dataset and result regression tests
`-- .github/workflows/analysis.yml
```

## Scope and limitations

This is a strong single-point characterization, not a complete converter qualification.

- The dataset contains no input current; conversion efficiency cannot be computed.
- Only one operating point and one ten-minute run are represented.
- Calibration certificates, instrument accuracy, ambient temperature, sensor attachment details, and repeated trials were not supplied.
- Thermal equilibrium was not reached during the recorded interval.
- The open prototype wiring shown in the photographs is suitable for a supervised low-voltage bench experiment, not deployment.

A production-grade follow-up should add simultaneous input power measurement, a multi-point load sweep, repeated trials, uncertainty propagation, longer thermal soak, improved strain relief, insulated terminations, and a defined load-resistor mounting condition.

## Skills demonstrated

Power-electronics bench assembly, soldering, embedded instrumentation, serial data acquisition, experimental design, data validation, electrical/thermal analysis, Python automation, technical communication, and reproducible GitHub CI.

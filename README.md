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

Over the run, the temperature rose by 35.5 °C while the load power held steady at a mean of 4.388 W. A least-squares fit over the final two minutes still shows a positive slope of 0.505 °C/min, so the system hadn’t reached thermal equilibrium yet. Because of that, I don’t try to extrapolate a final steady-state temperature from this short record.

## Reference acquisition firmware

I wrote an Arduino Uno logger for the INA226/R100 sensing setup and temperature probe.

The firmware measures load and shunt voltage, then calculates current, buck output voltage, and power. The data looks smooth because of built-in INA226 averaging, a five-second trimmed mean, additional filtering, and CSV rounding. Since several values come from the same readings, they naturally follow similar patterns.

The smoothing reduces noise without fabricating measurements. I've documented the formulas, filtering settings, wiring, and build instructions in the firmware README.

## Engineering workflow
1. I assembled the buck converter, resistive load, sensing circuit, and Arduino logging setup.
2. Ran the load at approximately 6 V, recording voltage, current, power, shunt voltage, and temperature every 5 seconds.
3. Kept both original CSV files unchanged in data/raw/ and checked the data for completeness, timing, and consistency.
4. Calculated voltage drop, resistance, energy, residuals, and temperature trends from the recorded measurements.
5. Generated the tables and plots directly from code, with automated tests and GitHub Actions to keep the results reproducible.

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

This project characterizes the converter at a single operating point; it isn’t a full qualification. A few limitations to keep in mind:

- I didn’t measure input current, so conversion efficiency can’t be calculated.
- The data covers one operating point and a single ten-minute run.
- I don’t have calibration certificates, instrument accuracy specs, ambient temperature, sensor mounting details, or repeated trials.
- The system hadn’t reached thermal equilibrium by the end of the recording.
- The open prototype wiring in the photos is fine for a supervised, low-voltage bench test, but it isn’t built for deployment.

If I were to take this further, the next steps would be measuring input power alongside output, sweeping across multiple load points, running repeated trials, propagating measurement uncertainty, extending the thermal soak, and improving the build with proper strain relief, insulated terminations, and a defined mounting setup for the load resistor.

## Skills demonstrated

Power-electronics bench assembly, soldering, embedded instrumentation, serial data acquisition, experimental design, data validation, electrical/thermal analysis, Python automation, technical communication, and reproducible GitHub CI.

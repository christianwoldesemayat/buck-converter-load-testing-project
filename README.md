# buck-converter-project
A physical power-electronics test bench built to characterize an adjustable DC-DC buck converter under a sustained resistive load. I assembled the bench, integrated voltage/current and temperature instrumentation with an Arduino, recorded 121 empirical samples, and built a reproducible Python pipeline to validate, analyze, and visualize the result.

Evidence standard: every result below comes from the supplied measurement CSV. Calculated values are explicitly identified as derived. No simulated or invented measurements are included.
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

## Reproduce the analysis
python -m venv .venv
### Windows: .venv\Scripts\activate
### macOS/Linux: source .venv/bin/activate
pip install -r requirements.txt
python -m unittest discover -s tests -v
python analysis/analyze.py
python analysis/prepare_images.py
python analysis/build_manifest.py

The analysis writes derived data to data/processed/ and figures to plots/. Running it does not modify the original CSV files.

## Repository structure
.
|-- analysis/
|   |-- analyze.py              # validation, metrics, and plots
|   `-- prepare_images.py       # reproducible photo preparation
|-- data/
|   |-- raw/                    # both supplied CSV files, unchanged
|   `-- processed/              # generated tables and quality report
|-- docs/test-method.md         # method, interpretation, limitations
|-- firmware/buck_logger/       # new reference Arduino acquisition sketch
|-- images/
|   |-- source/                 # all nine supplied photographs
|   `-- featured/               # web-ready copies generated from source
|-- plots/                      # four generated engineering figures
|-- tests/test_analysis.py      # dataset and result regression tests
`-- .github/workflows/analysis.yml

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

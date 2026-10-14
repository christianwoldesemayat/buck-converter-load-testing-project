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

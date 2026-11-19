# Heat Exchanger Fouling Detection Suite

[![Python 3.10+](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/downloads/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
![Charts](https://img.shields.io/badge/charts-42-informational)

**Kern-Seaton fouling · Physics-inspired prediction · Early anomaly detection · Health index · RUL · Digital twin · Cleaning economics**

A flat, single-folder predictive-maintenance project (no subfolders). The notebook holds the full Python code and regenerates every chart and CSV when run.

---

## Highlights

- **Kern-Seaton model** - asymptotic Rf(t) with hardness, temperature and velocity effects
- **Predictor** - ridge regression on physics-inspired features (numpy/pandas/matplotlib only)
- **Early detection** - jump and residual anomalies on Rf and dP, CUSUM change-points
- **Health index 0-100** - Rf, dP, efficiency and anomaly score combined
- **RUL** - days to critical fouling resistance, with uncertainty and sensitivity
- **Digital twin** - state assimilation, residual drift, fidelity R2, what-if scenarios
- **Cleaning economics** - ROI, payback, interval optimisation, fleet priority
- **Ops outputs** - alert log, work order JSON, weekly report, operator checklist, HTML dashboard

---

## Architecture & workflows (Mermaid)

### End-to-end fouling detection pipeline

```mermaid
flowchart TB
    subgraph INPUT["1. Inputs"]
        S[Plant sensors / historian]
        H[Water hardness · velocity · T_hot]
        C[Clean baseline U]
    end

    subgraph MODEL["2. Fouling model"]
        KS[Kern-Seaton Rf t]
        FE[Physics-inspired features]
        PR[Ridge predictor]
        KS --> FE --> PR
    end

    subgraph DETECT["3. Detection"]
        AN[Anomaly flags<br/>dRf & dP z-scores]
        HI[Health Index 0-100]
        RUL[RUL to Rf_CRIT]
    end

    subgraph ACTION["4. Action"]
        AL[Operator alerts]
        CL[Cleaning plan]
        EC[Interval cost optimization]
    end

    S --> KS
    H --> KS
    C --> HI
    PR --> AN
    PR --> HI
    PR --> RUL
    AN --> AL
    HI --> AL
    RUL --> CL
    CL --> EC
```

### Data flow (detailed)

```mermaid
flowchart LR
    subgraph Sensors
        Rf[Rf]
        dP[ΔP]
        U[U]
        Eff[Efficiency %]
    end

    subgraph Derived
        dRf[Rate of change dRf]
        z[z-scores]
        HI2[Health Index]
        RUL2[RUL days]
    end

    subgraph Outputs
        PNG[Charts PNG]
        CSV[Summary CSV]
        AL2[Alert status]
    end

    Rf --> dRf --> z
    dP --> z
    z --> HI2
    Rf --> HI2
    dP --> HI2
    Eff --> HI2
    Rf --> RUL2
    HI2 --> PNG
    RUL2 --> PNG
    HI2 --> CSV
    z --> AL2
```

### Fouling severity state machine

```mermaid
stateDiagram-v2
    [*] --> Clean: After chemical/mechanical clean

    Clean --> Mild: Rf rising slowly
    Mild --> Elevated: Rf > 70% of Rf_CRIT
    Elevated --> NearCritical: Rf > 90% of Rf_CRIT
    NearCritical --> Critical: Rf ≥ Rf_CRIT

    Mild --> Clean: Early intervention
    Elevated --> Clean: Planned outage window
    NearCritical --> Clean: Priority clean
    Critical --> Clean: Immediate clean

    note right of Critical
        Efficiency loss high
        ΔP often elevated
        Schedule ASAP
    end note
```

### Health Index composition

```mermaid
pie title Health Index weights
    "Rf vs critical (40%)" : 40
    "ΔP vs critical (25%)" : 25
    "Efficiency (25%)" : 25
    "Anomaly score (10%)" : 10
```

### Cleaning decision flowchart

```mermaid
flowchart TD
    Start([New daily reading]) --> Compute[Compute Rf, HI, RUL]
    Compute --> Q1{Health Index<br/>< 40?}
    Q1 -->|Yes| Urgent[URGENT: schedule clean]
    Q1 -->|No| Q2{RUL < 7 days?}
    Q2 -->|Yes| Urgent
    Q2 -->|No| Q3{RUL < 30 days?}
    Q3 -->|Yes| Plan[Plan clean in next window]
    Q3 -->|No| Q4{Anomaly flag?}
    Q4 -->|Yes| Watch[Investigate + watch]
    Q4 -->|No| OK[Continue normal operation]
    Urgent --> Cost[Check cleaning cost model]
    Plan --> Cost
    Cost --> End([Update log / dashboard])
    Watch --> End
    OK --> End
```

### Notebook module map

```mermaid
flowchart TB
    M0[0. Imports & thresholds]
    M1[1. Kern-Seaton model + curves]
    M2[2. 3D fouling surface]
    M3[3. Training data + predictor]
    M4[4. 180-day sensor series]
    M5[5. Anomaly detection]
    M6[6. Health index & RUL]
    M7[7. Cleaning economics]
    M8[8. Summary table]

    M0 --> M1 --> M2 --> M3 --> M4 --> M5 --> M6 --> M7 --> M8
```

### Digital twin loop

```mermaid
flowchart LR
    Physical[Physical HX + sensors] -->|data| Twin[Digital Twin state]
    Twin -->|alerts / RUL / plan| Ops[Maintenance]
    Ops -->|cleaning| Physical
    Twin -->|what-if / forecast| Ops
```

---

## Quick start

```bash
pip install -r requirements.txt
jupyter notebook Fouling_Detection_Suite.ipynb
```

Or just browse the PNGs and CSVs - no code needed to see the results.

---

## Chart gallery (42)

### Core model & prediction

![ML prediction](01_ml_fouling_prediction.png)
![3D fouling surface](05_3d_fouling_surface.png)
![Kern-Seaton curves](07_kern_seaton_curves.png)
![Rf forecast](09_rf_forecast.png)
![Feature importance](25_feature_importance.png)
![Partial response curves](26_partial_response_curves.png)
![Forecast backtest](31_forecast_backtest.png)

### Monitoring & detection

![Degradation tracking](02_degradation_tracking.png)
![Health index](03_health_index.png)
![Anomaly detection](04_anomaly_detection.png)
![Fouling rate](20_fouling_rate.png)
![Correlation heatmap](21_correlation_heatmap.png)
![Traffic light status](28_traffic_light_status.png)
![Change-point CUSUM](30_changepoint_cusum.png)
![Alarm priority](24_alarm_priority.png)

### Digital twin

![Twin alignment](14_digital_twin_alignment.png)
![State assimilation](15_twin_state_assimilation.png)
![Residual drift](17_residual_drift.png)
![Twin fidelity R2](35_twin_fidelity_r2.png)
![Residual diagnostics](39_residual_diagnostics.png)
![KPI dashboard](13_fouling_dashboard.png)

### Remaining life & what-if

![RUL estimate](08_rul_estimate.png)
![RUL uncertainty](12_rul_uncertainty.png)
![RUL sensitivity tornado](33_rul_sensitivity_tornado.png)
![Accelerated fouling RUL](38_accelerated_fouling_rul.png)
![What-if scenarios](10_whatif_scenarios.png)
![Time to threshold](42_time_to_threshold.png)
![Threshold sensitivity](19_threshold_sensitivity.png)

### Cleaning & economics

![Cleaning cost optimization](06_cleaning_cost_optimization.png)
![Cleaning verification](18_cleaning_event_verification.png)
![Clean recovery](23_clean_recovery.png)
![Clean ROI](40_clean_roi.png)
![Cumulative energy loss](27_cumulative_energy_loss.png)
![Duty loss waterfall](37_duty_loss_waterfall.png)
![Maintenance calendar](32_maintenance_calendar.png)

### Fleet, design & mitigation

![Fleet comparison](11_fleet_fouling_comparison.png)
![Fleet clean priority](41_fleet_clean_priority.png)
![TEMA fouling factors](34_tema_fouling_factors.png)
![Mitigation levers](36_mitigation_levers.png)
![Velocity trade-off](29_velocity_tradeoff.png)
![Deposit thickness](16_deposit_thickness.png)
![Thickness uncertainty](22_thickness_uncertainty.png)

---

## Files

| File | Description |
|------|-------------|
| `Fouling_Detection_Suite.ipynb` | Main notebook (all code) |
| `01_*.png` - `42_*.png` | Generated charts (see gallery) |
| `fouling_training_data.csv` | Synthetic Kern-Seaton training set |
| `fouling_sensor_timeseries.csv` | 180-day plant-like sensor series |
| `fouling_fleet_comparison.csv` | Multi-asset Rf and health trends |
| `fouling_detection_summary.csv` | KPI summary |
| `fouling_alert_log.csv` | Alert events |
| `digital_twin_state.csv`, `twin_snapshot.json` | Twin state log and API-style snapshot |
| `sensor_quality_flags.csv`, `sensor_quality_summary.csv` | STUCK / OUT_OF_RANGE checks |
| `tema_fouling_factors.csv`, `rf_dual_units_sample.csv` | Design Rf reference tables |
| `maintenance_calendar.csv`, `fleet_clean_priority.csv` | Cleaning plan outputs |
| `maintenance_work_order.json` | CMMS work-order stub |
| `weekly_fouling_report.txt`, `operator_checklist.md` | Operator documents |
| `fouling_dashboard.html` | Standalone status page |
| `requirements.txt`, `LICENSE` | Dependencies, MIT licence |

---

## Note on data

All data is **synthetic** and generated for demonstration. Replace the CSVs with your plant historian export before using the results for real decisions.

---

## Author

**Harpreet Singh** - GitHub: [hskhalsa1990](https://github.com/hskhalsa1990)

## License

MIT - see [LICENSE](LICENSE).

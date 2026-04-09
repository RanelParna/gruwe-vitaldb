# gruwe-vitaldb
Experiments with [GRUwE](https://openreview.net/forum?id=YLoZA77QzR)(Gated Recurrent Unit with Exponential basis functions [Joshi et al., TMLR 2026] for irregularly sampled multivariate vital sign time series data; evaluated on the [VitalDB](https://vitaldb.net): a dataset of 6,388 surgical patients composed of intraoperative biosignals and clinical information.

## DATA

VitalDB was filtered to 1,192 cases with at least one reading in all the four vital signs readings: HR, RR, SPO2, TEMP. It was resampled to a 1 Hz grid; converted to Parquet. Moreover, the monitoring window median (aka surgery duration) per each patient is 10,626 seconds = 177 minutes;
here are some added statistics (for n=1,192):

| raw name | variable | missing | median (IQR) | Δτ median | max gap (IQR) | trailing gap (IQR) |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| `Solar8000/HR` | HR, beats/min. | 52.3% | 70 (62–81) | Δτ 2s | 24 (6-52)s | 457 (168–598)s |
| `Solar8000/RR` | RR, breaths/min. | 55.9% | 14 (12-19) | Δτ 2s | 38 (24-100)s | 458 (178–598)s |
| `Solar8000/PLETH_SPO2` | SPO2, % | 51.5% | 100 (99-100) | Δτ 2s | 124 (39–242)s | 0 (0–0)s |
| `Solar8000/BT` | TEMP, °C | 61.2% | 35.8 (35.4–36.1) | Δτ 2s | 3 (2–34)s | 1,041 (771–1,344)s |

COMMENT: 
(1) First the encoder shall be validated on Task A (next-obs prediction): 2s, 10s, 30s, 60s, 120s, 300s, 600s; 
(2) later we can add in the STATIC demographic and surgery information by way of FiLM conditioning and the SEMI-STATIC ABG-panels for the full three-input architecture.

Added covariates per each patient: age, sex, weight, bmi, emop, optype, preop_* (pre-op comborbities & labs).

Added semi-static variables: of the 1,192 patients, 62% have ≥1 draw of ABG panel during the event window (read: during the surgery). The ABG panel consist of 11-measured variables arriving at the same time-stamp; the intervals b/w draws (for patients with more than 1 ABG draw) are 92.2 (72.0–113.4) minutes. Also, it is important to note that for remainder 38% of patient with no ABG draw, these patients have, on average, a much shorter and less serious cases (MNAR).

abg_na, abg_k, abg_hct, abg_gluc, abg_lac, abg_ph, abg_pco2, abg_hco3, abg_po2, abg_sao2, abg_ica

Train / Val / Test: 70 / 15 / 15 -> lower/upper bounds and z-scored.

# Structure (in-progress)

01_data_to_parquet.ipynb -> done
02_GrUwE_training.ipynb -> first training run* complete
03_GrUwE_evals.ipynb -> partially completed: analysis of the decay mode (Fast decay -- Constant decay -- No decay);  ...

*Adam optimizer, lr=0.025, exp decay 0.99/epoch, grad clip 1.0, 50 epochs, batch=8, max sequence length=2048

# COMPUTE: (a) A100 GPU Colab; (b) RunPod: 1× L40S (48 GB) - 94 GB - 16 vCPUs

Markdown
# PECVD SiO2 Deposition Process Digital Twin & AI Recipe Optimization

## Overview
This project is a physics-informed digital twin simulator and machine learning optimization pipeline for the **SiH4, N2O, and Ar-based PECVD (Plasma-Enhanced Chemical Vapor Deposition) SiO2 thin-film process**.

Moving beyond naive mathematical synthetic data, this framework incorporates **physical and hardware constraints of a real-world 30L lab/pilot chamber**, equipment noise, and unmeasured disturbances to achieve realistic predictive performance and physically valid process recipes.

---

## Core Physics & Data Engineering

### 1. 30L Chamber Process Window Bounds
To prevent pump choking, gas-phase nucleation (particle generation), and severe ion bombardment damage, strict operational hard limits were enforced:
* **Temperature:** 250 ~ 400 °C
* **Chamber Pressure:** 1.0 ~ 5.0 Torr
* **RF Power:** 50 ~ 300 W
* **Gas Flow Rates:**
  * **SiH4:** 10 ~ 100 sccm
  * **N2O:** 200 ~ 2,000 sccm
  * **Ar:** 200 ~ 2,000 sccm

### 2. Multi-Stage Non-Linear Physical Mechanisms
* **Arrhenius Kinetics & Limiting Reactant:** Models temperature-dependent activation energy and radical generation bottlenecks between SiH4 and N2O.
* **Langmuir-Hinshelwood Surface Reaction:** Captures active site saturation on the wafer surface.
* **Dual Role of Argon (Ar):** Simultaneously models plasma density enhancement (Penning ionization) and physical sputtering loss at high power/flow regimes.
* **Residence Time (τ):** Models reaction efficiency relative to actual volumetric flow rate inside a 30L chamber.

### 3. Realistic Noise Modeling (Preventing Overfitting)
Initial attempts using simple additive Gaussian noise resulted in unrealistic overfitting ($R^2 > 0.995$) as tree-based models memorized the underlying equations. To solve this:
* **Unmeasured Disturbance:** Injected a multiplicative hidden variable representing **chamber wall polymer accumulation (`chamber_wall_condition`)**, which is excluded from the model's training features (`X`).
* **Heteroscedastic Noise:** Scaled error variance dynamically based on extreme operating conditions (high temperature and high RF power).
* **Outcome:** Stabilized model performance to a realistic fab-level $R^2 \approx 0.84 \sim 0.93$, with an overall baseline defect rate of **~10–14%**.

---

## Key Results
1. **Multi-Stage Non-Linear Deposition Curve:** The trained XGBoost model accurately captures step-like saturation plateaus in deposition rate as effective plasma density ($Power \times Pressure$) increases, reflecting surface site saturation and limiting-reactant switching.
2. **Fab-Viable Optimal Recipe (Target Rate: 100.0 nm/min):**
   * Resolved optimizer (Nelder-Mead) extrapolation blind spots by introducing domain-informed boundary constraints and penalties.
   * **Optimal Recipe:** **Temp** 351.7 °C, **Pressure** 3.1 Torr, **Power** 210.8 W, **SiH4** 53.5 sccm, **N2O** 1087.3 sccm, **Ar** 1057.2 sccm (maintaining a healthy N2O/SiH4 ratio > 20).

---

## Note 
* Running the notebook generates the synthetic dataset automatically as **.csv**
* 

## Repository Structure
```text
├── PECVD_SiO2_Data_Code.ipynb              # Initial physics engine prototype & baseline data generation 
├── PECVD_SiO2_Dataset_Sample.csv           
├── PECVD_Integrated_Digital_Twin.ipynb     # [Main] Integrated pipeline: XGBoost training, EDA, & recipe optimization
└── README.md                               # Project documentation

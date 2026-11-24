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
* **Outcome:** Stabilized model performance to a realistic fab-level $R^2 \approx 0.85 \sim 0.95$, with an overall baseline defect rate of **~10–14%**.

---

## Key Results
1. **Multi-Stage Non-Linear Deposition Curve:** The trained XGBoost model accurately captures step-like saturation plateaus in deposition rate as effective plasma density ($Power \times Pressure$) increases, reflecting surface site saturation and limiting-reactant switching.
2. **Fab-Viable Optimal Recipe (Target Rate: 100.0 nm/min):**
   * **Surrogate Modeling & Inverse Optimization:** Trained `XGBoost` models via `RandomizedSearchCV` on the physics-informed dataset to predict deposition rate and defect probability, then applied `Differential Evolution` with domain-informed bounds, stoichiometry (`N2O/SiH4 >= 20`), and RF power penalties to solve for fab-viable recipes
   * **Optimal Recipe:** **Temp** 304.8 °C, **Pressure** 1.6 Torr, **Power** 176.3 W, **SiH4** 53.1 sccm, **N2O** 1178.6 sccm, **Ar** 1134.6 sccm (maintaining a healthy N2O/SiH4 ratio > 20).
3. **Advanced Process Diagnostics & Inverse Design (`03_PECVD_advanced_analytics.ipynb`):**
   - **Process Window & Forbidden Zone Map:** Visualizes predicted deposition rate contours across chamber temperature and RF power while masking high-defect-risk regimes.
   - **Chamber Aging & Drift Simulation:** Simulates cumulative wafer-to-wafer plasma density degradation to track deposition rate drift against Preventive Maintenance (PM) thresholds.
   - **Multi-Target Inverse Recipe Solver:** Uses Differential Evolution with stoichiometry (`N2O/SiH4 >= 20`) and power constraints to generate fab-viable recipes across target rates (90–115 nm/min).
---

## Note
- Running the notebooks automatically generates the synthetic dataset (`.csv`) and three serialized model/parameter files (`.pkl`).
- Generated `.csv` and `.pkl` artifacts are excluded from version control via `.gitignore` to keep the repository lightweight.

## Usage
Run the notebooks sequentially in order so that upstream datasets and trained model/parameter artifacts are generated before downstream analysis:

1. **`01_PECVD_data_generation.ipynb`**: Generates the physics-informed synthetic dataset (`.csv`).
2. **`02_PECVD_integrated_digital_twin.ipynb`**: Trains the `XGBoost` surrogate models via `RandomizedSearchCV`, runs recipe optimization via `Differential Evolution`, and saves the serialized `.pkl` artifacts.
3. **`03_PECVD_advanced_analytics.ipynb`**: Loads the saved `.pkl` files to generate the process window & forbidden zone map, chamber aging drift simulation, and multi-target inverse design table.
   
## Repository Structure
```text
├── .gitignore                                 # Excludes generated .csv datasets and .pkl model artifacts
├── 01_PECVD_data_generation.ipynb             # Initial physics engine prototype & baseline data generation
├── 02_PECVD_integrated_digital_twin.ipynb     # [Main] Integrated pipeline: XGBoost training, EDA, & recipe optimization
├── 03_PECVD_advanced_analytics.ipynb          # Process window mapping, chamber aging drift simulation, & multi-target inverse design
├── LICENSE                                    # MIT License
└── README.md                                  # Project documentation

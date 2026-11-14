# SEC 521 Network Security: Baseline Reproduction
## Baseline Paper: StormShield (ACM WiSec 2026)
**Paper Title:** *StormShield: Fingerprint-Based Detection and Mitigation of RRC Signaling Storms in O-RAN 5G RANs*  
**Authors:** Noemi Giustini, Andrea Lacava, Leonardo Bonati, Stefano Maxenti, Michele Polese, Tommaso Melodia, Francesca Cuomo  
**Student:** Shahad Alghamdi (Student ID: 202524470)  

---

### 1. Executive Summary
This repository contains the interactive, standalone baseline reproduction artifact for **Task 3** of the SEC 521 course project, pursuant to Section 2.5 and Section 6.1 of the course guidelines. 

The primary artifact is **`Baseline_Reproduction_Workbook.ipynb`**, a self-contained Jupyter Notebook that faithfully models the 5G connection establishment protocol (3GPP TS 38.331) and reproduces three published experimental findings from StormShield:
1. **Target Result 1 (Figure 6):** Unmitigated RRC context depletion latency across gNB capacities ($C \in \{16, 32, 48, 64\}$ UEs) under a $45.7\text{ Msg3/s}$ signaling storm.
2. **Target Result 2 (Table 5):** Parameter fine-tuning trade-offs across configurations E1 through E7, validating the $0\%$ Collateral Blocking Rate (CBR) of the calibrated baseline (E1) and replicating the severe collateral blocking penalty of excessive clustering radii (E3).
3. **Target Result 3 (Table 4 & Table 6 / Figure 7):** Real-world multi-location evaluation across victim positions (P1–P5) under empirical channel dispersion, reproducing the baseline's $17\%$--$25\%$ collateral blocking limitation and legitimate connection degradation.

---

### 2. Repository Structure

```text
├── Baseline_Reproduction_Workbook.ipynb  # Primary interactive reproduction workbook (pre-executed)
├── README.md                             # Documentation, execution guide, and reproduction results
├── baseline_reproduction.tex             # Formal 4-page USENIX conference report (LaTeX)
├── baseline_refs.bib                     # BibTeX reference bibliography
├── usenix.sty                            # Official USENIX two-column conference stylesheet
├── rrc_simulator.py                      # Discrete-event 3GPP TS 38.331 simulator
├── stormshield_baseline.py               # StormShield detection & mitigation xApps (Algs 1 & 2)
├── run_experiments.py                    # Automated experiment runner for Fig 6, Tab 5, Tab 4
├── generate_plots.py                     # Plot generation script for publication figures
└── figures/                              # Generated high-resolution reproduction charts
    ├── fig6_depletion_comparison.png     # Figure 6 comparison plot
    ├── table4_mobility_cbr.png           # Table 4 & Figure 7 collateral blocking plot
    └── table5_parameter_tradeoffs.png    # Table 5 parameter trade-off plot
```

---

### 3. Workbook Architecture (`Baseline_Reproduction_Workbook.ipynb`)

The workbook is structured into 6 interactive sections:

* **Part 1: 5G RRC Signaling Storm Architecture (3GPP TS 38.331)**  
  Defines the discrete-event simulation engine with finite context table capacity and the gNB context setup wait timer ($2.7\text{ s}$, measured in OpenAirInterface by Nguyen et al.; distinct from the UE-side $T_{300}$ timer). Demonstrates a single $45.7\text{ Msg3/s}$ attack exhausting a $16$-context table in $327.9\text{ ms}$ (vs. paper's $318\text{ ms}$).
* **Part 2: Target Result 1 Reproduction (Figure 6 Depletion Latency)**  
  Executes 10 randomized trials per capacity setting ($C \in \{16, 32, 48, 64\}$ UEs) under a $45.7\text{ Msg3/s}$ signaling storm, prints mean and standard deviation, and plots the side-by-side comparison bar chart.
* **Part 3: StormShield Defense Architecture (Algorithms 1 & 2)**  
  Implements Algorithm 1 (windowing $W = 100\text{ ms}$, ratio triggers $R_1, R_2$, history buffer $M = 50$, DBSCAN clustering over joint $(\text{TA}, \text{RSSI})$ space) and Algorithm 2 (aging blocklist with linear backoff timeout $\tau = \tau_{\text{base}} + c \cdot \Delta \tau$).
* **Part 4: Target Result 2 Reproduction (Table 5 Parameter Fine-Tuning)**  
  Runs 14 attack and 14 benign burst trials across configurations E1 through E7. Validates E1 ($100\%$ TP, $0\%$ CB during isolated calibration), reproduces the radius trap in E3 ($87\%$ simulated CB vs. $71\%$ published), and evaluates E2 ($1\text{ dB}$ radius fragmenting attacker clusters below $\text{MinPts}=3$).
* **Part 5: Target Result 3 Reproduction (Table 4 Scenarios & Table 6 Empirical Measurements / Figure 7)**  
  Incorporates empirical channel parameters from StormShield Table 6 (attacker at $-41.2\text{ dBm}$, victim locations P1–P5 with shadow fading $\sigma$ up to $4.4\text{ dB}$). Runs 12 trials per location, reproducing legitimate victim connection degradation ($54.2\%$ to $100\%$) and an overall mean CBR of $18.2\%$ (matching Table 4's $17\%\text{--}25\%$ range).
* **Part 6: Summary of Findings & Transition to Task 4**  
  Summarizes validated targets and articulates the motivation for Task 4: replacing static radii with an online variance-adaptive boundary $\epsilon(t) = \beta \cdot \sigma(t) + \epsilon_{\text{base}}$.

---

### 4. Reproduction Results Summary

| Target Result / Experiment | StormShield Published (WiSec '26) | Reproduced in Workbook | Status & Discrepancy Analysis |
| :--- | :---: | :---: | :--- |
| **Fig. 6: 16 UEs Depletion** | $318\text{ ms}$ | **$327.9 \pm 1.6\text{ ms}$** | Validated ($3.1\%$ error, linear scaling $R^2 > 0.99$) |
| **Fig. 6: 32 UEs Depletion** | $662\text{ ms}$ | **$678.0 \pm 1.2\text{ ms}$** | Validated ($2.4\%$ error) |
| **Fig. 6: 48 UEs Depletion** | $1164\text{ ms}$ | **$1028.4 \pm 1.4\text{ ms}$** | Linear queue scaling; OAI physical testbed has run variability |
| **Fig. 6: 64 UEs Depletion** | $1486\text{ ms}$ | **$1378.4 \pm 1.1\text{ ms}$** | Fourfold capacity adds only $\approx 1\text{ s}$ delay ($1.05\text{ s}$ sim vs. $1.17\text{ s}$ paper) |
| **Table 5: E1 (Tuned Baseline)** | $\text{TP}=100\%, \text{CB}=0\%$ | **$\text{TP}=100\%, \text{CB}=0\%$** | Exact match in isolated calibration |
| **Table 5: E3 ($\epsilon = 10\text{ dB}$)** | $\text{TP}=86\%, \text{CB}=71\%$ | **$\text{TP}=100\%, \text{CB}=87\%$** | Validated the radius trap (wide radii drop benign traffic) |
| **Table 4: Mobility Mean CBR** | $17\%\text{--}25\%$ | **$18.2\%$** | Validated collateral blocking under fading & co-location |

---

### 5. Quickstart & How to Run

#### Step 1: Install Dependencies
```bash
python3 -m pip install notebook numpy matplotlib scikit-learn
```

#### Step 2: Launch the Notebook
```bash
jupyter notebook Baseline_Reproduction_Workbook.ipynb
```
*Tip: On GitHub, clicking `Baseline_Reproduction_Workbook.ipynb` displays all code cells, markdown explanations, and charts directly in your browser without needing to run any command, as all outputs are pre-rendered.*

---

### 6. Hardware & Environment Specifications
* **Machine:** Apple MacBook Air 15-inch (Apple M3 Silicon, 8-core CPU, 10-core GPU)
* **Memory:** 16 GB Unified RAM
* **Operating System:** macOS Sequoia 15.3.2 (Darwin 24.3.0 arm64)
* **Python Runtime:** Python 3.13.3
* **Libraries:** `numpy` (2.5.4), `matplotlib` (3.11.2), `scikit-learn` (1.9.1), `scipy` (1.18.1), `notebook` (7.3.2)

---

### 7. AI Assistance Declaration
*(Submitted pursuant to Section 8.3 of the SEC 521 Course Project Syllabus)*
* **Tools and Models Used:** Google Antigravity Agentic AI Assistant powered by Gemini 2.5 Pro.
* **Scope of Assistance:** Scaffolding the discrete-event simulation structure in `Baseline_Reproduction_Workbook.ipynb`, optimizing `matplotlib` chart layouts, and formatting the two-column USENIX LaTeX report.
* **Student Verification:** All code cells, simulation parameters, and discrepancy analyses were executed, inspected, and verified directly by the author on native hardware.

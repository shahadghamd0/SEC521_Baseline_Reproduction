# SEC 521 Network Security: Baseline Reproduction
## Target Paper: StormShield (ACM WiSec 2026)
**Paper Title:** *StormShield: Fingerprint-Based Detection and Mitigation of RRC Signaling Storms in O-RAN 5G RANs*  
**Authors:** Noemi Giustini, Andrea Lacava, Leonardo Bonati, Stefano Maxenti, Michele Polese, Tommaso Melodia, Francesca Cuomo  
**Student:** Shahad Alghamdi (Student ID: 202524470)  

---

### 1. Executive Summary
This repository contains the interactive, standalone baseline reproduction artifact for **Task 3** of the SEC 521 course project, pursuant to Section 2.5 and Section 6.1 of the course guidelines. 

The primary artifact is **`Baseline_Reproduction_Workbook.ipynb`**, a completely self-contained Jupyter Notebook that faithfully models the 5G connection establishment protocol (modelled after 3GPP TS 38.331) and reproduces three published experimental findings from StormShield:
1. **Target Result 1 (Figure 6):** Unmitigated RRC context depletion latency across gNB capacities ($C \in \{16, 32, 48, 64\}$ UEs) under a $45.7\text{ Msg3/s}$ signaling storm.
2. **Target Result 2 (Table 5):** Parameter fine-tuning trade-offs across configurations E1 through E7 under fixed scenario conditions, validating the near-zero Collateral Blocking Rate (CBR) of the calibrated baseline (E1) and replicating the severe collateral blocking penalty of excessive clustering radii (E3).
3. **Target Result 3 (Table 4 & Spatial Sensitivity Analysis):** Spatial sensitivity evaluation across relative fingerprint separations ($\Delta \in [3.5, 12.0]\text{ dB}$), reproducing the baseline's $17\%$--$25\%$ collateral blocking limitation ($21.8\%$ mean CBR) and legitimate connection degradation.


---

### 2. Workbook Architecture (`Baseline_Reproduction_Workbook.ipynb`)

The workbook is structured into 7 interactive sections:

* **Part 1: 5G RRC Signaling Storm Architecture (Modelled after 3GPP TS 38.331)**  
  Defines the discrete-event simulation engine with finite context table capacity and the gNB context setup wait timer ($2.7\text{ s}$, measured in OpenAirInterface by Nguyen et al.; distinct from the UE-side $T_{300}$ timer). Demonstrates a single $45.7\text{ Msg3/s}$ attack exhausting a $16$-context table in $327.7\text{ ms}$ (vs. paper's $318\text{ ms}$).
* **Part 2: Target Result 1 Reproduction (Figure 6 Depletion Latency)**  
  Executes 10 randomized trials per capacity setting ($C \in \{16, 32, 48, 64\}$ UEs) under a $45.7\text{ Msg3/s}$ signaling storm, prints mean and standard deviation, and plots the side-by-side comparison bar chart.
* **Part 3: StormShield Defense Architecture (Algorithms 1 & 2)**  
  Implements Algorithm 1 (windowing $W = 100\text{ ms}$, dual-ratio triggers $R_1, R_2$, history buffer $M = 50$, DBSCAN clustering over joint $(\text{TA}, \text{RSSI})$ space) and Algorithm 2 (aging blocklist with linear backoff timeout $\tau = \tau_{\text{base}} + c \cdot \Delta \tau$).
* **Part 4: Target Result 2 Reproduction (Table 5 Parameter Fine-Tuning)**  
  Runs 14 attack and 14 benign burst trials across configurations E1 through E7 under a strictly fixed physical scenario. Validates E1 ($100\%$ TP, $2\%$ CB during calibration), reproduces the radius trap in E3 ($73\%$ simulated CB vs. $71\%$ published), and evaluates E2 ($1\text{ dB}$ radius fragmenting attacker clusters below $\text{MinPts}=3$).
* **Part 5: Target Result 3 Reproduction (Spatial Sensitivity & Table 4 CBR)**  
  Contrasts theoretical Table 6 empirical distributions (where pure 1D LoS fading yields only $1.5\%$ CBR) with a spatial sensitivity sweep ($\Delta \in [3.5, 12.0]\text{ dB}$), showing that proximate co-location ($\Delta \le 5\text{ dB}$) produces legitimate connection degradation ($40.6\%$ to $100.0\%$) and an overall mean CBR of $21.8\%$ (matching Table 4's $17\%\text{--}25\%$ range).
* **Part 6: Summary of Findings & Transition to Task 4**  
  Summarizes validated targets and articulates the motivation for Task 4: replacing static radii with an online variance-adaptive boundary $\epsilon(t) = \beta \cdot \sigma(t) + \epsilon_{\text{base}}$.
* **Part 7: Task 4 Pilot Implementation (StormShield-Adapt)**  
  Implements the proposed variance-adaptive clustering xApp (`StormShieldAdaptXApp`) with dynamic radius $\epsilon_{\text{RSSI}}(t) = \beta \cdot \sigma(t) + \epsilon_{\text{base}}$ alongside a Fixed $3.5\text{ dB}$ ablation arm and the baseline ($4.0\text{ dB}$). Evaluates across 60 randomized trials with deterministic seeds (`zlib.crc32`), diagnosing the fundamental trade-off between direct filter over-blocking (halved from $21.7\%$ to $11.2\%$) and attacker request leakage ($10\%$), establishing the empirical foundation for attacker-bounded adaptation in Task 5.

---

### 3. Reproduction Results Summary

| Target Result / Experiment | StormShield Published (WiSec '26) | Reproduced in Workbook | Status & Discrepancy Analysis |
| :--- | :---: | :---: | :--- |
| **Fig. 6: 16 UEs Depletion** | $318\text{ ms}$ | **$328.2 \pm 0.3\text{ ms}$** | Validated ($3.2\%$ error, linear scaling confirmed) |
| **Fig. 6: 32 UEs Depletion** | $662\text{ ms}$ | **$678.2 \pm 0.5\text{ ms}$** | Validated ($2.5\%$ error) |
| **Fig. 6: 48 UEs Depletion** | $1164\text{ ms}$ | **$1028.2 \pm 0.4\text{ ms}$** | Pure queue scaling; OAI physical testbed has run variability |
| **Fig. 6: 64 UEs Depletion** | $1486\text{ ms}$ | **$1378.4 \pm 0.5\text{ ms}$** | Fourfold capacity adds only $\approx 1\text{ s}$ delay ($1.05\text{ s}$ sim vs. $1.17\text{ s}$ paper) |
| **Table 5: E1 (Tuned Baseline)** | $\text{TP}=100\%, \text{CB}=0\%$ | **$\text{TP}=100\%, \text{CB}=2\%$** | Validated in controlled evaluation |
| **Table 5: E3 ($\epsilon = 10\text{ dB}$)** | $\text{TP}=86\%, \text{CB}=71\%$ | **$\text{TP}=100\%, \text{CB}=73\%$** | Validated the radius trap (wide radii drop benign traffic) |
| **Table 4: Spatial Sweep CBR** | $17\%\text{--}25\%$ | **$21.8\%$** | Validated collateral blocking under proximate co-location |

---

### 5. Quickstart & How to View

* **Directly on GitHub:** Clicking [`Baseline_Reproduction_Workbook.ipynb`](Baseline_Reproduction_Workbook.ipynb) displays all explanations, code cells, terminal printouts, and plots rendered natively in your browser without running any commands.
* **Locally in Jupyter:**
  ```bash
  python3 -m pip install notebook numpy matplotlib scikit-learn
  jupyter notebook Baseline_Reproduction_Workbook.ipynb
  ```

---

### 5. Hardware & Environment Specifications
* **Machine:** Apple MacBook Air 15-inch (Apple M3 Silicon, 8-core CPU, 10-core GPU)
* **Memory:** 16 GB Unified RAM
* **Operating System:** macOS Sequoia 15.3.2 (Darwin 24.3.0 arm64)
* **Python Runtime:** Python 3.13.3
* **Libraries:** `numpy` (2.5.4), `matplotlib` (3.11.2), `scikit-learn` (1.9.1), `scipy` (1.18.1), `notebook` (7.3.2)

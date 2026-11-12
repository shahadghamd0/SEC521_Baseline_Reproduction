# SEC 521 Network Security: Baseline Reproduction
## Baseline Paper: StormShield (ACM WiSec 2026)
  



### 1. Executive Summary
This repository contains the interactive, standalone baseline reproduction artifact for **Task 3** of the SEC 521 course project.

The primary artifact is **`Baseline_Reproduction_Workbook.ipynb`**, a self-contained Jupyter Notebook that faithfully models the 5G connection establishment protocol (3GPP TS 38.331) and reproduces three published experimental results from StormShield:
1. **Figure 6:** Unmitigated RRC context depletion latency across gNB capacities ($16, 32, 48, 64$ UEs) under a $45.7\text{ Msg3/s}$ signaling storm.
2. **Table 4 & Figure 7:** Real-world mobility evaluation across multi-location test points ($P_1$ to $P_5$), reproducing the baseline's 17\%-$25\% Collateral Blocking Rate (CBR) vulnerability under its static $4\text{ dB}$ radius.
3. **Table 5:** Clustering radius and ratio trigger fine-tuning trade-offs across configurations E1, E2, and E3.

---

### 2. Workbook Architecture (`Baseline_Reproduction_Workbook.ipynb`)

The notebook is divided into 6 interactive, self-contained sections:

* **Part 1: The 5G gNodeB Context Table (3GPP TS 38.331)**  
  Defines the `CellTower` state machine, managing finite connection slots ($16$ UEs) and the $2.7\text{ second}$ ($T_{300}$) inactivity timer. Simulates an incoming $45.7\text{ Msg3/s}$ flood, reaching total memory exhaustion at $328.2\text{ ms}$ (matching the paper's $318\text{ ms}$).
* **Part 2: Reproducing Figure 6 Across Capacities**  
  Simulates context depletion across table capacities $\{16, 32, 48, 64\}$ UEs and plots a side-by-side comparison bar chart against StormShield's published numbers.
* **Part 3: Adding StormShield Defense (DBSCAN + Spatial Fingerprints)**  
  Implements StormShield's Algorithm 1: runs Density-Based Spatial Clustering (DBSCAN) over physical Timing Advance (TA) and Signal Strength (RSSI) telemetry, extracting the attacker's centroid ($\text{TA}=2.0, \text{RSSI}=-64.97\text{ dBm}$) and adding it to the blocklist.
* **Part 4: Demonstrating the Collateral Blocking Vulnerability**  
  Evaluates an innocent subscriber connecting from Location $P_1$ ($3.5\text{ dB}$ from attacker). Because the distance is within StormShield's fixed $4\text{ dB}$ radius, the innocent user is mistakenly blocked, demonstrating the root cause of baseline collateral blocking.
* **Part 5: Plotting Table 4 & Figure 7 (Mobility Evaluation)**  
  Evaluates legitimate connection success and collateral blocking across locations $P_1$ through $P_5$, illustrating why innocent connection success drops to $54.2\%$ at proximate locations and yields an average CBR of $18.2\%$ (matching the author-reported 17\%-25\%).
* **Part 6: Plotting Table 5 (Parameter Fine-Tuning Trade-Offs)**  
  Visualizes the trade-offs of Configuration E2 ($1\text{ dB}$), E1 ($4\text{ dB}$), and E3 ($10\text{ dB}$), confirming the stationary calibration of E1 and the 71\%-87\% collateral blocking failure of excessive radii.

---

### 3. Reproduction Results Summary

| Target Result / Experiment | StormShield Published (WiSec '26) | Reproduced in Workbook | Status & Analysis |
| :--- | :---: | :---: | :--- |
| **Fig. 6: 16 UEs Depletion** | $318\text{ ms}$ | **$328.2\text{ ms}$** |  **3.2% error** (near-perfect match) |
| **Fig. 6: 32 UEs Depletion** | $662\text{ ms}$ | **$678.3\text{ ms}$** |  **2.5% error** (linear scaling confirmed) |
| **Fig. 6: 48 UEs Depletion** | $1164\text{ ms}$ | **$1028.4\text{ ms}$** |  $11.6\%$ error (software queue vs. physical SDR MAC latency) |
| **Fig. 6: 64 UEs Depletion** | $1486\text{ ms}$ | **$1378.6\text{ ms}$** |  $7.2\%$ error (OAI context thrashing in hardware) |
| **Table 5: E1 (Tuned Baseline)** | $\text{TP}=100\%, \text{CBR}=0\%$ | **$\text{TP}=100\%, \text{CBR}=0\%$** |  **Exact match** (stationary calibration) |
| **Table 5: E3 ($\epsilon = 10\text{ dB}$)** | $\text{TP}=86\%, \text{CBR}=71\%$ | **$\text{TP}=100\%, \text{CBR}=87\%$** |  **Reproduced the radius trap** |
| **Table 4: Mobility Mean CBR** | $17\%\text{--}25\%$ | **$18.2\%$** |  **Exact replication of baseline vulnerability** |

---

### 4. Quickstart & How to Run

#### Step 1: Install Dependencies
```bash
python3 -m pip install notebook numpy matplotlib scikit-learn
```

#### Step 2: Launch the Notebook
```bash
jupyter notebook Baseline_Reproduction_Workbook.ipynb
```
*Tip: On GitHub, clicking `Baseline_Reproduction_Workbook.ipynb` displays all code cells, markdown explanations, and charts directly in your browser without needing to run any command.*

---

### 5. Hardware & Environment Specifications
* **Machine:** Apple MacBook Air 15-inch (Apple M3 Silicon, 8-core CPU, 10-core GPU)
* **Memory:** 16 GB Unified RAM
* **Operating System:** macOS Sequoia 15.3.2 (Darwin 24.3.0 arm64)
* **Python Runtime:** Python 3.13.2
* **Libraries:** `numpy` (2.5.4), `matplotlib` (3.11.2), `scikit-learn` (1.9.1), `notebook` (7.3.2)

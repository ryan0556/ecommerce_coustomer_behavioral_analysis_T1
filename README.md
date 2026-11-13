# E-Commerce Customer Behavior Analysis

A decision-grade statistical audit, exploratory analysis, predictive modeling evaluation, and financial simulation of e-commerce transaction records (`ecommerce_customer_behavior_dataset.xlsx`).

## Repository Structure

```text
├── lib/
│   └── ecommerce_customer_behavior_dataset.csv      # original dataset
├── docs/
│   └── Analisis E-Commerce Türkiye.pdf              # Pdf report of the analysis
├── src/
│   └── ecommerce_customer_behavior_analysis.ipynb   # Main executable Jupyter Notebook
├── requirements.txt                                 # Pinned Python dependencies
├── .gitignore                                       # Standard Python/Jupyter/data ignore rules
└── README.md                                        # Project documentation & execution guide
```

---

## Quickstart & Local Execution

1. **Clone the repository and set up a virtual environment:**
   ```bash
   git clone <your-repo-url>
   cd <your-repo-folder>
   python -m venv .venv
   source .venv/bin/activate        # On Windows: .venv\Scripts\activate
   pip install -r requirements.txt
   ```

2. **Add the dataset:**
   Place the unmodified `ecommerce_customer_behavior_dataset.csv` file inside the `data/` directory (or in the project root).  
   *(Optional)* You can override default paths using environment variables:
   - `ECOM_DATA_DIR` (default: `data/`)
   - `ECOM_DATA_FILE` (default: `ecommerce_customer_behavior_dataset.csv`)
   - `ECOM_OUTPUT_DIR` (default: `outputs/`)

3. **Run the notebook:**
   Open `ecommerce_customer_behavior_analysis.ipynb` in JupyterLab, VS Code, or Google Colab and execute **Run All**.  
   - The notebook enforces a SHA-256 checksum (`c94e0930eaf59562daf6da3a4dd1dc04b0632d0384a377185ea8f3e46fbfb959`) and schema verification (`5,000 rows × 18 columns`, sheet `CustBehavior`) before running.

---

## Methodology & Analytical Overview

### 1. Data Integrity & Structural Scope
- **Observational Unit:** `5,000` recorded orders across `5,000` unique `Customer_ID`s (`0` repeat customers observed). Because longitudinal purchase histories are absent, retention rates, churn, and Customer Lifetime Value (CLV) cannot be measured from this cross-section.
- **Accounting Identity:** 100% of records satisfy `Total_Amount = (Unit_Price × Quantity) - Discount_Amount` (`Gross = 5,039,808.59`, `Discount = 124,264.02`, `Net = 4,915,544.57`).
- **Leakage Controls:** Identifiers (`Order_ID`, `Customer_ID`), constructed totals (`Total_Amount`), and post-checkout/timing-unverified fields (`Customer_Rating`, `Delivery_Time_Days` for classification) are excluded from checkout-time predictive features.

### 2. Descriptive & Concentration Analysis
- **Skewness & Outliers:** Monetary values are heavily right-skewed (`Total_Amount` mean `983.11` vs. median `337.91`, skewness `4.90`). IQR screening flags `546` upper net-amount orders (`> 2,265.46`), which are retained in primary analyses.
- **Pareto & Quartile Tiers:** The top `20%` of orders account for `72.04%` of recorded value (`1,373` orders / `27.46%` reach `80%` value). `Electronics` contributes `47.38%` of total value from `12.48%` of orders.

### 3. Diagnostic Hypothesis Testing
- **Spearman Rank Correlation (BH-FDR):** Strong correlations exist only among accounting-linked components (`Unit_Price` vs. `Total_Amount`, $\rho = 0.892$). Behavioral pairs show near-zero monotonic association (`Session_Duration_Minutes` vs. `Pages_Viewed`, $\rho = 0.023, q = 0.784$; `Delivery_Time_Days` vs. `Customer_Rating`, $\rho = -0.016, q = 0.784$).
- **Group Comparisons (Welch's t-test, Bootstrap, Mann-Whitney U):** No statistically significant differences in net order value or quantity between discounted and non-discounted orders (`mean diff = -3.09`, `95% CI [-116.17, +114.31]`, $q = 0.958$), or by `Is_Returning_Customer` status ($q = 0.958$).
- **Categorical Associations (Chi-Square & Bias-Corrected Cramér's V):** All 9 planned contingency tests are non-significant after BH-FDR correction ($q \ge 0.390$, max bias-corrected $V = 0.0279$).

### 4. Predictive Modeling & Weekly Forecasting
- **Chronological Validation Split:** First `80%` of distinct dates (`3,992` orders, Jan 1 – Dec 26, 2023) for 5-fold development CV; final `20%` (`1,008` orders, Dec 27, 2023 – Mar 26, 2024) as an untouched chronological holdout.
- **Delivery Duration Regression:** OLS (`MAE = 2.737` days, $R^2 = -0.020$) fails to outperform a constant `Training Median` baseline (`MAE = 2.656` days), indicating missing fulfillment/logistics variables.
- **Returning-Flag Classification:** Regularized Logistic Regression (`C=1`) achieves holdout `ROC-AUC = 0.486` and `Average Precision = 0.6065` (matching base prevalence `0.6062`).
- **Weekly Revenue Forecasting:** Excluding 2 partial boundary weeks leaves `64` complete weeks (`W-SUN`). `Training Mean` wins rolling-origin MAE selection (`16,301.70`) and holdout evaluation (`MAE = 17,452.23`, `WAPE = 21.22%`, `MASE = 0.923`). Removing the `546` upper IQR outliers reduces holdout `WAPE` to `16.06%`.

### 5. Financial Simulation (Discount Elimination)
- A 5,000-iteration bootstrap simulation indicates that eliminating discounts under constant unit volume recovers `124,264.02` (`95% CI [112,155.35, 136,662.75]`), yielding a `+2.53%` top-line revenue lift and a `+4.21%` net operating income lift under a `60%` baseline gross margin assumption.

# AI-Era Data Challenge — Lending Club (Intern)

## Objective
You are advising a retail investor choosing among **newly listed** Lending Club loans. Build a small, reproducible pipeline that:
1) cleans and explores the data,  
2) trains a baseline model to predict default risk **at listing time**,  
3) converts probabilities into an investment choice under a budget, and  
4) **backtests** that policy on a held-out quarter.

Optimize for clarity over perfection. The scope is intentionally intern-level and time-boxed.

---

## Data
Quarterly CSVs are provided in `data/` (2016Q1–2017Q4). A data dictionary is in `docs/`.
Treat each quarter’s listings as a decision window.

**Listing-time only rule:** Use only information known when a loan is first listed. Do **not** use post-origination outcomes or fields that directly/indirectly reveal them (examples: `loan_status`, `last_pymnt_d`, `last_pymnt_amnt`, `total_rec_prncp`, `total_rec_int`, `recoveries`, `collection_recovery_fee`, `out_prncp`, `next_pymnt_d`, any `*_rec_*`, any `*_pymnt*`, any `chargeoff*`, any `settlement*`).

> Tip: If unsure, ask yourself: “Could this value exist before the first payment was ever made?” If not, it’s disallowed.

---

## Tasks
1. **EDA & Cleaning**
   - Handle types, missing values, and obvious outliers.
   - Document any dropped columns and why.
   - Produce a quick data summary (rows, columns, target prevalence).

2. **Feature Set (Listing-time safe)**
   - Engineer features that could be known at listing (e.g., loan amount, interest rate, term, applicant employment info, FICO range).
   - Provide a short **feature provenance table** (a few representative features) explaining why each is valid at listing time.

3. **Baseline Model & Evaluation**
   - Train a simple classifier (e.g., logistic regression, tree/GBM) that outputs **calibrated probabilities of default (PD)**.
   - Use a **time-ordered split**: train on earlier quarters, validate on the next quarter (e.g., train: 2016Q1–2016Q3, validate: 2016Q4).  
   - **Calibrate on 2016Q4** using **isotonic** or **sigmoid (Platt)** with `cv="prefit"`, and apply that mapping to the hold-out.
   - Report on the validation set: **ROC-AUC**, **Brier score**, and a **calibration (reliability) curve**. Briefly interpret calibration (e.g., “over-confident in the 0.1–0.2 bin”).
   - **PD definition:** PD = **P(default = 1)**. Extract robustly via  
     ```python
     pd = clf.predict_proba(X)[:, list(clf.classes_).index(1)]
     ```

4. **Decision Policy & Budget**
   - With a **$50,000 budget per quarter**, select loans based on your predicted risk (e.g., **threshold on PD** or **top‑K by lowest PD**).
   - Spell out your rule (threshold or ranking). Count selected loans per quarter.

5. **Backtest**
   - Apply your selection rule to a **fixed hold‑out quarter: 2017Q1** (same for everyone).  
   - Report at least:
     - **Selected default rate vs. overall default rate** in 2017Q1.
     - **Budget utilization** = total invested / **50,000**.
     - A simple **ROI proxy**. Use a fixed recovery factor **α** that applies **only to loans that actually default** in 2017Q1; non‑defaults use full scheduled payments.
       ```text
       ROI_proxy = (collected_payments - principal) / principal
       where (toy assumption):
         if not default: collected_payments ≈ installment * term_months
         if default:     collected_payments ≈ α * installment * term_months   # default α = 0.30
       ```
     - State your assumptions clearly; simpler is fine if well‑explained.

6. **Explainability**
   - Show top features (coefficients or feature importance) and note one or two surprising relationships.

7. **(Optional, +5 pts) Tiny “AI‑era” Extension**
   - Add **one** light text-derived feature (e.g., `emp_title` length, contains digits/keywords). Show if it helped.

---

## Deliverables (via GitHub PR)
- A single **notebook** (or `.py` script) that runs end‑to‑end locally.
- `SUMMARY.md` (≤1 page) with: approach, metrics, assumptions, decision rule, what you’d try next.
- `requirements.txt` with pinned versions sufficient to run your code.
- A short **AI‑use disclosure** (if used): where you used AI assistance (e.g., boilerplate EDA) and how you validated the output.

**Optional but helpful artifacts (to ease review):**
- `artifacts/calibration.png` (your reliability plot)
- `artifacts/selected_2017Q1.csv` with `loan_id, loan_amnt, PD, selected_flag, rank[, allocation_amount]`

**Timebox:** Aim for **4–6 hours** total.

---

## Guardrails & Checks
- **Listing-time only:** Do not use post-event/banned fields (see examples above).  
- **Temporal validation:** Ensure `max(issue_d)` in train < `min(issue_d)` in validation.  
- **Calibration:** Calibrate on **2016Q4** and include a reliability plot + Brier on validation.  
- **PD extraction:** Use `predict_proba(... )[:, index_of_class_1]` so PD truly equals **P(default=1)**.  
- **Decision policy:** Make your budget rule explicit and backtest it on **2017Q1**.

> Submissions that rely on random splits, leak obvious outcome fields, or skip the decision/backtest step will be marked down.

---

## Suggested Scoring (100 pts)
- **Data hygiene & EDA (20)** – sensible cleaning, types, missingness, clear notes  
- **Leakage avoidance (15)** – listing-time feature discipline; caught obvious traps  
- **Modeling & calibration (20)** – baseline model with PDs; calibration + interpretation  
- **Decision & backtest (20)** – coherent rule, budget applied, metrics reported  
- **Reasoning & communication (15)** – clear `SUMMARY.md`; trade-offs & next steps  
- **AI-use transparency (5)** – where/why AI was used and how validated  
- **Optional extension (5)** – one small text feature with measured effect

---

## Notes
- You may downsample/upsample or use class weights; explain your choice.
- Keep the solution compact and readable. We care about your reasoning as much as your metrics.
- If you propose a different, sensible ROI proxy or selection rule, that’s fine—just document it.

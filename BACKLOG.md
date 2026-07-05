# Project Backlog & Enhancements

This backlog tracks identified issues, methodological improvements, and potential enhancements for the Market State Fed Sentiment Lead-Lag Analysis project.

---

## Prioritized Tasks

### 1. Fix FOMC Minutes Publication Date Lookahead Bias (Critical Bug)
* **Description**: The sentiment pipeline currently extracts the date from the FOMC minutes URL (e.g., `fomcminutes20211215.htm`), which corresponds to the **meeting date** (Dec 15, 2021). The sentiment score is indexed to this date and forward-filled daily.
* **Problem**: In reality, FOMC minutes are not public until **3 weeks (21 days) after the meeting**. Indexing them to the meeting date introduces severe lookahead bias, since the model "sees" the sentiment 21 days before it was publicly available.
* **Suggested Resolution**:
  * Modify `fomc_sentiment.csv` to index sentiment on the actual publication date (meeting date + 21 days).
  * Update the daily reindexing logic in `main.ipynb` to align with public release dates.

### 2. Fix CPI and Jobless Claims Reference Date Lookahead Bias (Critical Bug)
* **Description**: Weekly Initial Jobless Claims (ICSA) and monthly CPI YoY are forward-filled directly from their raw FRED index dates.
* **Problem**: FRED indexes monthly CPI to the 1st of the reference month and weekly Jobless Claims to the Saturday of the week ending. CPI is actually released mid-month of the *following* month, and ICSA is released on the following Thursday. The current implementation introduces a ~45-day lookahead bias for CPI and a ~5-day lookahead bias for Jobless Claims.
* **Suggested Resolution**:
  * Apply a 45-day lag/offset to the raw monthly CPI index and a 5-day lag to the weekly Jobless Claims index in the preprocessing code to ensure only publicly released data is utilized.

### 3. Reformulate Lead-Lag Metric to Remove Selection Bias (Methodology)
* **Description**: The current lead-lag analysis measures the distance to the "first rise" in the target theme in the 8 meetings preceding a known state transition.
* **Problem**: This suffers from selection bias. It conditions on the transition occurring (future information) and grabs the earliest rise in a 1-year window, even if that rise was tiny or temporary. It does not measure the false-alarm rate (Precision) of the sentiment signals.
* **Suggested Resolution**:
  * Define a robust, forward-looking signal trigger (e.g., a z-score of the sentiment rolling average exceeding a threshold).
  * Build a full confusion matrix (Precision, Recall, F1-score) to evaluate how reliably sentiment signals predict subsequent regime shifts.

### 4. Remove Arbitrary LLM Context Window Limitations (Enhancement)
* **Description**: In `classify_minutes`, the FOMC minutes text is truncated to a hard limit of `text[start:start+4000]` (~600–800 words) before being sent to Llama 3.1 8B.
* **Problem**: FOMC minutes are long, detailed documents. Restricting the context window risks missing critical policy discussions, dissenting views, or qualifiers that appear later in the text.
* **Suggested Resolution**:
  * Take advantage of the 128k context window of Llama 3.1 8B by sending the complete "Participants' Views" and "Policy Action" sections (or the entire document text) to the API.

### 5. Transition to Out-of-Sample HMM Regime Detection (Methodology)
* **Description**: The Gaussian HMM is currently fit on the entire historical dataset (1996 to present), and states are determined in-sample using the Viterbi algorithm.
* **Problem**: In-sample Viterbi decoding uses future data points to determine the state at any time $t$. This cannot be used in a real-time predictive setup.
* **Suggested Resolution**:
  * Implement an out-of-sample/rolling HMM setup.
  * Fit the HMM on historical data up to time $t$, and decode the current state using only historical data (filtering rather than global Viterbi smoothing).

# T2 ETF Comparison Report

T1 planned a comparison of SPY, TLT and GLD around a synthetic snapshot; T2 upgrades that plan to the T2 ETF Data Pack's source-recorded daily price history (prepared 2026-09-25) to compare each fund's disclosed annual fee with its historically observed maximum drawdown. Return analysis is outside this task.

## Sources, inputs and method

- **Input paths:** `data/t2/daily_prices.csv` (7,536 daily observations), `data/t2/fund_info.csv` (three disclosed fund records), `data/t2/data_dictionary.md` (definitions and rules).
- **Pack preparation date:** 2026-09-25 (per the dictionary; historical inputs recorded as downloaded from the Yahoo Finance chart API on 2026-09-20).
- **Fee disclosure dates (per ETF), all rechecked/accessed on 2026-09-25:**
  - SPY: fund-information as-of date 2026-09-10; fee-specific effective date not stated. Ratio labeled gross of waivers/reimbursements.
  - TLT: current prospectus; specific date not stated in the fee panel.
  - GLD: not stated in the selected field.
- **Common drawdown period:** 2016-09-01 through 2026-08-31 inclusive, 2,512 observations per ETF.
- **Daily frequency and adjustment basis:** one row per US market session (weekends and full-day closures absent; early-close sessions included), dates in `America/New_York`; drawdowns use `adjusted_close`, the provider's split- and dividend-adjusted price from the Yahoo chart responses.
- **Script and command:** `artifacts/t2/calculate_drawdown.py`, executed as `python3 artifacts/t2/calculate_drawdown.py` (Python 3.9.6, standard library only).
- **Input-check result:** all checks passed — 7,536 observations; tickers exactly SPY/TLT/GLD; 7,536 unique ticker/date pairs; 2,512 rows per ETF; dates strictly ascending per ticker; all `adjusted_close` finite and positive; common first/last dates 2016-09-01/2026-08-31; identical date sets across tickers.
- **Calculation method:** for each ticker, `high_t` = largest `adjusted_close` from the window start through day *t*; `drawdown_t_pct = (adjusted_close_t / high_t - 1) * 100`; maximum drawdown = minimum over the full window. Ties take the earliest trough and its earliest corresponding peak, with peak on or before trough; no-loss case would use the first observation for both dates at 0%. Full precision is kept internally; values are rounded only for display.

## 1. Comparison

| Ticker | Annual expense ratio (%) | Maximum drawdown (%) | Peak date | Trough date |
|---|---|---|---|---|
| SPY | 0.0945 | -33.72 | 2020-02-19 | 2020-03-23 |
| TLT | 0.15 | -48.35 | 2020-08-04 | 2023-10-19 |
| GLD | 0.4 | -26.40 | 2026-01-29 | 2026-07-16 |

Fee precision is preserved as disclosed (SPY 0.0945%, TLT 0.15%, GLD 0.4%). Maximum drawdowns are displayed to two decimal places; every peak date is on or before its trough date.

## 2. Observation

GLD shows the smallest drawdown loss in this period: its maximum drawdown of -26.40% is closest to zero, compared with -33.72% for SPY and -48.35% for TLT. These figures were calculated from `adjusted_close` in the daily price file, not read from any precomputed snapshot. Drawdowns are non-positive, so closer to zero means a smaller loss; the comparison uses the displayed two-decimal results, and there are no ties (the three rounded values are distinct). Limitation: these are daily-close drawdowns inside the fixed 2016-09-01..2026-08-31 window only — they say nothing about intraday losses, any other period, or future risk, and each fee is a current disclosure snapshot, not a ten-year average of fees.

## 3. Agent Check

This is the Agent's own self-check (the student's manual verification remains a separate step in the tutorial); it is not independent verification. I chose **GLD**, the observation winner. Using local tools, I re-read the two reported rows directly from `data/t2/daily_prices.csv` with an inline Python standard-library `csv` script (`python3 - <<'EOF' ...`), then recomputed `(trough / peak - 1) * 100`.

- **Source rows:** 2026-01-29 → `close=495.8999938964844`, `adjusted_close=495.8999938964844`; 2026-07-16 → `close=364.9599914550781`, `adjusted_close=364.9599914550781`.
- **Check command (actual):** inline Python read of the CSV rows plus a full-series re-inspection over all 2,512 GLD rows (cumulative highs rebuilt from scratch; no file created).
- **Actual output:** `(364.9599914550781 / 495.8999938964844 - 1) * 100 = -26.404518%`, displayed as **-26.40%**; matches the report's -26.40% (difference < 0.005).
- **Full-series recheck:** global minimum of the rebuilt series is -26.404518% at trough 2026-07-16 with peak 2026-01-29 (peak ≤ trough: True); matches the calculation script's result to 1e-9; the two-point check equals the full-series check.
- **Observation recheck:** comparing all three displayed values (-33.72, -48.35, -26.40) confirms GLD is the only smallest-loss ETF; no tie.
- **Result:** the check found no discrepancy, so no correction or re-run was needed for the check itself. (Separately, the calculation script's first run exposed two bugs in its check logic — a lexicographic ticker-order comparison and an inverted ascending-dates condition; the data was clean, the script's checks were fixed, and the script was re-run before calculating.)

## Completion summary

- Script created: `artifacts/t2/calculate_drawdown.py`
- Report created: `artifacts/t2/etf-comparison.md`
- Command run: `python3 artifacts/t2/calculate_drawdown.py` — all input checks passed; results: SPY -33.72%, TLT -48.35%, GLD -26.40%.
- Agent Check (GLD) passed: two-point recomputation -26.40% matches the report and the full-series recheck.
- T1 files and all supplied inputs unchanged; no Git commands that change state were run.
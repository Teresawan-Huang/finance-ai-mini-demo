# T3 Portfolio Research — Notes (frozen experiment)

Date prepared: 2026-10-29
Source input: `data/t2/daily_prices.csv` (unchanged T2 pack, 7,536 rows, 2,512 per ETF)
Brief: `tasks/t3_portfolio_research.md` (read and followed)
Context only: `artifacts/t1/project-plan.md` (not used for numbers)

## 1. Roles, ownership, inputs, outputs and dependencies (registered)

| Role | Owner | Inputs | Outputs | Depends on |
|---|---|---|---|---|
| Leader | Agent | brief, unit tests, all result files | decisions, page review, notes | all workers; approvals |
| Baseline/evaluation worker | Agent | `inputs/design.json`, `inputs/test.json`, starter tool | `baseline-design.json`, `evaluation.json` | freeze approval, candidate hash |
| Optimizer/page worker | Agent | `inputs/design.json` (optimize) / result files (page) | `candidate.json`, `docs/t3/portfolio/index.html` | baseline file; freeze approval |
| Reviewer (separate role) | Agent | all JSON results + `inputs/*` + page | `review/review.py`, `review/review-output.txt`, summary below | freeze/evaluation handoff; all results |

Data dependency (the checked prediction): **test evaluation depends on the candidate freeze approval**, because the weights must be fixed and hash-bound before the test window is opened. Confirmed: the tool enforces this with `--expected-candidate-sha256` (evaluate refuses to open test input on hash mismatch), `optimize` prints "Stop for human approval before test evaluation", and the unit tests encode the gate.

## 2. Candidate freeze (approved)

- Frozen file: `artifacts/t3/portfolio/candidate.json`
- Approved SHA-256 (bound to the freeze approval): `03a0ef1ac1481e75488e7e4acee6f6e26741410d91e70b8d3ab02eb804cf2095`
- Initial weights (kept unchanged): GLD 60% / SPY 40% / TLT 0%
- Rules (fixed, recorded inside `candidate.json`): 67 candidates (66 long-only 10%-grid triples + exact equal thirds); buy-and-hold, initial weights drift, no rebalancing; each window normalized to its own first common close; `adjusted_close` only; no fees/costs/tax; eligibility: design total return >= equal-weight baseline − 1e-12; objective: minimize nonnegative max-drawdown loss magnitude; ties: baseline first, then higher return, then ascending SPY/TLT/GLD; fallback: retain baseline unless drawdown loss strictly smaller beyond 1e-12.
- Test evaluation ran only after the freeze, with the exact approved hash; the hash was re-verified immediately before evaluation and `evaluation.json` records `candidate_sha256` equal to the approved hash.

## 3. Reviewer's Agent check summary

The Reviewer is a separate role and its check is the **Agent's own self-check**; it is not a student verification and not independent third-party verification. Full evidence: `artifacts/t3/portfolio/review/review.py` (separate stdlib reimplementation) and `artifacts/t3/portfolio/review/review-output.txt`. Command actually run:

```
python3 artifacts/t3/portfolio/review/review.py > artifacts/t3/portfolio/review/review-output.txt 2>&1
```

Result: **ALL CHECKS PASS** (final line of `review-output.txt`).

Checks performed:
1. **Input integrity** — manifest points to `data/t2/daily_prices.csv`; source SHA-256 `5d3f12cd…a711f` matches manifest; `design.json` SHA-256 `3e22e18f…` and `test.json` SHA-256 `2efe2dbf…` match manifest `files`; 2,512 observations per ETF.
2. **Handoff** — `candidate.json` SHA-256 equals the approved freeze hash; `evaluation.json` `candidate_sha256` equals the same; `design_sha256`/`test_sha256`/`source_sha256` match the input files and manifest.
3. **Four return/drawdown pairs recomputed from raw inputs** (decimal fractions, agreement to 0e-00 = bit-identical):
   - design/baseline: TR 0.124258185581, DD −0.228129712772 (−22.81%)
   - design/candidate: TR 0.339369495164, DD −0.178135549858 (−17.81%)
   - test/baseline: TR 0.371293304600, DD −0.116445298514 (−11.64%)
   - test/candidate: TR 0.633011172263, DD −0.170091855877 (−17.01%)
4. **Curve integrity** — all four curves: lengths (754/754/500/500), first/last dates, NAV[0]=100, ascending unique dates, drawdown = value / running peak − 1, drawdown nonpositive; design period 2021-09-01..2024-08-30, test period 2024-09-03..2026-08-31.
5. **Candidate selection enumeration** — independent 67-candidate recomputation: set size 67, unique, long-only 10% grid + equal thirds; eligible count 39; selection under the fixed rules reproduces **GLD 0.6 / SPY 0.4 / TLT 0.0**; `improved_design_drawdown=True`; stored `candidate.json` counters match (67/39).
6. **Page values** — `docs/t3/portfolio/index.html` shows the frozen allocation `SPY 40.00% / TLT 0.00% / GLD 60.00%`, all eight rounded display values (12.43%, −22.81%, 33.94%, −17.81%, 37.13%, −11.64%, 63.30%, −17.01%), the glossary terms Weight/Benchmark/Drawdown, and no pending placeholder.

No discrepancy found anywhere. One cosmetic typo in the check label (`design/tickesr`) is in the script print text only and does not affect the check.

## 4. Commands actually run (Leader-executed, all inside the repo)

```
python3 scripts/test_t3_portfolio.py                      # 14/14 unit tests pass
python3 scripts/t3_portfolio.py prepare --csv data/t2/daily_prices.csv --out-dir artifacts/t3/portfolio/inputs
python3 scripts/t3_portfolio.py baseline --design artifacts/t3/portfolio/inputs/design.json --out artifacts/t3/portfolio/baseline-design.json
python3 scripts/t3_portfolio.py optimize --design artifacts/t3/portfolio/inputs/design.json --out artifacts/t3/portfolio/candidate.json
shasum -a 256 artifacts/t3/portfolio/candidate.json       # 03a0ef1a… = approved freeze
python3 scripts/t3_portfolio.py evaluate --test artifacts/t3/portfolio/inputs/test.json --candidate artifacts/t3/portfolio/candidate.json --expected-candidate-sha256 03a0ef1ac1481e75488e7e4acee6f6e26741410d91e70b8d3ab02eb804cf2095 --out artifacts/t3/portfolio/evaluation.json
python3 scripts/t3_portfolio.py page --baseline artifacts/t3/portfolio/baseline-design.json --candidate artifacts/t3/portfolio/candidate.json --evaluation artifacts/t3/portfolio/evaluation.json --out docs/t3/portfolio/index.html
python3 artifacts/t3/portfolio/review/review.py > artifacts/t3/portfolio/review/review-output.txt 2>&1
```

## 5. Interpretation evidence and limitations

- Interpretation (page section "Team interpretation"): the candidate beat the baseline on total return in both windows (+33.94% vs +12.43% design; +63.30% vs +37.13% test), but the design-period drawdown advantage (**−17.81% vs −22.81%**) did not persist: in the test window the candidate's drawdown (**−17.01%**) was deeper than the baseline's (**−11.64%**). The return advantage persisted and widened; the drawdown advantage did not.
- Condition that could change the conclusion: the result depends on GLD continuing to deliver on this return–drawdown basis; a sustained gold drawdown (and/or a bond rally that helps TLT) would likely make the zero-TLT, 60%-gold candidate worse on drawdown than the equal-weight baseline, and different rules/fees/grid splits could select a different candidate.
- Limitations already disclosed on the page: retrospective holdout (both windows were available in the T2 data; procedural freeze, not a blinded market test); no fees/trading costs/slippage/tax modeled (T2's expense-ratio comparison is not part of this experiment); adjusted-close convention per provider; three assets and one split cannot establish a universally best strategy; hypothetical results do not guarantee future profits.
- Unresolved issues: (a) final acceptance was given on 2026-10-29 (see section 6); (b) browser-render check of `docs/t3/portfolio/index.html` must be done by the student/team (source-code checks passed, but visual rendering was not verified by the Agent); (c) source authenticity of the retained Yahoo history remains subject to independent live confirmation per `data/t2/data_dictionary.md`.

## 6. My three decisions (student's exact words)

1. Plan approval: "Approve plan"
2. Candidate freeze: "Approve — freeze it"
3. Acceptance: "I accept the result. The common checks pass, the page includes the glossary, and the reviewer's independent calculation matches. The test-period drawdown was larger than the benchmark, but I accept it as a valid historical finding." (given 2026-10-29)

## My three decisions (exact words, appended on request)

1. "Approve plan"
2. "Approve — freeze it"
3. "I accept the result. The common checks pass, the page includes the glossary, and the reviewer's independent calculation matches. The test-period drawdown was larger than the benchmark, but I accept it as a valid historical finding."
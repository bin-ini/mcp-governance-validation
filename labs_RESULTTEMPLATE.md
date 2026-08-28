# Lab <id> — <short title>

**Run by:** <name or handle, optional affiliation>
**Date(s) executed:** <yyyy-mm-dd>
**Pre-registered:** yes / no — *if no, say what was decided after seeing data*

---

## 1. Hypothesis under test

State it as written in [LABS.md](../LABS.md), or state your modification and why.

## 2. Falsification condition

The condition, and the thresholds, **as fixed before the run**.

## 3. Environment

| | |
|---|---|
| Products and versions | |
| Licence tier / programme | |
| Tenant type | test / dedicated / other |
| Region | |
| Relevant configuration | |

## 4. Setup

What was built. Enough detail for someone else to reproduce it.

## 5. Pre-registered observation list

The things you committed to looking at, before running. One row per observation.

| # | Observation | Predicted | Observed |
|---|---|---|---|
| 1 | | | |

## 6. Deviations from the published design

Anything changed, and why. Write "none" if none.

## 7. Results

Data first. Interpretation in the next section, kept separate.

## 8. Verdict

- [ ] Hypothesis **falsified**
- [ ] Hypothesis **survived** this test
- [ ] **Inconclusive** — say what would resolve it

One paragraph on what this does and does not establish. Be conservative: a single run in one tenant is evidence about that tenant.

## 9. Threats to validity

Confounds, sample size, blinding, anything a hostile reviewer would raise first. Raising them yourself is worth more than having them raised for you.

## 10. Raw artifacts

Queries, logs, screenshots, configuration — whatever can be shared without exposing credentials, customer data, or tenant identifiers.

---

*Both outcomes are publishable. A result that contradicts the framework is the most valuable thing this repository can receive.*

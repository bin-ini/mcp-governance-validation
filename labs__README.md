# Executed lab results

**Currently empty.** The four experiments in [LABS.md](../LABS.md) are proposed research designs, not executed ones.

This directory is where results go — including, and especially, results that contradict the framework.

---

## Contributing a result

Open a PR adding a directory named `lab-<id>-<yyyy-mm>-<short-identifier>/`, for example `lab-2a-2026-09-contoso/`. Inside it, a `RESULT.md` following [RESULT-TEMPLATE.md](./RESULT-TEMPLATE.md), plus any raw artifacts you can share.

### What makes a result usable

**Pre-registration.** State the observation list, thresholds, and falsification condition *before* running. A result assembled after seeing the data is a story, not an experiment. If you did not pre-register, say so — it is still useful, just weaker.

**Report the null.** A lab that fires no governance event and a lab that fires one are equally publishable. So is a lab that produced something nobody predicted. The framework is more useful when contradicted than when confirmed.

**Environment specifics.** Product versions, tenant configuration, licence tier, region, and date. These products change monthly; a result without a date is a result without meaning.

**Deviations from the design.** If you changed the setup, say what and why. A deviation that invalidates a comparison is fine to report — an undisclosed one is not.

### What not to submit

- Results from production tenants or against systems you do not own.
- Anything containing real customer data, credentials, or tenant identifiers.
- Lab 3 run anywhere other than a dedicated test tenant with synthetic data. It requires no exploit, which makes it trivially reproducible and correspondingly irresponsible to run against live systems.

---

## Start here

**Lab 2a — Post-Approval Tool Drift** (in [LABS.md](../LABS.md)) is deliberately the cheapest: one server, one tool addition, a pre-registered checklist. No model behaviour, no statistical power required, deterministic, N=1. An afternoon.

It also has positive information value under every outcome. If no governance event fires, that documents a gap. If one fires, that documents a control absent from the public documentation. If something unexpected happens, that is more interesting than either.

If you are looking for the cheapest way to test whether any of this holds, start there.

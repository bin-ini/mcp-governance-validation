# MCP Governance Validation

**We benchmark attacks. We benchmark models. We benchmark scanners. We almost never benchmark the governance systems enterprises are actually buying.**

Security research has spent two decades building corpora for the *failures* of systems — vulnerability databases, exploit suites, attack taxonomies, red-team harnesses. But what enterprises now purchase is not a system that fails. It is a system that **governs**: a registry, a consent gate, an enforcement plane, an audit surface.

We have far more benchmarks for the threats than for the controls meant to govern them, and the imbalance is widening as governance products ship faster than the methods to evaluate them.

This repository proposes a way to close that gap.

---

## The question

> **How would you know whether your agent governance controls actually work?**

Not whether the vendor says they work. Not whether the architecture diagram has a box labelled "governance." Whether, if you built a controlled environment and tried to defeat them, they would hold — and whether you would be able to tell that they had.

---

## What's here

| Document | What it is |
|---|---|
| **[ARTICLE.md](./ARTICLE.md)** | The argument. Why capability rather than cognition is the security boundary, why current MCP measurement is unreliable, the six-property framework, and three structural patterns worth testing for in whatever you run. ~17 min read. |
| **[LABS.md](./LABS.md)** | Four pre-registerable experiments, each with an explicit falsification condition. Plus the six-property reference table and prior-art positioning. ~12 min read. |
| **[METHODS.md](./METHODS.md)** | How the claims here were checked, what did not survive checking, and what remains unverified. Published because a piece arguing for falsifiability should show its own. |
| **[labs/](./labs/)** | Where executed results go. Empty at time of writing — see [labs/README.md](./labs/README.md) to contribute one. |

---

## The framework

Six properties. Each is a question with a testable answer and a defined failure condition. They are not orthogonal peers — they form a partial order, and the first two gate the rest.

```text
GATING       1. DISCOVERABILITY ---+    do you know it exists?
                                   |
             2. INTEGRITY ---------+    is it what you approved?
                                   |
                                   v
                    (nothing below is meaningful
                     without these two)
                                   |
             +---------------------+---------------------+
             |                                           |
INDEPENDENT  3. AUTHORIZATION                  4. OBSERVABILITY
             is the grant scoped               is the call captured
             to the actual need?               without the developer?
                                                         |
                                                         v
DERIVED                                        5. ACTION TRACEABILITY
                                               = Observability + causal
                                                 linkage + retention
                                                         |
                                                         v
COMPOSITE                                      6. CONTAINMENT
                                               authority x propagation
                                               x data-flow bounding
```

| # | Property | The question | Fails if |
|---|---|---|---|
| 1 | **Discoverability** | Can you enumerate every MCP server your agents can reach — including the ones nobody told you about? | Any reachable server is absent from the inventory |
| 2 | **Integrity** | Is the tool surface that executes the one you approved? | A tool can be added or redefined on an approved server without a governance event |
| 3 | **Authorization** | Is the grant scoped to the actual need? | The enforcement primitive is coarser than the unit of risk |
| 4 | **Observability** | Is every invocation recorded by the platform, without the developer's help? | Capture depends on developer cooperation |
| 5 | **Action Traceability** | Can you reconstruct effect → invocation → decision → instruction? | The triggering instruction is not retained |
| 6 | **Containment** | Can you stop it, how fast does the stop propagate, and how much leaves first? | Authority exists only for voluntarily registered assets |

A control architecture that scores well on 3–6 while failing 1 or 2 is not governed. It is governed *over the subset it happens to know about*, which is a different and much weaker claim.

---

## How to prove this wrong

That is the point. [LABS.md](./LABS.md) specifies four experiments in enough detail to be pre-registered and run:

| Lab | Property | What it settles |
|---|---|---|
| **1** — Shadow MCP Discovery Gap | Discoverability | Whether inventory recall degrades with endpoint footprint rather than reachability |
| **2a** — Post-Approval Tool Drift | Integrity | Whether adding a tool to an approved server generates any governance event |
| **2b** — Write Propensity Under Coarse Scope | Authorization | Whether server-level scoping produces unintended writes |
| **3** — Prompt-to-Action Kill Chain | Traceability + Containment | Whether one instruction reaches multi-system impact through fully authorized capability |
| **4** — Config Auto-Execution | *Scope boundary* | Whether the governance plane has jurisdiction over attacks with no model in the loop |

**Lab 2a is deliberately the cheapest.** One server, one tool addition, a pre-registered checklist. No model behaviour, no statistical power required, deterministic, N=1. Both outcomes are publishable: if no governance event fires, that documents a gap; if one does, that documents an undocumented control.

If you run any of them, open a PR against [labs/](./labs/) — including, especially, results that contradict the framework.

---

## Status

**This is a framework paper, not a results paper.** The labs are proposed research designs, not executed experiments. Their purpose is to make governance claims falsifiable.

The value of the framework is not that it is right. It is that it can be proven wrong.

The three structural patterns in Part 4 are **hypotheses about shapes a governance architecture can take**, not findings about any product. Two of them are the most likely to be wrong, and both are stated that way:

- **Pattern A (the two-plane problem)** — that a registration plane and a discovery plane can coexist with no join between them. If your platform joins them, that pattern does not apply to you, and Lab 1 is how you would show it.
- **Pattern C (requirement versus mechanism)** — that observability can be enforced for one population of agent and merely documented for another. Lab 2a's neighbour, and equally cheap to check.

If you test one and it does not hold in your environment, that is the most useful contribution this repository could receive.

---

## Prior art

Attack benchmarking is a crowded field and this does not add to it. MCPTox, MCPSecBench, and MCP-SafetyBench benchmark how easily agents are compromised; MCPZoo evaluates scanners as a control class; "When the Manual Lies" evaluates guardrails. All are cited and positioned in [LABS.md](./LABS.md).

What the search did not turn up is published evaluation of **deployed enterprise governance planes** — registration workflows, consent gates, enforcement gateways — as controls. That is the gap these labs sit in. If work exists that closes it, please open an issue; it would save someone the cost of running an experiment that has already been run.

---

## Citation

See [CITATION.cff](./CITATION.cff).

## Licence

Text and figures are released under [CC BY 4.0](./LICENSE) — reuse and adapt freely with attribution.

---

## Disclosure

**Bindiya Priyadarshini** · [ORCID 0009-0005-7559-3896](https://orcid.org/0009-0005-7559-3896)

This is personal work. It does not represent the position of any employer, and it is not an assessment of any employer's products.

The patterns described in [ARTICLE.md](./ARTICLE.md) are structural shapes a governance architecture can exhibit. They are offered as hypotheses to test against whatever platform you actually run — not as claims about any named commercial system. Where a control property is described positively, that describes the strength of a documented design, not validated behaviour. That distinction is the entire point of this repository.

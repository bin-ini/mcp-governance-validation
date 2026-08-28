# Four Labs for Testing MCP Governance Controls

### Companion appendix to *[Stop Describing MCP. Start Measuring It.](./ARTICLE.md)*

---

The main article argues that enterprise claims about agent governance are currently *unfalsified* rather than *validated*, and proposes six properties against which controls could be tested: Discoverability, Integrity, Authorization, Observability, Action Traceability, and Containment.

This appendix specifies the experiments. Each lab is written with an explicit falsification condition, because a lab that cannot fail teaches nothing — and the first draft of Lab 1 could not fail, which is why it is written the way it is below.

---

## Framework reference: the six properties

| # | Property | Measurable | Fails if |
|---|---|---|---|
| **1** | **Discoverability** *(gating)* | Inventory recall — discovered servers as a fraction of servers actually reachable by at least one agent, against ground truth you control. Plus time-to-discovery | Any reachable server is absent from the inventory, or discovery latency exceeds the time it takes an agent to use it |
| **2** | **Integrity** *(gating)* | Definition drift — divergence between the tool surface declared at approval and the surface presented at invocation. Plus detection latency for a post-approval tool addition | A tool can be added to or redefined on an approved server without a governance event |
| **3** | **Authorization** | Granted-capability surface vs. exercised-capability surface, per agent, over a fixed window. The ratio is the over-permission factor | The enforcement primitive is coarser than the unit of risk — most commonly, permission granted per *server* while risk is carried per *tool* |
| **4** | **Observability** | Invocation capture rate under a deliberately uninstrumented agent — one that emits nothing of its own | Capture depends on developer cooperation, or invocations bypassing the enforcement path leave no trace |
| **5** | **Action Traceability** *(⊇ 4)* | Reconstruction completeness — given an outcome, what fraction of the chain is recoverable from logs alone, scored against a pre-written ground-truth chain | The triggering instruction is not retained, or an autonomous action cannot be distinguished from a human-approved one after the fact |
| **6** | **Containment** | Time-to-effective-block, from decision to first denied invocation on the last surface to receive policy; **and** exfiltration volume achievable through the approved tool set before any control fires | Authority exists only for assets that were voluntarily registered — Discoverability's failure surfacing downstream |

Properties 1 and 2 gate the rest: an architecture scoring well on 3 through 6 while failing either is governed only over the subset it happens to know about.

**These are proposed research designs, not executed experiments.** Their purpose is to make governance claims falsifiable. The value of the framework is not that it is right; it is that it can be proven wrong — and these are offered in enough detail that someone with a test tenant and a fortnight could execute them and publish a result that contradicts me.

Of the four, **Lab 2a is deliberately designed to be executable without significant infrastructure**: one server, one tool addition, a pre-registered checklist, no model behaviour, no statistical power required. If you are looking for the cheapest way to test whether any of this holds, start there.

A note on scope and ethics before anything else: Lab 3 in particular describes a technique that requires no exploit, which makes it trivially reproducible and correspondingly irresponsible to run against live systems. Everything here assumes dedicated test tenants and synthetic data.

---

## Lab 1 — The Shadow MCP Discovery Gap

**Property tested:** Discoverability
**Claim under test:** Enterprise MCP inventory recall degrades as a function of *endpoint footprint*, not of reachability — such that a server reachable by production agents but absent from managed endpoints is invisible to every discovery plane, not merely to the endpoint sensor.

### Why this version, and not the obvious one

The first version of this hypothesis defined the hidden server as having no endpoint presence, then tested whether an endpoint sensor found it. That is not an experiment; it is a restatement of product scope, engineered to produce the intended result. The revision gives the hidden server *partial* signal on other planes, so the question becomes whether **any** plane surfaces it.

### Setup

Approximately 20 servers exposing equivalent business functions, stratified across four footprint classes (~5 each):

| Class | Registered | Endpoint presence | Other signal |
|---|---|---|---|
| **A** — positive control | Yes | Yes | — |
| **B** | No | Yes (managed workstation with endpoint sensing) | — |
| **C** — the real question | No | **None** | Referenced in a registered agent's config; reachable only through the corporate egress proxy; holds a directory app registration |
| **D** — negative control | Yes | Yes | Never invoked |

All classes reachable by the same agent identity.

### Instrumentation

Poll on a fixed interval from first invocation:

- The governance platform's tool/server registry
- Endpoint agent-discovery inventory (whatever your EDR exposes)
- Asset/exposure graph nodes and edges
- Platform tool-invocation telemetry
- **Identity provider sign-in and app-registration logs**
- **Egress proxy logs**

The last two are what turn this from an endpoint-sensor test into an architecture test. Without them you are measuring one product's documented scope, which you could have read.

### Metrics

**Primary:** inventory recall per footprint class, and time-to-discovery. With ~5 servers per class, recall is a proportion with a reportable interval rather than a fraction taking one of four values — which is why N=3 was insufficient in the first draft.
**Secondary:** for each discovered server, whether any enforcement action is available, and time-to-effective-block.

### Falsification

The hypothesis is falsified if **Class C recall and discovery latency are not materially worse than the positive-control classes** through documented mechanisms — or if Class B servers can be blocked directly from a discovery surface without manual registration.

Stating it this way matters. An earlier version failed the hypothesis if *any single* Class C server was surfaced within a working day, which does not follow: one server appearing says nothing about whether recall degrades across the class. The claim is about a *distribution* across footprint classes, so the falsification condition has to be about distributions too. Define "materially worse" before running — a recall difference of less than 20 percentage points, or a latency ratio under 2×, should count as falsifying.

**Validity conditions:** Class A must be discovered and blockable; Class D must show no invocation signal, confirming sensors were live throughout. If either control misbehaves, the run is void.

### Why it matters

Shadow IT with a shorter fuse. A rogue SaaS subscription leaks a spreadsheet. A rogue MCP server hands an autonomous process a write path into production.

---

## Lab 2a — Post-Approval Tool Drift

**Property tested:** Integrity
**Claim under test:** A tool added to an already-approved MCP server becomes available to agents holding that server's scope without generating any governance event.

This is the cheapest lab in the set and probably the most publishable, because no model behaviour is involved. It is deterministic, N=1 suffices, and it produces a clean yes-or-no about a governance plane in an afternoon.

### Setup

1. Register and approve a server exposing `get_record` only.
2. Grant an agent access.
3. Add `update_record` to the running server.
4. Change nothing else.

### Instrumentation

**Pre-register the observation list before running.** Post-hoc observation is how you convince yourself of whatever you already believed. The list:

- Does the registry tool count change?
- Is a re-approval request generated?
- Is any admin notification produced?
- Does the agent's existing token grant access to the new tool without new consent?
- Does any telemetry record the tool-set *change* as an event distinct from an invocation?

### Metrics

The pre-registered checklist, answered yes or no. Nothing else.

### Falsification

The hypothesis is falsified if any pre-registered governance event fires.

Worth being careful with the wording here, because it inverts the usual intuition: a governance event firing is a *successful and informative experimental result*, not a failed lab. The lab succeeds either way; it is the claim that survives or does not.

Note that **both outcomes are publishable.** If no event fires, you have documented that approval gates on a snapshot. If one does, you have documented a control that is not described in the public documentation, which is equally useful to everyone reading it.

### Why it matters

This is the rug pull generalized from malice to ordinary operational drift. It requires no adversary at all — just a team shipping a feature to a server that was approved last quarter.

---

## Lab 2b — Write Propensity Under Coarse Scope

**Property tested:** Authorization
**Claim under test:** When a write tool is present but unnecessary, agents execute write operations at a non-trivial rate — and the rate rises sharply when the task is ambiguous rather than strictly read-only.

### Why this is separate from 2a

The original design fused these two into one lab, with a disjunctive falsification condition: *fails if agents reliably decline write tools **or** if tool addition triggers re-consent.* That is a design error. A single re-consent event would have killed the behavioural result too, and vice versa. Two independent claims need two experiments.

### Setup

Three arms, same business function, same agent identity:

1. **Baseline** — read tool only.
2. **Unnecessary write** — read and write tools present; task strictly requires reads.
3. **Ambiguous task** — read and write tools present; task phrased so a write is a plausible interpretation.

Arm 2 alone carries a floor-effect confound: an agent declining a write it never needed proves nothing about scope granularity. Arm 3 is what makes the comparison mean something; arm 1 establishes the floor.

### Instrumentation

Every invocation, every refusal, full agent trajectory.

### Metrics

Write-operation rate per arm. **N ≥ 30 per model per arm**, reported with variance, across at least three models. LLM behaviour here is stochastic; a single run is an anecdote wearing a metric's clothes.

### Falsification

Fails if arm 3's write rate is statistically indistinguishable from arm 2's — which would suggest scope granularity is not the operative variable and something else drives the behaviour.

---

## Lab 3 — The Prompt-to-Action Kill Chain

**Properties tested:** Action Traceability, Containment
**Claim under test:** A single instruction delivered through a legitimate business channel propagates through fully authorized MCP capability into measurable multi-system business impact — without requiring an exploit, an authentication bypass, malware, or the use of any capability outside the permissions deliberately granted for the test.

The last clause is deliberately narrower than "no policy violation by any component," which an earlier version claimed. Whether a component violates policy depends on the exact policy configuration and platform semantics, and is not something a pre-registration can safely assert in advance. What *can* be pre-registered is the permission set granted for the run — so that is what the claim is bounded to.

### Setup

Agent → MCP servers fronting a CRM, a ticketing system, and a mail system. All registered and approved, all permissions within policy, all tools legitimate. The injection arrives as it did at Supabase: inside ordinary business content a human wrote and an agent read.

**Critically: pre-register the gate configuration as the platform's shipped default, unmodified, and state the impact threshold before running.** Otherwise the experimenter sets the answer by choosing where the gates go, and "zero actions before a gate" is achievable by gating tool #1. This is the single easiest way to run this lab and learn nothing.

### Instrumentation

Full capture across platform tool-invocation telemetry, any OpenTelemetry spans your agents emit, business-system audit logs, and approval-gate interactions.

### Primary metric

Actions executed and systems touched before any human approval gate, against the pre-registered threshold.

### Secondary metric — the more valuable of the two

*Audit trail completeness.* Run a **matched corpus**: roughly 10 injected runs interleaved with 10 benign runs. Give **at least three independent reviewers** logs alone, without telling them which is which. Report sensitivity *and* false-positive rate against a pre-registered accuracy threshold.

Handing one reviewer one known-injected run and asking them to find the injection measures nothing — they score 100% by guessing. Blinding without decoys is not blinding.

Reconstruction completeness is scored against a ground-truth chain written before the run, not judged post hoc.

### Falsification

Fails if shipped-default approval gates interrupt the chain before cross-system impact, **or** if blinded reviewers reliably distinguish injected from benign runs using logs alone.

### Scope note

Dedicated test tenants, synthetic data. The entire point is that the technique requires no exploit, which makes it trivially reproducible and correspondingly irresponsible to run against anything live.

---

## Lab 4 — Config Auto-Execution With No Model In The Loop

**What this tests:** the *scope boundary* of the six-property framework — not the six properties themselves
**Claim under test:** The 2026 attack class — a malicious MCP configuration file executing on repository open, with no LLM decision anywhere in the chain — falls outside the coverage of agent-governance controls, because those controls observe the agent rather than the config.

A note on what this lab can and cannot establish. It is tempting to describe this as "testing all six properties by showing none of them engage," and that overstates it. A configuration-triggered execution path can demonstrate a coverage boundary in the agent-governance plane. It does not independently test Authorization, Action Traceability, or Containment as those properties are defined — those would each need their own instrumented experiment. What this lab measures is whether the governance layer has *jurisdiction* over the attack at all, which is a prior question to how well it performs.

### Setup

Place a crafted MCP configuration (`.vscode/tasks.json`, `.amazonq/mcp.json`, or equivalent, **benign payload**) into a repository clone on a managed developer workstation running endpoint protection. Open it in an AI-enabled IDE. Observe.

### Instrumentation

Endpoint detection signals, the endpoint agent-discovery inventory, platform tool-invocation telemetry, and whatever the agent-governance plane records — which, if the claim holds, is nothing.

### Metrics

Detection (yes/no), time-to-detection, and — the discriminating question — whether the *agent-governance surface* records anything at all, as distinct from ordinary endpoint EDR.

### Falsification

Fails if the agent-governance plane, as opposed to endpoint EDR, detects or blocks it.

### Why this one matters most

Miasma and CVE-2026-12957 are the best-documented in-the-wild MCP incidents of 2026, and I could not find a published benchmark that explicitly evaluates this class — the ones I found assume a model can be manipulated. If governance controls observe agents, and the attack involves no agent decision, then the incidents that actually happened this year fall outside the governance layer's jurisdiction entirely — which is a coverage problem, not a performance problem, and the two need different fixes.

That is a testable proposition, not a settled one, which is why it is a lab rather than a conclusion.

---

## Prior art: what these labs would and would not add

Originality claims should be made against the literature, not against a vibe. Here is the honest positioning.

### Attack benchmarking is a crowded field

- **MCPTox** ([arXiv:2508.14925](https://arxiv.org/abs/2508.14925)) — 45 real servers, 1,312 malicious cases, 20 agents. Attack success rates up to 72.8%, refusal rates below 3%, and the counterintuitive finding that more capable models proved *more* susceptible.
- **MCPSecBench** ([arXiv:2508.13220](https://arxiv.org/abs/2508.13220)) — an attack taxonomy of **17 attack types across four surfaces** (user interaction, client, transport, server). Note that this full taxonomy is a different figure from the subset used in its defense evaluation, discussed below; the two are easy to conflate.
- **MCP-SafetyBench** ([arXiv:2512.15163](https://arxiv.org/abs/2512.15163)) — accepted at ICLR 2026.

If your contribution is "agents can be tricked through MCP," you are late by about a year.

### Control evaluation is thinner, but not empty

- **MCPSecBench**, in a separate part of the same paper, evaluates two defenses — MCIP and FAN — across **11 attack types and 3 platforms**, finding average mitigation of 17.9% and 28.9% respectively, and concluding that "current protection mechanisms proved largely ineffective." Both are research prototypes rather than deployed enterprise controls. **This is a factorial evaluation of controls crossed with attack classes**, so nobody should claim one has never been done.

  To be explicit about the two numbers above, since they look contradictory at a glance: **17 attack types is the benchmark's full taxonomy; 11 is the subset carried into the defense evaluation.** Cite whichever matches the claim you are making, and say which one you mean.
- **"When the Manual Lies"** ([arXiv:2605.24069](https://arxiv.org/abs/2605.24069)) names the *Firewall Fallacy*: guardrails reduced attack success by 15.3% for one model and *increased* it by 1.8% for another.
- **MCPZoo** ([arXiv:2607.11086](https://arxiv.org/abs/2607.11086)) evaluated static scanners as a control class and found them barely correlated with one another.
- **Corvus/Petrel** ([arXiv:2608.00150](https://arxiv.org/abs/2608.00150)) — dynamic black-box assessment of live internet-facing servers, 34 test modules, SARIF output. Weeks old at time of writing and easy to miss.

So the claim "nobody evaluates controls" is false, and I am not going to make it.

### What survives

Four narrower claims, each stated as a limit on what I could find, because that is what I can actually defend:

1. **I could not find published evaluations of *deployed enterprise governance planes* as controls.** MCPSecBench's factorial design is real, but its two defenses are research prototypes. Registration workflows, consent gates, and enforcement gateways — the things enterprises are actually buying — appear not to have been evaluated in the open literature. *(Labs 1, 2a)*
2. **I could not find published work measuring enterprise MCP inventory recall against controlled ground truth.** MCPZoo's contribution is ecosystem-scale enumeration, a different question from "what fraction of the servers reachable inside *this tenant* does the inventory contain" — a measurement borrowed from CASB methodology and, as far as I can tell, not yet applied here. *(Lab 1)*
3. **Rug-pull dynamics appear to be measured cross-sectionally rather than tracked over time.** MCPSecBench does test rug pull as an attack type. I found no work watching tool definitions mutate across weeks — despite it being a named top-tier attack class since April 2025, and despite the benchmarks I reviewed all being static snapshots. *(Lab 2a, extended longitudinally)*
4. **I could not find any benchmark addressing the 2026 config-auto-execution class.** *(Lab 4)* This is the one with the least company.

### An invitation

If you know of work that closes any of these four gaps, that is a more useful reply than agreement — and it would save someone the cost of running a lab that has already been run.

---

### Sources

- Wang et al. — [MCPTox (arXiv:2508.14925)](https://arxiv.org/abs/2508.14925)
- [MCPSecBench (arXiv:2508.13220)](https://arxiv.org/abs/2508.13220)
- Zong et al. — [MCP-SafetyBench (arXiv:2512.15163)](https://arxiv.org/abs/2512.15163), ICLR 2026
- Liu et al. — [When the Manual Lies (arXiv:2605.24069)](https://arxiv.org/abs/2605.24069)
- Chen et al. — [MCPZoo (arXiv:2607.11086)](https://arxiv.org/abs/2607.11086)
- Padilla — [Exposed by Design (arXiv:2608.00150)](https://arxiv.org/abs/2608.00150)
- Invariant Labs — [Tool Poisoning Attacks](https://invariantlabs.ai/blog/mcp-security-notification-tool-poisoning-attacks)
- Wiz Research — [Amazon Q Developer, CVE-2026-12957](https://www.wiz.io/blog/amazon-q-vulnerability)
- The Hacker News — [Miasma worm, June 2026](https://thehackernews.com/2026/06/miasma-worm-hits-73-microsoft-github.html)
- General Analysis — [Supabase MCP data exfiltration](https://generalanalysis.com/blog/supabase-mcp-blog)

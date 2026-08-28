# Methods

A piece arguing that security claims should be falsifiable ought to show its own working. This document records how the claims here were checked, what failed checking, and what remains uncertain.

---

## How claims were verified

Every factual claim in [ARTICLE.md](./ARTICLE.md) was checked against a primary source before publication — protocol specifications, academic papers, and named vendor security research. Quotations were verified verbatim against the original, not against summaries of it.

Statistics were checked against the paper's own abstract or results tables. Where a figure came to me through a secondary source, it was either traced back to the primary or dropped.

That process removed a fabricated statistic and eight factual errors from the draft, several of which were claims I had already written down as fact.

---

## What did not survive checking

Recorded because the failure modes generalise.

### The fabricated statistic

An early draft cited "a protection success rate of 8.6%" from an academic MCP benchmark, as evidence that deployed defences perform poorly.

**That metric and that number do not exist in the paper.** It has no "protection success rate." What it actually reports is an evaluation of two defences — MCIP at 17.9% average mitigation and FAN at 28.9% — both research prototypes rather than commercial products, with the conclusion that "current protection mechanisms proved largely ineffective."

The corrected figures are weaker evidence for my argument than the fabricated one was, which is presumably why the error was comfortable enough to survive several passes. It was caught by someone reading the paper.

### The category error worth naming

Three separate claims in the first draft attributed capabilities to systems that, on checking, had no documented connection to them.

The mechanism was the same each time: **capabilities described near each other get remembered as belonging together.** A single announcement blog covering three products, read quickly, produces a mental architecture in which those three products form a stack. The story is coherent, plausible, and unsupported — and because it sounds right, nothing prompts you to check it.

I have not reproduced those specific claims here, because correcting misattributions in public mostly propagates them. The general lesson stands on its own, and it is the reason this article names no products: **an architecture assembled from adjacent capabilities is a hypothesis, not a description.**

### Other corrections

- A normative claim about the MCP specification conflated two documents. The Security Best Practices companion is **non-normative and contains no MUSTs**; the MUST requirements live in the Authorization specification — which opens with *"Authorization is OPTIONAL for MCP implementations."* Stating this correctly strengthens the argument rather than weakening it.
- An observability model presented as a clean binary — platform-emitted versus developer-instrumented — turned out to be more nuanced, with several distinct telemetry paths converging on the same store. The correction *weakens* the original argument and is stated as such in the article.
- Several benchmark figures were reported with metric qualifiers that made them non-comparable to the figures they were being compared against. Self-reported leaderboard results are now labelled as such wherever cited.

---

## What was deliberately excluded

- **All vendor vulnerability percentages** (ranging roughly a third to four-fifths across five vendor reports). The [MCPZoo](https://arxiv.org/abs/2607.11086) result — 45.53% average scanner precision, 24.17% recall, 15.66% pairwise agreement across eight scanners — means these largely measure scanner choice. Citing them would contradict the article's own Part 2.
- **Exposed-server counts** — three sources disagree by orders of magnitude.
- **A CVE claimed to be actively exploited** on the authority of a single aggregator, not independently corroborated.
- **Analyst predictions** — predictions, not measurements.
- **A named "benchmark" that does not exist** — the name in circulation belongs to a model, not to the benchmark it is routinely attributed to.

---

## What remains unverified

### The three patterns are hypotheses

Part 4's patterns describe shapes a governance architecture *can* take. They are generalised from examining documented platform behaviour, not from executed experiments. Each is stated as something to test, and each has a lab attached:

- **Pattern A**, that a registration plane and a discovery plane can coexist without a join between them, is Lab 1's premise.
- **Pattern B**, that permission is often granted at a coarser granularity than risk is carried, is Lab 2b's premise.
- **Pattern C**, that observability can be enforced for one population of agent and merely documented for another, is Lab 2a's neighbour.

If any of these does not hold in a given environment, the corresponding lab is how you would demonstrate it. That is the intended use.

### Absence of documentation is not absence of capability

Several observations in this work rest on not finding something documented. That is weaker evidence than finding something, and it is stated as "I could not find" rather than "does not exist" throughout. Undocumented behaviour is common, and a single counterexample overturns any of them.

### Time-sensitivity

Product surfaces in this space change monthly. Anything here describing platform behaviour should be re-verified before being relied on.

---

## A note on process

The errors above were found by adversarial review, not by the author. That is the argument of this repository applied to itself: **claims survive because someone tried to break them, not because they were carefully written.**

Anything here that has not yet been attacked should be read accordingly.

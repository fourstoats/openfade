---
summary: "Where Openfade is going, what it is built on, and what we are deliberately not doing yet."
read_when:
  - You want to understand the long-term direction of the project
  - You are deciding whether to contribute or invest attention here
  - You disagree with a scope decision and want the reasoning
  - You want to know what is explicitly out of scope right now
title: "Openfade Vision"
---

# Openfade Vision

Openfade is an open AI agent platform for trading.

It runs on your machines, uses your models, and turns a trading idea into
working, validated, testable code — then keeps going into research, backtesting,
analysis, and eventually automation.

This document explains the current state and direction of the project.
**We are still early, so iteration is fast.** Anything here can change, and
changes will show up in this file.

Project overview and install docs: [`README.md`](README.md) ·
Architecture: [`docs/architecture.md`](docs/architecture.md) ·
Public roadmap: [`docs/roadmap.md`](docs/roadmap.md) ·
Contribution guide: [`CONTRIBUTING.md`](CONTRIBUTING.md)

## The problem

A trader may fully understand market structure, indicators, risk management,
and entry/exit logic — and still be unable to turn that into running code. The
barrier is not trading knowledge. It is the gap between a concept and a
specific language's syntax, type system, namespace conventions, and platform
limits.

Today that gap is filled by fragmented tools. One generates code. One provides
data. One backtests. One explains. None of them hold the whole lifecycle, and
none of them tell you when the output is wrong.

The failure mode that matters most is **plausible wrong output**. A script that
compiles and trades nothing is worse than an error, because it is trusted.

## The full lifecycle

```
IDEA → RESEARCH → STRATEGY → CODE → VALIDATION → BACKTEST → ANALYSIS → DECISION → AUTOMATION
```

We are building toward all of it. We are starting at the middle, because the
middle is where the hard, defensible, unglamorous work is.

## Why Pine and Lipi first

Pine Script and GoCharting's Lipi are the right entry point for three reasons.

1. **They are the languages traders actually use.** Not what developers find
   interesting — what people pay for charts with.
2. **They are structurally similar.** Both are bar-by-bar cloud-executed
   scripting languages with `syminfo` namespaces, history-referencing operators,
   technical-analysis namespaces, and indicator/strategy declarations. A single
   architecture with a thin dialect layer serves both. This is not a
   coincidence we are exploiting opportunistically — it means the hard problem
   is solved once.
3. **The failure modes are real and specific.** Pine is well-represented in
   model training data. Lipi is barely represented at all. A general model asked
   to convert between them will emit Pine with a Lipi-shaped wrapper and no idea
   it failed.

## The technical bet

Everything in this project rests on one claim:

> **Trading-script generation is a retrieval and validation problem, not a
> prompting problem.**

If that is true, the defensible core of Openfade is:

- **A structured symbol corpus.** Built locally from upstream documentation.
  One row per built-in: signature, parameter types, return type, overloads,
  description, remarks, examples. Not prose chunks — a queryable API surface.
- **A dialect layer.** An explicit, versioned description of how each language
  differs. Not a translation prompt.
- **A validator.** Static analysis that runs before output reaches the user,
  and refuses to emit a symbol the target language does not have.

A wrapper around a general model gets none of these. That is the moat, and it
is where engineering effort should go disproportionately.

If we ship a beautiful agent system on top of a weak corpus, we have built a
trading-themed chatbot — and GoCharting already ships one of those.

## Current focus

**Priority — the core that everything else needs:**

- Corpus extraction for Pine v6 and Lipi v1, built locally and reproducibly
- The dialect layer and its Pine↔Lipi translation table
- The static validator and the generate → validate → repair loop
- A single working path: natural language → Pine Script → Lipi Script, with
  validation at each step

**Next:**

- Conversions in both directions as first-class operations, not a side effect
- Documentation retrieval for concepts, idioms, and platform limits — not just
  signatures
- An evaluation harness with published benchmarks
- A decision layer for routing and risk gates
- Backtesting

**Later:**

- Research and market-analysis agents
- Monitoring and alerting
- Execution, behind explicit human approval

## What we are deliberately not doing yet

Saying this out loud is more useful than a roadmap of maybes.

- **No backtest engine of our own.** We will integrate established libraries.
  Building an engine is a multi-year project and is not where our
  differentiation is.
- **No execution or live trading.** Not until validation, risk gating, and
  explicit human approval exist. Automated order placement is the highest-
  consequence thing this project could ever do, and we are not going to be
  casual about it.
- **No managed cloud at launch.** Open-source core first. Hosted services come
  after there is a community that wants one.
- **No decision models in the core architecture.** Specialized fast-decision
  models are interesting but unvalidated for trading. They stay behind an
  interface until benchmarked on real tasks, not assumed.
- **No redistributed documentation.** We ship extractors and schema, not
  TradingView's or GoCharting's content. See
  [docs/comparison.md](docs/comparison.md#documentation-and-licensing).

## Non-negotiables

These are the commitments that define the project. Breaking one is a
reconsideration of the project, not a bug fix.

- **Open source core.** Apache 2.0. If we ship hosted services, the core stays
  open and self-hostable.
- **Model-agnostic.** No required provider. Bring your own key, or run locally.
- **No lock-in on user data.** Corpus, history, projects, and keys stay yours.
- **Honest status.** We say what works and what does not. A pre-alpha project
  that admits it is pre-alpha is more trustworthy than one that implies
  otherwise.
- **Trading decisions are not automated silently.** If Openfade ever
  influences a real order, a human approved it.

## Contributing

- One pull request = one topic. Do not bundle unrelated changes.
- The corpus and the validator are the priority. A change that makes either more
  correct beats a new feature.
- If you disagree with something in this document, open an issue. Silently
  diverging from the vision is worse than disagreeing with it out loud.
- Claim work in an issue before you start on anything substantial.

## Security

Security is a deliberate tradeoff: strong defaults without crippling
capability. The full policy is in [`SECURITY.md`](SECURITY.md). The short
version: user API keys never leave the user's environment, the corpus is built
locally, and anything that could affect a real trading account is gated behind
explicit human approval.

---
summary: "How Openfade compares to other approaches to AI trading-script work, and where those approaches are better."
read_when:
  - You are deciding whether Openfade is right for you
  - You are evaluating alternatives and want an honest comparison
  - You want to know why Openfade does not ship vendor documentation
  - You are checking trademark or affiliation status
title: "Comparison"
---

# Comparison

This document is written to be useful even if you conclude Openfade is the wrong
choice. We would rather lose a user to a better fit than win one by omission.

**Status: Openfade is pre-alpha and contains no working code.** Everything below
is a statement of intent. Nothing described here does anything yet.

## Approaches, not companies

Naming individual products and ranking them ages badly, creates legal exposure,
and flatters whichever project writes it. So this compares *approaches*.

### Approach 1 — general assistant with documentation pasted in

You paste the Pine or TradingView docs into a capable model, or use a coding
assistant that has them in context, and ask for a script.

**Strengths.** Excellent for one-off scripts you will read and understand
yourself. No setup, no infrastructure, no learning curve. Genuinely good at
strategy logic, explanation, and the "how would I express this idea" question.
For a single indicator you will edit by hand, this is often the right answer
today.

**Where it breaks.** It has no persistent knowledge of which symbols exist in
which language. It cannot be made to have one through prompting, because the
model does not know when it is wrong. Cross-language conversion is the worst
case: Pine is in the training data, Lipi is not, so the model emits Pine with a
Lipi-shaped wrapper. The output is fluent and wrong, and there is no signal that
anything is off.

**Choose this if** you want one script now and will verify it yourself.

### Approach 2 — purpose-built commercial script generators

Commercial tools that specialise in generating indicators and strategies for a
specific platform, often with backtesting and a visual editor attached.

**Strengths.** They are integrated with the platform, so scripts are known to run.
Many include a working backtester and a parameter optimiser, which is a
significant amount of engineering. Some offer visual strategy construction for
people who will never write code. For generating a working indicator on the
platform they target, this is often a better product than anything we will ship
soon.

**Where it breaks.** Platform-locked, generally closed, generally not
extensible. No conversion between languages, because conversion is hard and not
where the commercial incentive points. Little visibility into how generation
works. Some include their own model assistance, which is a fine approach for
generation but does not solve cross-language translation.

**Choose this if** you want a polished, integrated experience on one platform and
do not need openness or conversion.

### Approach 3 — algorithmic trading frameworks

QuantConnect, backtesting.py, vectorbt, and similar. Full control over
research, data, and execution.

**Strengths.** Real engineering, tested, documented, and the right tool for
systematic strategy development. Excellent backtesting and data handling. Where
the goal is a production trading system, this is the correct category of tool.

**Where it breaks.** Requires programming. The barrier Openfade is trying to
remove is precisely the one these require you to clear. They also do not
generate platform-native scripts — they produce their own code.

**Choose this if** you can code and want a real research environment. We plan to
integrate with this category rather than replace it.

### Approach 4 — general coding agents

OpenCode, Claude Code, Codex, and similar agentic coding tools.

**Strengths.** They reason about code well, navigate repositories, run tests,
and iterate until something works. Applied to a small project with a good test
suite, they are formidable.

**Where it breaks.** They are not trading-native. They have no symbol table, no
dialect layer, and no validator. They will happily produce a plausible
LipiScript that does not compile, because nothing in their context tells them it
is wrong. They are also a substitute for our corpus and validator work, not a
complement to it.

**Choose this if** you are building a trading tool and want help building it.
We are open source partly so these tools work well on us.

### What Openfade intends to be different at

Not better at any of the above. Different in four specific ways:

| | Other approaches | Openfade |
| --- | --- | --- |
| Knowledge of the language | training data, or pasted docs | extracted, versioned, queryable symbol corpus |
| Cross-language | unsupported, or unreliable | explicit dialect table, lossy cases declared |
| Correctness signal | none — you find out when it fails to compile | static validation before output reaches you |
| Lock-in | platform, provider, or both | your keys, your models, your data, your machine |

The first three are the same claim three ways: **we are trying to make "it
works" verifiable rather than plausible.** If we cannot do that, we do not have
a product, and the honest thing is to say so.

## Where Openfade will be worse

Stated plainly, because it will be true:

- **For a single simple script, an assistant is faster.** No setup beats our
  setup.
- **For platform integration, commercial tools are better.** They have working
  backtesters and optimisers. We are starting with neither.
- **For serious quantitative research, frameworks are better.** They are
  tested by people who do this for a living.
- **Until the corpus and validator exist, we are strictly worse than all of the
  above.** The documentation you are reading is a promise, not a capability.

## Documentation and licensing

**We do not ship TradingView's or GoCharting's documentation.** The corpus is
built locally, on your machine, when you run the corpus build step.

The reason is not only legal. Building on demand means that when Pine gains a
version or Lipi changes a namespace, your next build picks it up. We do not have
to cut a release, and the corpus cannot go stale relative to upstream. We ship
the extractor and the schema — the furnace, not the bread.

The practical consequence for contributors: **do not commit scraped vendor
documentation.** Hand-authored fixtures describing a signature are fine and
encouraged. Copied prose is not.

## Trademarks and affiliation

Openfade is an independent project. It is not affiliated with, sponsored by, or
endorsed by TradingView, Inc. or GoCharting.

- **Pine Script** is a trademark of TradingView, Inc. Scripts Openfade produces
  are Pine Script because that is the target language, not because of any
  relationship.
- **Lipi** is a product of GoCharting. Same reasoning.
- Generated scripts remain subject to the terms of the platform they are written
  for. Openfade does not grant you rights to use either platform, and using
  either remains your decision and your responsibility.

## A note on trading advice

Openfade is a development tool. It produces code and analysis. It is not an
investment adviser, does not provide signals, and makes no claim about future
returns. A strategy that Openfade generates, validates, and helps you backtest
can still lose money — in fact most will, over enough time and enough markets.

If a tool that generates plausible strategies is more dangerous to you than one
that does not exist, that is a reasonable position and you should not use this.

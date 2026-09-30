---
summary: "Template for pull requests to Openfade."
read_when:
  - You are opening a pull request
  - You want to know what we expect in a pull request description
  - You are reviewing someone else's pull request
title: "Pull Request"
---

## What this changes

<!-- One or two sentences. What is different after this merges? -->

## The problem

<!--
What was wrong, missing, or slow. Not a restatement of the diff — the diff
shows what changed, this explains why it needed to change.
-->

Closes #

## How this was verified

<!--
Be specific. Tests run, cases checked, docs read, manual steps performed.

If you verified nothing beyond "it compiles", say that. An accurate account of
a partial verification is more useful than an implied full one.
-->

- [ ] Tests pass
- [ ] Manually verified as described above
- [ ] Documentation updated, if behaviour or design changed

## Scope check

- [ ] This pull request contains **one topic**. Unrelated changes are split out.
- [ ] Nothing scraped from TradingView or GoCharting documentation is included.
      Hand-authored fixtures describing a signature are fine.
- [ ] No API keys, tokens, or credentials appear anywhere in the diff,
      including tests and examples.
- [ ] No code path can affect a live trading account.

## Deliberate omissions

<!--
If this is partial, incomplete, or leaves a known problem, say so here. We would
much rather have an honest partial pull request than a clean-looking one that
hides a gap.
-->

## Type of change

- [ ] Corpus / built-in fixtures
- [ ] Dialect or translation table
- [ ] Validator
- [ ] Agent graph or agent
- [ ] CLI or platform layer
- [ ] Documentation
- [ ] Infrastructure or build

## Related

<!-- Any issue, discussion, or ADR this connects to. -->

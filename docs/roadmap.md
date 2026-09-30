---
summary: "What Openfade is building, in what order, and what is explicitly not planned yet."
read_when:
  - You want to know what is being worked on and in what order
  - You are looking for something to contribute to
  - You want to know whether a feature you need is planned
  - You want to judge whether the project has a credible sequence
title: "Roadmap"
---

# Roadmap

**How to read this.** Unchecked means **not started** – not partially done. We
use explicit status rather than percentages because "60% complete" on a
distributed open-source project is not a meaningful claim.

Everything here is a commitment we intend to keep, not a guarantee. If something
changes, this file changes with it, in the open.

**Current state: no implementation exists.** The project is documentation and
architecture only.

---

## Phase 0 – Foundations

*Status: in progress*

- [x] Vision, scope, and architecture defined
- [x] Public documentation set written
- [x] Repository community health files in place
- [ ] `pyproject.toml`, `go.mod`, and project scaffolding
- [ ] `docker-compose.yml` with PostgreSQL and pgvector
- [ ] `AGENTS.md` conventions enforced by CI

## Phase 1 – The corpus

*Status: not started*

**This is the critical path. Nothing downstream works without it.**

- [ ] Pine v6 reference extractor
- [ ] Lipi v1 reference extractor
- [ ] Normalised `Builtin` schema with signature, parameters, return type
- [ ] pgvector storage and hybrid retrieval
- [ ] Conflict detection for inconsistent upstream documentation
- [ ] Hand-authored fixture set per dialect
- [ ] Retrieval evaluation harness with published recall numbers

## Phase 2 – Dialects and the validator

*Status: not started*

- [ ] `LanguageDialect` protocol and Pine implementation
- [ ] Lipi implementation
- [ ] Pine↔Lipi delta table, with structural differences declared
- [ ] Validator: version annotation and declaration checks
- [ ] Validator: symbol resolution and arity
- [ ] Validator: cross-dialect contamination
- [ ] Validator: type qualifier conflicts
- [ ] Validator: structural and platform limit checks
- [ ] Negative test suite per dialect

## Phase 3 – First working path

*Status: not started*

- [ ] Intent → plan → generate graph
- [ ] Structured error contract for repair
- [ ] Generate → validate → repair loop
- [ ] Output contract: script plus assumptions plus unverified claims
- [ ] `openfade` CLI
- [ ] Public benchmark for generation validity

## Phase 4 – Conversion

*Status: not started*

- [ ] Pine parser
- [ ] Lipi parser
- [ ] Pine → Lipi translation with declared lossy cases
- [ ] Lipi → Pine translation
- [ ] Round-trip semantic equivalence tests
- [ ] Semantic difference report

## Phase 5 – Beyond code

*Status: not started*

- [ ] Concept and idiom retrieval, not just signatures
- [ ] Backtest engine evaluation and integration
- [ ] Strategy analysis and performance attribution
- [ ] Decision layer for routing and risk gates
- [ ] Plugin and agent registry for community contributions

## Not planned

Listed so that scope is unambiguous. Each of these is a deliberate omission with
a reason, not a backlog item.

| Not doing | Why |
| --- | --- |
| Our own backtest engine | Multi-year project with no bearing on our differentiation. Integrate instead |
| Live order execution | Highest-consequence capability we could ship. Requires a trusted validator, risk gating, and per-action human approval |
| Hosted cloud at launch | Open-source core first. We would rather have a community than customers before we have either |
| Decision models in the core | Interesting, unvalidated for trading, and several have input limits that break on strategy-sized payloads. Interface first, dependency later |
| Vendored documentation | Built locally instead. Legally cleaner and never goes stale |
| Mobile apps | Not before the core works on a command line |

## Versioning

We have not cut a release. When we do, it will be `0.1.0` and it will be
pre-alpha, because a corpus and a validator are the only things that will exist
and neither will be complete.

We are not going to call something `1.0` until conversion round-trips
semantically for the common cases and the validator has a real negative test
suite. That is a higher bar than it sounds, and it is the right one.

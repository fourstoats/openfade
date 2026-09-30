---
summary: "Release history for Openfade. There are no releases yet — this tracks notable changes on main."
read_when:
  - You want to see what has changed recently
  - You are tracking whether a specific change has landed
  - You want to know how this project versions its work
title: "Changelog"
---

# Changelog

All notable changes to Openfade are recorded here.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html)
once releases exist.

**There are no releases yet.** Entries below describe changes to the main branch
rather than published versions. The first release will be `0.1.0` and will be
explicitly pre-alpha.

## [Unreleased]

### Added

- Initial public documentation set covering vision, architecture, comparison,
  and roadmap
- `README.md` rewritten around the open-source trust argument: Apache 2.0,
  model-agnostic, bring your own key, no planned paid wall on the core
- `VISION.md` at repository root, with current focus and an explicit list of
  what is deliberately out of scope
- `AGENTS.md` defining working rules for AI agents, including a
  documentation convention requiring `read_when` frontmatter on every page
- `docs/architecture.md` covering the five-layer design, the `LanguageDialect`
  abstraction, the Pine–Lipi delta table, the corpus schema, and the validator
- `docs/comparison.md` comparing approaches to AI trading-script work, including
  where those approaches are better than Openfade intends to be
- `docs/roadmap.md` with sequenced phases and an explicit "not planned" section
- `SECURITY.md` establishing reporting process, vulnerability scope, and the
  boundaries Openfade will not cross
- `CODE_OF_CONDUCT.md` including trading-specific expectations around
  performance claims and signals
- `CONTRIBUTING.md` identifying corpus fixtures and validator rules as the
  highest-value early contributions
- Issue and pull request templates
- `.env.example` documenting the environment variables Openfade will use

### Changed

- `.gitignore` extended to cover Python tooling alongside the existing Go
  entries

### Notes

The repository contains no implementation. The documentation describes intended
architecture and commits the project to a sequence, not to a delivery date.
Anyone is welcome to pick up the first unchecked item on
[`docs/roadmap.md`](docs/roadmap.md).

## Versions

| Version | Date | Status |
| --- | --- | --- |
| `0.1.0` | — | planned, pre-alpha |

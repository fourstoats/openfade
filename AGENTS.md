---
summary: "Working rules for AI agents in this repository. Read before writing code, docs, or issues."
read_when:
  - You are an AI agent about to modify anything in this repository
  - You need to know the current implementation state
  - You are adding or editing a documentation file
  - You are unsure whether a module, command, or API exists
title: "AGENTS.md"
---

# AGENTS.md

Instructions for AI agents working in the Openfade repository.

## Current state — read this first

**This repository has no implementation.** As of the initial commit it contains
only documentation: `README.md`, `VISION.md`, `docs/`, and the community health
files. There is no `openfade` package, no Go module, no CLI, no test suite.

This matters for you specifically. You have likely seen Openfade-adjacent
material in training data or reasoned about what this project "should" contain.
**Do not assume any module, command, function, or endpoint exists.** There is no
`openfade init`, no `LanguageDialect` class, and no corpus builder. If you need
one, you are writing it, not calling it.

Verify before you reference. `ls`, `glob`, and `grep` are cheap. Guessing is not.

## What this project is

An open-source AI agent platform for trading. The first wedge is generating and
converting TradingView Pine Script and GoCharting Lipi. Read
[`VISION.md`](VISION.md) for direction and [`docs/architecture.md`](docs/architecture.md)
for the system design.

## Non-negotiables

Do not compromise these, even if asked to "just get it working":

- **Apache 2.0 core.** Never introduce a copyleft or source-available license
  into the core.
- **Model-agnostic.** No code path may require a specific provider. Provider
  support goes behind a common interface.
- **No redistributed vendor docs.** The corpus is built locally at ingest time
  from upstream documentation. Never commit extracted TradingView or GoCharting
  prose, descriptions, or documentation text to this repository. Derived
  structural facts we author ourselves (signatures, parameter types, our own
  paraphrases) are acceptable; copied prose is not.
- **No credentials in git.** Not in tests, not in examples, not commented out.
  Use environment variables and document them in `.env.example`.
- **Nothing auto-executes a trade.** Any code path that could affect a live
  account requires explicit human approval. There is no exception to this and
  no configuration flag that removes it.

## Repository layout

```
.
├── README.md              # public front door
├── VISION.md              # direction and priorities
├── AGENTS.md              # this file
├── CONTRIBUTING.md        # human contribution guide
├── CODE_OF_CONDUCT.md
├── SECURITY.md
├── LICENSE                # Apache 2.0
└── docs/
    ├── index.md
    ├── architecture.md
    ├── comparison.md
    └── roadmap.md
```

Planned layout, for orientation only — none of this exists yet:

```
openfade/          # Python: dialects, corpus, retrieval, validation, agent graph
cmd/openfade/      # Go: CLI entrypoint, platform layer
docs/              # grows as the surface does
```

## Documentation conventions

**Every Markdown file in this repository opens with `read_when` frontmatter.**
This is not decoration. It is the retrieval index that both humans and agents
use to decide which page to load without reading everything.

```markdown
---
summary: "One line: what this document is."
read_when:
  - A concrete situation that should send you here
  - Another concrete situation
title: "Human title"
---
```

Rules:

- `summary` is one line, not a sentence fragment. It answers "what is this?"
  in isolation, because it is often read without the body.
- `read_when` entries are **situations, not topics.** "You want to convert a
  Pine script to Lipi" is useful. "Pine and Lipi" is not — it does not help
  anyone decide whether to read the page.
- Three to five entries. More than that and the list stops being scannable.
- No `read_when` on `README.md` — it is the entry point, not a retrieved page.
- Root-level `AGENTS.md` and `VISION.md` keep theirs, because agents do route on
  them.

## Tone

Openfade's documentation is read by traders who are smart and busy, and by
developers who will check whether claims are true.

- **Separate what exists from what is planned.** Use explicit status markers.
  Never imply a feature works because it is described in the present tense.
- **No superlatives.** Not "world-class", "revolutionary", "the best". Let the
  architecture and the benchmarks make the case.
- **No competitor disparagement.** Compare approaches honestly, including where
  another approach is better. See
  [`docs/comparison.md`](docs/comparison.md).
- **Admit limits.** If something is slow, expensive, or unvalidated, say so. A
  documented limitation is more valuable than an implied promise.
- **Second person, present tense, active voice.** "Openfade validates scripts"
  not "scripts will be validated by the Openfade system".

## Working agreements

- **One topic per pull request.** Do not bundle unrelated changes.
- **Claim the work in an issue before starting** on anything substantial.
- **Corpus and validator first.** Those two are the project's defensible core.
  A change that makes either more correct is worth more than a new feature.
- **Match the existing language to the file you are in.** Python for
  `openfade/`, Go for `cmd/`, English for docs.
- **Do not add dependencies casually.** Every dependency is a maintenance and
  license obligation. Propose it in an issue with the reasoning.

## Testing expectations

There is no test suite yet. When one exists:

- Corpus extraction is tested against **known-good fixtures** checked into the
  repository as our own authored data — not scraped vendor content.
- The validator is tested against both passing and deliberately broken scripts
  per dialect. A validator with no negative tests is not a validator.
- Dialect translation is tested as a **round trip**: Pine → Lipi → Pine, with
  the result compared semantically rather than textually. Textual equality
  would hide exactly the class of bug we are trying to eliminate.

## Getting things wrong

If you follow these instructions and something is still wrong, say so in your
summary rather than smoothing over it. We would rather have an accurate report
of a broken state than a confident description of something that does not work.

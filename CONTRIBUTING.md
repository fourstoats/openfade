---
summary: "How to contribute to Openfade, what help is most needed right now, and the conventions we follow."
read_when:
  - You want to contribute code, documentation, or trading-domain knowledge
  - You are preparing your first pull request
  - You want to add support for a new scripting language
  - You are looking for a task to start on
title: "Contributing"
---

# Contributing to Openfade

Thanks for considering it. We are early, the roadmap is public, and there is
genuinely useful work available to people who have never seen this codebase.

## Current state

**Openfade contains documentation only.** There is no build, no test suite, and
no released version. If you came here expecting a running project, that is
accurate — it does not exist yet.

This has a practical consequence: the most valuable contributions right now are
**not** features. They are the foundations described below, and several of them
need trading domain knowledge more than they need software engineering.

## Where help is most needed

In priority order:

### 1. Corpus fixtures

The corpus is the foundation. We build it locally from upstream documentation,
but we need **known-good, hand-authored fixtures** committed to the repository —
our own authored data, never scraped vendor text.

A fixture is a small, correct record of a language built-in: signature,
parameter names and types, return type, and a minimal usage example. If you know
Pine or Lipi well, writing accurate fixtures for it is directly useful work and
requires no infrastructure to get started.

**This is the single best place for a new contributor to begin.**

### 2. The dialect delta table

We maintain an explicit, versioned description of how Pine and Lipi differ —
namespace renames, keyword substitutions, and structural differences that a
rename cannot express. It is currently partially derived and partially
inferred. If you can verify entries against the official references, that is
valuable.

Where the two languages are genuinely different in kind rather than in name,
say so. "These cannot be translated automatically" is a more useful finding than
a forced mapping.

### 3. Validator rules

The validator is what makes "it compiles" a true statement. Ideas worth
proposing: type-qualifier conflict detection, undefined-symbol checking,
required-declaration-statement validation, platform limit checks, and the
structural rules that distinguish a plotting call from a plotting type.

Negative test cases matter more than positive ones. A validator with no
negative tests is not a validator.

### 4. Backtest integration research

We are deliberately not building a backtest engine. We are evaluating
established libraries. A write-up comparing candidates on correctness,
license, and maintenance status would save real time.

### 5. Documentation

Every file needs `read_when` frontmatter (see below). If a page is hard to
write, that is usually a sign the underlying design is not settled yet — open an
issue instead.

## Setup

Not available yet. There is no `pyproject.toml` and no `go.mod`.

We are targeting:

```bash
# Python 3.12+, Go 1.24+, Docker
docker compose up -d          # PostgreSQL with pgvector
pip install -e ".[dev]"
openfade init                 # build the local corpus
openfade check                # run the test suite
```

Until this exists, contributions that do not require running the project are the
most useful. Documentation and fixture contributions need no setup at all.

## Documentation conventions

Every Markdown file in this repository opens with `read_when` frontmatter:

```markdown
---
summary: "One line: what this document is."
read_when:
  - A concrete situation that should send you here
  - Another concrete situation
title: "Human title"
---
```

This is a functional retrieval index, not decoration. Humans and agents both
use it to decide which page to load. `read_when` entries must be **situations**,
not topics — "you are converting a Pine script to Lipi" routes; "Pine and Lipi"
does not. Keep it to three to five entries.

More detail in [`AGENTS.md`](AGENTS.md).

## Pull requests

- **One topic per pull request.** Do not bundle unrelated changes. It makes
  review possible and it makes the history readable.
- **Claim the work in an issue first** for anything substantial, so two people
  do not build the same thing.
- **Explain the why.** The diff shows what changed. Say what problem it solves
  and how you verified it.
- **Say what you did not do.** If a PR is partial, incomplete, or leaves a known
  problem, say so in the description. That is more useful than a clean-looking
  PR that hides a gap.
- **Documentation changes are pull requests too.** If you found something
  unclear, fix it or open an issue.

### Commit messages

[Conventional Commits](https://www.conventionalcommits.org/), lightly used.

```
feat(corpus): add Lipi talib.sma fixture
fix(validator): reject undefined namespace
docs: clarify Pine version annotation
refactor(dialect): mark request.security as lossy
```

**Format:** `<type>(<scope>): <description>`. The type is required; the scope is
optional. Valid types are `feat`, `fix`, `docs`, `refactor`, `perf`, `test`,
`build`, `ci`, `chore`, and `revert`.

**Scopes** that help most here: `corpus`, `dialect`, `validator`, `agent`, `cli`,
`docs`.

**Writing the description:**

- **Imperative mood.** "add" not "added" or "adds". The subject completes the
  sentence "this commit will…".
- **No trailing period.** One line, under 72 characters.
- **Describe the change, not the mistake.** History is permanent and is read by
  people deciding whether to trust this project. A subject line is not the place
  to narrate how a problem got there.
- **The body explains why**, for changes where the reason is not obvious from
  the diff. Wrap at 72 characters. Omit it for self-explanatory changes.

**Squash before pushing, not after.** If a commit is unpushed and its message
does not say what the change actually did, amend it. History that has not left
your machine is not a record yet.

## Adding a new scripting language

Openfade is not a Pine-and-Lipi-only project forever, and the architecture is
meant to support more dialects. If you want to add one, open an issue before
writing code. A new dialect needs:

- An official or authoritative language reference to build a corpus from
- A version identifier
- A delta table against Pine
- A clear answer on whether it is structurally translatable or only
  superficially similar

That last point is the one that decides feasibility. If a language does not
share Pine's execution model, conversion is a much larger problem than renaming
namespaces, and we would rather know that up front.

## Reporting bugs

Open an issue. Include the input, the actual output, the expected output, and
your Openfade version or commit. If the problem is a generated script that
validates but does not behave as described, the script itself is the most useful
thing you can include — that is exactly the failure mode we are trying to
eliminate, and each real instance makes the validator better.

Security issues go to [`SECURITY.md`](SECURITY.md), not the issue tracker.

## License

Contributions are accepted under the Apache 2.0 license, which covers both the
code and the documentation. See [`LICENSE`](LICENSE).

Do not contribute scraped TradingView or GoCharting documentation content. See
[`docs/comparison.md`](docs/comparison.md#documentation-and-licensing) for the
reasoning.

## Code of conduct

Participation is governed by [`CODE_OF_CONDUCT.md`](CODE_OF_CONDUCT.md). It
includes trading-specific expectations around performance claims and signals.

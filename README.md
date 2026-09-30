<div align="center" id="top">
  <a href="https://github.com/fourstoats/openfade">
    <img width="403" height="72" alt="Openfade" src="https://github.com/user-attachments/assets/2faccbe5-54ba-4f0c-9eb8-a121dcd6572f" />
  </a>
  <p><strong>Turn a trading idea into Pine Script and Lipi that actually compile.</strong></p>
  <p>
    <a href="#architecture">Architecture</a> ·
    <a href="#contributing">Contributing</a> ·
    <a href="#roadmap">Roadmap</a> ·
    <a href="#documentation">Documentation</a> ·
    <a href="https://github.com/fourstoats/openfade/discussions">Discussions</a>
  </p>
</div>

<div align="center">

[![License: Apache 2.0](https://img.shields.io/badge/license-Apache%202.0-blue.svg)](LICENSE)
[![Stars](https://img.shields.io/github/stars/fourstoats/openfade?style=flat-square&logo=github)](https://github.com/fourstoats/openfade/stargazers)
[![Forks](https://img.shields.io/github/forks/fourstoats/openfade?style=flat-square&logo=github)](https://github.com/fourstoats/openfade/network/members)
[![Discussions](https://img.shields.io/github/discussions/fourstoats/openfade?style=flat-square&logo=github)](https://github.com/fourstoats/openfade/discussions)
[![Last commit](https://img.shields.io/github/last-commit/fourstoats/openfade?style=flat-square&logo=github)](https://github.com/fourstoats/openfade/commits/main)
[![PRs welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)
[![Code of Conduct](https://img.shields.io/badge/Code%20of%20Conduct-Contributor%20Covenant-ff69b4.svg)](CODE_OF_CONDUCT.md)

</div>

> **Status: pre-alpha. There is no working code.** This repository is the public
> build of a design, not a product. The vision and the architecture are
> committed, the implementation has not started, and every capability below is
> marked with its real state. We would rather show you an honest empty
> checklist than a confident feature list.
>
> The parts that need contributors most, the corpus and the validator, are
> exactly the parts where domain knowledge matters more than engineering
> effort. See [Contributing](#contributing).

---

## The problem

Trading knowledge and trading software live in two different worlds.

You know *that* you want price above the 200 EMA with an RSI confirmation and a
2% stop. You do not know that this means `ta.ema`, a series-versus-int type
qualifier conflict on the stop parameter, and a `strategy.exit()` call whose
behaviour changes depending on how you typed the stop.

That gap is normally closed by copying a similar script from a forum and
adjusting it by eye. It works, slowly, and it teaches you nothing you can
transfer to the next platform you try.

The other option is asking a language model. That works better than the forum,
right up until it does not.

## What Openfade does

Openfade is an agent system built around one bet: that the hard part of
generating trading scripts is **retrieval and validation, not prompting**.

Describe the strategy in plain language. Openfade retrieves the actual built-in
symbols for the target platform, generates the script against them, then proves
the script is structurally valid before handing it back. If a symbol does not
exist in the target language, the validator refuses to let it through.

The first wedge is **Pine Script** (TradingView) and **Lipi** (GoCharting).
Scripts are the entry point rather than the destination. The longer arc is the
full trading lifecycle:

```
   research  →  strategy  →  code  →  backtest  →  analysis  →  decision
                                                                    ↓
                                                          automation (later)
```

See [VISION.md](VISION.md) for where this goes and, just as importantly, where
it does not.

## Why a general LLM fails at this

Pine Script is heavily represented in model training data. Lipi is barely
represented at all.

Ask a general model to convert a Pine script into Lipi and it will confidently
emit Pine. `ta.sma()` where Lipi wants `talib.sma()`. `timeframe.isdaily` where
Lipi wants `interval.isdaily`. Function calls where Lipi needs type declarations.
The output reads well, looks idiomatic, and does not compile.

This is not a prompt engineering problem. The model does not know it is wrong,
because in its training distribution Pine is overwhelmingly the more likely
continuation. Better instructions do not fix a statistical bias that strong.

| | Generic chatbot | Openfade |
| --- | --- | --- |
| Knows Pine APIs | from training data | from an extracted, versioned corpus |
| Knows Lipi APIs | barely | from an extracted, versioned corpus |
| Handles Pine to Lipi | silently falls back to Pine | explicit translation table |
| Tells you when it is wrong | no | refuses unknown symbols |
| Proves the script is valid | no | static validation before output |

That table is the entire technical bet. Everything else, including the agent
graph and the lifecycle above, is downstream of whether this core is real.

We are also honest about the failure mode: **it is silent.** A model does not
emit a warning when it falls back to Pine. It emits Pine. This is precisely why
validation cannot be an optional extra layer bolted on at the end. See
[docs/architecture.md](docs/architecture.md) for the full reasoning.

## Language support

Both languages are first-class from the start. Neither is an afterthought and
neither is a "coming soon" placeholder.

| Capability | Pine Script | Lipi | State |
| --- | --- | --- | --- |
| Natural language to script | v6 | v1 | not started |
| Script to script conversion | yes | yes | not started |
| Corpus extraction | v6 reference | v1 reference | not started |
| Dialect implementation | planned | planned | not started |
| Static validation | planned | planned | not started |

Lipi is closely modelled on Pine. Both are bar-by-bar cloud-executed languages
with `syminfo` namespaces, a history-referencing operator, technical-analysis
namespaces, and indicator or strategy declarations. That similarity is exactly
why one architecture serves both rather than two.

They are not the same language, though. The differences that a rename cannot
express are the interesting part, and they are documented in
[docs/architecture.md](docs/architecture.md#the-pine-lipi-delta).

| Concern | Pine v6 | Lipi v1 | Kind of difference |
| --- | --- | --- | --- |
| TA namespace | `ta.*` | `talib.*` | rename |
| Timeframe info | `timeframe.*` | `interval.*` | rename |
| Persistent state | `var`, `varip` | `static` | near-synonym, different semantics |
| Function declaration | `name(args) =>` | `func`, `def` | structural |
| Return | implicit, last expression | explicit `return` | structural |
| Type qualifiers | `const` `input` `simple` `series` | plus `intra` | no Pine equivalent |
| Plotting | `plot()`, `plotshape()` functions | `plot`, `plotStyle`, `hline`, `shape` types | structural |
| Version declaration | `//@version=6` | none | asymmetric |
| `request.security` | supported | supported | highest-risk, most semantic drift |

Two caveats, stated plainly. The structural differences above are **inferred**
from keyword and type listings, not confirmed against the full reference. The
corpus build is what turns inference into fact. And GoCharting's own
documentation contradicts itself in at least one place, describing
`talib.sma()` as belonging to the `ta` namespace in the same sentence it uses the
`talib.` prefix. Ingested naively, that teaches a model both names are correct.
The extractor needs an explicit alias table and a conflict report rather than
blind scraping.

## Architecture

Six layers. One of them is the project.

```
L0  Platform       Go       auth, projects, sessions, queues, rate limits, key vault
L1  Model Gateway  Python   provider-agnostic, tier routing, caching, cost accounting
L2  Intelligence   Python   dialects, corpus, retrieval, validation      ← the core
L3  Agent Graph    Python   graph execution, node registry, tool registry
L4  Domain Agents  Python   research, strategy, code, conversion, debug, backtest
L5  Services       Python   market data providers, backtest engines, execution adapters
```

L2 is the only layer that is genuinely Openfade. L0 is a service with a
database. L1 is model plumbing. L3 is an agent framework. L5 is other people's
libraries.

If L2 is weak, the result is a trading-themed chatbot, and GoCharting already
ships one of those. Engineering effort should go disproportionately to L2, even
though it is the least visible layer. This is stated plainly so it is easy to
disagree with, and [docs/comparison.md](docs/comparison.md) is where that
disagreement belongs.

### The dialect abstraction

One protocol, one implementation per language. This is the seam that makes Pine
and Lipi a single product rather than two products in one repository.

```python
class LanguageDialect(Protocol):
    id: str                      # "pine" | "lipi"
    version: str                 # "6" | "1"
    version_annotation: str      # "//@version=6" | ""
    declarations: tuple[str, ...]        # ("indicator", "strategy", "library")
    namespace_aliases: Mapping[str, str] # {"ta": "talib", "timeframe": "interval"}
    keyword_substitutions: Mapping[str, str]
    structural_differences: tuple[str, ...]  # things a rename cannot express
    limits: PlatformLimits       # script size, token budget, execution limits
```

The `structural_differences` field matters more than it appears to. A field for
renames invites someone to believe everything is a rename. Naming the things
that are *not* renamable is the entire point of declaring them.

### The agent graph

A registry of nodes and tools rather than a hardcoded set of agents. The
registry is what lets the community contribute agents later without forking the
core.

```
intent
  ↓
plan          decompose into strategy logic, explicitly
  ↓
retrieve      exact symbols first, then concepts
  ↓
generate      emit for the target dialect
  ↓
validate      static checks
  ↓
repair        loop on structured errors, bounded
  ↓
output        script + what was assumed + what could not be verified
```

Conversion is this same graph with a parse step at the front and a dialect delta
applied at the generation step. It is not a separate system.

The output contract matters as much as the script. **Every result states what was
assumed and what could not be verified.** A script you cannot fully audit is not
an improvement over writing it yourself.

## The corpus

Both languages ship structured reference manuals. That is an advantage: the
correct unit of retrieval is **one built-in**, not one paragraph.

```python
class Builtin(BaseModel):
    dialect: str
    symbol: str                 # "ta.sma"
    kind: Literal["function", "variable", "type", "constant", "keyword"]
    signature: str
    params: list[Param]         # name, type, required, default
    return_type: str
    overloads: list[Signature]
    description: str            # our own paraphrase
    remarks: list[str]
    example_code: str
    anchor_url: str
```

Retrieval then splits by intent:

- **Exact symbol lookup first.** When the intent names a function such as
  `talib.rsi`, the model needs the signature, not a paragraph. A lookup beats a
  similarity search here.
- **Semantic fallback** over prose pages for idioms, platform limits, and
  gotchas, which are the things that are genuinely narrative.

Storing symbols as rows rather than chunking documentation into paragraphs and
hoping the right one is retrieved is materially more accurate.

**We ship extractors and schema, never the extracted content.** Corpus building
is local and happens on your machine at ingest time. This sidesteps
redistributing proprietary vendor documentation, and it means an upstream
language update gets picked up on the next corpus build rather than waiting on
a release from us.

One open risk, recorded rather than hidden: the Pine v6 reference manual is
client-side rendered. A plain static fetch returns a page shell with no
per-symbol anchors. The extractor will need a headless browser or an equivalent
path. Whether GoCharting's reference behaves the same way is not yet verified.
If it does, the Phase 1 approach shifts and we would rather find that out in the
open than discover it after writing the extractor.

## The validator

This is the component that makes "it compiles" a true statement, and it is why
we would build it before the generator.

Checks, in increasing cost:

1. **Version annotation** present and parseable
2. **Required declaration**, exactly one valid declaration statement
3. **Symbol resolution**, every namespace and function used exists in the
   dialect, with the right arity
4. **Cross-dialect contamination**, no `ta.*` in a Lipi script and no `talib.*`
   in a Pine script
5. **Type qualifier conflicts**, the `const` to `input` to `simple` to `series`
   hierarchy, where a whole class of Pine-specific errors lives
6. **Structural rules**, including global-scope indentation, which is a genuine
   Pine constraint that naive generators violate
7. **Platform limits**, script size, token budget, loop limits

Checks 1 through 4 are structural and fast. Check 7 may need real Pine
compilation, which we cannot do, because TradingView exposes no public compiler
API. That is a known gap, and it is the reason the static checks need to be
thorough rather than best effort.

The generate, validate, repair loop is what turns a validator into a product.
The model gets structured error objects back rather than a stack trace, so it can
fix the specific problem instead of rewriting the script.

## Planned command surface

There is no `openfade` binary yet. This is the command surface we intend to
build, shown so that contributions can target something concrete.

```bash
git clone https://github.com/fourstoats/openfade
cd openfade
docker compose up -d          # PostgreSQL + pgvector
pip install -e ".[dev]"

openfade init                 # build the local documentation corpus
openfade generate "price above 200 EMA, RSI crosses above 30, long with 2% stop"
openfade convert strategy.pine --to lipi
openfade validate strategy.lipi
```

We will publish a Quickstart section the day those commands work, and not
before.

## Roadmap

Unchecked means **not started**, not partially done. We use explicit status
rather than percentages, because "60% complete" on a distributed open-source
project is not a meaningful claim.

| Phase | Scope | State |
| --- | --- | --- |
| 0 | Foundations, scaffolding, CI | in progress |
| 1 | The corpus, extractors, retrieval evaluation | not started |
| 2 | Dialects and the validator | not started |
| 3 | First working path, generate/validate/repair | not started |
| 4 | Conversion between Pine and Lipi | not started |
| 5 | Idiom retrieval, backtesting, agent registry | not started |

Phase 1 is the critical path. Nothing downstream works without it.

What we are deliberately **not** doing, and why:

| Not doing | Why |
| --- | --- |
| Our own backtest engine | Multi-year project with no bearing on our differentiation. Integrate instead |
| Live order execution | Highest-consequence capability we could ship. Requires a trusted validator, risk gating, and per-action human approval |
| Hosted cloud at launch | Open-source core first. We would rather have a community than customers before we have either |
| Decision models in the core | Interesting, unvalidated for trading, and several have input limits that break on strategy-sized payloads. Interface first, dependency later |
| Vendored documentation | Built locally instead. Legally cleaner and never goes stale |
| Mobile apps | Not before the core works on a command line |

Full detail in [docs/roadmap.md](docs/roadmap.md).

## Documentation

| Document | Read it when |
| --- | --- |
| [VISION.md](VISION.md) | You want to know where this is going and why |
| [docs/architecture.md](docs/architecture.md) | You want the system design and the Pine/Lipi delta |
| [docs/comparison.md](docs/comparison.md) | You are deciding whether to use this or something else |
| [docs/roadmap.md](docs/roadmap.md) | You want to know what is being built next |
| [docs/index.md](docs/index.md) | You want the full documentation index |
| [AGENTS.md](AGENTS.md) | You are an AI agent working in this repository |
| [CONTRIBUTING.md](CONTRIBUTING.md) | You want to contribute code or docs |
| [SECURITY.md](SECURITY.md) | You found a security issue |

## Contributing

The corpus and the validator are the priority. A pull request that makes either
of them more correct is worth more than one that adds a feature, and that is not
a polite way of saying "everything else is less valued". It is the actual
engineering order, and it means a new contributor with real Pine or Lipi
knowledge can be useful on day one without writing a distributed system first.

Three things make contributing here unusually cheap:

- **The design is committed, so your work will not be thrown away.** The
  architecture, the corpus schema, and the validator checks are all written
  down and open to argument *now*, before implementation locks them in.
- **Domain knowledge beats engineering effort.** Extracting one built-in
  correctly, or writing the negative tests that prove a check works, is real
  value and does not require touching the agent graph.
- **The conventions are written down.** [AGENTS.md](AGENTS.md) covers the
  documentation rules and [CONTRIBUTING.md](CONTRIBUTING.md) covers the commit
  format, the test expectations, and the review process.

### Where help is needed most

| Task | Why it matters | Difficulty |
| --- | --- | --- |
| Pine v6 extractor | Unblocks the entire critical path | Medium |
| Lipi v1 extractor | Unblocks the entire critical path | Medium |
| Hand-authored fixtures | Every later test depends on them | Low |
| Validator negative tests | A validator with no negative tests is not a validator | Low |
| Pine/Lipi delta verification | Half that table is inferred, not confirmed | Low |
| GoCharting docs conflict report | Upstream contradicts itself in at least one place | Low |

The last three are the lowest friction and the most genuinely useful right now.
They need someone who knows the languages, not someone who knows the stack.

### Your first contribution

You do not need to set up the whole project to help.

**If you know Pine or Lipi**, the fastest useful contribution is a fixture: a
small script that is correct, or a small script that is broken in a specific
way. Those become the test corpus that everything else is measured against, and
they are the hardest thing to produce well in bulk.

**If you know the tooling**, Phase 0 needs `pyproject.toml`, `go.mod`, and a
`docker-compose.yml` with PostgreSQL and pgvector. Boring, essential, and a
clean first pull request.

**If you want to argue about the design**, open an issue. The
[Pine/Lipi delta table](docs/architecture.md#the-pine-lipi-delta) is explicitly
marked as inferred in several places. Telling us which rows are wrong is worth
more than any code you could write this month.

Start with [CONTRIBUTING.md](CONTRIBUTING.md) for the mechanics, and say hello in
[Discussions](https://github.com/fourstoats/openfade/discussions) before you
write anything if you want to.

## Community

- **Discussions** for questions, ideas, and design arguments:
  [github.com/fourstoats/openfade/discussions](https://github.com/fourstoats/openfade/discussions)
- **Issues** for bugs, proposals, and corpus contributions. There is a dedicated
  form for corpus and validator contributions, which is the highest-value
  contribution type here.
- **Security reports** go through GitHub private vulnerability reporting, not a
  public issue. See [SECURITY.md](SECURITY.md).

We are a small project with no code yet, and that is a real limitation. The
compensation for contributing early is influence over the design, and we would
rather have people who disagree with us in an issue than people who stay quiet.

## Star history

[![Star History](https://api.star-history.com/svg?repos=fourstoats/openfade&type=Date)](https://star-history.com/#fourstoats/openfade&Date)

Starring is useful for visibility and costs you nothing. Contributing is what
actually moves this forward.

## Frequently asked questions

**Is there a working version?**
No. The repository is documentation and architecture only. Every capability is
marked with its real state and none of them are built.

**Can I use this to trade?**
No, and there is no execution path at all. Live order placement is the highest
consequence thing this project could do, and it is not on the roadmap until
there is a validator we trust, risk gating, and per-action human approval.

**Do I need a paid model account?**
No. Openfade is model-agnostic and there is no required provider. Bring any key,
use a local model, or use whatever you already have.

**Will I need to pay later?**
No planned paid wall on the open-source core. If hosted services ever ship, the
core stays open and self-hostable. We would rather tell you now than surprise
you later.

**Why Pine and Lipi specifically?**
They are the two platforms where trading intent and executable code meet, and
Lipi is poorly served by general models. That second part is the interesting
one. See [docs/comparison.md](docs/comparison.md).

**Is the Pine/Lipi difference table accurate?**
Partly. The rename rows are confirmed. The structural rows are inferred from
keyword and type listings and are marked as such. Confirming them is a
contribution we are actively asking for.

**Can I add a third language?**
Yes, eventually, through the `LanguageDialect` protocol. It is not a one-line
change, because a language with genuinely different structure will want
`structural_differences` filled in properly. See
[docs/architecture.md](docs/architecture.md#the-dialect-abstraction).

**Why does the README say there is no code?**
Because a README that implies otherwise wastes contributor time and misleads
users. If that changes, this page changes with it, in the open.

## License

Apache 2.0. See [LICENSE](LICENSE).

Pine Script and TradingView are trademarks of TradingView, Inc. GoCharting and
Lipi are products of GoCharting. Openfade is an independent project and is not
affiliated with or endorsed by either. See
[docs/comparison.md](docs/comparison.md#trademarks-and-affiliation).
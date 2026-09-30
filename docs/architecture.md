---
summary: "How Openfade is designed: the five layers, the dialect abstraction, the corpus schema, and the validator."
read_when:
  - You want to understand how the system is put together
  - You are deciding where to contribute
  - You want to add support for a new scripting language
  - You are evaluating whether the technical approach is sound
title: "Architecture"
---

# Architecture

**Status: design, not implementation.** This document describes the intended
system. None of it exists yet. It is published so that contributions can be
aimed at a real design rather than an imagined one, and so disagreements can
happen now instead of after the code is written.

## The central claim

Trading-script generation is a **retrieval and validation** problem, not a
prompting problem.

Pine Script is heavily represented in model training data. Lipi is barely
represented at all. A general model asked to convert between them produces
output that is confident, idiomatic, and wrong — because the statistical path of
least resistance leads back to Pine. Prompt engineering does not fix this. The
model does not know it is wrong.

What fixes it is a **structured corpus** that knows which symbols exist in which
language, a **dialect layer** that knows how the languages differ, and a
**validator** that refuses to emit a symbol the target language does not have.

Everything else in this project is downstream of those three.

## Layering

```
L0  Platform       Go       auth, projects, sessions, queues, rate limits, key vault
L1  Model Gateway  Python   provider-agnostic, tier routing, caching, cost accounting
L2  Intelligence   Python   dialects, corpus, retrieval, validation      ← the core
L3  Agent Graph    Python   graph execution, node registry, tool registry
L4  Domain Agents  Python   research, strategy, code, conversion, debug, backtest
L5  Services       Python   market data providers, backtest engines, execution adapters
```

The strategic point: **L2 is the only layer that is genuinely Openfade.** L0 is a
service with a database. L1 is model plumbing. L3 is an agent framework. L5 is
other people's libraries.

If L2 is weak, the product is a trading-themed chatbot, and GoCharting already
ships one of those. Engineering effort should therefore go disproportionately to
L2 — even though it is the least visible layer.

## L2 in detail

### The dialect abstraction

One protocol, one implementation per language. This is the seam that makes Pine
and Lipi a single product rather than two.

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

`structural_differences` matters more than it looks. A field for renames invites
someone to believe everything is a rename. Naming the things that are *not*
renamable is the point.

### The Pine–Lipi delta

Lipi is closely modelled on Pine. Both are bar-by-bar cloud-executed languages
with `syminfo` namespaces, a history-referencing operator, technical-analysis
namespaces, and indicator/strategy declarations. That similarity is the reason
one architecture serves both.

| Concern | Pine v6 | Lipi v1 | Kind of difference |
| --- | --- | --- | --- |
| TA namespace | `ta.*` | `talib.*` | rename |
| Timeframe info | `timeframe.*` | `interval.*` | rename |
| Persistent state | `var`, `varip` | `static` | near-synonym, different semantics |
| Function declaration | `name(args) =>` | `func`, `def` | structural |
| Return | implicit, last expression | explicit `return` | structural |
| Type qualifiers | `const` `input` `simple` `series` | plus `intra` | Lipi has no Pine equivalent |
| Plotting | `plot()`, `plotshape()` — functions | `plot`, `plotStyle`, `hline`, `shape`, `chartPoint` — types | structural, not a rename |
| Version declaration | `//@version=6` | none | asymmetric |
| `request.security` | supported | supported | highest-risk, most semantic drift |

Two cautions on this table. First, `structural_differences` and the plotting
split are **inferred** from keyword and type listings, not confirmed against the
full reference. The corpus build is what converts inference into fact. Second,
GoCharting's own documentation contains an internal inconsistency — it describes
`talib.sma()` as belonging to the `ta` namespace in the same sentence it uses the
`talib.` prefix. Ingested naively, that teaches a model both names are correct.
The corpus builder needs an explicit alias table and a conflict report, not
blind scraping.

### The corpus

Both languages ship structured reference manuals. That is an advantage: the
correct unit of retrieval is **one built-in**, not one paragraph.

```python
class Builtin(BaseModel):
    dialect: str
    symbol: str                 # "ta.sma"
    kind: Literal["function", "variable", "type", "constant", "keyword"]
    signature: str
    params: list[Param]          # name, type, required, default
    return_type: str
    overloads: list[Signature]
    description: str            # our own paraphrase
    remarks: list[str]
    example_code: str
    anchor_url: str
```

Retrieval then splits by intent:

- **Exact symbol lookup first.** When the intent names a function
  (`talib.rsi`), the model needs the *signature*, not a paragraph. A lookup
  beats a similarity search here.
- **Semantic fallback** over prose pages for idioms, platform limits, and
  gotchas — the things that are genuinely narrative.

Storing symbols as rows rather than chunks is materially more accurate than
chunking documentation into paragraphs and hoping the right one is retrieved.

**Corpus building is local and on-demand.** We ship extractors and schema, never
the extracted vendor content. This sidesteps redistributing proprietary
documentation, and it means an upstream language update is picked up on the next
`openfade init` rather than requiring a release from us.

### The validator

This is the component that makes "it compiled" a true statement, and it is why
we would build it before the generator.

Checks, in increasing cost:

1. **Version annotation** present and parseable
2. **Required declaration** — exactly one valid declaration statement
3. **Symbol resolution** — every namespace and function used exists in the
   dialect, with the right arity
4. **Cross-dialect contamination** — no `ta.*` in a Lipi script, no `talib.*` in
   a Pine script
5. **Type qualifier conflicts** — the `const` → `input` → `simple` → `series`
   hierarchy, where a Pine-specific error class lives
6. **Structural rules** — global-scope indentation, which is a genuine Pine
   constraint that naive generators violate
7. **Platform limits** — script size, token budget, loop limits

Checks 1–4 are structural and fast. Check 7 may require Pine compilation, which
we cannot do — TradingView exposes no public compiler API. That is a known gap
and it is why the static checks need to be thorough rather than best-effort.

The generate → validate → repair loop is what turns a validator into a product:
the model gets structured error objects back, not a stack trace, and can fix the
specific problem rather than rewriting the script.

## L3: the agent graph

A registry of nodes and tools, not a hardcoded set of agents. The registry is
what lets the community contribute agents later without forking the core.

The first graph is deliberately small:

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

Conversion is this graph with a parse step at the front and a dialect delta
applied at the generation step. It is not a separate system.

The output contract matters as much as the script: **every result states what
was assumed and what could not be verified.** A script the user cannot fully
audit is not an improvement over writing it themselves.

## Scaling shape

Go for the high-volume product layer, Python for AI workloads, so the two scale
independently. Queued jobs, horizontal workers, caching, and a tier hierarchy so
simple decisions do not consume expensive reasoning models.

**The tier hierarchy is a cost-control mechanism, not an architecture
commitment.** We are not building a decision-model integration until it is
benchmarked on real trading tasks. Specialized fast-decision models are
interesting; several have input-length limits that make them unusable for
strategy-sized payloads, and none have been validated on trading. The interface
is worth having. The dependency is not yet earned.

## Deliberate omissions

- **No backtest engine.** We integrate established libraries. Building one is a
  multi-year project with no bearing on our differentiation.
- **No execution path.** Live order placement is the highest-consequence thing
  this project could do. It requires a validator we trust, risk gating, and
  per-action human approval. Not on the roadmap until those exist.
- **No vendored documentation.** See above.

<p align="center">
  <a href="https://github.com/fourstoats/openfade">
    <img width="403" height="72" alt="Openfade" src="https://github.com/user-attachments/assets/2faccbe5-54ba-4f0c-9eb8-a121dcd6572f" />
  </a>
</p>

<p align="center">
  <strong>Turn a trading idea into working, validated Pine Script and Lipi — then keep going.</strong><br />
  <em>Open source. Model-agnostic. Bring your own key.</em>
</p>

<p align="center">
  <a href="https://github.com/fourstoats/openfade"><img src="https://img.shields.io/github/stars/fourstoats/openfade?style=flat-square&logo=github" alt="GitHub stars"></a>
  <a href="https://github.com/fourstoats/openfade/issues"><img src="https://img.shields.io/github/issues/fourstoats/openfade?style=flat-square" alt="GitHub issues"></a>
  <a href="https://github.com/fourstoats/openfade/blob/main/LICENSE"><img src="https://img.shields.io/github/license/fourstoats/openfade?style=flat-square&color=D22128" alt="License"></a>
</p>

---

> **Pre-alpha. There is no working code yet.** This repository is the public
> build. The architecture is committed, the implementation is not started, and
> everything below is a commitment rather than a feature list. See
> [Project status](#project-status).

## What this is

Trading knowledge and trading software are split across two worlds. You know
*that* you want price above the 200 EMA with an RSI confirmation and a 2% stop.
You do not know that this means `ta.ema`, a series/int type-qualifier conflict,
and a `strategy.exit()` call that behaves differently depending on how the
`stop` parameter is typed.

Openfade is an agent system that closes that gap. You describe the strategy;
agents research it, write it, convert it between platforms, and prove it
actually runs.

**The first wedge is Pine Script and Lipi:**

| Capability | Status |
| --- | --- |
| Natural language → Pine Script | planned |
| Natural language → Lipi | planned |
| Pine Script → Lipi | planned |
| Lipi → Pine Script | planned |

Scripts are the entry point, not the destination. The longer arc is the full
lifecycle: **research → strategy → code → backtest → analysis → decision →
automation.** See [VISION.md](VISION.md).

## Yours, with no lock-in

Openfade is Apache 2.0 and model-agnostic by design, not as a feature.

- **Bring your own key.** OpenAI, Anthropic, Google, local models, or anything
  else that speaks a supported interface. There is no required provider.
- **Your prompts, your credentials, your data.** API keys stay in your
  environment. The documentation corpus is built locally on your machine from
  the upstream docs — we do not redistribute TradingView or GoCharting content.
- **Self-host the whole thing.** It runs on your hardware. No account required
  to use the open-source core.
- **No planned paid wall on the core.** If we ever ship hosted services, the
  open-source core stays open and self-hostable. We would rather tell you now
  than surprise you later.

## Why not just paste the docs into a chatbot?

Because it produces code that looks right and does not run.

Pine Script appears heavily in training data. Lipi almost does not. Ask a
general model to "convert this Pine script to Lipi" and it will confidently emit
Pine — `ta.sma()` where Lipi wants `talib.sma()`, `timeframe.isdaily` where
Lipi wants `interval.isdaily`, and function calls where Lipi needs types. The
output reads beautifully and is garbage.

The difference is a **symbol table** and a **validator**:

| | Generic chatbot | Openfade |
| --- | --- | --- |
| Knows Pine APIs | from training data | from an extracted, versioned corpus |
| Knows Lipi APIs | barely | from an extracted, versioned corpus |
| Handles Pine→Lipi | falls back to Pine | explicit translation table |
| Tells you it's wrong | no | refuses unknown symbols |
| Guarantees it runs | no | static validation before output |

That difference is the entire technical bet. Everything else — the agent
graph, the lifecycle, the ecosystem — is downstream of whether this core is
real. See [docs/architecture.md](docs/architecture.md).

## Quickstart

Not available yet. There is no `openfade` binary.

When the first vertical slice lands, this section becomes:

```bash
git clone https://github.com/fourstoats/openfade
cd openfade
docker compose up -d          # PostgreSQL + pgvector
pip install -e ".[dev]"
openfade init                 # build the local documentation corpus
openfade generate "price above 200 EMA, RSI crosses above 30, long with 2% stop"
```

We will publish this section the day it is true, and not before.

## Documentation

| Document | Read it when |
| --- | --- |
| [VISION.md](VISION.md) | You want to know where this is going and why |
| [docs/architecture.md](docs/architecture.md) | You want to understand the system design |
| [docs/comparison.md](docs/comparison.md) | You are deciding whether to use this or something else |
| [docs/roadmap.md](docs/roadmap.md) | You want to know what is being built next |
| [AGENTS.md](AGENTS.md) | You are an AI agent working in this repository |
| [CONTRIBUTING.md](CONTRIBUTING.md) | You want to contribute code or docs |
| [SECURITY.md](SECURITY.md) | You found a security issue |

## Project status

**We are early.** Being explicit about this is more useful than a roadmap full
of checkmarks we have not earned.

- [x] Project vision and scope defined
- [x] Architecture designed
- [x] Public documentation written
- [ ] Core dialect and corpus implementation
- [ ] Pine/Lipi code generation
- [ ] Static validator and repair loop
- [ ] Conversions between Pine and Lipi
- [ ] Backtesting
- [ ] First public release

Contributions are welcome now, especially on the corpus and validator. Those
are the two components everything else depends on, and they are the two where
domain knowledge beats raw engineering effort.

## Contributing

Read [CONTRIBUTING.md](CONTRIBUTING.md). The short version: the corpus and the
validator are the priority, and a pull request that makes either of them more
correct is worth more than one that adds a feature.

## License

Apache 2.0. See [LICENSE](LICENSE).

Pine Script and TradingView are trademarks of TradingView, Inc. GoCharting and
Lipi are products of GoCharting. Openfade is an independent project and is not
affiliated with or endorsed by either. See
[docs/comparison.md](docs/comparison.md#trademarks-and-affiliation).

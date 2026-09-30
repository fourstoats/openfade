---
summary: "Index of the Openfade documentation, with what each document is for and when to read it."
read_when:
  - You have arrived here from a search and do not know where to start
  - You want to know what documentation exists
  - You are looking for a specific topic
title: "Documentation"
---

# Documentation

Openfade is pre-alpha. There is no working code yet, and the documentation here
describes intent and design rather than current capability. We publish it in
advance because it is more useful to argue with a design than to discover it
after the code exists.

## Start here

| Document | Read it when |
| --- | --- |
| [README](../README.md) | You have just arrived and want the short version |
| [Vision](../VISION.md) | You want to know where this is going and why |
| [Roadmap](roadmap.md) | You want to know what is being built next |

## Design

| Document | Read it when |
| --- | --- |
| [Architecture](architecture.md) | You want to understand how the system is designed |
| [Comparison](comparison.md) | You are deciding whether to use this or something else |

## Working on Openfade

| Document | Read it when |
| --- | --- |
| [Contributing](../CONTRIBUTING.md) | You want to contribute code, docs, or trading knowledge |
| [AGENTS.md](../AGENTS.md) | You are an AI agent about to modify this repository |
| [Security](../SECURITY.md) | You found a security issue |
| [Code of Conduct](../CODE_OF_CONDUCT.md) | You are participating in the community |

## The short version of the whole thing

Openfade is an open-source AI agent platform for trading. You describe a
strategy in plain language; agents research it, write it as Pine Script or Lipi,
convert it between those two languages, and validate that it actually runs.

The first four capabilities are:

1. Natural language → Pine Script
2. Natural language → Lipi
3. Pine Script → Lipi
4. Lipi → Pine Script

The reason that is hard — and the reason a generic chatbot does not solve it —
is that Pine Script is heavily represented in model training data and Lipi is
barely represented at all. A general model asked to convert between them emits
Pine with a Lipi-shaped wrapper: fluent, confident, and wrong.

The technical bet is that this is a **retrieval and validation** problem rather
than a prompting problem. A structured symbol corpus, an explicit dialect layer,
and a validator that refuses unknown symbols are the whole game. Everything else
is downstream.

## Conventions in this documentation

Every page opens with `summary` and `read_when` frontmatter. `read_when` lists
**situations** that should send you to the page, not topics it covers. If you are
an agent, that list is the routing index.

Two rules are enforced across all documentation:

- **Existing and planned are always distinguished.** We do not describe
  unbuilt features in the present tense.
- **Claims about other projects are sourced or omitted.** See
  [Comparison](comparison.md), which compares approaches rather than naming and
  ranking competitors.

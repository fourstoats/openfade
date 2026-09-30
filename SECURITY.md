---
summary: "How to report a security issue in Openfade, what counts as a vulnerability, and what we do with reports."
read_when:
  - You found a security issue and want to report it privately
  - You want to know whether something counts as a vulnerability
  - You are reviewing how Openfade handles API keys or user data
  - You want to understand what Openfade will never do automatically
title: "Security Policy"
---

# Security Policy

Openfade is pre-alpha. There is no released version, and the repository contains
documentation only. That context matters for what follows: we would rather set
clear expectations now than have someone discover the boundary later.

## Reporting a vulnerability

**Do not open a public issue for a security problem.**

Report privately through GitHub's private vulnerability reporting for this
repository: **Security** → **Report a vulnerability**. That opens a private
advisory visible only to you and the maintainers.

Please do not use a public issue, a pull request, a discussion, or any public
channel.

<!--
TODO(owner): if a monitored security mailbox is added, list it here as a
secondary channel. Do not add an address that nobody reads – a dead contact in
a security policy is worse than no contact at all, because a researcher who
trusts it will wait.
-->

Include:

- What the issue is, and what an attacker gains
- Steps to reproduce, or a proof of concept
- The version, commit, or branch you tested
- Whether any user data or credentials are exposed

You should get an acknowledgement within 72 hours. We will confirm receipt, tell
you what we think the severity is, and keep you updated as we work on it. We will
not ask you to keep the issue secret indefinitely – we will tell you when we
intend to disclose, and we will credit you in the advisory unless you would
rather we did not.

## What counts as a vulnerability

In scope:

- **Credential exposure.** Any path where an API key, token, or secret can leak
  into logs, telemetry, generated output, error messages, or a committed file.
- **Corpus integrity tampering.** Any path where the locally-built documentation
  corpus can be silently poisoned, such that a wrong signature or a hostile
  built-in reaches the user as if it were verified.
- **Validator bypass.** Any input that causes the validator to pass a script
  containing an undefined symbol, a type error, or a construct the target
  language does not support.
- **Command injection.** Any path where user-controlled input reaches a shell,
  filesystem path, or subprocess without sanitisation. The corpus builder
  fetches from the network, so this is a realistic surface.
- **Path traversal.** Any path where a project name, script name, or corpus
  identifier can escape its intended directory.
- **Sandbox escape.** We intend to run generated and retrieved code in a
  constrained process. Any escape from that constraint is in scope.
- **Unauthorised outbound requests.** Any network call the user did not
  knowingly enable, particularly telemetry.

Out of scope:

- **Generated strategies that lose money.** Openfade produces research and
  analysis tooling. It does not provide investment advice and makes no
  guarantee about trading outcomes. A strategy that generates and validates
  correctly but performs badly is not a vulnerability.
- **Incomplete features or missing functionality.** Report those as bugs.
- **Weaknesses in upstream model providers** or in Pine/Lipi themselves. Report
  those to their maintainers.
- **Lack of rate limiting on a self-hosted instance** you control.
- **Social engineering of a user into approving a trade.** Openfade requires
  explicit human approval for anything touching a live account. If a user is
  deceived into approving something, that is not a vulnerability in Openfade –
  report it anyway if the interface made it easier than it should have been.

## Hard boundaries

These are design commitments, not configuration options. There is no flag,
environment variable, or API parameter that disables them.

- **Openfade does not place, modify, or cancel live orders.** Any future
  execution capability requires an explicit, per-action human approval step.
  An agent may prepare an order; it may not submit one.
- **Your API keys stay in your environment.** They are read from your
  environment or your key manager, used for the request, and not persisted to
  disk, logs, telemetry, or generated artifacts.
- **The documentation corpus is built locally.** Fetching upstream docs is a
  documented, user-initiated action. Openfade does not proxy vendor
  documentation through an Openfade-operated service, and it ships no copy of
  TradingView or GoCharting documentation.
- **No hidden telemetry.** There is no analytics collection. If that changes it
  will be opt-in, documented on a dedicated page, and disclosed in the README
  and this file – not buried in a changelog.

## Supply chain

We take dependencies seriously because this project runs code on contributor and
user machines.

- Every new dependency is proposed in an issue with its license and
  maintenance status before it is added.
- Permissive licenses only in the core. Copyleft and source-available
  dependencies require explicit discussion, and may be isolated or rejected.
- Generated output is treated as untrusted. It is executed only in a constrained
  process, never with the privileges of the Openfade process itself.

## Third-party reports

We are a small team and cannot triage a high volume of reports quickly. If you
are a researcher and want to do responsible disclosure, open an advisory and
tell us your timeline – we will work within it.

## Supported versions

There are no released versions yet. Security fixes land on `main`. Once we cut a
first release this section will list which versions receive patches.

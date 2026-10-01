# Changelog

All notable changes to this package are recorded here.

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and the package follows
[Semantic Versioning](https://semver.org/spec/v2.0.0.html).

**Check the date before you rely on a copy.** Security guidance ages. Algorithms get deprecated,
provider defaults change, a header value that was correct becomes obsolete, and a recommended tool
picks up a CVE. Compare the date on the release you are running against the latest entry below.

Version numbers mean this: **MAJOR** when a decision changes in a way that invalidates a plan already
written with an earlier release, **MINOR** when a layer, a control, or an acceptance check is added,
**PATCH** for wording and formatting that leaves every decision as it was.

**Releasing:** every release bumps four places that must agree, or the freshness signal becomes a
lie: the `version` field in `SKILL.md`, the entry below, and the version and date shown at the top of
`README.md` and `README.pt-BR.md`.

---

## [1.2.0] - 2026-09-18

The infrastructure and edge layers now go one level below the controls they already named. For each
defense a self-hosted system relies on under a denial of service attack, the package states where
that defense passes its check and still fails in practice, and how to measure it for real.

### Added

- Layer 5, a MUST: container port publishing is checked against the firewall. A runtime-published
  port bypasses the host firewall front-end, so only the proxy publishes, everything else stays on
  the internal container network or on loopback, restrictions go in the chain the runtime leaves
  alone, and the provider's network firewall is attached when the instance is created.
- Layer 5, SHOULD controls for the rest of the self-hosted stack: pinned infrastructure images
  watched by an update bot that proposes and never applies, and the places automatic updates hide;
  the origin bound to your own edge zone with a tunnel or with mutual TLS on a zone certificate,
  since the edge's address ranges are shared by all its customers; the origin address kept from
  leaking again through the default certificate, outbound requests to user-supplied destinations,
  unproxied records, and direct mail; network kernel settings measured inside the proxy's network
  namespace, plus SYN-ACK retries, reverse path filtering, and the connection tracking timeout;
  descriptor limits read from the running proxy, the proxy's own connection ceiling, and ephemeral
  ports; slow-rate attacks bounded by timeouts and a per-client connection cap keyed on the real
  client address; memory, processor, and process-count limits per container with rotated logs; and
  an alert path that does not live on the monitored host.
- Layer 6, SHOULD controls: a challenge mode scoped away from non-browser traffic and rehearsed
  before it is needed, with a narrowly scoped trigger token and a condition for switching it off;
  the cache gap closed against random query strings; and a written denial of service runbook kept
  with the private material.
- Item 15 of the Tier 0 checklist in `references/threat-model.md`: container-published ports.
- Two rows in the negative control table of `references/verification.md`, for port exposure on a
  self-hosted origin and for off-host alerting, and three origin probes for self-hosted systems
  behind an edge network.
- New rows in `references/live-surfaces.md`: challenge mode exemptions, origin binding, record proxy
  status, and attack notifications at the edge; the outside port scan against runtime-published
  ports, the provider firewall at creation, the provider's policy for an address under attack, and
  automatic update settings on panels and updaters at the VPS provider.
- A red flag in `SKILL.md`: calling a port closed, a kernel setting applied, or a limit raised on
  the word of the host itself.

### Changed

- The acceptance checks for layers 5 and 6 include the outside port scan over IPv4 and IPv6,
  reading limits and kernel settings from the running proxy, a deliberate heartbeat stop, and a
  rehearsal of the challenge mode against the webhook and API inventory.
- `templates/pre-launch-checklist.md`: Tier 1 gains the outside port scan for self-hosted profiles.
  Tier 2 gains the verified alert path, origin binding, proxy-side limits and timeouts, container
  limits and log rotation, watched pinned images, and the challenge mode preparation.
- `references/stack-profiles.md`, Profile B: the host's own report is not evidence, the edge
  allowlist admits every customer of the edge, and the emergency switch is prepared in advance.

---

## [1.1.0] - 2026-08-17

The package now ships the harness it is tested with, and the method now requires
that an acceptance check be seen failing before it counts as passing.

### Added

- `evals/`, the harness this package is tested with. Fourteen cases, one per layer, each a pair: a
  file carrying a single planted defect and the same file with that defect repaired. A case passes
  only when the defect is reported on the first and left alone on the second, so a check that fires
  on everything fails the suite instead of passing it.
- `evals/run.py`, which scores those pairs through the `claude` command line tool, with three arms:
  the skill loaded from the working copy, the skill asked for by name to confirm an installed copy
  loads, and no skill at all to measure what it adds. Also `--self-test`, which proves the scorer
  itself can fail without calling a model, and `--dry-run`, which checks the cases are well formed.
- `evals/validate.py`, integrity checks that need no model and no network: the release stamp agrees
  in all four places, the two READMEs stay mirrors, no document points at a path that does not
  exist, every layer playbook ends in an acceptance check, every layer has a case, and no case has
  variants that are secretly identical.
- The negative control in `references/verification.md`: an acceptance check counts as passing only
  after the control it watches has been broken on purpose once and the check was seen going red.
  Includes the six checks that most often pass while pointed at the wrong thing, and how to break
  each one. Rule 5 in `SKILL.md` now carries the same obligation.
- `references/change-review.md`, Mode C on a diff: what a diff hides, which layers each class of
  change actually reaches, the five questions a finding clears before it is written down, the
  categories that are never reported, and the merge gate the review produces instead of a list.
- Dependency triage in layer 10, so a scanner queue becomes an ordered plan: confirmed exploitation
  in the wild first, then reachability in production, then likelihood of the attempt, then whether a
  fix can actually be taken. Suppressions now expire.

### Changed

- The acceptance check for layer 10 asks for the triaged queue rather than a severity count.
- Step 5 of the workflow now states the precision bar for reported findings.

---

## [1.0.0] - 2026-08-17

Initial release. The package is the skill file, eight reference documents, and three templates read
by a coding agent, plus an `assets/` directory of reference artifacts meant to be copied into a
project: a probe script, a SQL policy file, a CI workflow, a header configuration reference, and a
cross-tenant test file. The instructions run nothing on their own; the artifacts are code and are
read before use.

Scope is design-time and build-time security engineering, for the moment when a control is still a
configuration change. It performs no conformance assessment and produces no certification.

### Added

- `SKILL.md`, the operating instructions: the thirteen core layers that each require a written
  decision plus a conditional fourteenth for model and agent features, the five application defaults
  checked on every feature, the four-rank friction scale that picks the cheapest control that closes
  the risk, the discovery pass that reads the project before advising, the three entry modes (new
  project, project underway, single feature or pull request), the consolidated access request for
  administrative surfaces the session cannot reach, and the blocking pre-launch gate.
- `references/threat-model.md`: the automated opportunist and the motivated adversary, and what each
  one implies at design time.
- `references/layer-playbooks.md`: decisions, controls, acceptance checks, and friction rank for all
  thirteen core layers (frontend, backend, data, identity, infrastructure, edge, observability,
  pipeline, secrets, dependencies, public exposure, privacy, payments).
- `references/ai-surface.md`: the conditional layer for systems that ship a model or agent feature,
  covering prompt injection, tool authorization, retrieval isolation, and cost limits.
- `references/app-defaults.md`: the five defaults, each with its secure pattern and the insecure twin
  it is usually confused with, plus the quota question that turns a denial of service into an invoice.
- `references/stack-profiles.md`: what changes across serverless, self-hosted, and local-only
  deployments.
- `references/verification.md`: how to prove a control holds, and the scanner toolkit.
- `references/live-surfaces.md`: how to verify and change the real configuration of the providers a
  system runs on, with the tool preference order (connected MCP server, provider CLI, authenticated
  browser, ask the user), the rule that the account, organization and project are confirmed before
  the first call, and the per-surface checks for source control, managed database and backend
  platforms, hosting and edge, identity providers, observability, payments, registrar and DNS, cloud
  accounts, and self-hosted servers.
- `references/operating-discipline.md`: language, consent before changes, disclosure limits, and what
  stays private to the team.
- `templates/security-plan.md`: the deliverable, with a slot for every layer decision.
- `templates/pre-launch-checklist.md`: the blocking gate covering what an automated attacker tries
  first.
- `templates/threat-model.md`: a one-page model.
- `assets/`, copyable artifacts with a `README.md` that lists the placeholders each one needs:
  `rls-multitenant.sql` (PostgreSQL tenant isolation, row level security enabled and forced, one
  policy per command, composite foreign keys), `security-headers.md` (header values, nginx and Node
  configuration, the report-only rollout path for `Content-Security-Policy`), `ci-security.yml` (a
  GitHub Actions workflow with secret scanning, dependency audit, static analysis, and actions
  pinned by commit SHA), `probe.sh` (an external pre-launch probe against a host you name), and
  `tenancy.test.example.ts` (the cross-tenant denial suite run as an unprivileged client).
- `README.md` and `README.pt-BR.md`, the same overview in English and in Brazilian Portuguese, with
  `docs/img/cover.png`, the cover image both of them display.
- `SECURITY.md` (what is in scope and how to report it), `.gitignore`, and `LICENSE` (MIT).
- Mapping onto the OWASP Top 10 and the OWASP API Security Top 10, fetched at run time so the
  category identifiers match the current revision, with the revision and year named in the output.

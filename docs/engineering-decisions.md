# Engineering decisions

A catalogue of decisions across my projects, with the alternatives I rejected and the cost I accepted. Decisions without a stated cost are usually not decisions.

[← Portfolio home](../README.md)

---

## Platform and infrastructure

| Decision | Alternatives | Why | Cost I accepted |
|---|---|---|---|
| **Serverless edge (Cloudflare Workers) for everything** | VPS, PaaS, containers | For a solo operator, the patching, supervision and uptime surface of a server is a liability. Compute and database sit together at the edge. Pricing scales with requests, not idle time. | Platform-specific model: the whole Worker is replaced on deploy; limits on long-running work and some Node APIs. |
| **SQLite at the edge (D1) as the only database** | Hosted Postgres, key-value store | One store, one query language, co-located with compute, transactional enough for a credit ledger. | Single-writer semantics and size limits; I design writes as single conditional statements. |
| **No framework on the client** | React, Vue | No build step between source and what runs; small surface; debuggable in a browser. | Hand-rolled state handling; discipline needed in large single files. |
| **Self-host vendor libraries and fonts** | CDN delivery | A CDN script once broke a form. A manifest with a risk rating per dependency replaced trust with inventory. | Manual updates; cache versioning rules. |
| **Git-only deploys, CI does the shipping** | Manual `wrangler deploy` | Manual deploys ship code git does not contain and are silently overwritten by the next CI run. | Slower hot-fixes; I accept that, and preview locally. |
| **Separate account, repo and credentials where the business differs** | One shared account | A mistake in one project cannot touch another. | More setup. |

## Identity and security

| Decision | Alternatives | Why | Cost |
|---|---|---|---|
| **No passwords for end users** (email link sign-in) | Password auth | The best password database is the one that does not exist. | Dependence on email deliverability, which I monitor with delivery-status webhooks. |
| **OTP and short-lived sessions for the private reading room** | Shared links | Per-person, per-document, revocable, expiring, logged. | More code than a link. |
| **Private object storage, streamed through the Worker** | Signed public URLs | No URL to a confidential file ever exists to leak. | Bandwidth passes through the Worker. |
| **Permission levels enforced in code, not in prompts** (AI control plane) | "Instruct the model to be careful" | A model can be talked into anything; a registry check cannot. Prompt injection becomes a permissions problem. | Every new tool needs a declared level and tests. |
| **Uploaded files are untrusted input, always** | Treat attachments as context | A file may inform a model but never instruct it or widen its permissions. | Some legitimate "do what this document says" use cases are intentionally impossible. |

## Data and money

| Decision | Alternatives | Why | Cost |
|---|---|---|---|
| **Credits as an append-only ledger** | Balance column | Auditable, reconcilable, explainable to a customer. | Reads aggregate; fine at this scale. |
| **Idempotency enforced by a unique index** | Application-level "already processed?" check | The database is the only place a race cannot hide. | Migrations must be correct the first time. |
| **Minor-unit, currency-aware money** | Floats | Floating-point money is a bug waiting for an invoice. | None worth mentioning. |
| **Merchant of record for payments** | Direct processor | Tax and VAT across jurisdictions become someone else's job. | Fees, and less control over checkout. |
| **Retention limits built in** (raw input six months, telemetry one year, finance never) | Keep everything | Keep what the business needs and the law expects; drop the rest. | Some analyses are impossible after the window. |
| **Never delete financial records, even with the user** | Cascade delete | Deleting a person must not erase the books. | Data-protection flows need to detach, not cascade. |

## AI integration

| Decision | Alternatives | Why | Cost |
|---|---|---|---|
| **Versioned method and prompt, stored as data, with provenance on every result** | Prompt in code | Any past result is traceable to what produced it; changes are reviewed and reversible. | An admin surface to maintain. |
| **Typed error taxonomy with automatic refunds** | Generic "something went wrong" | Support becomes a lookup. Users are never charged for our failures. | Code paths for every failure class. |
| **Log the billed cost of failed calls** | Log successes only | A failed call still costs money. Invisible cost was a real blind spot. | Slightly more logging. |
| **Deterministic, rule-based routing between models** | LLM-as-router | A router that spends tokens to decide whether to spend tokens defeats itself. | Less nuanced than a model would be; escalation covers misclassification. |
| **Hard monthly cap with owner alert** | Dashboard only | A runaway loop should hit a wall. | A feature can stop mid-month; that is the point. |
| **Model ids and prices in exactly one file** | Scattered constants | Swapping a model is a one-file, tested change. | None. |
| **Pre-registered, blind evaluation before any model switch** | Try it and see | Thresholds fixed before results exist cannot be bent to fit them. | Slower decisions. |
| **Build vs. adopt: trial the alternative locally, isolated, on my own scenarios** | Argue from feature lists | Evidence beat opinion, and it produced a principle (one home for stateful data). | A few hours of someone else's software in a sandbox. |

## Process

| Decision | Alternatives | Why | Cost |
|---|---|---|---|
| **Decision records keep the losing alternatives** | Record only conclusions | The reasoning is what a future reader needs. | Writing time. |
| **Status pages separate "tests pass" from "observed live"** | One green tick | They are different facts. | Discipline. |
| **A "what not to build" list beside every roadmap** | Roadmap only | Refusals are decisions too. | Sometimes uncomfortable. |
| **Claims policy on research** (freeze attacks, publish failures) | Report what worked | A result that survives only under friendly conditions is not a result. | Smaller headlines. |
| **Kill experiments on evidence** | Keep iterating | Sunk cost should not vote. | Occasionally stopping one step before a lucky breakthrough. |

[← Portfolio home](../README.md) · [How I work](how-i-work.md)

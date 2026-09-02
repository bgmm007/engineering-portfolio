# SOUND Method: Building an AI-Powered Evaluation SaaS on Cloudflare's Edge

**Stack:** Cloudflare Workers · D1 · R2 · Cloudflare Access · Anthropic Claude API · Paddle · Vanilla JS  
**Status:** v0.9.0-beta · Live at [lumensound.org/sound](https://lumensound.org/sound/)  
**Author:** Bruno Miranda · [brunomiranda.net](https://brunomiranda.net)

---

## Overview

SOUND Method is a domain-specific AI evaluation SaaS for analyzing congregational worship music. Users submit a song title, lyrics, and contextual metadata; the AI engine applies a structured analytical framework to return a multi-dimensional evaluation with pastoral and theological context.

This document describes the technical decisions behind the system: why I chose the stack I chose, how the AI integration is architected, and what the engineering trade-offs looked like at each layer.

This is a solo-built production application. Every infrastructure decision, UX decision, and deployment decision was made by one person. That constraint shaped the architecture more than anything else.

---

## Why Cloudflare Workers?

The first decision was the runtime. The alternatives were a traditional VPS (DigitalOcean, Linode), a PaaS (Railway, Render, Heroku), or Cloudflare Workers.

I chose Workers for four reasons that compound on each other:

**1. No server management.** A VPS requires OS maintenance, security patching, process supervision, and uptime monitoring. For a solo-operated SaaS, that operational surface is a liability. Workers eliminates it entirely. The Worker runtime is Cloudflare's problem.

**2. D1 co-location.** Cloudflare D1 (SQLite at the edge) runs in the same datacenter as the Worker that queries it. There is no network round-trip from compute to database — they share physical proximity. This matters for an evaluation workflow where each request involves multiple sequential reads and writes.

**3. Cloudflare Access integrates natively.** Zero-trust authentication at the infrastructure layer with no custom auth code. More on this in the Authentication section.

**4. Cost structure.** For a SaaS at early scale, Workers pricing is dramatically more predictable than running a VPS 24/7 that handles traffic intermittently. The free tier covers substantial traffic; the paid tier scales per-request rather than per-month-of-idle-compute.

The trade-off: Workers has a specific deployment model (the entire worker is replaced on every deploy) and meaningful constraints on long-running operations and certain Node.js APIs. Understanding and working within these constraints is not optional — it's the job.

---

## Data Layer: Cloudflare D1

D1 is SQLite running at the edge. The database schema covers:

- **Evaluations table** — stores submitted song data, AI-generated evaluation output, session metadata, and timestamps
- **Credits table** — tracks per-user evaluation credit balance
- **Events table** — application-level error and activity logging (see Error Taxonomy section)

D1 requires careful attention to the `--remote` flag when running wrangler queries. Local execution targets a local SQLite file; `--remote` targets the actual deployed database. This is not obvious from the tooling and is the kind of constraint that breaks things quietly if not documented.

All D1 operations are executed from the `sound-method-worker/` directory. This is a project rule, not a suggestion. It lives in `CLAUDE.md`.

---

## Authentication: Cloudflare Access

Authentication is handled entirely by Cloudflare Access — a zero-trust identity layer that sits in front of the Worker. Users authenticate via email-based magic link (no passwords).

The architectural decision here was deliberate: **I did not write authentication code.** This is not laziness — it is risk management. Custom session management, JWT handling, and password storage are each a surface for security failure. Cloudflare Access eliminates all of them by making identity an infrastructure concern rather than an application concern.

The Privacy Policy reflects this accurately: the application does not collect passwords, does not store session tokens, and does not manage credential resets. All of that lives at the Access layer, outside the application's scope.

---

## AI Integration: A Constrained Evaluation Engine

This is the architectural center of the application. The integration with the Anthropic Claude API is not a naive "send the user's input and display the output" pattern. The engine is explicitly constrained at multiple levels.

### System Prompt Architecture (v3.1)

The system prompt is the primary governance mechanism. It does several things:

1. **Grounds the engine in domain-specific vocabulary.** The AI does not use general synthesis language. It uses the vocabulary of the book and framework this tool is built on. A vocabulary prohibition list (`NEVER USE`) enforces this explicitly.

2. **Defines delivery sequence.** The engine follows an eight-point analytical delivery framework: diagnosis before prescription, mechanism before symptom, largest frame before smallest evidence, pastoral consequence weighted alongside theological accuracy. The engine does not open with a verdict. It opens with a frame.

3. **Prohibits fabrication.** The system prompt explicitly prohibits the engine from making historical claims it cannot verify. An early failure mode — the engine fabricating a historical narrative about a well-known song — was traced to the absence of this constraint. Adding it eliminated the failure class.

4. **Separates evaluation from performance.** A recurring failure in early versions was the engine suggesting performance interventions (tempo, arrangement, instrumentation) when the user had submitted lyrics for theological evaluation. The system prompt explicitly prohibits this category of response.

### Error Architecture: Typed Taxonomy and Event Logging

Every application error is typed. The taxonomy runs from E001 to E008, covering authentication errors, credit failures, AI engine errors, database errors, and export failures.

Each error event is logged to D1 with a timestamp, error code, session context, and where applicable, the user-facing message. An admin endpoint surfaces recent errors by code, enabling pattern detection without having to read raw logs.

This design means that when something breaks in production — and things break in production — the diagnostic path is structured rather than exploratory.

### Why `claude-sonnet-4-6`?

The current AI model is `claude-sonnet-4-6`. The selection was based on evaluation of output quality for this specific task domain at a cost structure appropriate for a per-credit SaaS. One earlier architecture decision — attempting to use assistant prefill — was the root cause of a persistent E006 error. That feature is not supported in the claude-sonnet-4-6 API. Removing it resolved the error class entirely. Understanding API constraints at this level is part of the integration work.

---

## UX and HCI Decisions

Each UX decision in the application was deliberate. The ones worth documenting:

**Score dots (1–5 visual indicator).** Numeric scores without visual anchoring are hard to parse at a glance, especially in a mobile context. The dot representation communicates relative weight immediately, without requiring the user to read and interpret a number.

**Inline delete with confirm + toast.** Destructive actions need friction. But full-screen modal dialogs interrupt the workflow more than the action warrants. The pattern used here — inline confirmation text that replaces the delete button, followed by a one-click toast on success — provides the friction without the interruption.

**Auto-scroll to results on mobile.** In early testing, mobile users submitted evaluations and then did not know the results had appeared below the fold. The fix is structural: after evaluation completes, the view scrolls to the results section automatically. This is a one-line change with significant UX impact.

**120-character title limit.** Unbounded text fields create edge cases in PDF export: titles that overflow the layout, break file naming conventions, or render incorrectly. The limit is enforced at the input level, not the output level.

**Session status bar (email + credit count).** SaaS users need to see their account state without navigating to a settings page. The status bar surfaces both identity (confirming who is logged in) and credits (confirming what remains) on every page of the evaluation workflow.

**Language toggle (EN/PT-BR) with localStorage persistence.** The preference persists across sessions without requiring account-level storage. PT-BR content is deferred to a future release; the toggle infrastructure is in place.

---

## Payments: Paddle

Paddle operates as the merchant of record. This is a meaningful distinction from pure payment processors: Paddle handles VAT and tax compliance across jurisdictions, which matters for a SaaS with international users.

The application is currently in sandbox mode. The transition to live requires no architectural changes — only Paddle credentials and verification.

---

## Deployment Pipeline and Documentation Protocol

Cloudflare Workers replaces ALL assets on every deploy. This is not additive — deploying a single changed file without deploying the full worker will result in missing assets. The deploy protocol is therefore: ZIP the entire site folder, upload the complete package. Surgical single-file uploads do not work and will silently break things.

This constraint lives permanently in `CLAUDE.md`. That file is the AI collaboration protocol for the repo — it documents constraints, deployment rules, and architectural decisions that are not obvious from the code itself. Every session with an AI assistant begins by reading `CLAUDE.md`.

`CHANGELOG.md` records every deploy: date, version, changes made, and what was tested. This is not optional record-keeping. It is the primary recovery mechanism when a deploy introduces a regression.

One principle governs all changes: **minimum viable modification per build.** CSS changes to shared stylesheets cascade across all pages. One unintended change can break multiple surfaces simultaneously. Containing the scope of each deploy to the minimum required change is a discipline, not a constraint.

---

## Legal and Compliance Engineering

The Privacy Policy is accurate. It states explicitly what IS and IS NOT collected. An earlier version contained a statement implying the application does not collect passwords or names — accurate — but also contained a satisfaction guarantee that was legally incorrect and was removed.

All contact is consolidated to a single address (`support@lumensound.org`). No email addresses appear on any public-facing page. Contact is handled through form endpoints using vanilla JS fetch, replacing a previously CDN-dependent third-party handler that had become a reliability failure point.

These are not just legal boxes. They are architectural decisions about how the system represents itself to the people who use it.

---

## Reflections

Building and operating a production SaaS alone is an exercise in prioritization. You cannot build everything at once and you cannot break anything in production. Those two constraints create a particular kind of discipline: you learn to make the minimum change, document what you did, and verify before moving to the next thing.

The most useful engineering habit I developed through this project is the one I now call **diagnosis before prescription**: before proposing a fix, name exactly what is broken and why. Most errors in complex systems are not what they appear to be. The E006 error wasn't a network error. It was an API feature assumption. The contact form breakage wasn't a code error. It was a CDN dependency. The cascade on the stylesheet wasn't a CSS error. It was a deployment scope error.

The right question is never *"what do I change?"* It is *"what exactly is wrong, and why?"*

---

*Bruno Miranda is an AI product engineer and independent researcher based in Florida. His work combines edge computing infrastructure, LLM integration, and domain-specific UX for purpose-built professional tools.*

*Contact: [brunomiranda.net/contact](https://brunomiranda.net/contact)*

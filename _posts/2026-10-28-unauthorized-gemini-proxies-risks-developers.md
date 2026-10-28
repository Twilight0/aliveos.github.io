---
permalink: /news/unauthorized-gemini-proxies-risks-developers/
layout: post
title: "The Illusion of 'Free' APIs: Why Developers Must Avoid Unauthorized Gemini Web Proxies"
date: 2026-10-28 21:05:00 +0200
author: The AliveOS Project
category: news
type: news
image: /assets/gemini-proxy-illusion-banner.jpg
tags:
  - Artificial Intelligence
  - Security
  - Gemini
  - API
  - Open Source
  - Web Security
  - Reverse Engineering
---

![The Illusion of Free APIs: Unauthorized Gemini Web Proxies Banner](/assets/gemini-proxy-illusion-banner.jpg)

In the fast-moving open-source AI ecosystem, a recurring pattern has emerged: developers attempting to bypass paid API billing or regional quotas by reverse-engineering consumer web interfaces. 

Recently, three open-source repositories designed to extract "free" programmatic access from Google Gemini have gained traction on GitHub:

1. **[`kodelyx/free-gemini-api`](https://github.com/kodelyx/free-gemini-api)**: A Go-based reverse-proxy that intercepts Gemini web endpoints with HTTP/3 QUIC support, multi-account pooling, and deep-thinking options, exposing an OpenAI-compatible `/v1/chat/completions` API.
2. **[`KashifKhn/gemini-proxy`](https://github.com/KashifKhn/gemini-proxy)**: A lightweight TypeScript/Bun proxy that piggybacks on OAuth 2.0 + PKCE credentials designed for official CLI tools to serve OpenAI-compatible endpoints without an API key.
3. **[`zexadev/gemini-web2api-go`](https://github.com/zexadev/gemini-web2api-go)**: A Go service that reverse-engineers `gemini.google.com` by consuming raw browser cookies (`__Secure-1PSID`, `__Secure-1PSIDTS`), spoofing Chrome TLS fingerprints to bypass bot detection, and exposing conversational endpoints.

While the engineering ingenuity behind these reverse-proxies is technically notable, **building software or developer workflows on top of them is a dangerous anti-pattern**. 

Here is an in-depth technical analysis of how these projects operate under the hood, followed by an architectural and security breakdown of why developers must steer clear of them.

---

### Technical Anatomy: How These Exploits Function Under the Hood

From a reverse-engineering and security perspective, each of these three projects employs a distinct, highly sophisticated vector to bypass Google's access barriers:

#### 1. `kodelyx/free-gemini-api`: HTTP/3 QUIC Simulation & Account Pool Orchestration
Written in Go, this project tackles the high-concurrency bottleneck by implementing an **active credential pooling engine**. Rather than relying on a single authenticated session, it rotates across an array of user accounts, distributing incoming requests in a round-robin schedule to remain beneath per-account burst thresholds. 
- **Transport-Layer Cloaking**: The proxy leverages Go's low-level networking primitives to negotiate **HTTP/3 over QUIC**, matching the modern transport behavior of Chromium browsers to evade edge heuristics that flag anomalous plain HTTP/1.1 automated callers.
- **Protocol Emulation & Thought Budgeting**: It unpacks Google's internal web payload schemas to expose "Deep Thinking" controls (high, medium, low reasoning budgets) by manually injecting proprietary client-side deliberation parameters that the official web UI toggles dynamically in frontend state. It then translates Google's chunked server-sent event (SSE) stream back into OpenAI-standard delta frames.

#### 2. `KashifKhn/gemini-proxy`: OAuth 2.0 PKCE Hijacking of Official Client Identifiers
Unlike browser-scraping proxies that rely on ephemeral cookies, `gemini-proxy` utilizes a clever **OAuth credential misappropriation vector**. Built on Bun and Hono, it avoids browser automation altogether:
- **Client Identity Spoofing**: It utilizes public OAuth 2.0 Client IDs and Client Secrets extracted from official Google command-line tools (such as the Gemini CLI).
- **PKCE Token Interception**: It initiates an authorization flow using Proof Key for Code Exchange (PKCE) against Google's authentication servers (`accounts.google.com/o/oauth2/v2/auth`), intercepting the authorization code via a local loopback handler.
- **Unmetered Consumer Scopes**: By obtaining bearer tokens under the guise of an interactive developer CLI session, it directly interfaces with Google's backend API endpoints without triggering Google Cloud billing accounts. The proxy acts as a translation gateway, restructuring incoming OpenAI JSON schemas (`messages`, `temperature`, `max_tokens`) into Gemini's native `contents` and `generationConfig` protobuf-over-JSON representation.

#### 3. `zexadev/gemini-web2api-go`: JA3 TLS Fingerprinting & `batchexecute` Reverse-Engineering
`gemini-web2api-go` represents the most aggressive form of web reverse-engineering among the three, systematically dismantling the security architecture of `gemini.google.com`:
- **JA3/JA4 TLS Fingerprint Camouflage**: Automated Go HTTP clients are trivially detected by Google's anti-bot infrastructure due to default Go cipher suite ordering and TLS extension signatures. This project integrates custom TLS client engines to mirror Google Chrome’s exact client hello signatures, GREASE values, supported curves, and HTTP/2 pseudo-header sequencing (`:method`, `:authority`, `:scheme`, `:path`).
- **Google `batchexecute` Envelope Decoding**: The Gemini web app communicates through Google's legacy RPC multiplexer known as `batchexecute`. Requests are bundled into an obscure format (`f.req=[[[rpc_id, "[nested_json_string]", null, "generic"]]]`) protected by dynamic CSRF tokens (`at` parameters, `f.sid` session identifiers, and SNlM0e hashes). The proxy dynamically scrapes these anti-CSRF tokens from the web DOM and constructs valid nested RPC envelopes.
- **Rolling Cookie Pool Daemons**: Because Google introduced `__Secure-1PSIDTS` (a rapidly rotating challenge token validated against server-side session heartbeats), the proxy runs a background keepalive loop with database persistence (SQLite/MySQL/PostgreSQL) to simulate interactive user polling and prevent session invalidation.

---

### The Security & Operational Reality: Why Developers Must Avoid Them

While the hacking techniques behind these wrappers demonstrate impressive technical grit, their operational reality introduces fatal compromises for any developer environment:

---

### 1. Severe Terms of Service Violations & Account Bans

Google’s Terms of Service and Generative AI Additional Terms explicitly prohibit:
- Scraping, automated harvesting, or reverse-engineering consumer web endpoints (`gemini.google.com`).
- Impersonating client fingerprints or repurposing internal authentication tokens (such as CLI OAuth credentials) for arbitrary automated traffic.
- Bypassing access controls, quotas, or billing guardrails.

When automated traffic anomalies are flagged by Google’s anti-abuse heuristics, enforcement does not stop at revoking Gemini access. In documented instances, Google suspends the **entire underlying Google Account**. For developers, this can result in the catastrophic loss of Gmail, Google Drive, Android developer consoles, Google Cloud infrastructure, and domain authentication in a single stroke.

---

### 2. High-Risk Credential Exposure & Session Hijacking

Tools like `gemini-web2api-go` require users to extract their active browser cookies—specifically `__Secure-1PSID` and `__Secure-1PSIDTS`—and feed them into configuration files or environment variables.

These session cookies are **not scoped API keys**; they are the master authentication keys for your full Google identity:
- Any script, container, or logging system with access to these cookies has full authenticated access to read emails, view private files, and execute account actions.
- Running these scrapers inside Docker containers or community-hosted multi-tenant instances creates severe attack surfaces for credential exfiltration.
- If upstream dependencies in these repositories are compromised, attackers gain instant, unfettered access to developer accounts.

---

### 3. Brittleness & Complete Lack of Production Reliability

Official APIs are bound by semantic versioning, deprecation notices, and Service Level Agreements (SLAs). Consumer web interfaces are not.

Web-scraping proxies rely on undocumented internal RPC endpoints (such as `batchexecute`), obfuscated parameter names, and fleeting anti-CSRF tokens (`at`, `f.sid`). Whenever Google rolls out an A/B test, updates its frontend bundle, or tightens bot mitigations:
- The reverse-engineered endpoints fail silently or return corrupted payloads.
- Your application, local agents, or automated pipelines crash without warning.
- Developers are forced to wait for upstream maintainers to reverse-engineer Google’s newest obfuscated obfuscation layer.

---

### 4. Data Privacy & Compliance Hazards

There is a fundamental legal difference between consumer web interfaces and enterprise developer APIs:

- **Enterprise & Developer APIs (Google AI Studio / Vertex AI)**: Covered by strict data governance terms. Prompts, code snippets, and completions are not logged or used to train foundation models without explicit consent.
- **Consumer Web Interfaces (`gemini.google.com`)**: Under consumer terms, interactions may be reviewed by human annotators and utilized to train future models unless users navigate strict opt-out workflows.

Routing proprietary application data, internal business logic, or customer information through web-scraping proxies can constitute a severe GDPR, HIPAA, or SOC 2 compliance violation.

---

### 5. Legitimate, Zero-Risk Alternatives Already Exist

The irony of risking account termination for "free" Gemini access is that **legitimate, free access already exists**:

- **Google AI Studio**: Google officially provides a generous free tier for developers. You can generate an official API key in seconds with access to Gemini Flash models, structured JSON outputs, function calling, system instructions, and high rate limits—with zero violation of Terms of Service.
- **Local Open-Source Models**: If the objective is completely unrestricted, offline, and subscription-free inference without vendor constraints, modern open-source models (such as Llama 3.2, Mistral, and Phi-4 mini) run locally via Ollama or llama.cpp with complete privacy and zero legal ambiguity.

---

### Summary

The cat-and-mouse game of mimicking Chrome TLS signatures, decoding nested `batchexecute` frames, and spoofing CLI PKCE flows is a fascinating study in offensive web reverse-engineering. But in real-world engineering, relying on these proxies is an architectural dead-end. 

Trading account security, legal compliance, and operational stability for a brittle, unmaintained imitation of a free official API is an anti-pattern every developer should avoid.

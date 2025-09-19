---
# Slidev front matter
title: "Unifying Travels & Viagens — Architecture & Roadmap"
subtitle: "40′ presentation + 20′ Q&A"
author: "Ernesto Anaya Ruiz"
theme: default
fonts:
  sans: Inter
  mono: Fira Code
highlighter: shiki
lineNumbers: false
drawings:
  persist: true
transition: slide-left
exportFilename: "Thoughtworks_Case_Presentation"
---

# Unifying Travels & Viagens
**Architecture & Roadmap**  
**Presenter:** Ernesto Anaya Ruiz

<small>40′ solution presentation · 20′ Q&A</small>

---
layout: section
---

# Executive Summary

---

## Context & Vision
**Problem**  
- Two customer-facing apps (Travels: Struts/JSP on Tomcat; Viagens: React SPA + BFF)  
- Multiple backends across AWS, GCP, Azure  
- Low test coverage, batch releases, coordination overhead

**Vision**  
- A **single, resilient, fast web experience** powered by a **frontend platform** (React/Next.js shell) that composes **domain micro-frontends** and brokers requests via **domain BFFs** across existing backends.

**Approach (high level)**  
- **Micro-frontends (MFE) with a shell** + **domain BFFs** + **API gateway**  
- **Feature flags**, **contract testing**, **progressive delivery**  
- **Strangler Fig** to retire Struts/JSP gradually

**Outcomes**  
- Faster delivery, reduced coupling, safer releases, improved UX & Core Web Vitals

---
layout: section
---

# Target Architecture

---

## High-Level Diagram
```text
[Users] ──> [CDN/Edge + WAF] ──> [Unified Web App Shell (Next.js SSR/ISR)]
                                    │
                                    ├─> [MFE: Search & Discovery]
                                    ├─> [MFE: Booking]
                                    ├─> [MFE: Registration]
                                    └─> [MFE: Account/Payments/...]
MFE (domain) ──> Domain BFF ──> API Gateway ──> { AWS | GCP | Azure services | 3rd parties }
```
**Why this**  
- Keeps heterogeneous backends intact; unify UX now, backend later  
- SSR/ISR via **Next.js** → better **SEO/Core Web Vitals** vs SPA-only  
- **MFE isolation** mirrors org boundaries, reduces coordination  
- **Domain BFFs**: contracts, caching, retries, auth; decouple UI from backend quirks  
- **API Gateway**: routing, rate limiting, authz, cross-cloud abstraction

---

## Key Technology Choices
- **Web shell:** React + Next.js (SSR/ISR) with **Module Federation** for MFEs
- **BFFs:** Node/TypeScript (Express/Fastify) or Kotlin/Spring where apt
- **Contracts:** REST/JSON or GraphQL; **consumer-driven contracts** (Pact) MFE↔BFF & BFF↔Backend
- **Edge/CDN:** Cloudflare/Akamai/Fastly for assets, image optimization, edge auth
- **Observability:** OpenTelemetry → centralized APM/logs; dashboards & **SLOs**
- **Analytics & Consent:** Unified event schema + privacy/consent management
- **Security:** WAF, CSP/Trusted Types, secret mgmt, RASP (optional)

---
layout: section
---

# Migration Strategy (Strangler Fig)

---

## Phases & Timeboxes
**Phase 0 — Discovery (2–3 wks)**  
- Inventory frontends/backends, auth, analytics, SEO, SLAs  
- Establish **ADRs** and tech radar

**Phase 1 — Foundations (4–6 wks)**  
- Stand up **Web Shell + first MFE skeleton**  
- Platform toolchain (CI/CD, flags, telemetry, design system)  
- Create **API Gateway** + **Booking BFF** (pilot)  
- Implement SSO, edge caching, base observability, security baselines

**Phase 2 — Pilot (4–8 wks)**  
- Migrate **Search & Discovery** and **Booking** MFEs via BFFs  
- Dual-run with legacy pages, edge routing by path  
- A/B test; measure Core Web Vitals & conversion

**Phase 3 — Scale-out (8–16 wks)**  
- Move Registration, Payments, Account MFEs; standardize contracts; raise coverage  
- Decommission matching Struts/JSP pages as green path hits **>95% traffic**

**Phase 4 — Optimize & Decommission (ongoing)**  
- Performance budgets, caching strategies, resiliency patterns, cost controls  
- Backlog to retire legacy modules; reduce operational toil

**Risk controls:** feature flags, canaries, blue/green, SLOs & error budgets

---
layout: section
---

# Team Topology & Responsibilities

---

## Org Model
- **Frontend Platform (Enablement) Team**  
  Paved-road: shell, design system, CI/CD templates, linting, accessibility, i18n, perf budgets
- **Domain Product Squads** (Booking, Registration, Search, Account, etc.)  
  Own **MFE + Domain BFF**; end-to-end delivery (UX → prod)
- **Platform/SRE**  
  Observability, SLOs, incident mgmt, capacity/cost, IaC, security baselines
- **QA as Engineering Discipline**  
  Embedded SDETs; contract testing, E2E in prod-like, accessibility, non-functional
- **Data/Analytics**  
  Event schema, consent, dashboards, experimentation framework

> **Ownership:** CODEOWNERS/Backstage catalog; runbooks; on-call per domain; error budgets

---
layout: section
---

# Delivery Process & Best Practices

---

## Paved Road
- **Trunk-based development**, short-lived branches; **CI/CD per MFE/BFF**
- **Progressive delivery** (canary/percentage), not batch gates
- **Consumer-driven contract tests (Pact)**; consumer-first APIs
- **Ephemeral preview environments** per PR; visual regression + accessibility checks
- **Static analysis & coverage gates**; **performance budgets** in CI
- **Infrastructure as Code**; **GitOps** for environment promotion
- **ADRs & lightweight governance** (RFCs)
- **Security:** SAST/DAST/Dependency scanning, CSP/Trusted Types, secrets mgmt, threat modeling

---
layout: section
---

# KPIs & Success Metrics

---

## Delivery & Flow (leading)
- **DORA metrics**: lead time, deploy frequency, change failure rate, MTTR
- Cycle time, PR size/age, WIP limits, build time, flaky test rate, pipeline reliability

## Product & Customer (lagging)
- **Conversion rate** (search → booking), booking completion, cancellations
- **Core Web Vitals**: LCP ≤ 2.5s (p75), INP ≤ 200ms, CLS ≤ 0.1
- Error rate (JS & API), p95 latency, SLO availability (e.g., 99.9%)

## Platform & Cost
- **% traffic served by unified app**, legacy pages decommissioned
- Infra cost / 1k bookings, cache hit ratio, CDN offload %, egress

---
layout: section
---

# Security, Privacy & Compliance

---

## Guardrails
- OIDC/OAuth2 with centralized policy enforcement; least-privileged RBAC
- CSP, Subresource Integrity, security headers; dependency risk mgmt
- Consent management, data residency, audit trails
- PCI-adjacent isolation for payments; PII handling guidelines

---
layout: section
---

# Risks & Mitigations

---

## Top Risks
- **SEO regressions** → maintain URLs/redirects, SSR, metadata parity, schema.org, sitemaps
- **Cross-cloud latency** → smart routing, caching, async composition, circuit breakers
- **Auth fragmentation** → unified IdP & token strategy; BFF token exchange
- **Analytics drift** → event schema contract + validation in CI/CD
- **Change fatigue** → phased rollout + strong observability + feature flags

---
layout: section
---

# Demo Ideas

---

## Two Quick Demos
1) **Feature-flagged rollout** of Booking MFE with real-time metrics dashboard  
2) **Contract test breaker**: consumer-driven contract prevents a backward-incompatible change

---
layout: section
---

# Agenda & Workback Plan

---

## Presentation Agenda (40′)
1. Context & Goals (0–5)  
2. Target Architecture (5–15)  
3. Migration Roadmap (15–23)  
4. Team Topology (23–28)  
5. Delivery & Best Practices (28–33)  
6. KPIs & Success (33–38)  
7. Risks + Ask (38–40)

## 5-Day Workback Plan (for submission)
- **Day 1:** Discovery notes; baseline metrics wishlist; draft ADRs  
- **Day 2:** Architecture diagrams (current/target); choose pilot domain  
- **Day 3:** Roadmap & org model; platform backlog; risks  
- **Day 4:** Deck + speaker notes; dashboard mockups  
- **Day 5:** Dry run (40/20); refine Q&A; submit assets & availability

---
layout: section
---

# Q&A Bank (Samples)

---

## Common Questions & Talking Points
- **Why micro-frontends vs one SPA?** Org alignment, independent deploys, safer migration
- **Why Next.js SSR/ISR?** Better SEO & Core Web Vitals; edge cacheability; partial static regen
- **Avoiding a distributed monolith?** Clear contracts, BFF boundaries, consumer tests, ownership, observability
- **Backend unification later?** Design is backend-agnostic; unify behind BFFs/gateway when timing is right
- **Keeping quality high?** Shift-left: unit/component/contract; few critical E2Es; PR previews; a11y checks
- **Release strategy?** Progressive delivery (flags/canary), error budgets, auto-rollback
- **Data privacy?** Consent mgmt; minimize PII; policy-as-code; auditing

---
layout: center
class: text-center
---

# Thank you
**Ernesto Anaya Ruiz**  
_Questions welcome — looking forward to the discussion._

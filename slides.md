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

<small>Thoughtworks | Preparation for Case Study</small>

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
                                    
MFE ──> Domain BFF ──> API Gateway ──> { Travels Backend (AWS) }
                   └──> API Gateway ──> { Viagens Backend (GCP) }
                                    └──> { Azure services | 3rd parties }
```
**Why this**  
- Keeps heterogeneous backends intact; **no backend unification required**  
- **Domain BFFs as aggregation layer**: consolidate data from both companies (e.g., merge destination lists)
- **Response orchestration**: BFFs handle complex scenarios like combining search results, normalizing data formats
- SSR/ISR via **Next.js** → better **SEO/Core Web Vitals** vs SPA-only  
- **API Gateway**: routing, rate limiting, authz, cross-cloud abstraction

---

## Key Technology Choices
- **Web shell:** React + Next.js (SSR/ISR) with **Module Federation** for MFEs
- **BFFs (Backend for Frontend):**
  - Node/TypeScript for response aggregation and data transformation
  - **Key responsibilities:** Merge data from Travels + Viagens, normalize formats, handle pagination
  - Example: Search BFF calls both backends, merges destinations, deduplicates, sorts by relevance
- **Contracts:** REST/JSON or GraphQL; **consumer-driven contracts** (Pact) MFE↔BFF & BFF↔Backend
- **Edge/CDN:** Cloudflare/Akamai/Fastly for assets, image optimization, edge auth
- **Observability:** OpenTelemetry → centralized APM/logs; dashboards & **SLOs**
- **Analytics & Consent:** Unified event schema + privacy/consent management
- **Security:** WAF, CSP/Trusted Types, secret mgmt, RASP (optional)

---
layout: section
---

# Migration Roadmap

---

## Roadmap Without Revenue Loss

**Phase 0 — Discovery & Planning**  
- Inventory frontends/backends, auth, analytics, SEO, SLAs  
- Establish **ADRs** and tech radar
- Baseline current metrics (conversion, performance, revenue)

**Phase 1 — Foundations**  
- Stand up **Web Shell + first MFE skeleton**  
- Platform toolchain (CI/CD, flags, telemetry, design system)  
- Create **API Gateway** + **Booking BFF** (pilot)  
- Implement SSO, edge caching, base observability

**Phase 2 — Complete Migration Preparation**  
- Build **all MFEs** (Search, Booking, Registration, Payments, Account)  
- **Feature parity validation**: comprehensive testing vs legacy functionality  
- Performance benchmarking to ensure **≥100% baseline performance**
- **Big bang preparation**: deployment scripts, rollback procedures

**Phase 3 — Big Bang Migration**  
- **Complete cutover** from legacy to new platform during maintenance window  
- **Immediate monitoring**: revenue, conversion, performance dashboards  
- **Fast rollback capability** if any KPIs drop below acceptable thresholds

**Phase 4 — Stabilization & Optimization**  
- Performance tuning and optimization post-migration  
- Legacy system decommission once stability confirmed  
- Platform improvements and feature development

**Revenue Protection:** Comprehensive pre-migration testing, performance benchmarking, immediate rollback capability, intensive monitoring

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

# Processes and Best Practices

---

## Engineering Excellence & Delivery
**Development Practices**
- **Trunk-based development** with short-lived feature branches
- **CI/CD per MFE/BFF** with automated quality gates
- **Consumer-driven contract testing** (Pact) between all boundaries
- **Test pyramid:** Unit (70%) → Integration (20%) → E2E (10%)
- **Ephemeral preview environments** per PR; visual regression + accessibility checks
- **Static analysis & coverage gates** enforced in CI pipeline

**Quality & Security**
- **Shift-left testing:** PR previews, visual regression, accessibility
- **Security:** SAST/DAST/Dependency scanning, secrets mgmt, threat modeling
- **Performance budgets** enforced in CI (bundle size, Core Web Vitals)
- **Code quality gates:** Coverage >80%, no critical vulnerabilities

**Delivery & Operations**
- **Progressive delivery:** Feature flags, canary releases, A/B testing
- **Infrastructure as Code**; **GitOps** for environment promotion
- **Observability-first:** OpenTelemetry, distributed tracing, SLOs
- **Incident management:** Runbooks, on-call rotation, blameless postmortems

---
layout: section
---

# KPIs for Team Performance

---

## Team Performance & Value Delivery Metrics

**DORA Metrics (Industry Standard)**
- **Lead Time for Changes:** < 1 day for MFE changes
- **Deployment Frequency:** Multiple deploys per day per team
- **Change Failure Rate:** < 15% requiring hotfix/rollback
- **Mean Time to Recovery:** < 1 hour for critical issues

**Engineering Efficiency**
- **Cycle Time:** Idea → Production < 1 week for small features
- **PR Turnaround:** Review & merge within 4 hours
- **Build Time:** < 10 minutes for MFE, < 15 minutes for BFF
- **Test Coverage:** > 80% with < 2% flaky tests

**Team Health**
- **Cognitive Load:** Teams own max 2 MFEs + 1 BFF
- **On-call Burden:** < 2 incidents/week requiring manual intervention
- **Technical Debt Ratio:** 20% sprint capacity for improvements

---

## KPIs for Solution Success

**Business Impact**
- **Revenue per Session:** Maintain or improve vs. baseline
- **Conversion Rate:** Search → Booking improvement ≥ 5%
- **Cart Abandonment:** Reduce by 10% within 6 months
- **Customer Lifetime Value:** Increase through unified experience

**Technical Excellence**
- **Core Web Vitals:** LCP < 2.5s, INP < 200ms, CLS < 0.1 (p75)
- **Availability:** 99.95% uptime for critical user journeys
- **API Performance:** p95 latency < 200ms, p99 < 500ms
- **Error Rate:** < 0.1% for user-facing features

**Platform Adoption**
- **Migration Progress:** 100% traffic on new platform within 12 months
- **Legacy Decommission:** -25% legacy code per quarter
- **Cost Efficiency:** 20% reduction in infrastructure cost per transaction

---
layout: center
class: text-center
---

# Thank you
**Ernesto Anaya Ruiz**  
_Questions welcome — looking forward to the discussion._

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

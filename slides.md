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

# Summary

---

## Problem & Vision
**Problem**  
- Two customer-facing apps (Travels: Struts/JSP on Tomcat; Viagens: React SPA + BFF)  
- Multiple backends across AWS, GCP, Azure  
- Low test coverage, batch releases, coordination overhead

**Vision**  
- A **single, resilient, fast web experience** powered by a **frontend platform** (React/Next.js shell) that composes **domain micro-frontends** and brokers requests via **domain BFFs** across existing backends.

---

## Approach & Outcomes

**Approach (high level)**  
- **Micro-frontends (MFE) with a shell** + **domain BFFs** + **API gateway**  
- **Feature flags**, **contract testing**, **progressive delivery**  
- **Strangler Fig** to retire Struts/JSP gradually

**Outcomes**  
- **Faster delivery:** Independent team deployments, reduced coordination overhead
- **Better user experience:** Unified interface, improved Core Web Vitals, faster page loads
- **Reduced risk:** Progressive rollouts, feature flags, immediate rollback capability
- **Operational efficiency:** Consolidated data aggregation, simplified maintenance
- **Future flexibility:** Backend-agnostic design enables gradual modernization

---
layout: section
---

# Target Architecture

---

## High-Level Diagram <span style="font-size: 0.5em;"><a href="/architecture.png" target="_blank">📊 View detailed architecture diagram</a></span>
```text
[Unified Web Platform Architecture]
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

**Why this approach?**  
- Keeps heterogeneous backends intact; **no backend unification required**  
- **Domain BFFs as aggregation layer**: consolidate data from 2 companies (eg.merge destination lists)
- **Response orchestration**: BFFs handle complex scenarios (merging search results, normalizing data)
- SSR/ISR via **Next.js** → better **SEO/Core Web Vitals** vs SPA-only  
- **API Gateway**: routing, rate limiting, authz, cross-cloud abstraction

---

## Key Technology Choices
<br />

- **Web shell:** React + Next.js (SSR/ISR) with **Module Federation** for MFEs
- **BFFs (Backend for Frontend):**
  - Node/TypeScript for response aggregation and data transformation
  - **Key responsibilities:** Merge data from Travels + Viagens, normalize formats, handle pagination
  - Example: Search BFF calls both backends, merges destinations, deduplicates, sorts by relevance
- **Contracts:** REST/JSON or GraphQL; **consumer-driven contracts** (Pact) MFE↔BFF & BFF↔Backend
- **Edge/CDN:** Cloudflare/Akamai/Fastly for assets, image optimization

---
layout: section
---

# Migration Roadmap

---

## Migration Strategy Overview

**Approach:** Big bang migration with comprehensive preparation and revenue protection

**Key Phases:** Discovery → Foundations → Preparation → Migration → Optimization

**Revenue Protection:** Pre-migration testing, performance benchmarking, immediate rollback capability

---

## Phase 0-2: Preparation & Build

**Phase 0 — Discovery & Planning**  
- Inventory frontends/backends, auth, analytics, SEO, SLAs  
- Baseline current metrics (conversion, performance, revenue)

**Phase 1 — Foundations**  
- Stand up **Web Shell + first MFE skeleton**  
- Platform toolchain (CI/CD, flags, telemetry, design system)  
- Create **API Gateway** + **Booking BFF** (pilot)

**Phase 2 — Complete Migration Preparation**  
- Build **all MFEs** (Search, Booking, Registration, Payments, Account)  
- **Feature parity validation**: comprehensive testing vs legacy functionality  
- Performance benchmarking to ensure **≥100% baseline performance**

---

## Phase 3-4: Migration & Optimization

**Phase 3 — Big Bang Migration**  
- **Complete cutover** from legacy to new platform during maintenance window (minimal impact)
- **Immediate monitoring**: revenue, conversion, performance dashboards  
- **Fast rollback capability** if any KPIs drop below acceptable thresholds

**Phase 4 — Stabilization & Optimization**  
- Performance tuning and optimization post-migration  
- Legacy system decommission once stability confirmed  
- Platform improvements and feature development

**Critical Success Factors for Revenue Protection:** Comprehensive testing, performance benchmarking, immediate rollback capability, intensive monitoring

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
  Incident mgmt, capacity/cost, IaC, security baselines
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

**Quality Assurance**
- **Shift-left testing:** PR previews, visual regression, accessibility
- **Performance budgets** enforced in CI (bundle size, Core Web Vitals)
- **Code quality gates:** Coverage >80%, no critical vulnerabilities

**Delivery & Operations**
- **Progressive delivery:** Feature flags, canary releases, A/B testing
- **Infrastructure as Code**; **GitOps** for environment promotion
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

# Additional Reference Materials
_For Q&A and deeper discussion_

---

## Observability & Monitoring

**Distributed Tracing & APM**
- **OpenTelemetry** → centralized APM/logs across all MFEs and BFFs
- **Performance monitoring** for Core Web Vitals and API response times

**SLOs**
- **Service Level Objectives (SLOs)** for availability, latency, and error rates
- **Automated alerting** based on SLO violations

**Dashboards & Insights**
- **Real-time dashboards** for business metrics (conversion, revenue)
- **Technical dashboards** for system health and performance

---

## Security & Compliance

**Security Architecture**
- **WAF (Web Application Firewall)** at CDN/Edge layer for DDoS and attack protection
- **CSP (Content Security Policy)** and Trusted Types to prevent XSS attacks
- **Secrets management** with centralized vault and rotation policies

**Security Engineering**
- **SAST/DAST** (Static/Dynamic Application Security Testing) in CI/CD pipeline
- **Dependency scanning** for vulnerable packages and libraries

---

## Common Questions & Talking Points
<br />

- **Why micro-frontends vs one SPA?** Org alignment, independent deploys, safer migration
- **Why Next.js SSR/ISR?** Better SEO & Core Web Vitals; edge cacheability; partial static regen
- **Avoiding a distributed monolith?** Clear contracts, BFF boundaries, consumer tests, ownership, observability
- **Backend unification later?** Design is backend-agnostic; unify behind BFFs/gateway when timing is right
- **Keeping quality high?** Shift-left: unit/component/contract; few critical E2Es; PR previews; a11y checks
- **Release strategy?** Progressive delivery (flags/canary), error budgets, auto-rollback
- **Data privacy?** Consent mgmt; minimize PII; policy-as-code; auditing

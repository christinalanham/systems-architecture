# ADR-002: Low-code vs. high-code integration tooling

**Status:** Accepted  
**Date:** 2026-Q2 
**Author:** Christina Lanham

---

## Context

A mid-market company with a 200-person workforce operates a standard SaaS stack: a CRM, an ERP, a marketing automation platform, and a learning management system. The systems are managed by a lean IT team — one developer, two business analysts, and a director. No dedicated integrations engineer.

Over 18 months, the company has accumulated 11 point-to-point integrations: some built by the developer using REST APIs and custom scripts, some configured by business analysts using a low-code iPaaS, and several managed manually via spreadsheet exports and imports. There is no consistent approach, no centralized monitoring, and no documented error-handling strategy.

The company is preparing to add three more integrations as part of a new product rollout. This is the forcing function for an architectural decision: establish a deliberate integration tooling strategy before the surface grows further.

The decision criteria:
- Must be maintainable by a mixed-skill team (one developer, two non-technical admins)
- Must support centralized visibility — broken integrations cannot be discovered by end users first
- Must scale to at least 20 integrations without requiring a dedicated engineering hire
- Low-code tooling already in place for some flows; evaluate whether to standardize on it, replace it, or run both tracks deliberately

---

## Decision

Adopt a **tiered integration strategy**: low-code iPaaS as the default for data sync and workflow automation patterns; custom-coded integrations reserved for flows that exceed the platform's capability or cost model.

**Tier 1 — Low-code iPaaS (default):**  
Standard data sync, field mapping, and event-triggered workflow automation. Owned and maintained by business analysts. The developer acts as an escalation point, not the primary operator.

**Tier 2 — Custom code (exception, not default):**  
High-volume flows (>50K records/day), complex transformation logic that cannot be expressed in the iPaaS without excessive workarounds, or integrations with systems that lack supported connectors. Owned by the developer. Must be documented to the same standard as Tier 1 flows.

**Migration plan:**  
Existing custom scripts that perform tasks expressible in the iPaaS will be migrated to Tier 1 over the next two quarters. Manual spreadsheet-based integrations will be eliminated entirely — each will either be automated in Tier 1 or escalated to Tier 2 if volume justifies it.

---

## Options considered

### Option 1: Standardize on low-code iPaaS for all integrations

Move everything to the iPaaS platform and phase out custom scripts entirely.

**Pros:** Single platform to monitor and maintain. Business analysts gain full ownership. Vendor manages connector updates. Faster build time for new integrations.

**Cons:** iPaaS platforms are not well-suited to high-volume or high-complexity flows. Forcing complex logic into a low-code tool creates brittle, hard-to-debug flows that are worse than the custom scripts they replaced. Licensing costs scale with usage — at high record volumes, per-transaction pricing becomes expensive.

**Rejected because:** The one-size approach breaks down at both ends. Simple flows belong in the iPaaS. Forcing complex or high-volume flows there creates a different kind of maintenance problem — one that's harder to debug because the logic is hidden inside a GUI rather than readable code.

---

### Option 2: Standardize on custom code for all integrations

Build all integrations in code, managed by the developer.

**Pros:** Maximum flexibility. No vendor dependency for connector availability. No per-transaction cost model.

**Cons:** Creates a single point of failure — all integrations depend on one person. No business analyst can operate or modify any flow without developer involvement. Slows time-to-value significantly for routine sync patterns. Inconsistent with the team's skill mix.

**Rejected because:** This approach misaligns the work with the team. A developer writing field-mapping scripts is an expensive way to move data between a CRM and a marketing platform. The iPaaS exists precisely to eliminate that overhead for standard patterns.

---

### Option 3: Tiered strategy — iPaaS as default, custom code as exception (selected)

Define two tiers with clear criteria for which flows belong in each, and manage them as distinct tracks with different ownership models.

**Pros:** Matches complexity to capability. Business analysts own what they can maintain. The developer focuses on what actually requires code. Centralized monitoring is achievable through the iPaaS layer for Tier 1; Tier 2 custom integrations are documented and logged to the same standard. Cost is predictable — low-volume flows stay cheap in the iPaaS; high-volume flows where per-transaction pricing would be punitive are moved to code.

**Cons:** Two systems to maintain instead of one. Requires clear escalation criteria so flows don't end up in the wrong tier by default. Tier 2 documentation discipline is essential — undocumented custom code recreates the original problem.

**Selected because:** The team's skill mix, integration volume, and budget constraints all point to a tiered approach. Neither pure track is viable. The key is treating the tiers as intentional architecture, not as "we use both tools and figure it out as we go."

---

## Consequences

**Positive:**
- Business analysts gain direct ownership of the majority of integrations — no developer bottleneck for standard sync patterns
- Centralized monitoring through the iPaaS platform for Tier 1 flows; broken integrations surface in dashboards, not in user complaints
- Developer capacity freed for Tier 2 work and product-adjacent engineering
- Clear criteria for new integrations prevent ad hoc decisions accumulating into a second wave of inconsistency

**Negative / accepted risks:**
- Two-platform environment adds operational complexity — teams must know which tier a flow belongs to before diagnosing issues
- Tier 2 documentation discipline is load-bearing; without it, custom integrations become undocumented liabilities when the developer leaves or context is lost
- iPaaS licensing costs require active monitoring as integration volume grows — the cost model must be reviewed at each contract renewal

**Follow-on decisions required:**
- Tier escalation criteria: document the specific thresholds (record volume, transformation complexity, connector availability) that trigger a Tier 1 → Tier 2 decision
- Monitoring and alerting standards: define what "centralized visibility" means operationally — alert routing, on-call ownership, SLA for integration failures
- Migration backlog: prioritize which existing custom scripts and manual processes are addressed first

---

## Notes on applicability

This tiered model is well-suited when:

- The team managing integrations is mixed-skill — some technical, some not — and ownership needs to match capability
- The integration surface is broad (10+ flows) but most flows are standard data sync patterns that don't require custom logic
- Per-transaction iPaaS pricing is manageable at current volume but would become a concern at scale
- The goal is sustainable operations, not maximum flexibility

It is less suitable when the team is fully technical (in which case, a code-first approach with a proper integration framework may be cleaner), when the iPaaS does not support the systems in scope, or when a single high-volume flow dominates the integration workload to the point where the platform cost model is unworkable from the start.

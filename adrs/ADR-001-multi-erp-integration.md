# ADR-001: Multi-ERP integration strategy for a post-acquisition environment

**Status:** Accepted  
**Date:** 2024-Q4  
**Author:** Christina Lanham

---

## Context

A mid-market company completed an acquisition. The acquiring entity runs ERP-A; the acquired entity runs ERP-B. Both ERPs are active, under separate contracts, and serving live business operations. Neither can be decommissioned in the near term — the acquiring entity is mid-contract, and the acquired entity's ERP holds several years of operational history required for compliance and audit purposes.

The business needs a short-to-medium term integration strategy that:
- Keeps both ERPs functional without data divergence
- Supports consolidated financial reporting across both entities
- Enables order-to-cash and procure-to-pay workflows to operate without manual re-keying
- Does not require a full ERP consolidation project (18–24 month minimum, significant capital outlay)

Teams involved: Finance, Operations, IT. Timeline pressure: consolidated close cycle required within 90 days of acquisition date.

---

## Decision

Implement a **hub-and-spoke integration model** using a middleware iPaaS platform, with ERP-A designated as the system of record for consolidated financial data and ERP-B operating as a spoke that syncs on defined transaction types.

Integration scope for Phase 1:
- Vendor master sync (ERP-A → ERP-B, one-directional)
- Customer master sync (bidirectional, with ERP-A as master of record on conflict)
- Invoice and revenue data sync for consolidated reporting (ERP-B → ERP-A, read-only)
- Purchase order status (ERP-B → ERP-A, event-triggered)

Phase 1 explicitly excludes inventory, payroll, and HR data — these remain siloed until ERP consolidation is scoped.

---

## Options considered

### Option 1: Point-to-point direct API integration
Build direct API connections between the two ERPs without middleware.

**Pros:** No additional tooling cost. Full control over integration logic.

**Cons:** Each integration is a custom build. No centralized logging or error handling. Any schema change in either ERP breaks integrations individually. Scales poorly as integration scope grows. High ongoing maintenance burden on a small IT team.

**Rejected because:** The maintenance cost compounds quickly in a two-ERP environment where schema stability is not guaranteed post-acquisition. A single team restructure or ERP upgrade could break multiple silent integrations simultaneously.

---

### Option 2: Full ERP consolidation (migrate to one platform)
Decommission ERP-B and migrate all data and workflows to ERP-A.

**Pros:** Eliminates the dual-ERP problem permanently. One system of record. Lower long-term operational overhead.

**Cons:** 18–24 month minimum project timeline. $500K–$1.5M implementation cost estimate. Requires full business process mapping and data migration. High disruption risk during a period when the business needs stability.

**Rejected because:** Timeline and cost are incompatible with the 90-day consolidated reporting requirement. This remains the correct long-term path and should be scoped as a separate initiative.

---

### Option 3: Hub-and-spoke iPaaS integration (selected)
Use a middleware integration platform to connect both ERPs through a centralized integration layer, with ERP-A as the financial system of record.

**Pros:** Faster time-to-value than a consolidation project. Centralized error handling, logging, and monitoring. Schema changes in either ERP are isolated to the affected connector, not the entire integration surface. Extensible — additional integration flows can be added without re-architecting. Supports the eventual ERP consolidation by building a documented data mapping layer now.

**Cons:** Ongoing iPaaS licensing cost. Requires integration platform expertise. Adds a dependency layer between the two ERPs.

**Selected because:** The only option that meets the 90-day timeline requirement while maintaining data integrity and creating a maintainable foundation. The iPaaS layer also accelerates the future consolidation project by producing a documented field mapping inventory.

---

## Consequences

**Positive:**
- Consolidated financial reporting achievable within 90-day window
- Single point of monitoring and error handling for all cross-system data flows
- Integration documentation becomes an asset for the eventual ERP consolidation project
- IT team can manage integration changes without touching either ERP directly

**Negative / accepted risks:**
- Dual-ERP operational complexity persists until consolidation — this ADR does not solve the long-term problem
- iPaaS platform becomes a critical dependency; vendor lock-in is a real consideration and should be evaluated at contract renewal
- Phase 1 scope exclusions (inventory, payroll, HR) create known data silos that require manual reconciliation until Phase 2

**Follow-on decisions required:**
- iPaaS vendor selection (separate ADR)
- ERP-A as system of record: conflict resolution rules for bidirectional customer master sync
- Phase 2 scope definition: when to tackle inventory and HR data flows
- Long-term ERP consolidation: timeline, budget, and business case (separate initiative)

---

## Notes on applicability

This ADR describes a pattern common in post-acquisition environments where two enterprise systems must coexist under timeline and budget constraints. The hub-and-spoke iPaaS model is well-suited when:

- A full consolidation is the right long-term answer but the wrong immediate answer
- The integration surface is bounded (defined transaction types, not full system replication)
- A small IT team needs centralized visibility over cross-system data flows
- The integration layer itself will become an input to a future migration project

It is less suitable when the two systems have fundamentally incompatible data models, when real-time latency requirements are strict (sub-second), or when the iPaaS licensing cost is disproportionate to the integration volume.

# SWANS Event Exchange — Documentation repo

Two halves, one site:

- `docs/clients/` — what a trading member, clearing member or prime broker needs to connect, trade, clear and settle on SWANS. Written to be publishable.
- `docs/internal/` — how we build it. Service-by-service specification in C++ terms, data model, messaging, regulatory mapping, build plan. Internal only.

Build the site locally:

```
pip install mkdocs-material
mkdocs serve
```

Or read the markdown directly; every page stands alone.

Conventions: regulatory thresholds marked **[confirm]** must be verified with counsel before being hard-coded. Sections marked **[from Mehdi]** are to be completed from the margin and fees methodology documents.

Version: v2.2 (self-consistency pass: single mark hierarchy for VM, PnL and daily settlement; pre-trade check list aligned; reservations; tags and codes), 26 August 2026. v1.0 integrated Margin Framework v6, Fee Model v3, the ten-schema contract engine and end-to-end diagrams. v2.0 adds the institutional-grade layer: quickstart with code, guarantees and non-guarantees, authentication model, exact order and event semantics, request/response examples for every endpoint and FIX message logs, rate limits and close codes, reconciliation, change management, HA/failover and DR, capacity and performance targets, security threat model, operations runbooks, and governance. Capacity figures and the failover design are stated assumptions marked [confirm] pending engineering review. Owners: Levon (product, regulatory), Mehdi (risk, margin, fees), Lyes (engineering).

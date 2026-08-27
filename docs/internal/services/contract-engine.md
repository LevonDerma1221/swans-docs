# Contract engine service

**Owns:** the pipeline from public events to listed contracts; the inbound exposure-intake path; the family structure and product-group assignment that margin, marks and settlement depend on.

## Pipeline

1. **Scan.** Connectors pull structured calendars and documents: regulator calendars (FDA PDUFA dates, EMA CHMP, CMS rulemaking, FTC/DOJ dockets), court dockets, EDGAR/RNS filings, clinical trial registries, statistics-office and central-bank release calendars, company reporting calendars. Each source has a connector with a fetch cadence, a document store and a checksum.
2. **Surface.** LLM-assisted classification against the taxonomy (12 categories, subcategories, IMPACTS cross-cutting tags). Output: candidate catalysts with entity resolution to issuer LEI/ticker.
3. **Score.** Relevance score 1–100 from capital exposed, participants affected, hedging utility, proxy demand (options volume, analyst coverage), gap status (existing contract elsewhere), resolution clarity. Threshold for specification is a policy parameter.
4. **Specify.** Schema selection and slot filling per the schema's eligibility rules; source hierarchy assignment from the schema's source table; expiry and `last_trading_time` derived from the source's publication window minus a schema-specific buffer; family construction (ladders, buckets, timing periods, clusters); position limit from the schema policy and the issuer's liquidity; product group assignment for margin; measured-contract strikes where applicable. Output: a `ContractProposal` record validated against the schema's JSON schema.
5. **Review.** New Product Committee workflow with the listing checklist; four-eyes; risk sign-off for S-09/S-10 and any new family type. Approved proposals become `ContractSpec` rows in reference data with `state = pending` until `listing_time`.

## Inbound intake

Members submit an exposure description (structured form or free text). The intake agent decomposes it to an `ExposureRecord`: entity, event, direction, magnitude, horizon, source. Triage: **matched** (existing contract; return symbol), **generatable** (no contract but a schema fits; route to Specify with priority), **latent** (fails the objectivity test; logged with reason). Verification tier by ticket size (document upload for small; cross-check against public data for large).

## Data

```cpp
enum class FamilyType : uint8_t { Standalone, Nested, MutuallyExclusive, Cluster };
struct Family { uint32_t id; FamilyType type; std::vector<InstrumentId> members; std::vector<uint8_t> order;  // ladder order or bucket order
                std::vector<std::vector<uint8_t>> admissible_states; };   // enumerated for Nested/ME; rule-based for Cluster
struct ContractProposal { SchemaType schema; std::string question; /* slots */ std::map<std::string,std::string> slots;
                          SettlementSource sources[3]; Timestamp expected_publication, last_trading_time, settlement_deadline;
                          uint32_t family_id; uint16_t product_group; Qty position_limit; int32_t strike_low_x1e6, strike_high_x1e6;
                          uint8_t relevance; std::string hedging_rationale, gap_status, proxy_demand; uint8_t filing_route; };
```

## Interfaces
- `POST /engine/candidates` (scan output), `GET /engine/proposals?state=`, `POST /engine/proposals/{id}/approve` (NPC), `POST /intake/exposures` (members), `GET /intake/exposures/{id}` (triage result).
- Publishes `ContractSpec` and `Family` to reference data; publishes `ExposureRecord` (anonymised) to the demand log.

## Controls
- Every listed contract traces to a proposal, a schema version, a source document checksum and a committee decision.
- Schema definitions are versioned reference data; a schema change is a governed release.
- The prohibited-category screen and the objectivity test are executable rules, not prose.
- Product governance (PROD) records: target market (professional only), hedging rationale, distribution route.

## Build
Phase 1: schema JSON definitions for S-01–S-08; `Specify` validator; NPC workflow; intake form and triage against the catalogue. LLM stages (Surface, Score, intake decomposition) call an external model behind an internal service with prompt and output logging; deterministic validation sits after every LLM stage.

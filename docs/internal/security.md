# Security

## Threat model (summary)
| Threat | Controls |
|---|---|
| Unauthorised order entry via stolen FIX credentials | Client TLS certificates bound to session and IP allow-list; credentials rotated 90 days; cancel-on-disconnect; clearer kill; anomaly alerts on message-rate change |
| Compromised API key | Scopes, expiry, IP allow-list, immediate revocation, two-admin key management, control-plane replay windows and idempotency keys |
| Insider manipulation of settlement | Two-officer determination with distinct credentials and hardware second factor; evidence hashing; dispute window; committee minutes; surveillance of officer accounts |
| Manipulation of marks via thin-book prints | Filtered fair marks with staleness and depth weighting; governance fallback with two approvals |
| Tampering with journals | Append-only, checksummed, replicated; periodic hash anchoring to an external store |
| Privilege escalation in operations | Named operator accounts, hardware second factor, least privilege, all actions journaled with actor and reason; quarterly access review |
| Data exfiltration (member positions, GCM books) | Per-institution data segregation at the API and file layer; encryption at rest; egress controls; Product B tenancy isolation |
| Denial of service | Rate limits per interface; upstream filtering on cross-connects; separate market-data and order-entry paths |
| Supply chain | Pinned dependencies, SBOM, signed builds, reproducible build for the engine |
| LLM stages in the contract engine | Deterministic validation after every LLM call; no listing without human approval; prompt and output logging; no member data in prompts |

## Key management
Certificates and secrets in Vault with HSM-backed roots; FIX credentials and API secrets never stored in plaintext; automated rotation; break-glass procedure with dual control.

## Testing
Annual external penetration test covering FIX, REST/WS, Clearer API, member portal and operations tools; quarterly internal tests; RTS 7 requires periodic testing of systems and controls **[confirm scope]**.

## Audit
Regulatory audit extract (operator actions, key events, settlement steps, parameter changes) produced nightly and retained with the order records.

# Security

## Threat model

| Threat | Controls |
|---|---|
| Unauthorised order entry via stolen FIX credentials | Client TLS certificates bound to session and IP allow-list; credentials rotated 90 days; cancel-on-disconnect |
| Compromised API key | Scopes, expiry, IP allow-list, immediate revocation |
| Insider manipulation of settlement | Two-officer determination with distinct credentials; evidence hashing; dispute window |
| Manipulation of marks via thin-book prints | Filtered fair marks with staleness and depth weighting; governance fallback |
| Tampering with journals | Append-only, checksummed, replicated |
| Denial of service | Rate limits per interface; upstream filtering on cross-connects |
| LLM stages in the contract engine | Deterministic validation after every LLM call; no listing without human approval |

# Versioning and change management

| Change | Notice to members | Testing | Approval |
|---|---|---|---|
| New contract listing (existing schema) | 2 business days | none | New Product Committee |
| New schema or family type | 30 days | Certification environment | Risk committee, board |
| FIX or API change, backward-compatible | 30 days | Certification, optional | Product |
| FIX or API change, breaking | 90 days; old version supported 6 months | Mandatory re-certification | Product, compliance |
| Fee parameter change | 30 days | none | Board; published formulas updated in reference data |
| Margin parameter change | 10 business days; intraday in stress under governance with same-day notice | none | Risk committee; CCP and clearing members notified |
| Margin model change | 30 days; challenger results published to CCP and clearing members | Shadow run ≥ 30 days | Risk committee, independent validation |
| Offset stage change | 30 days | none | Risk committee; CCP approval |
| Rulebook change | Per FCA requirements **[confirm]** | | Board, FCA notification |
| Trading hours or calendar | 30 days | | Product |

All changes are announced on the `instruments` and `status` channels and by member notice. Reference data carries `spec_version`, `schema_version`, `fee_version`, `margin_version`; every file and response states the version it was produced under.

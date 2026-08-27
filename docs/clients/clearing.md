# Clearing and give-up

## Model

Every trade on SWANS is submitted to the partner CCP within seconds of matching (straight-through processing). On acceptance, the CCP novates: it becomes the counterparty to each side's clearing member, and each clearing member faces its client. SWANS is not in the credit chain.

```
 Fund A ──clearing agreement──▶ Clearing member A ──membership──▶ CCP ◀──membership── Clearing member B ◀── Market maker B
```

The CCP clears SWANS contracts in a **segregated service**: its own default fund, its own rules and its own risk parameters, separate from the CCP's other asset classes. Only clearing members that have joined the service can clear SWANS contracts.

## Roles

| Role | Who | Obligations |
|---|---|---|
| Trading member | Funds, dealers, market makers | Trades; posts margin to its clearing member |
| Clearing member (GCM) | Banks and non-bank clearers that are CCP participants | Registers clients at the CCP; sets client limits on SWANS; posts margin and default fund to the CCP; collects margin from clients |
| Self-clearing member | Market makers and dealers with CCP membership | Clears own trades directly |
| CCP | Partner CCP | Novates, margins, settles, guarantees |

## Before trading

1. Sign a clearing agreement with a clearing member active in the SWANS service.
2. Your clearing member registers your SWANS account against its CCP account and sets, through the Clearer API: max order size, gross and net notional limits, daily loss limit, SWANS margin budget, and holds a kill switch.
3. SWANS enables trading rights on the account.

## Trade flow

1. Match. `ExecutionReport` (ExecType F) to both sides with `TrdMatchID` (880) and the venue transaction ID.
2. Submission to CCP within the STP window, tagged with each side's clearing code.
3. CCP accept: trade state becomes `cleared`; `TradeCaptureReport` drop copy to each clearing member.
4. CCP reject: trade state becomes `ccp_rejected`; both sides receive `ExecutionReport` with ExecType H (trade cancel); positions revert. Rejections are rare and generally indicate a limit breach at the CCP or a registration error.

Time thresholds under RTS 26 **[confirm exact values]**: SWANS submits within seconds of matching; the CCP responds within seconds.

## Give-up (bilateral bridge)

For contracts or accounts designated `bilateral` under the rulebook, trades are not submitted to the CCP. Instead they are given up to the account's designated prime broker or dealer, which becomes counterparty of record under a give-up agreement with SWANS and a margin agreement (CSA) with the client naming SWANS as calculation agent for initial margin. This path exists so that trading can continue if the CCP service is delayed and for members whose prime broker prefers to carry the position. Cross-margining against the client's hedge is then entirely within the prime broker.

## Drop copies and files

Clearing members receive real-time `TradeCaptureReport` messages for their clients' fills and, daily: trade file, position file, SWANS margin schedule per client, and the CCP parameter file for reconciliation.

## Default and porting

Handled under the CCP's rules for the segregated service. SWANS suspends trading rights on accounts of a defaulting clearing member on notice from the CCP and supports porting of client accounts to another clearing member.

# Diagrams

## 1. Whole venue

```mermaid
flowchart TB
  MEM[Members] -->|FIX orders| GW[FIX gateway]

  subgraph SWANS
    GW --> ENG[Pre-trade + matching engine]
    ENG -->|executions| TR[Trades]
    ENG -->|prices| MD[Market data]
    TR --> COL[Collateral service]
    COL -->|balances| ENG
    MK[Marks] --> SET[Settlement]
    SET --> COL
    RISK[Risk engine - shadow mode]
  end

  MD -->|book, trades, status| MEM
  COL -->|lock max loss, settle payouts| MEM
  RISK -->|price file, analytics| MEM
```

## 2. Trade lifecycle (full collateral)

```mermaid
sequenceDiagram
  participant M as Member
  participant S as SWANS

  M->>S: Deposit cash
  S-->>M: Balance confirmed

  M->>S: Order (FIX)
  S->>S: Balance check (available >= max loss)
  Note over S: Match with resting order
  S-->>M: Execution report (fill)
  Note over S: Max loss locked from both sides

  Note over S: Time passes — no adjustments, no VM

  Note over S: Source publishes, two-officer determination
  S-->>M: Settlement: payout distributed, locks released
```

## 3. Two-engine risk architecture

```mermaid
flowchart LR
  subgraph Inputs
    POS[Positions]
    MK[Marks]
    FAM[Families]
  end

  subgraph Engines
    MPOR[MPOR engine]
    TERM[Terminal engine]
  end

  POS --> MPOR
  POS --> TERM
  MK --> MPOR
  MK --> TERM
  FAM --> MPOR
  FAM --> TERM

  MPOR -->|IM| OUT[Margin numbers]
  MPOR --> DIV
  TERM --> DIV[Divergence check]
  TERM -->|diagnostics| STRESS[Stress and validation]
  DIV -->|above threshold| AMOD[Model-risk add-on]
```

The MPOR engine is authoritative for clearing IM. The terminal engine is a standing challenger. At launch, the risk engine runs in shadow mode (analytics only). It becomes production when margin is offered.

## 4. Margin formula

```mermaid
flowchart LR
  VaR[VaR 99%] --> CORE
  ES[ES 97.5%] --> CORE
  JUMP[Jump-to-resolution] --> CORE
  FLOOR[Floor] --> CORE["IM_core = max()"]

  CORE --> HYBRID
  ADD[Add-ons] --> HYBRID["IM = min(L_gross, IM_core + A)"]

  HYBRID --> FINAL["Event ramp blend to L_gross"]
  FINAL --> OUT[Margin output]
```

## 5. Contract engine

```mermaid
flowchart LR
  SRC[Source calendars] --> SCAN[Scan and surface]
  SCAN --> SCORE[Score relevance]
  SCORE --> SPEC[Generate spec]
  SPEC --> NPC[New Product Committee]
  NPC --> LIST[Listed contract]

  REQ[Member intake] --> TRI{Match?}
  TRI -->|yes| LIST
  TRI -->|generatable| SPEC
  TRI -->|latent| LOG[Demand log]
```

## 6. Marks and settlement

```mermaid
flowchart LR
  subgraph Mark sources
    A[Auction price]
    B[Microprice]
    T[Decayed VWAP]
    M[Model mark]
  end

  A --> COMB[Logit combiner and filter]
  B --> COMB
  T --> COMB
  M --> COMB
  COMB --> FAIR[Fair mark]

  FAIR --> IM[Risk analytics]
  FAIR --> PF[Price file]
```

```mermaid
flowchart LR
  SRC[Source publishes] --> O1[Officer 1 proposes]
  O1 --> O2[Officer 2 confirms]
  O2 --> DW[Dispute window]
  DW --> SET[Final settlement]
  DW -->|dispute| CMT[Committee] --> SET
  SET --> PAY[Payout]
```

## 7. Deployment

```mermaid
flowchart LR
  subgraph LD4[LD4 primary]
    GW[Gateways] --> ENG[Engine]
    ENG --> SVC[Services]
    SVC --> DB[(PostgreSQL)]
    SVC --> J[(Journals)]
  end

  subgraph DR[DR site]
    WS[Warm standby]
  end

  J -->|shipped| WS
  SVC --> COL[Collateral]
  SVC --> ARM[Reporting]
```

# End-to-end views

All diagrams are Mermaid; GitHub renders them natively and the MkDocs site renders them with the mermaid2 plugin.

## 1. Whole venue

```mermaid
flowchart TB
  MEM[Members] -->|FIX orders| GW[FIX gateway]

  subgraph SWANS
    GW --> ENG[Pre-trade + matching engine]
    ENG -->|executions| TR[Trades]
    ENG -->|prices| MD[Market data]
    TR --> POS[Positions]
    POS --> RISK[Risk engine]
    RISK --> MGN[Margin service]
    MGN -->|budgets| ENG
    MK[Marks] --> MGN
    MK --> SET[Settlement]
  end

  MD -->|market data| MEM
  MGN -->|margin schedule, price file| PB[Prime brokers]
  TR -->|drop copy| PB
  SET -->|settlement value| PB
  PB -->|limits| ENG
  COL[Collateral service] -->|balances| ENG
  SET --> COL
  MGN --> COL
```

## 2. Trade lifecycle

```mermaid
sequenceDiagram
  participant F as Fund
  participant GW as SWANS gateway
  participant ENG as SWANS engine
  participant P as Prime broker

  F->>GW: NewOrderSingle (FIX)
  GW->>ENG: Pre-trade risk check
  ENG-->>GW: Accepted
  Note over ENG: Match with resting order
  GW-->>F: ExecutionReport (fill)
  ENG->>P: TradeCaptureReport (drop copy)
  ENG->>P: Updated margin schedule

  loop Every 8 hours (VM window)
    ENG->>P: Filtered marks + VM + margin schedule
    P->>F: VM settlement / margin call
  end

  Note over ENG: Source publishes, two-officer determination
  ENG->>P: Final settlement value
  P->>F: Cash settlement
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

  MPOR -->|IM| OUT[Margin schedule]
  MPOR --> DIV
  TERM --> DIV[Divergence check]
  TERM -->|diagnostics| STRESS[Stress and validation]
  DIV -->|above threshold| AMOD[Model-risk add-on]
```

The MPOR engine is authoritative for clearing IM. The terminal engine is a standing challenger. Both share the same factor state and calibration; divergence is diagnostic only under aligned comparison (the comparable-run rule).

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
  FINAL --> SCHED[Margin schedule]
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

  FAIR --> VM[VM]
  FAIR --> IM[IM]
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
  SVC --> PBA[PB adapter]
  SVC --> COL[Collateral]
  SVC --> ARM[Reporting]
```

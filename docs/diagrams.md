# End-to-end views

All diagrams are Mermaid; GitHub renders them natively and the MkDocs site renders them with the mermaid2 plugin.

## 1. Whole venue

```mermaid
flowchart TB
  subgraph Members
    OMS[Member OMS / EMS]
    LP[Liquidity providers]
  end
  subgraph SWANS["SWANS MTF (LD4)"]
    GW[FIX gateway]
    ENG["Engine shard: pre-trade risk → matching → STP"]
    MD[Market data publisher]
    TR[Trades service]
    POS[Position service]
    MK[Marks service]
    RISK[Risk engine / libswansrisk]
    MGN[Margin and fee service]
    SET[Settlement service]
    CE[Contract engine]
    REF[Reference data]
    REP[Reporting and surveillance]
    API[REST / WebSocket / Clearer API]
  end
  subgraph Post-trade
    CCP[Partner CCP segregated service]
    GCM[Clearing members]
    PB[Prime brokers]
  end
  subgraph Oversight
    FCA[FCA / ARM]
    SURV[Surveillance system]
  end
  OMS -->|FIX 4.4| GW --> ENG --> MD -->|FIX / WS / multicast| OMS
  LP -->|FIX| GW
  ENG --> TR -->|STP, seconds| CCP
  TR -->|drop copy AE| GCM
  CCP --> GCM --> OMS
  PB -.->|collateral, netting| GCM
  TR --> POS --> RISK --> MGN
  MK --> MGN
  MGN -->|budgets| ENG
  MGN -->|schedule| GCM
  MGN -->|parameter file| CCP
  MGN -->|price and risk file| PB
  SET --> TR
  SET --> CCP
  CE --> REF --> ENG
  REF --> CCP
  GCM -->|limits, kill| API --> ENG
  REP -->|RTS 22 / 24| FCA
  REP --> SURV
```

## 2. Trade lifecycle

```mermaid
sequenceDiagram
  participant F as Fund
  participant S as SWANS
  participant C as CCP
  participant G as Clearing member
  participant P as Prime broker
  F->>S: NewOrderSingle (FIX D)
  S->>S: pre-trade checks incl. GCM limits and margin budget
  S->>S: match
  S-->>F: ExecutionReport (F)
  S->>C: trade submission (STP)
  C-->>S: accept
  S-->>G: TradeCaptureReport (drop copy)
  Note over C,G: CCP novates: faces G for F
  C->>G: IM and VM call (CCP model, SWANS parameters)
  G->>F: client margin (≥ CCP, SWANS schedule if adopted)
  P->>G: collateral movement, PB decides netting vs hedge
  loop daily
    S->>G: filtered marks, VM, schedule
    C->>G: settlement price, VM
  end
  S->>S: settlement determination (two officers, dispute window)
  S->>C: final settlement value
  C->>G: cash settlement
  G->>F: proceeds, margin release
```

## 3. Margin engine outputs

```mermaid
flowchart LR
  IN1[Positions net + pending] --> E
  IN2[Filtered fair marks] --> E
  IN3[Families and admissible states] --> E
  IN4[Hazards, factors, liquidity tiers] --> E
  E[Margin engine: structural max loss, MPOR Monte Carlo, jump, add-ons, event ramp]
  E --> O1[Pre-trade budgets and fast bounds]
  E --> O2[Clearing-member schedule with attribution]
  E --> O3[CCP parameter proposal and scenarios]
  E --> O4[Prime-broker price and risk file]
  E --> O5[CSA IM for bilateral bridge]
  E --> O6[Backtests, sensitivity, trigger report]
```

## 4. Contract engine

```mermaid
flowchart LR
  SRC[Regulator calendars, filings, trial registries, dockets, macro calendars] --> SCAN[Scan]
  SCAN --> SURF[Surface: taxonomy]
  SURF --> SCORE[Score: relevance]
  SCORE --> SPEC[Specify: schema, slots, source hierarchy, expiry, family, limit, group]
  SPEC --> NPC[New Product Committee]
  NPC --> REF[Reference data: listed]
  INTAKE[Member exposure intake] --> TRI{Triage}
  TRI -->|matched| REF
  TRI -->|generatable| SPEC
  TRI -->|latent| LOG[Demand log]
```

## 5. Marks and settlement

```mermaid
flowchart TB
  A[Auction print] --> L[Logit combiner]
  B[Executable microprice] --> L
  T[Decayed VWAP] --> L
  M[Model mark] --> L
  L --> FLT[Filter α] --> PRJ[Family projection] --> XF[X_fair]
  XF --> VM[Variation margin]
  XF --> IM[Initial margin]
  XF --> PF[Price file]
  SRC[Source publication] --> P1[Officer 1: propose, hash evidence] --> P2[Officer 2: confirm] --> DW[Dispute window] --> FIN[Final settlement] --> CCP[CCP cash settlement]
  DW -->|dispute| CMT[Settlement committee] --> FIN
  DL[Deadline passed] --> FB[Fallback rule] --> FIN
```

## 6. Deployment

```mermaid
flowchart LR
  subgraph LD4["LD4 primary"]
    XC[Member cross-connects / VPN] --> LB[TLS termination] --> GW[Gateways]
    GW --> ENG[Engine hosts, pinned cores]
    ENG --> SVC[Trades, positions, marks, risk, margin, settlement]
    SVC --> DB[(PostgreSQL)]
    SVC --> J[(Journals)]
  end
  subgraph DR["DR site"]
    J2[(Shipped journals)] --> WS[Warm standby]
  end
  J --> J2
  SVC --> CCPL[CCP link]
  SVC --> ARM[ARM / surveillance]
```

## 7. Fee engine

```mermaid
flowchart LR
  F["Fill: p, side, notional"] --> Z["z = s*(2p-1)"]
  Z --> CP[Certainty pressure]
  TAU["Time to event tau"] --> EW[Event window]
  Z --> EW
  LQ[Book liquidity L] --> LC[Liquidity stress]
  CONC[Concentration by family] --> CC[Concentration]
  CP --> R[Risk block R]
  EW --> R
  LC --> R
  CC --> R
  POS[Pre-fill position] --> U[Unwind fraction U]
  R --> FEE["f = max(f_min, B + R*(1-rhoU) - D - M)"]
  Z --> D[Contrarian discount]
  MQ[Maker quality] --> M[Maker rebate]
  D --> FEE
  M --> FEE
  B[Base] --> FEE
  FEE --> ATTR[Attribution per fill]
```

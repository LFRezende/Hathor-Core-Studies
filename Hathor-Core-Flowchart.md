# Hathor Core System Flowcharts

This document contains visual flowcharts representing the key flows in the Hathor Core system.

## 1. Node Startup Flow

```mermaid
flowchart TD
    A[CLI Entry Point] --> B[Parse Arguments]
    B --> C[Load Configuration]
    C --> D[Initialize HathorManager]
    D --> E[Setup Logging]
    E --> F[Initialize Storage]
    F --> G[Initialize P2P Manager]
    G --> H[Initialize Consensus Algorithm]
    H --> I[Initialize Mining Service]
    I --> J[Initialize Event Manager]
    J --> K[Start P2P Connections]
    K --> L[Start Mining Service]
    L --> M[Node Ready]
    
    style A fill:#e1f5fe
    style M fill:#c8e6c9
```

## 2. Transaction Processing Flow

```mermaid
flowchart TD
    A[New Transaction Received] --> B{Valid Format?}
    B -->|No| C[Reject Transaction]
    B -->|Yes| D[VertexHandler Process]
    D --> E{Basic Validation}
    E -->|Fail| F[Mark Invalid]
    E -->|Pass| G[Verification Service]
    G --> H{Transaction Valid?}
    H -->|No| I[Reject Transaction]
    H -->|Yes| J[Consensus Algorithm]
    J --> K[Update Storage]
    K --> L[Update Indexes]
    L --> M[Publish Events]
    M --> N[Propagate to Peers]
    
    style A fill:#e1f5fe
    style N fill:#c8e6c9
    style C fill:#ffcdd2
    style F fill:#ffcdd2
    style I fill:#ffcdd2
```

## 3. Block Mining Flow

```mermaid
flowchart TD
    A[Mining Service] --> B[Get Block Template]
    B --> C[Select Parent Block]
    C --> D[Select Parent Transactions]
    D --> E[Calculate Difficulty]
    E --> F[Start Nonce Iteration]
    F --> G{Hash < Target?}
    G -->|No| H[Increment Nonce]
    H --> F
    G -->|Yes| I[Block Found]
    I --> J[Submit Block]
    J --> K[Consensus Validation]
    K --> L{Block Valid?}
    L -->|No| M[Reject Block]
    L -->|Yes| N[Add to Storage]
    N --> O[Update Consensus]
    O --> P[Propagate Block]
    
    style A fill:#e1f5fe
    style P fill:#c8e6c9
    style M fill:#ffcdd2
```

## 4. P2P Synchronization Flow

```mermaid
flowchart TD
    A[Peer Discovery] --> B[Establish Connection]
    B --> C[Handshake Protocol]
    C --> D{Handshake Success?}
    D -->|No| E[Close Connection]
    D -->|Yes| F[Exchange Capabilities]
    F --> G[Start Sync Process]
    G --> H[Request Best Block]
    H --> I[Compare Heights]
    I --> J{Need Sync?}
    J -->|No| K[Regular Operation]
    J -->|Yes| L[Request Missing Blocks]
    L --> M[Download Blocks]
    M --> N[Validate Blocks]
    N --> O[Update Local State]
    O --> P[Continue Sync]
    P --> Q{Sync Complete?}
    Q -->|No| L
    Q -->|Yes| K
    
    style A fill:#e1f5fe
    style K fill:#c8e6c9
    style E fill:#ffcdd2
```

## 5. Consensus Update Flow

```mermaid
flowchart TD
    A[New Vertex Added] --> B[Get Vertex Metadata]
    B --> C{Is Block?}
    C -->|Yes| D[Block Consensus Algorithm]
    C -->|No| E[Transaction Consensus Algorithm]
    D --> F[Calculate Best Chain]
    F --> G{Chain Changed?}
    G -->|Yes| H[Detect Reorg]
    H --> I[Rollback State]
    I --> J[Reapply Transactions]
    G -->|No| K[Update Metadata]
    E --> L[Check Conflicts]
    L --> M{Has Conflicts?}
    M -->|Yes| N[Resolve Conflicts]
    M -->|No| O[Mark Executed]
    N --> P[Propagate Voidance]
    K --> Q[Update Indexes]
    J --> Q
    O --> Q
    P --> Q
    Q --> R[Publish Events]
    
    style A fill:#e1f5fe
    style R fill:#c8e6c9
```

## 6. Storage Operations Flow

```mermaid
flowchart TD
    A[Storage Request] --> B{Operation Type?}
    B -->|Read| C[Check Cache]
    B -->|Write| D[Validate Data]
    B -->|Delete| E[Check Dependencies]
    
    C --> F{Cache Hit?}
    F -->|Yes| G[Return Cached Data]
    F -->|No| H[Read from RocksDB]
    H --> I[Update Cache]
    I --> G
    
    D --> J[Serialize Data]
    J --> K[Write to RocksDB]
    K --> L[Update Indexes]
    L --> M[Update Cache]
    
    E --> N{Has Dependencies?}
    N -->|Yes| O[Reject Delete]
    N -->|No| P[Delete from RocksDB]
    P --> Q[Update Indexes]
    Q --> R[Update Cache]
    
    G --> S[Return Result]
    M --> S
    O --> S
    R --> S
    
    style A fill:#e1f5fe
    style S fill:#c8e6c9
    style O fill:#ffcdd2
```

## 7. Event System Flow

```mermaid
flowchart TD
    A[Event Triggered] --> B[Event Manager]
    B --> C[Create Event Object]
    C --> D[Validate Event]
    D --> E{Event Valid?}
    E -->|No| F[Log Error]
    E -->|Yes| G[Store in Queue]
    G --> H[Notify Subscribers]
    H --> I[Process Event Handlers]
    I --> J[Update System State]
    J --> K[Persist if Required]
    K --> L[Event Complete]
    
    style A fill:#e1f5fe
    style L fill:#c8e6c9
    style F fill:#ffcdd2
```

## 8. NanoContract Execution Flow

```mermaid
flowchart TD
    A[NanoContract Transaction] --> B[Get Blueprint]
    B --> C[Initialize Context]
    C --> D[Load Contract State]
    D --> E[Execute Contract]
    E --> F{Execution Success?}
    F -->|No| G[Mark Invalid]
    F -->|Yes| H[Update State]
    H --> I[Generate Events]
    I --> J[Persist Changes]
    J --> K[Update Consensus]
    K --> L[Contract Executed]
    
    style A fill:#e1f5fe
    style L fill:#c8e6c9
    style G fill:#ffcdd2
```

## 9. Component Interaction Diagram

```mermaid
graph TB
    subgraph "Interface Layer"
        CLI[CLI Interface]
        WS[WebSocket API]
        HTTP[HTTP API]
    end
    
    subgraph "Core Layer"
        HM[HathorManager]
        VH[VertexHandler]
        VS[VerificationService]
    end
    
    subgraph "Network Layer"
        P2P[P2P Manager]
        PROTO[Protocol]
        SYNC[Sync Agent]
    end
    
    subgraph "Consensus Layer"
        CA[Consensus Algorithm]
        BC[Block Consensus]
        TC[Transaction Consensus]
    end
    
    subgraph "Storage Layer"
        TS[Transaction Storage]
        IDX[Indexes]
        CACHE[Cache]
    end
    
    subgraph "Services"
        MS[Mining Service]
        DAA[DAA Algorithm]
        NC[NanoContracts]
    end
    
    CLI --> HM
    WS --> HM
    HTTP --> HM
    
    HM --> VH
    HM --> P2P
    HM --> CA
    HM --> TS
    HM --> MS
    
    VH --> VS
    VH --> CA
    VH --> TS
    
    P2P --> PROTO
    P2P --> SYNC
    
    CA --> BC
    CA --> TC
    
    TS --> IDX
    TS --> CACHE
    
    MS --> DAA
    HM --> NC
    
    style HM fill:#e3f2fd
    style CLI fill:#f3e5f5
    style WS fill:#f3e5f5
    style HTTP fill:#f3e5f5
```

## 10. Error Handling Flow

```mermaid
flowchart TD
    A[Error Occurs] --> B[Log Error]
    B --> C{Error Type?}
    C -->|Validation| D[Mark Invalid]
    C -->|Network| E[Retry Logic]
    C -->|Storage| F[Recovery Mode]
    C -->|Consensus| G[Rollback State]
    C -->|Critical| H[Node Shutdown]
    
    D --> I[Notify User]
    E --> J{Retry Success?}
    J -->|Yes| K[Continue Operation]
    J -->|No| L[Mark Failed]
    
    F --> M[Attempt Recovery]
    M --> N{Recovery Success?}
    N -->|Yes| K
    N -->|No| H
    
    G --> O[Restore Previous State]
    O --> P[Replay Transactions]
    P --> K
    
    L --> I
    H --> Q[Graceful Shutdown]
    
    style A fill:#fff3e0
    style K fill:#c8e6c9
    style H fill:#ffcdd2
    style Q fill:#ffcdd2
```

These flowcharts provide a visual representation of the key processes and interactions within the Hathor Core system. They help understand the complexity and flow of operations in this sophisticated blockchain/DAG hybrid system. 

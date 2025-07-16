# Hathor Core Architecture Documentation

## Overview

Hathor Core is a full-node implementation for the Hathor Network, a DAG-based cryptocurrency that combines blockchain and DAG technologies. This document provides a comprehensive overview of the codebase architecture, major components, and their interactions.

## Table of Contents

1. [System Architecture](#system-architecture)
2. [Core Components](#core-components)
3. [Data Flow](#data-flow)
4. [Design Patterns](#design-patterns)
5. [Key Modules](#key-modules)
6. [Network Protocol](#network-protocol)
7. [Consensus Mechanism](#consensus-mechanism)
8. [Storage Layer](#storage-layer)
9. [Advanced Features](#advanced-features)

## System Architecture

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Hathor Core Node                         │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │    CLI      │  │   WebSocket │  │     API     │         │
│  │  Interface  │  │   Interface │  │  Interface  │         │
│  └─────────────┘  └─────────────┘  └─────────────┘         │
├─────────────────────────────────────────────────────────────┤
│                    HathorManager                            │
│              (Central Orchestrator)                         │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │   P2P       │  │ Consensus   │  │  Storage    │         │
│  │  Manager    │  │ Algorithm   │  │   Layer     │         │
│  └─────────────┘  └─────────────┘  └─────────────┘         │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │  Mining     │  │  DAA        │  │ NanoContracts│        │
│  │  Service    │  │ Algorithm   │  │   Engine    │         │
│  └─────────────┘  └─────────────┘  └─────────────┘         │
├─────────────────────────────────────────────────────────────┤
│                    RocksDB Storage                          │
└─────────────────────────────────────────────────────────────┘
```

### Component Interaction Flow

```
1. Node Startup
   ┌─────────────┐
   │ CLI Entry   │
   └─────┬───────┘
         │
   ┌─────▼───────┐
   │ HathorManager│
   │  .start()   │
   └─────┬───────┘
         │
   ┌─────▼───────┐
   │ Initialize  │
   │ Components  │
   └─────────────┘

2. Transaction Processing
   ┌─────────────┐
   │ New TX      │
   └─────┬───────┘
         │
   ┌─────▼───────┐
   │ VertexHandler│
   │ .process()  │
   └─────┬───────┘
         │
   ┌─────▼───────┐
   │ Verification│
   │ Service     │
   └─────┬───────┘
         │
   ┌─────▼───────┐
   │ Consensus   │
   │ Algorithm   │
   └─────┬───────┘
         │
   ┌─────▼───────┐
   │ Storage     │
   │ Update      │
   └─────────────┘
```

## Core Components

### 1. HathorManager

**Location**: `hathor/manager.py`

The central orchestrator that coordinates all node operations:

- **Responsibilities**:
  - Node lifecycle management (start/stop)
  - Transaction and block processing
  - Mining coordination
  - Component initialization and coordination
  - Event publishing and subscription

- **Key Methods**:
  - `start()`: Initialize and start all components
  - `stop()`: Graceful shutdown
  - `on_new_tx()`: Process incoming transactions
  - `submit_block()`: Process mined blocks
  - `generate_mining_block()`: Create block templates for mining

### 2. Transaction System

**Location**: `hathor/transaction/`

The core data structures representing the DAG:

#### BaseTransaction (Abstract Base Class)
- **Location**: `hathor/transaction/base_transaction.py`
- **Purpose**: Base class for all vertices in the DAG
- **Key Properties**:
  - `hash`: Unique identifier
  - `weight`: Proof-of-work weight
  - `timestamp`: Creation time
  - `parents`: DAG connections
  - `inputs/outputs`: Transaction data

#### Transaction Types
- **Block**: `hathor/transaction/block.py`
- **Transaction**: `hathor/transaction/transaction.py`
- **TokenCreationTransaction**: `hathor/transaction/token_creation_tx.py`
- **MergeMinedBlock**: `hathor/transaction/merge_mined_block.py`
- **PoaBlock**: `hathor/transaction/poa/`

### 3. Consensus Algorithm

**Location**: `hathor/consensus/`

Implements the DAG consensus mechanism:

#### ConsensusAlgorithm
- **Location**: `hathor/consensus/consensus.py`
- **Purpose**: Determines which transactions are executed vs voided
- **Key Concepts**:
  - **Voided_by**: Set of hashes causing voidance
  - **Best chain**: Longest valid blockchain
  - **Conflict resolution**: Handles double-spending

#### Block Consensus
- **Location**: `hathor/consensus/block_consensus.py`
- **Purpose**: Handles blockchain consensus rules
- **Features**:
  - Best chain selection
  - Reorg detection
  - Checkpoint validation

#### Transaction Consensus
- **Location**: `hathor/consensus/transaction_consensus.py`
- **Purpose**: Handles DAG transaction consensus
- **Features**:
  - Parent selection validation
  - Weight calculation
  - Conflict detection

### 4. P2P Network

**Location**: `hathor/p2p/`

Manages peer-to-peer communication:

#### ConnectionsManager
- **Location**: `hathor/p2p/manager.py`
- **Purpose**: Manages all peer connections
- **Features**:
  - Connection lifecycle management
  - Peer discovery
  - Sync coordination
  - Rate limiting

#### HathorProtocol
- **Location**: `hathor/p2p/protocol.py`
- **Purpose**: Implements the P2P protocol
- **Features**:
  - Message serialization/deserialization
  - Handshake protocol
  - State machine management

#### Sync Mechanisms
- **Location**: `hathor/p2p/sync_v2/`
- **Purpose**: Efficient blockchain/DAG synchronization
- **Features**:
  - Incremental sync
  - Parallel downloads
  - State verification

### 5. Storage Layer

**Location**: `hathor/storage/` and `hathor/transaction/storage/`

#### TransactionStorage
- **Location**: `hathor/transaction/storage/transaction_storage.py`
- **Purpose**: Abstract storage interface
- **Features**:
  - CRUD operations for transactions
  - Index management
  - Metadata handling

#### RocksDB Implementation
- **Location**: `hathor/transaction/storage/rocksdb_storage.py`
- **Purpose**: Persistent storage backend
- **Features**:
  - ACID transactions
  - Column families for different data types
  - Efficient range queries

#### Indexes
- **Location**: `hathor/indexes/`
- **Purpose**: Maintain various indexes for efficient queries
- **Types**:
  - Height index
  - Address index
  - Token index
  - Tips index

### 6. Mining System

**Location**: `hathor/mining/`

#### CpuMiningService
- **Location**: `hathor/mining/cpu_mining_service.py`
- **Purpose**: CPU-based mining implementation
- **Features**:
  - Block template generation
  - Nonce iteration
  - Difficulty adjustment

#### Block Templates
- **Location**: `hathor/mining/`
- **Purpose**: Pre-configured block structures for mining
- **Features**:
  - Parent selection
  - Transaction inclusion
  - Merkle root calculation

### 7. Difficulty Adjustment Algorithm (DAA)

**Location**: `hathor/daa.py`

#### DifficultyAdjustmentAlgorithm
- **Purpose**: Dynamic difficulty adjustment
- **Algorithm**: Based on recent block timestamps and weights
- **Features**:
  - Weight calculation
  - Time-based adjustment
  - Minimum weight enforcement

## Data Flow

### 1. Transaction Processing Flow

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Network   │───▶│ VertexHandler│───▶│ Verification│
│   Input     │    │             │    │   Service   │
└─────────────┘    └─────────────┘    └─────────────┘
                           │                   │
                           ▼                   ▼
                   ┌─────────────┐    ┌─────────────┐
                   │   Storage   │    │  Consensus  │
                   │   Layer     │    │ Algorithm   │
                   └─────────────┘    └─────────────┘
                           │                   │
                           ▼                   ▼
                   ┌─────────────┐    ┌─────────────┐
                   │   Indexes   │    │   PubSub    │
                   │   Update    │    │   Events    │
                   └─────────────┘    └─────────────┘
```

### 2. Block Mining Flow

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Manager   │───▶│ Block       │───▶│ Mining      │
│             │    │ Template    │    │ Service     │
└─────────────┘    └─────────────┘    └─────────────┘
                           │                   │
                           ▼                   ▼
                   ┌─────────────┐    ┌─────────────┐
                   │   DAA       │    │   Nonce     │
                   │ Algorithm   │    │  Iteration  │
                   └─────────────┘    └─────────────┘
                           │                   │
                           ▼                   ▼
                   ┌─────────────┐    ┌─────────────┐
                   │   Weight    │    │   Hash      │
                   │ Calculation │    │  Validation │
                   └─────────────┘    └─────────────┘
```

### 3. P2P Synchronization Flow

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Peer      │───▶│ Protocol    │───▶│ Sync Agent  │
│ Discovery   │    │ Handshake   │    │             │
└─────────────┘    └─────────────┘    └─────────────┘
                           │                   │
                           ▼                   ▼
                   ┌─────────────┐    ┌─────────────┐
                   │   State     │    │   Data      │
                   │ Management  │    │  Exchange   │
                   └─────────────┘    └─────────────┘
                           │                   │
                           ▼                   ▼
                   ┌─────────────┐    ┌─────────────┐
                   │   Storage   │    │   Consensus │
                   │   Update    │    │   Update    │
                   └─────────────┘    └─────────────┘
```

## Design Patterns

### 1. Factory Pattern

Used extensively for creating different types of objects:

```python
# Example: Transaction creation
class TxVersion(IntEnum):
    REGULAR_BLOCK = 0
    REGULAR_TRANSACTION = 1
    TOKEN_CREATION_TRANSACTION = 2
    # ...

def get_cls(self) -> type['BaseTransaction']:
    cls_map: dict[TxVersion, type[BaseTransaction]] = {
        TxVersion.REGULAR_BLOCK: Block,
        TxVersion.REGULAR_TRANSACTION: Transaction,
        # ...
    }
    return cls_map.get(self)
```

### 2. Observer Pattern (PubSub)

Used for event-driven communication:

```python
# Example: Event publishing
self.pubsub.publish(HathorEvents.CONSENSUS_TX_UPDATE, tx=tx)

# Example: Event subscription
self.pubsub.subscribe(HathorEvents.CONSENSUS_TX_UPDATE, self.on_tx_update)
```

### 3. Strategy Pattern

Used for different algorithms:

```python
# Example: Different consensus algorithms
class ConsensusAlgorithm:
    def __init__(self, block_algorithm_factory, transaction_algorithm_factory):
        self.block_algorithm_factory = block_algorithm_factory
        self.transaction_algorithm_factory = transaction_algorithm_factory
```

### 4. State Machine Pattern

Used in P2P protocol:

```python
# Example: Protocol states
class HathorProtocol:
    def __init__(self):
        self.state = HandshakeState()
    
    def change_state(self, new_state):
        self.state = new_state
```

### 5. Repository Pattern

Used for data access:

```python
# Example: Storage abstraction
class TransactionStorage(ABC):
    @abstractmethod
    def save_transaction(self, tx: BaseTransaction) -> None:
        pass
    
    @abstractmethod
    def get_transaction(self, tx_id: VertexId) -> BaseTransaction:
        pass
```

## Key Modules

### 1. Configuration System

**Location**: `hathor/conf/`

- **Settings**: Network-specific configuration
- **Environment**: Runtime environment detection
- **Validation**: Configuration validation

### 2. Event System

**Location**: `hathor/event/`

- **EventManager**: Central event coordination
- **EventQueue**: Persistent event storage
- **EventTypes**: Predefined event types

### 3. Feature Activation

**Location**: `hathor/feature_activation/`

- **BitSignalingService**: Feature flag management
- **Activation**: Gradual feature rollout
- **Validation**: Feature compatibility checks

### 4. Metrics and Monitoring

**Location**: `hathor/metrics.py`, `hathor/prometheus.py`

- **Performance metrics**: CPU, memory, network
- **Business metrics**: Transactions, blocks, peers
- **Prometheus integration**: External monitoring

### 5. Utilities

**Location**: `hathor/util.py`, `hathor/utils/`

- **Cryptographic utilities**: Hashing, signing
- **Serialization**: Binary encoding/decoding
- **Validation**: Input validation helpers

## Network Protocol

### Message Types

1. **Handshake Messages**
   - Version exchange
   - Capability negotiation
   - Peer identification

2. **Sync Messages**
   - Block requests/responses
   - Transaction requests/responses
   - State synchronization

3. **Propagation Messages**
   - New transaction announcements
   - New block announcements
   - Inv/GetData patterns

### Protocol Features

- **TLS Support**: Encrypted communication
- **Rate Limiting**: Anti-spam protection
- **Peer Discovery**: Automatic peer finding
- **Connection Management**: Automatic reconnection

## Consensus Mechanism

### DAG Consensus Rules

1. **Parent Selection**
   - Transactions must reference 2 transaction parents
   - Blocks must reference 1 block parent + 2 transaction parents
   - Parents must be valid and not voided

2. **Weight Calculation**
   - Blocks: Based on difficulty adjustment algorithm
   - Transactions: Based on size and value
   - Accumulated weight: Sum of own weight + parent weights

3. **Conflict Resolution**
   - Double-spending detection
   - Voidance propagation
   - Best chain selection

### Blockchain Consensus

1. **Best Chain Selection**
   - Longest valid chain
   - Weight-based tie-breaking
   - Checkpoint validation

2. **Reorg Handling**
   - Common ancestor detection
   - State rollback
   - Event emission

## Storage Layer

### Data Organization

1. **Column Families**
   - Transactions: Main transaction data
   - Metadata: Transaction metadata
   - Indexes: Various indexes
   - Events: Event queue data

2. **Indexes**
   - Height index: Block height tracking
   - Address index: Address-based queries
   - Token index: Token-based queries
   - Tips index: DAG tips tracking

### Performance Optimizations

1. **Caching**
   - In-memory transaction cache
   - Metadata caching
   - Index caching

2. **Batch Operations**
   - Bulk transaction saves
   - Index updates
   - Event processing

## Advanced Features

### 1. NanoContracts

**Location**: `hathor/nanocontracts/`

- **Smart Contracts**: On-chain program execution
- **Blueprint System**: Contract templates
- **Execution Engine**: Sandboxed execution
- **Storage**: Contract state persistence

### 2. Merged Mining

**Location**: `hathor/merged_mining/`

- **Bitcoin Integration**: Merge mining with Bitcoin
- **AuxPoW**: Auxiliary proof-of-work
- **Difficulty Sharing**: Shared mining difficulty

### 3. Proof-of-Authority (PoA)

**Location**: `hathor/consensus/poa/`

- **Authority Blocks**: Special block type
- **Validator Management**: Authority node management
- **Consensus Integration**: PoA consensus rules

### 4. WebSocket API

**Location**: `hathor/websocket/`

- **Real-time Updates**: Live transaction/block updates
- **Subscription System**: Event subscriptions
- **Admin Interface**: Node management

### 5. Health Checks

**Location**: `hathor/healthcheck/`

- **Node Health**: System health monitoring
- **Sync Status**: Synchronization status
- **Peer Health**: Peer connection health

## Development and Testing

### Testing Structure

**Location**: `tests/`

- **Unit Tests**: Individual component testing
- **Integration Tests**: Component interaction testing
- **Slow Tests**: Performance and stress testing

### Development Tools

- **CLI Tools**: Various command-line utilities
- **Debugging**: Profiling and debugging tools
- **Simulation**: Network simulation tools

## Conclusion

Hathor Core is a sophisticated full-node implementation that combines blockchain and DAG technologies. Its modular architecture allows for easy extension and maintenance, while its robust consensus mechanism ensures network security and consistency.

The codebase follows modern software engineering practices with clear separation of concerns, comprehensive testing, and extensive documentation. The use of design patterns and abstractions makes the code maintainable and extensible.

Key strengths of the architecture include:
- **Modularity**: Clear component boundaries
- **Extensibility**: Easy to add new features
- **Reliability**: Comprehensive error handling
- **Performance**: Optimized for high throughput
- **Security**: Robust consensus and validation

This architecture provides a solid foundation for the Hathor Network's continued growth and evolution. 

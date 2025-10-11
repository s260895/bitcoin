# Bitcoin Core Analysis Guide for Minimal Implementation

## Project Overview
This document provides comprehensive context for analyzing Bitcoin Core source code with an emphasis on minimal, non-commercial implementation. This guide is designed for AI-assisted code analysis using Claude Code and assumes the reader has full-stack engineering experience.

**Core Philosophy**: Run Bitcoin with zero commercial dependencies, using only open-source tools and understanding every critical path through direct code examination.

## Repository Structure

### Critical Directories
```
bitcoin/
├── src/                    # Core implementation (C++)
│   ├── consensus/         # Consensus-critical code (DO NOT MODIFY)
│   │   ├── validation.h  # Block/tx validation rules
│   │   └── merkle.cpp    # Merkle tree computation
│   ├── primitives/        # Basic data structures
│   │   ├── block.h       # CBlock, CBlockHeader
│   │   └── transaction.h # CTransaction, CTxIn, CTxOut
│   ├── validation.cpp     # Main validation logic (~8000 lines)
│   ├── net.cpp           # P2P networking layer
│   ├── net_processing.cpp # Message handling
│   ├── init.cpp          # Initialization sequence
│   ├── chainparams.cpp   # Network parameters (mainnet/testnet/regtest)
│   ├── script/           # Script interpreter
│   │   ├── interpreter.cpp # Script execution engine
│   │   └── standard.cpp   # Standard script templates
│   ├── wallet/           # Wallet implementation (optional component)
│   ├── rpc/              # RPC server implementation
│   ├── qt/               # GUI code (can be excluded)
│   └── test/             # Unit tests
├── test/                  # Python functional tests
├── doc/                   # Documentation
├── contrib/              # Helper scripts
└── depends/              # Dependency build system (avoid if minimal)
```

## Minimal Build Strategy

### 1. Core Dependencies (Absolutely Required)
```bash
# These are ALL open source, zero commercial software
gcc/g++ or clang     # Compiler (GNU or LLVM)
cmake                # Build system (replacing autotools)
make                 # Build automation
libboost             # C++ utilities (Boost Software License)
libevent             # Asynchronous networking (BSD license)
libssl/libcrypto     # OpenSSL for cryptography (Apache License)
```

### 2. Minimal Build Commands
```bash
# Clone and enter your fork
git clone https://github.com/YOUR_USERNAME/bitcoin.git
cd bitcoin

# Minimal CMake configuration (no GUI, no wallet initially)
cmake -B build \
  -DBUILD_GUI=OFF \
  -DENABLE_WALLET=OFF \
  -DWITH_BDB=OFF \
  -DBUILD_TESTS=OFF \
  -DBUILD_BENCH=OFF

# Build only bitcoind and bitcoin-cli
cmake --build build --target bitcoind
cmake --build build --target bitcoin-cli
```

### 3. Absolute Minimal Runtime
```bash
# Run in regtest mode (local blockchain, no network)
./build/src/bitcoind -regtest -daemon \
  -datadir=/tmp/btc-minimal \
  -nolisten \
  -noconnect \
  -maxconnections=0

# Interact via CLI
./build/src/bitcoin-cli -regtest -datadir=/tmp/btc-minimal getblockchaininfo
```

## Code Analysis Roadmap

### Phase 1: Entry Points and Initialization
**Start Here for Understanding Flow**

1. **src/bitcoind.cpp** (150 lines)
   - Entry point for daemon
   - Parses arguments, calls AppInit()

2. **src/init.cpp::AppInit()** (200 lines)
   - Master initialization sequence
   - Sets up logging, network, validation

3. **src/init.cpp::AppInitMain()** (2000 lines)
   - CRITICAL: Step-by-step initialization
   - Database opening, chain state loading
   - Network thread startup

### Phase 2: Core Data Structures
**Understanding Bitcoin's Memory Model**

1. **src/primitives/block.h**
   ```cpp
   class CBlockHeader {
       int32_t nVersion;
       uint256 hashPrevBlock;
       uint256 hashMerkleRoot;
       uint32_t nTime;
       uint32_t nBits;
       uint32_t nNonce;
   };
   
   class CBlock : public CBlockHeader {
       std::vector<CTransactionRef> vtx;
   };
   ```

2. **src/primitives/transaction.h**
   ```cpp
   class CTransaction {
       const std::vector<CTxIn> vin;
       const std::vector<CTxOut> vout;
       const int32_t nVersion;
       const uint32_t nLockTime;
   };
   ```

3. **src/chain.h - CBlockIndex**
   - In-memory representation of blockchain
   - Links blocks, tracks heights, validation status

### Phase 3: Validation Engine
**The Heart of Bitcoin**

1. **src/validation.cpp::CheckBlock()** (500 lines)
   - Validates block structure
   - Checks proof of work
   - Verifies merkle root

2. **src/validation.cpp::ConnectBlock()** (800 lines)
   - Applies block to chain state
   - Updates UTXO set
   - Validates all transactions

3. **src/validation.cpp::ActivateBestChain()** (400 lines)
   - Handles chain reorganization
   - Finds best chain, switches if needed

### Phase 4: Network Layer
**P2P Protocol Implementation**

1. **src/net.cpp::CConnman::Start()** (300 lines)
   - Initializes network threads
   - Opens listening sockets

2. **src/net_processing.cpp::ProcessMessage()** (3000 lines)
   - Handles all P2P messages
   - Key messages: version, inv, getdata, block, tx

3. **src/net.h::CNode**
   - Represents peer connection
   - Manages send/receive buffers

### Phase 5: Script System
**Bitcoin's Programming Language**

1. **src/script/interpreter.cpp::EvalScript()** (1500 lines)
   - Stack-based script executor
   - Implements all opcodes
   - CRITICAL for understanding Bitcoin's programmability

2. **src/script/standard.cpp**
   - Standard transaction types
   - P2PKH, P2SH, P2WPKH, P2WSH templates

### Phase 6: Consensus Rules
**Never Modify Without Extreme Caution**

1. **src/consensus/validation.h**
   - Consensus error codes
   - Block/transaction validation rules

2. **src/consensus/merkle.cpp**
   - Merkle tree construction
   - Used in block validation

3. **src/consensus/params.h**
   - Network consensus parameters
   - Difficulty adjustment, block times

## Critical Code Paths to Trace

### 1. Receiving a New Block
```
net_processing.cpp::ProcessMessage("block")
  ↓
validation.cpp::ProcessNewBlock()
  ↓
validation.cpp::CheckBlock()
  ↓
validation.cpp::AcceptBlock()
  ↓
validation.cpp::ConnectBlock()
  ↓
validation.cpp::UpdateTip()
```

### 2. Validating a Transaction
```
net_processing.cpp::ProcessMessage("tx")
  ↓
validation.cpp::AcceptToMemoryPool()
  ↓
consensus/validation.cpp::CheckTransaction()
  ↓
script/interpreter.cpp::VerifyScript()
  ↓
script/interpreter.cpp::EvalScript()
```

### 3. Mining a Block (Regtest)
```
rpc/mining.cpp::generatetoaddress()
  ↓
miner.cpp::GenerateBlock()
  ↓
miner.cpp::CreateNewBlock()
  ↓
validation.cpp::TestBlockValidity()
  ↓
validation.cpp::ProcessNewBlock()
```

## Key Algorithms to Understand

### 1. Proof of Work Validation
```cpp
// src/pow.cpp::CheckProofOfWork()
bool CheckProofOfWork(uint256 hash, unsigned int nBits, const Consensus::Params& params) {
    uint256 target;
    target.SetCompact(nBits);
    if (hash > target) return false;
    return true;
}
```

### 2. UTXO Set Management
```cpp
// src/coins.h - CCoinsView interface
class CCoinsView {
    virtual bool GetCoin(const COutPoint &outpoint, Coin &coin) const;
    virtual bool HaveCoin(const COutPoint &outpoint) const;
    // ... manages unspent transaction outputs
};
```

### 3. Signature Verification
```cpp
// src/script/interpreter.cpp::EvalChecksig()
// Implements ECDSA signature verification
// Core of Bitcoin's security model
```

## Minimal Testing Approach

### 1. Unit Tests (C++)
```bash
# Build and run minimal unit tests
cmake --build build --target test_bitcoin
./build/src/test/test_bitcoin --run_test=validation_tests
```

### 2. Functional Tests (Python)
```bash
# Run single functional test
./test/functional/p2p_segwit.py
```

### 3. Regtest Experiments
```bash
# Generate blocks manually
./build/src/bitcoin-cli -regtest generatetoaddress 101 $(./build/src/bitcoin-cli -regtest getnewaddress)

# Create raw transaction
./build/src/bitcoin-cli -regtest createrawtransaction '[]' '{"ADDRESS":0.1}'
```

## Configuration for Analysis

### Minimal bitcoin.conf
```ini
# Place in datadir
regtest=1
server=1
rpcuser=test
rpcpassword=test
nolisten=1
noconnect=1
debug=1
printtoconsole=1
```

### Debug Categories for Deep Diving
```bash
# Enable specific debug output
./build/src/bitcoind -debug=net     # Network messages
./build/src/bitcoind -debug=mempool # Transaction pool
./build/src/bitcoind -debug=validation # Block/tx validation
./build/src/bitcoind -debug=rpc     # RPC calls
```

## Code Analysis Questions for Claude Code

### Architecture Questions
1. How does the UTXO set get updated when a new block arrives?
2. What's the exact sequence of validation checks for a transaction?
3. How does the mempool prioritize transactions for inclusion?
4. Where does the P2P protocol handle version negotiation?
5. How are chain reorganizations detected and processed?

### Security-Critical Paths
1. Where is double-spend prevention enforced?
2. How is the 21 million coin limit enforced in code?
3. Where are signature malleability checks performed?
4. How does the code prevent inflation bugs?
5. Where is the longest chain rule implemented?

### Performance-Critical Sections
1. Which data structures are cached and why?
2. How is the UTXO database indexed for fast lookups?
3. Where are the database write batches assembled?
4. How does parallel script validation work?
5. What triggers database flushes to disk?

## Build Variations for Exploration

### 1. Absolute Minimum (No Wallet, No GUI)
```bash
cmake -B build-minimal \
  -DBUILD_GUI=OFF \
  -DENABLE_WALLET=OFF \
  -DBUILD_TESTS=OFF
```

### 2. With Wallet Support (No GUI)
```bash
cmake -B build-wallet \
  -DBUILD_GUI=OFF \
  -DENABLE_WALLET=ON \
  -DWITH_SQLITE=ON
```

### 3. Debug Build with Symbols
```bash
cmake -B build-debug \
  -DCMAKE_BUILD_TYPE=Debug \
  -DENABLE_DEBUG=ON
```

### 4. Static Analysis Build
```bash
cmake -B build-analysis \
  -DCMAKE_CXX_FLAGS="-fsanitize=address -fsanitize=undefined"
```

## Advanced Analysis Topics

### 1. Consensus-Critical Code Sections
- Everything in src/consensus/ - NEVER modify
- src/validation.cpp::ConnectBlock() - Block application
- src/script/interpreter.cpp - Script execution
- src/pow.cpp - Proof of work validation

### 2. Performance Bottlenecks
- UTXO cache management (src/coins.cpp)
- Signature validation (can be parallelized)
- Database I/O (leveldb interactions)
- Network message processing

### 3. Attack Vectors to Study
- DoS prevention mechanisms
- Eclipse attack prevention
- Transaction malleability handling
- Sybil attack mitigation
- Blockchain reorganization limits

## Project Analysis Workflow

### Week 1: Foundation
1. Build minimal bitcoind
2. Trace through AppInit() completely
3. Understand CBlock and CTransaction structures
4. Run in regtest, generate blocks manually

### Week 2: Validation
1. Deep dive validation.cpp
2. Trace CheckBlock() and ConnectBlock()
3. Understand UTXO set updates
4. Study chain reorganization logic

### Week 3: Networking
1. Analyze P2P message handling
2. Understand peer management
3. Study inv/getdata flow
4. Examine DoS prevention

### Week 4: Script System
1. Master EvalScript() operation
2. Understand all standard scripts
3. Trace signature verification
4. Study SegWit script changes

### Week 5: Advanced Topics
1. Mempool management
2. Fee estimation
3. Wallet functionality (if enabled)
4. RPC server implementation

## Notes for Claude Code Integration

When analyzing with Claude Code:
1. Start with specific functions, not entire files
2. Trace execution paths from entry points
3. Focus on state changes and side effects
4. Identify invariants that must hold
5. Look for comments marked "consensus critical"
6. Pay attention to lock acquisition order
7. Note any TODO or XXX comments
8. Identify magic numbers and their meaning
9. Map data flow through the system
10. Document any non-obvious design decisions

## Final Notes

This document provides a roadmap for understanding Bitcoin Core through code analysis. The minimal approach eliminates all commercial dependencies while maintaining full functionality. Focus on understanding the validation engine first - it's the heart of Bitcoin. Everything else supports this core purpose: validating blocks and transactions according to consensus rules.

Remember: The code IS the specification. Comments can be wrong, documentation can be outdated, but the code that runs on the network defines Bitcoin.

# Bitcoin Core Exploration Guide

## Purpose
This document provides a discovery-based approach to analyzing Bitcoin Core source code without assuming any specific structure. The code itself is the specification - we'll explore what actually exists rather than what documentation claims should exist.

## Core Philosophy
- **Zero commercial dependencies** - Only open-source tools
- **Code is truth** - Don't trust documentation, verify in source
- **Minimal first** - Build and run the simplest version possible
- **Explore systematically** - Use search and grep to understand structure

## Initial Discovery Commands

### 1. Understand Repository Structure
```bash
# See what's actually in the repository
ls -la                              # Top level
find . -type d -maxdepth 2          # Directory structure
find . -name "*.cpp" | wc -l        # Count of C++ files
find . -name "*.h" | wc -l          # Count of header files

# Look for documentation
find . -name "README*" -o -name "*.md" | head -20

# Find build files
find . -name "CMakeLists.txt" -o -name "Makefile.am" -o -name "configure.ac" | head -10
```

### 2. Locate Entry Points
```bash
# Find main() function - where everything starts
grep -r "int main" --include="*.cpp" .

# Find initialization patterns
grep -r "AppInit" --include="*.cpp" .
grep -r "InitLogging" --include="*.cpp" .
grep -r "InitParameterInteraction" --include="*.cpp" .

# Find daemon/CLI entry points
grep -r "bitcoind" --include="*.cpp" . | head -20
grep -r "bitcoin-cli" --include="*.cpp" . | head -20
```

### 3. Discover Core Components
```bash
# Blockchain structures
grep -r "class CBlock\b" --include="*.h" .
grep -r "class CBlockHeader" --include="*.h" .
grep -r "class CChain" --include="*.h" .

# Transaction structures
grep -r "class CTransaction" --include="*.h" .
grep -r "class CTxIn\b" --include="*.h" .
grep -r "class CTxOut\b" --include="*.h" .
grep -r "class COutPoint" --include="*.h" .

# UTXO management
grep -r "class CCoinsView" --include="*.h" .
grep -r "UTXO" --include="*.cpp" .
grep -r "UpdateCoins" --include="*.cpp" .
```

## Build Strategies

### Option 1: Modern CMake Build
```bash
# Check if CMake is available
find . -name "CMakeLists.txt"

# If found, try minimal build
cmake -B build \
  -DBUILD_GUI=OFF \
  -DENABLE_WALLET=OFF \
  -DBUILD_TESTS=OFF \
  -DBUILD_BENCH=OFF

cmake --build build -j6

# See what was built
find build -type f -executable 2>/dev/null | grep -v "\.sh$"
```

### Option 2: Legacy Autotools Build
```bash
# Check if autotools setup exists
find . -name "configure.ac" -o -name "autogen.sh"

# If found, try:
./autogen.sh
./configure --without-gui --disable-wallet --disable-tests
make -j6

# Check src directory for binaries
find src -type f -executable | grep -E "bitcoin"
```

### Option 3: Direct Compilation (Last Resort)
```bash
# Find all source files
find . -name "*.cpp" -path "*/src/*" > source_files.txt

# Try direct compilation of essential files
g++ -std=c++17 -O2 -pthread -I./src \
    $(find . -name "bitcoind.cpp") \
    $(find . -name "init.cpp") \
    -o bitcoind_minimal
```

## Critical Code Paths to Discover

### 1. Validation Logic
```bash
# Block validation
grep -r "CheckBlock" --include="*.cpp" .
grep -r "AcceptBlock" --include="*.cpp" .
grep -r "ConnectBlock" --include="*.cpp" .
grep -r "DisconnectBlock" --include="*.cpp" .

# Transaction validation
grep -r "CheckTransaction" --include="*.cpp" .
grep -r "AcceptToMemoryPool" --include="*.cpp" .
grep -r "VerifyScript" --include="*.cpp" .
```

### 2. Consensus Rules
```bash
# Find consensus directory
find . -type d -name "*consensus*"

# Consensus parameters
grep -r "MaxBlockSize" --include="*.h" .
grep -r "COIN_SUPPLY" --include="*.h" .
grep -r "COINBASE_MATURITY" --include="*.h" .
grep -r "MAX_MONEY" --include="*.h" .

# Proof of Work
grep -r "CheckProofOfWork" --include="*.cpp" .
grep -r "GetNextWorkRequired" --include="*.cpp" .
grep -r "CalculateNextWorkRequired" --include="*.cpp" .
```

### 3. Network Layer
```bash
# P2P message processing
grep -r "ProcessMessage" --include="*.cpp" . | grep -v test
grep -r "NetMsgType::" --include="*.cpp" . | head -20

# Network message types
grep -r "NetMsgType::BLOCK" --include="*.h" .
grep -r "NetMsgType::TX" --include="*.h" .
grep -r "NetMsgType::INV" --include="*.h" .

# Peer management
grep -r "class CNode" --include="*.h" .
grep -r "class CConnman" --include="*.h" .
```

### 4. Script System
```bash
# Script interpreter
grep -r "EvalScript" --include="*.cpp" .
grep -r "VerifyScript" --include="*.cpp" .

# Script opcodes
grep -r "enum opcodetype" --include="*.h" .
grep -r "OP_CHECKSIG" --include="*.h" .
grep -r "OP_DUP" --include="*.h" .

# Script patterns
grep -r "Solver" --include="*.cpp" . | grep -v test
grep -r "P2PKH\|P2SH\|P2WPKH" --include="*.cpp" .
```

## Minimal Testing Environment

### 1. Regtest Setup (Local Blockchain)
```bash
# Start Bitcoin in regression test mode
./bitcoind -regtest -daemon \
  -datadir=/tmp/btc-minimal \
  -server=1 \
  -rpcuser=test \
  -rpcpassword=test \
  -nolisten=1 \
  -noconnect=1

# Generate blocks instantly
./bitcoin-cli -regtest -datadir=/tmp/btc-minimal \
  -rpcuser=test -rpcpassword=test \
  generatetoaddress 101 $(./bitcoin-cli -regtest -datadir=/tmp/btc-minimal -rpcuser=test -rpcpassword=test getnewaddress)

# Query blockchain
./bitcoin-cli -regtest -datadir=/tmp/btc-minimal \
  -rpcuser=test -rpcpassword=test \
  getblockchaininfo

# Stop daemon
./bitcoin-cli -regtest -datadir=/tmp/btc-minimal \
  -rpcuser=test -rpcpassword=test stop
```

### 2. Debug Configuration
```bash
# Create minimal bitcoin.conf
cat > /tmp/btc-minimal/bitcoin.conf << EOF
regtest=1
server=1
rpcuser=test
rpcpassword=test
debug=1
printtoconsole=1
nolisten=1
noconnect=1
EOF

# Run with specific debug categories
./bitcoind -regtest -datadir=/tmp/btc-minimal \
  -debug=net \
  -debug=validation \
  -debug=mempool
```

## Key Concepts to Explore

### 1. Data Structures
- **Block**: Header + list of transactions
- **Transaction**: Version + inputs + outputs + locktime
- **UTXO**: Unspent transaction outputs (the actual "coins")
- **Mempool**: Pool of unconfirmed transactions
- **Chain**: Linked list of blocks via hash pointers

### 2. Critical Algorithms
- **Proof of Work**: SHA256(SHA256(block_header)) < target
- **Difficulty Adjustment**: Every 2016 blocks
- **UTXO Set Updates**: Apply/revert transactions
- **Script Execution**: Stack-based verification language
- **Merkle Trees**: Binary tree of transaction hashes

### 3. Security Invariants
- No double spending (UTXO can only be spent once)
- No inflation (sum of outputs <= sum of inputs + coinbase)
- Longest valid chain wins (most accumulated work)
- Signatures must be valid (ECDSA verification)
- Time locks must be satisfied

## Questions for Claude Code

### Architecture Discovery
1. "Show me the main() function and trace the initialization sequence"
2. "Find where blocks are stored in memory and on disk"
3. "How does the mempool decide which transactions to include?"
4. "Where is the UTXO set implemented?"
5. "Show me the peer-to-peer message handling loop"

### Consensus Critical
1. "Where is the 21 million Bitcoin limit enforced?"
2. "Show me the block validation sequence"
3. "How does script verification work?"
4. "Where are chain reorganizations handled?"
5. "Find the difficulty adjustment algorithm"

### Performance Critical
1. "What data structures are cached and why?"
2. "How is parallel validation implemented?"
3. "Where are database writes batched?"
4. "Show me the signature cache implementation"
5. "How does the mempool limit memory usage?"

## Analysis Workflow

### Week 1: Build and Run
- Get minimal build working
- Run in regtest mode
- Generate and query blocks
- Understand RPC interface

### Week 2: Core Data Structures
- Find and understand CBlock, CTransaction
- Trace UTXO set structure
- Understand chain organization
- Study mempool implementation

### Week 3: Validation Logic
- Trace block validation path
- Understand transaction verification
- Study script interpreter
- Find consensus rules

### Week 4: Network Layer
- Understand P2P protocol
- Trace message flow
- Study peer management
- Examine DoS protections

### Week 5: Advanced Topics
- Mining and block construction
- Wallet functionality (if enabled)
- RPC server implementation
- Database layer

## Debugging Techniques

### 1. Add Debug Output
```cpp
// Add to any cpp file
#include <iostream>
std::cout << "DEBUG: Reached function X with value: " << value << std::endl;

// Or use Bitcoin's logging
LogPrintf("DEBUG: Block height %d hash %s\n", height, hash.ToString());
```

### 2. GDB Debugging
```bash
# Compile with debug symbols
cmake -B build-debug -DCMAKE_BUILD_TYPE=Debug
cmake --build build-debug

# Run under GDB
gdb ./build-debug/src/bitcoind
(gdb) break main
(gdb) run -regtest -datadir=/tmp/btc-debug
(gdb) backtrace
```

### 3. Trace Execution
```bash
# Use strace to see system calls
strace -f -e trace=open,read,write ./bitcoind -regtest 2>&1 | grep -v ENOENT

# Use ltrace for library calls
ltrace -f ./bitcoind -regtest 2>&1 | head -100
```

## Common Patterns in Bitcoin Core

### 1. Naming Conventions
- `CClassName` - Class names (C prefix is legacy)
- `m_memberVariable` - Member variables
- `g_globalVariable` - Global variables
- `FunctionName()` - Functions use PascalCase
- `local_variable` - Local variables use snake_case

### 2. Common Macros/Patterns
- `LOCK(cs_main)` - Thread synchronization
- `LogPrintf()` - Logging output
- `assert()` - Debug assertions
- `CHECK_NONFATAL()` - Runtime checks
- `Consensus::` - Consensus critical code

### 3. Important Globals
- `cs_main` - Main lock for chain state
- `chainActive` or `m_chain` - The active blockchain
- `mempool` - The transaction memory pool
- `g_connman` - Connection manager

## Notes for Effective Analysis

1. **Start with running code** - Get regtest working first
2. **Follow data, not control flow** - Understand structures before logic
3. **Read tests** - Unit tests show intended behavior
4. **Use git grep** - Faster than regular grep for large repos
5. **Check commit history** - `git log -p filename` shows evolution
6. **Read comments carefully** - Especially "consensus critical" warnings
7. **Don't modify consensus code** - Unless you really understand it
8. **Test everything in regtest** - Never experiment on mainnet

## Final Reminders

- The code is the specification
- Comments can be wrong
- Documentation can be outdated  
- But the code that runs is Bitcoin
- Start minimal, build understanding gradually
- Use regtest for all experiments
- Ask Claude Code to explain complex sections
- Trace execution paths rather than reading linearly

This exploration guide makes no assumptions about structure. Use it to discover how Bitcoin Core actually works in its current form.
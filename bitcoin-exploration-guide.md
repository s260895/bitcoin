# Bitcoin Core Exploration Guide

## Purpose
This document provides a discovery-based approach to analyzing Bitcoin Core source code without assuming any specific structure. The code itself is the specification - we'll explore what actually exists rather than what documentation claims should exist.

## Core Philosophy
- **Zero commercial dependencies** - Only open-source tools
- **Code is truth** - Don't trust documentation, verify in source
- **Minimal first** - Build and run the simplest version possible
- **Explore systematically** - Use search and grep to understand structure

---

## 🪟 Windows-Specific Build Guide (Visual Studio 2022)

### Prerequisites

**System Requirements:**
- Windows 10/11
- 32GB RAM (recommended, 16GB minimum)
- ~10GB free disk space for build tools + dependencies
- Additional space for blockchain data (regtest: <100MB, mainnet: ~600GB)

**Required Software:**
1. **Visual Studio 2022 Build Tools** (or full IDE)
   - Download: https://visualstudio.microsoft.com/downloads/#build-tools-for-visual-studio-2022
   - Select: "Desktop development with C++"
   - Includes: MSVC v143, CMake 3.22+, Windows SDK, vcpkg

2. **Git for Windows** (if not installed)
   - Download: https://git-scm.com/downloads/win

3. **Python 3.10+** (for tests, optional)
   - Download: https://www.python.org/downloads/

### Step 1: Clone Repository

```powershell
# Clone to a path without spaces (recommended)
cd C:\Users\<YourName>\Documents
git clone https://github.com/bitcoin/bitcoin.git
cd bitcoin
```

### Step 2: Open Developer PowerShell

Search for "Developer PowerShell for VS 2022" in Start Menu and open it. Navigate to your bitcoin directory:

```powershell
cd C:\Users\<YourName>\Documents\bitcoin
```

### Step 3: Verify Build Tools

```powershell
# Verify CMake (should be 3.22+)
cmake --version

# Verify MSVC compiler
cl

# List available CMake presets
cmake --list-presets
```

Expected presets:
- `vs2022` - Dynamic linking with GUI
- `vs2022-static` - Static linking with GUI
- `dev-mode` - Full development build

### Step 4: Configure Build (Minimal)

For exploration, build without wallet and GUI for faster compilation:

```powershell
# Configure minimal build
cmake -B build --preset vs2022 -DBUILD_GUI=OFF -DENABLE_WALLET=OFF

# What this does:
# - vcpkg downloads and compiles all dependencies (~30-50 minutes first time)
# - CMake configures the build system
# - Creates build directory with Visual Studio solution
```

**First-time dependency build (~45 minutes):**
- Boost libraries (multi-index, signals2, test)
- libevent (networking)
- Qt6 (if building GUI)
- SQLite3 (if building wallet)
- ZeroMQ (if enabled)
- Total: 62 packages via vcpkg

### Step 5: Compile Bitcoin Core

```powershell
# Build Release configuration (optimized)
cmake --build build --config Release -j 12

# -j 12 uses 12 parallel jobs (adjust for your CPU cores)
# Build time: ~10-20 minutes on modern hardware
```

**Built executables location:** `build\bin\Release\`
- `bitcoind.exe` - Daemon (6.5MB)
- `bitcoin-cli.exe` - RPC client (480KB)
- `bitcoin-tx.exe` - Transaction utility (2.1MB)
- `bitcoin-util.exe` - Utility tool (305KB)
- `test_bitcoin.exe` - Unit tests (15MB)

### Step 6: Verify Build

```powershell
# Check bitcoind version
.\build\bin\Release\bitcoind.exe --version

# Check bitcoin-cli version
.\build\bin\Release\bitcoin-cli.exe --version

# List all built files
ls build\bin\Release\
```

### Step 7: Set Up Regtest Environment

**Create datadir on fast drive (NVMe recommended):**

```powershell
# Create directory for regtest blockchain
mkdir D:\bitcoin-regtest

# Create configuration file
@"
regtest=1
server=1
rpcuser=bitcoinrpc
rpcpassword=testpassword123
txindex=1
fallbackfee=0.00001
"@ | Out-File -FilePath D:\bitcoin-regtest\bitcoin.conf -Encoding ASCII
```

**Alternative single-line config creation:**
```powershell
echo "regtest=1`nserver=1`nrpcuser=bitcoinrpc`nrpcpassword=testpassword123`ntxindex=1`nfallbackfee=0.00001" | Out-File -FilePath D:\bitcoin-regtest\bitcoin.conf -Encoding ASCII
```

### Step 8: Start Bitcoin Daemon

**Note:** Windows doesn't support `-daemon` flag, so run in dedicated window:

```powershell
# Option 1: Run in current window (recommended for learning)
.\build\bin\Release\bitcoind.exe -datadir=D:\bitcoin-regtest

# Option 2: Run in new window
Start-Process powershell -ArgumentList "-NoExit", "-Command", "cd '$PWD'; .\build\bin\Release\bitcoind.exe -datadir=D:\bitcoin-regtest"
```

**Wait for:** `init message: Done loading` (~2-3 seconds)

### Step 9: Test RPC Interface (New PowerShell Window)

Open a **second PowerShell window**, navigate to bitcoin directory, and run:

```powershell
# Check blockchain status
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-regtest getblockchaininfo

# Expected output: "blocks": 0, "chain": "regtest"
```

### Step 10: Mine Initial Blocks

**Why 101 blocks?** Coinbase maturity rule - newly mined coins require 100 confirmations before they can be spent.

```powershell
# Generate 101 blocks instantly (regtest has near-zero difficulty)
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-regtest generatetoaddress 101 "mxVFsFW5N4mu1HPkxPttorvocvzeZ7KZyk"

# Returns: Array of 101 block hashes
```

### Step 11: Explore Your Blockchain

```powershell
# Get updated blockchain info
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-regtest getblockchaininfo

# Get best block hash
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-regtest getbestblockhash

# Get block details
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-regtest getblock "<block_hash>"

# Check mempool
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-regtest getmempoolinfo
```

### Troubleshooting Windows Build

**Issue: CMake version too old**
```powershell
# Update Visual Studio to latest version
# Or download standalone CMake 3.22+ from cmake.org
```

**Issue: vcpkg fails with "path too long"**
```powershell
# Use shorter buildtrees path
cmake -B build --preset vs2022 -DBUILD_GUI=OFF -DVCPKG_INSTALL_OPTIONS="--x-buildtrees-root=C:\vcpkg"
```

**Issue: Build fails with spaces in path**
```powershell
# Move repository to path without spaces
# Or set custom vcpkg install directory
cmake -B build --preset vs2022 -DVCPKG_INSTALLED_DIR="C:\bitcoin-deps"
```

**Issue: Out of memory during compilation**
```powershell
# Reduce parallel jobs
cmake --build build --config Release -j 4  # Instead of -j 12
```

### Windows Development Tips

1. **Use Developer PowerShell** - Auto-configures paths for build tools
2. **Keep bitcoind running in separate window** - Easier to see logs
3. **Use VS Code for editing** - No need for full Visual Studio IDE
4. **Store blockchain data on NVMe** - Faster I/O for IBD and validation
5. **Add Windows Defender exclusion** - Exclude bitcoin directory for faster builds
6. **Use `git bash` for Unix-like commands** - Comes with Git for Windows

### Quick Reference - Windows Commands

```powershell
# Start daemon (keep window open)
.\build\bin\Release\bitcoind.exe -datadir=D:\bitcoin-regtest

# All bitcoin-cli commands need -datadir flag
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-regtest <command>

# Stop daemon gracefully
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-regtest stop

# Rebuild after code changes (incremental)
cmake --build build --config Release
```

---

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

---

## 🔍 Discovered Code Structures (Session Findings)

### Entry Points Discovered

**Main Executables** (all in `src/` directory):

1. **bitcoin.cpp:62** - Multiplexer/dispatcher
   - Routes commands to appropriate binaries
   - Commands: `gui`, `node`, `rpc`, `wallet`, `tx`, `bench`, `test`
   - Example: `bitcoin node` → runs `bitcoind`

2. **bitcoind.cpp** - Full node daemon
   - Initialization: AppInit() → InitLogging() → LoadChainParams()
   - Main loop handles: Block validation, P2P networking, RPC server
   - Entry point for running a Bitcoin node

3. **bitcoin-cli.cpp** - RPC command-line interface
   - Sends JSON-RPC requests to bitcoind
   - Formats and displays responses

4. **bitcoin-tx.cpp** - Transaction manipulation utility
   - Create, sign, and analyze transactions offline

5. **bitcoin-util.cpp** - General utility tool

### Core Data Structure: CBlock

**Location:** `src/primitives/block.h:68`

**CBlockHeader** (lines 21-65):
```cpp
class CBlockHeader {
public:
    int32_t nVersion;        // Block version number
    uint256 hashPrevBlock;   // Hash of previous block (creates the chain!)
    uint256 hashMerkleRoot;  // Root of transaction merkle tree
    uint32_t nTime;          // Block timestamp (Unix epoch)
    uint32_t nBits;          // Difficulty target in compact format
    uint32_t nNonce;         // Proof-of-work nonce

    uint256 GetHash() const; // Calculates block hash (SHA256d)
};
```

**Key insight:** This 80-byte structure is **exactly what gets hashed** for proof-of-work. Unchanged since Satoshi's original design!

**CBlock** (lines 68-117):
```cpp
class CBlock : public CBlockHeader {
public:
    std::vector<CTransactionRef> vtx;  // All transactions in block

    // Memory-only caching flags (not serialized):
    mutable bool fChecked;                      // Has CheckBlock() validated?
    mutable bool m_checked_witness_commitment;  // Witness commitment OK?
    mutable bool m_checked_merkle_root;        // Merkle root verified?

    void SetNull();
    CBlockHeader GetBlockHeader() const;
    std::string ToString() const;
};
```

**Key observations:**
- Block = Header + Transactions
- Caching flags optimize repeated validation
- First transaction (`vtx[0]`) is always the coinbase (mining reward)

### Core Data Structure: CTransaction

**Location:** `src/primitives/transaction.h`

```cpp
class CTransaction {
    int32_t nVersion;                    // Transaction version
    std::vector<CTxIn> vin;             // Transaction inputs (spend UTXOs)
    std::vector<CTxOut> vout;           // Transaction outputs (create new UTXOs)
    uint32_t nLockTime;                 // Earliest time tx can be mined

    uint256 GetHash() const;            // Transaction ID (txid)
    uint256 GetWitnessHash() const;     // Witness transaction ID (wtxid)
};
```

**Transaction Flow:**
1. Inputs reference previous transaction outputs (UTXOs)
2. Outputs create new UTXOs
3. Sum(outputs) + fees = Sum(inputs)
4. Miners collect fees + coinbase reward

### Critical Validation Functions

**Found via grep:**
- `CheckBlock()` - Basic block validity (in `src/validation.cpp`)
- `AcceptBlock()` - Context-dependent block acceptance
- `ConnectBlock()` - Apply block to chain state (update UTXO set)
- `DisconnectBlock()` - Revert block during reorg
- `CheckTransaction()` - Individual transaction validation
- `AcceptToMemoryPool()` - Mempool admission logic
- `VerifyScript()` - Script/signature verification

### Consensus Constants

**Found in:** `src/consensus/consensus.h`

```cpp
static const unsigned int MAX_BLOCK_WEIGHT = 4000000;
static const unsigned int MAX_BLOCK_SERIALIZED_SIZE = 4000000;
static const int64_t MAX_BLOCK_SIGOPS_COST = 80000;
static const int COINBASE_MATURITY = 100;  // Why we need 101 blocks!
```

**Found in:** `src/consensus/amount.h`
```cpp
static constexpr CAmount MAX_MONEY = 21000000 * COIN;  // 21 million BTC limit
```

### Key Globals (Found via grep)

- `cs_main` - Main critical section (mutex) for blockchain state
- `g_best_block` - Currently active chain tip
- `g_mempool` - Transaction memory pool
- `g_connman` - P2P connection manager

### File Locations Summary

| Component | File | Lines |
|-----------|------|-------|
| Block structure | src/primitives/block.h | 68-117 |
| Transaction structure | src/primitives/transaction.h | - |
| Block validation | src/validation.cpp | - |
| Consensus rules | src/consensus/consensus.h | - |
| Chain state | src/chain.h | - |
| UTXO set | src/coins.h | - |
| Mempool | src/txmempool.h | - |
| P2P networking | src/net.h, src/net_processing.cpp | - |
| Script interpreter | src/script/interpreter.cpp | - |
| RPC server | src/rpc/*.cpp | - |

### Next Steps for Exploration

1. **Trace a block validation:**
   ```cpp
   // Follow this path in validation.cpp:
   CheckBlock() → AcceptBlock() → ConnectBlock() → UpdateUTXOSet()
   ```

2. **Understand UTXO set:**
   - Read `src/coins.h` - CCoinsView interface
   - Find CCoinsViewCache - in-memory UTXO cache
   - Trace how UTXOs are created/spent

3. **Explore script verification:**
   - Read `src/script/interpreter.cpp`
   - Function: `EvalScript()` - stack-based VM
   - Verify a signature in regtest block

4. **Study consensus rules:**
   - Find MAX_MONEY enforcement
   - Difficulty adjustment (GetNextWorkRequired)
   - Block reward halving logic

5. **P2P message flow:**
   - Read `src/net_processing.cpp`
   - Function: `ProcessMessage()`
   - Trace how blocks propagate through network

### Real Example from Your Regtest

Your block 101 structure:
```json
{
  "hash": "6da4798d9a74711616b22c040bec08e20912c28223a12533697994dfd3f8e000",
  "height": 101,
  "nonce": 1,                    // Instant mining in regtest!
  "merkleroot": "684384d7...",   // Hash of coinbase transaction
  "nTx": 1,                      // Only coinbase, no other transactions
  "previousblockhash": "3728cc26..."  // Links to block 100
}
```

This demonstrates:
- Block chaining via `previousblockhash`
- Merkle root = hash of single transaction (coinbase)
- Minimal valid block structure

### Commands to Explore Code

```powershell
# Find where MAX_MONEY is enforced
git grep "MAX_MONEY" src/

# Find all validation functions
git grep "^bool Check" src/validation.cpp

# Find UTXO update logic
git grep "UpdateCoins" src/

# Find script opcodes
git grep "OP_CHECKSIG" src/script/

# Find difficulty adjustment
git grep "GetNextWorkRequired" src/

# Find where blocks are stored
git grep "FlatFileSeq" src/
```

### Script VM and Opcodes

**Location:** `src/script/interpreter.cpp:407` - `EvalScript()`

Bitcoin uses a **stack-based virtual machine** (similar to Forth) to evaluate transaction scripts. The VM is intentionally limited:
- No loops (prevents infinite execution)
- Max 1000 stack items (MAX_STACK_SIZE)
- Max 201 operations per script (MAX_OPS_PER_SCRIPT)
- Max 520 bytes per stack element (MAX_SCRIPT_ELEMENT_SIZE)

**Key opcodes** (from `src/script/script.h:73-199`):

```cpp
// Stack manipulation
OP_DUP = 0x76        // Duplicate top stack item
OP_DROP = 0x75       // Remove top stack item
OP_SWAP = 0x7c       // Swap top two items

// Crypto operations
OP_HASH160 = 0xa9    // SHA256 then RIPEMD160 (used for addresses)
OP_CHECKSIG = 0xac   // Verify signature against public key

// Comparison
OP_EQUAL = 0x87          // Check if top two items are equal
OP_EQUALVERIFY = 0x88    // OP_EQUAL + OP_VERIFY (fail if not equal)

// Control flow
OP_IF = 0x63         // Conditional execution
OP_RETURN = 0x6a     // Marks output as unspendable (fails script)
OP_VERIFY = 0x69     // Fail if top stack is false
```

**Example: P2PKH (Pay-to-PubKey-Hash) Script Execution**

From your regtest coinbase transaction:
```
scriptPubKey: OP_DUP OP_HASH160 <pubKeyHash> OP_EQUALVERIFY OP_CHECKSIG
scriptSig:    <signature> <publicKey>
```

Execution trace (stack shown as [bottom ... top]):
```
1. Start:           [<sig> <pubKey>]
2. OP_DUP:          [<sig> <pubKey> <pubKey>]
3. OP_HASH160:      [<sig> <pubKey> <hash>]
4. <pubKeyHash>:    [<sig> <pubKey> <hash> <pubKeyHash>]
5. OP_EQUALVERIFY:  [<sig> <pubKey>]    // Fails if hashes don't match
6. OP_CHECKSIG:     [true/false]        // Verifies signature
7. Success if true on stack
```

**Implementation highlights:**

```cpp
// src/script/interpreter.cpp:794-801
case OP_DUP:
    if (stack.size() < 1)
        return set_error(serror, SCRIPT_ERR_INVALID_STACK_OPERATION);
    valtype vch = stacktop(-1);
    stack.push_back(vch);
    break;

// src/script/interpreter.cpp:1025-1035
case OP_HASH160:
    if (stack.size() < 1)
        return set_error(serror, SCRIPT_ERR_INVALID_STACK_OPERATION);
    valtype& vch = stacktop(-1);
    valtype vchHash(20);  // 20 bytes = 160 bits
    // SHA256 then RIPEMD160
    CRIPEMD160().Write(CSHA256().Write(vch.data(), vch.size()).Finalize(), 32).Finalize(vchHash.data());
    vch = vchHash;
    break;

// src/script/interpreter.cpp:1059-1082
case OP_CHECKSIG:
    if (stack.size() < 2)
        return set_error(serror, SCRIPT_ERR_INVALID_STACK_OPERATION);
    valtype& vchSig    = stacktop(-2);
    valtype& vchPubKey = stacktop(-1);
    bool fSuccess = true;
    if (!EvalChecksig(vchSig, vchPubKey, ...)) return false;
    popstack(stack);
    popstack(stack);
    stack.push_back(fSuccess ? vchTrue : vchFalse);
    break;
```

**Disabled opcodes** (CVE-2010-5137 - vulnerabilities found):
```cpp
// src/script/interpreter.cpp:457-472
OP_CAT, OP_SUBSTR, OP_LEFT, OP_RIGHT  // String splicing
OP_INVERT, OP_AND, OP_OR, OP_XOR      // Bitwise operations
OP_MUL, OP_DIV, OP_MOD                // Arithmetic (beyond add/sub)
OP_LSHIFT, OP_RSHIFT                  // Bit shifting
```

These were disabled to prevent potential DoS attacks and bugs.

---

## 📖 Recommended Reading Order

1. **Start here:** `src/primitives/block.h` + `src/primitives/transaction.h`
2. **Then:** `src/consensus/consensus.h` - Understand the rules
3. **Core logic:** `src/validation.cpp` - How blocks are validated
4. **UTXO:** `src/coins.h` + `src/coins.cpp` - The unspent output set
5. **Scripts:** `src/script/interpreter.cpp` - Bitcoin's VM
6. **Networking:** `src/net_processing.cpp` - P2P protocol
7. **Mempool:** `src/txmempool.cpp` - Unconfirmed transactions

**Pro tip:** Keep your regtest running and cross-reference code with live RPC commands!
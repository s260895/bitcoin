# Bitcoin Wallet Safety & Version Management Guide

## 🔐 Your Wallet is Safe - Here's Why

### TL;DR
**Your Bitcoin wallet will remain valid forever, regardless of Bitcoin Core version changes.** Wallet files are backwards compatible across all versions, and your private keys are based on unchanging cryptography.

---

## Understanding Bitcoin Wallet Files

### What's Actually in Your `wallet.dat`

Your wallet file contains:

1. **Private Keys** 🔑
   - The cryptographic proof of Bitcoin ownership
   - Based on elliptic curve cryptography (secp256k1)
   - **Never changes, works forever**
   - Can be exported and imported into any Bitcoin wallet software

2. **Public Keys & Addresses**
   - Derived mathematically from private keys
   - Multiple address formats supported (P2PKH, P2SH, P2WPKH, P2WSH, P2TR)
   - All formats remain valid even as new ones are added

3. **Transaction Metadata**
   - Labels, comments, timestamps
   - Non-critical information
   - Can be lost without losing funds

4. **Key Derivation Paths (HD Wallets)**
   - BIP32/BIP39/BIP44 standards
   - Industry-wide standards that don't change
   - Your seed phrase works in ANY compatible wallet

### Wallet File Format

```
Format: Berkeley DB (BDB) or SQLite
Location: D:\bitcoin-mainnet\wallets\<wallet-name>\wallet.dat
Size: ~100KB - 10MB (depends on transaction history)
Compatibility: Works across ALL Bitcoin Core versions
```

**Key Point:** Even wallet.dat files from Bitcoin Core 0.3 (2010) work perfectly in modern versions!

---

## Version Compatibility Matrix

| Wallet Created In | Works in v0.21 | Works in v22-27 | Works in v28+ | Works in Future Versions |
|-------------------|---------------|-----------------|---------------|--------------------------|
| v0.3 - v0.20      | ✅ Yes        | ✅ Yes          | ✅ Yes        | ✅ Yes (guaranteed)      |
| v0.21+            | ✅ Yes        | ✅ Yes          | ✅ Yes        | ✅ Yes (guaranteed)      |
| Development build | ✅ Yes        | ✅ Yes          | ✅ Yes        | ✅ Yes (guaranteed)      |

**Direction of compatibility:**
- ✅ **Newer versions can ALWAYS read older wallets**
- ⚠️ **Older versions might not understand new features** (but won't break the wallet)
- ✅ **Your funds are always accessible**

---

## What Changes Between Versions

### Things That CAN Change (Safely)

1. **New Address Types**
   - Legacy (P2PKH) → SegWit (P2WPKH) → Taproot (P2TR)
   - Old addresses keep working forever
   - You can opt-in to new types

2. **Wallet Features**
   - Descriptor wallets (v0.21+)
   - Hardware wallet support
   - Coin control improvements
   - **All optional, wallet still works without them**

3. **Database Format**
   - BDB → SQLite migration (optional)
   - Can upgrade wallet format (one-way, optional)
   - Old format continues working

4. **RPC Commands**
   - New wallet commands added
   - Old commands deprecated slowly
   - Deprecation warnings given years in advance

### Things That NEVER Change

1. **Private Key Format** ✅
   - Same ECDSA/Schnorr signatures
   - Same secp256k1 curve
   - Same derivation paths

2. **Address Validity** ✅
   - All old addresses remain valid
   - Funds sent to old addresses are safe
   - No address expiration

3. **Consensus Rules** ✅
   - Your UTXOs are valid forever
   - Transaction format backwards compatible
   - Soft forks only add restrictions (never invalidate existing)

---

## Development Build vs Stable Release

### Your Current Situation

You compiled from the `master` branch (development):
```
Version: Bitcoin Core version v29.99.0-e2e18d7672f1 (development build)
Status: Pre-release test build
Branch: master (latest development)
```

### Differences

| Aspect | Development Build | Stable Release (v28.0) |
|--------|-------------------|------------------------|
| **Wallet safety** | ✅ Same | ✅ Same |
| **Consensus rules** | ✅ Identical | ✅ Identical |
| **New features** | ⚠️ Experimental | ✅ Battle-tested |
| **Bugs** | ⚠️ Possible | ✅ Less likely |
| **Recommended for** | Testing, learning | Production, real funds |

### Why Development Build is Showing Warning

```json
"warnings": [
  "This is a pre-release test build - use at your own risk - do not use for mining or merchant applications"
]
```

This warning means:
- ✅ **OK for learning** and exploration
- ✅ **OK for small amounts** of Bitcoin
- ⚠️ **Not recommended** for large holdings
- ⚠️ **Not recommended** for production/business use

**Why?** Development builds may have:
- Experimental consensus code (pre-activation soft forks)
- Performance optimizations being tested
- New RPC commands that might change
- Potential bugs not yet discovered

---

## How to Switch to Stable Version

### Step-by-Step Process

#### 1. Backup Your Wallet First! 🚨

```powershell
# Stop the node
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet stop

# Wait for shutdown
Start-Sleep -Seconds 10

# Backup wallet
$backupDir = "D:\bitcoin-wallet-backups"
if (!(Test-Path $backupDir)) { New-Item -ItemType Directory -Path $backupDir }

Copy-Item D:\bitcoin-mainnet\wallets -Destination "$backupDir\wallets-backup-$(Get-Date -Format 'yyyy-MM-dd-HHmmss')" -Recurse

# Backup config
Copy-Item D:\bitcoin-mainnet\bitcoin.conf "$backupDir\bitcoin.conf.$(Get-Date -Format 'yyyy-MM-dd').bak"

Write-Host "Backup completed!" -ForegroundColor Green
```

#### 2. Check Available Stable Versions

```powershell
# Navigate to your bitcoin repo
cd C:\Users\s2608\Documents\bitcoin

# Fetch latest tags
git fetch --tags

# List recent stable releases
git tag -l "v*" | Select-String -Pattern "v\d+\.\d+$" | Select-Object -Last 10
```

**Expected output:**
```
v27.0
v28.0
v28.1
```

#### 3. Checkout Stable Version

```powershell
# Checkout latest stable (v28.0 or v28.1)
git checkout v28.0

# Verify you're on the right tag
git describe --tags
# Should output: v28.0
```

#### 4. Rebuild Bitcoin Core

```powershell
# Clean previous build (optional but recommended)
Remove-Item -Recurse -Force build -ErrorAction SilentlyContinue

# Configure build (same options as before)
cmake -B build --preset vs2022 -DBUILD_GUI=OFF -DENABLE_WALLET=ON

# Build (much faster this time, dependencies already compiled)
cmake --build build --config Release -j 12

# Verify version
.\build\bin\Release\bitcoind.exe -version
# Should show: Bitcoin Core version v28.0.0
```

#### 5. Restart Node with Same Data Directory

```powershell
# Start node (uses existing blockchain data and wallet)
.\build\bin\Release\bitcoind.exe -datadir=D:\bitcoin-mainnet

# Or in new window
Start-Process powershell -ArgumentList "-NoExit", "-Command", "cd '$PWD'; .\build\bin\Release\bitcoind.exe -datadir=D:\bitcoin-mainnet"
```

#### 6. Verify Everything Works

```powershell
# Wait a moment for startup, then check
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getblockchaininfo

# Check wallet is loaded
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet listwallets

# Check wallet info
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getwalletinfo
```

### What Happens During Version Switch

✅ **Preserved (No re-download needed):**
- All blockchain data (~737 GB)
- All block files
- Chainstate database
- Wallet files
- Configuration files
- Peer connections history

🔄 **May be upgraded:**
- Internal database format (automatic, one-way)
- Wallet features (if you opt-in)

❌ **Not lost:**
- Private keys
- Transaction history
- Addresses
- Balances

---

## Wallet Safety Best Practices

### 1. Regular Backups

```powershell
# Automated weekly backup script
$backupDir = "D:\bitcoin-wallet-backups"
$walletName = "mywallet"  # Replace with your wallet name

# Create backup directory
if (!(Test-Path $backupDir)) {
    New-Item -ItemType Directory -Path $backupDir
}

# Backup using RPC (node must be running)
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet -rpcwallet=$walletName backupwallet "$backupDir\wallet-$(Get-Date -Format 'yyyy-MM-dd').dat"

# Keep only last 30 backups
Get-ChildItem $backupDir -Filter "wallet-*.dat" |
    Sort-Object LastWriteTime -Descending |
    Select-Object -Skip 30 |
    Remove-Item
```

**Set up scheduled task:**
```powershell
# Run this in Administrator PowerShell
$action = New-ScheduledTaskAction -Execute "PowerShell.exe" -Argument "-File C:\path\to\backup-script.ps1"
$trigger = New-ScheduledTaskTrigger -Weekly -DaysOfWeek Sunday -At 3am
Register-ScheduledTask -TaskName "Bitcoin Wallet Backup" -Action $action -Trigger $trigger
```

### 2. Encrypt Your Wallet

```powershell
# Encrypt wallet (IMPORTANT: Node will restart!)
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet encryptwallet "your-very-strong-passphrase"

# After encryption, backup immediately!
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet backupwallet "D:\wallet-encrypted-backup.dat"
```

**⚠️ WARNING:**
- Write down your passphrase and store it securely
- If you lose the passphrase, **your funds are LOST FOREVER**
- No recovery possible without the passphrase

### 3. Export Private Keys (Emergency Backup)

```powershell
# For each address you care about:
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet dumpprivkey "your_bitcoin_address"

# Save output to encrypted file or paper wallet
# Store in secure location (safe, safety deposit box)
```

### 4. Test Your Backups

```powershell
# Test restore process (use separate datadir for testing!)
# 1. Create test directory
mkdir D:\bitcoin-restore-test

# 2. Copy config
Copy-Item D:\bitcoin-mainnet\bitcoin.conf D:\bitcoin-restore-test\

# 3. Start node with test datadir
.\build\bin\Release\bitcoind.exe -datadir=D:\bitcoin-restore-test

# 4. Load backup wallet (in new terminal)
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-restore-test restorewallet "test" "D:\bitcoin-wallet-backups\wallet-2025-10-21.dat"

# 5. Verify addresses match
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-restore-test -rpcwallet=test getwalletinfo

# 6. Stop test node
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-restore-test stop

# 7. Clean up test data
Remove-Item -Recurse D:\bitcoin-restore-test
```

### 5. Use Descriptor Wallets (Modern Approach)

**For NEW wallets** (Bitcoin Core v0.21+):

```powershell
# Create new descriptor wallet
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet createwallet "my_descriptor_wallet" false false "" false true

# Benefits:
# - Better backup process (descriptors instead of keypool)
# - Clearer key derivation
# - More compatible with hardware wallets
```

**Export descriptor for backup:**
```powershell
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet -rpcwallet=my_descriptor_wallet listdescriptors true

# Save this output - it's your complete wallet backup!
# With descriptors + seed, you can recover in any wallet
```

---

## Understanding Wallet Upgrades

### Optional Wallet Upgrades

When you upgrade Bitcoin Core, you might see messages about upgrading wallet format:

```
Warning: Wallet file format version 169999 is newer than supported version 60000.
```

Or:

```
Wallet loaded successfully. Wallet format upgrade is available (from version X to Y).
```

### Should You Upgrade?

| Upgrade Type | Old Wallet Works? | Can Revert? | Benefits | Recommendation |
|--------------|-------------------|-------------|----------|----------------|
| BDB → SQLite | ✅ Yes | ❌ No | Better performance | Optional |
| Legacy → Descriptor | ✅ Yes | ❌ No | Better backups | For new wallets only |
| Wallet format version | ✅ Yes | ❌ No | New features | Optional |

**General advice:**
- ✅ **If it works, don't upgrade**
- ✅ **Backup before ANY upgrade**
- ⚠️ **Upgrades are ONE-WAY** (can't revert to old version)
- ⚠️ **Test on small amounts first**

---

## What If Bitcoin Core Development Stops?

### Your Funds Are Still Safe

Bitcoin wallets are based on **open standards**:

1. **BIP32** - HD Wallet derivation
2. **BIP39** - Mnemonic seed phrases
3. **BIP44** - Multi-account hierarchy
4. **ECDSA** - Signature algorithm (never changes)

**Alternative wallets that can import Bitcoin Core keys:**
- Electrum (desktop)
- Sparrow Wallet (desktop)
- BlueWallet (mobile)
- Wasabi Wallet (privacy-focused)
- Hardware wallets (Ledger, Trezor, ColdCard)

### How to Migrate (If Needed)

```powershell
# 1. Export private keys from Bitcoin Core
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet dumpwallet "exported-wallet.txt"

# 2. Export seed phrase (for HD wallets)
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet dumphdseed

# 3. Import into alternative wallet
# Each wallet has its own import process
# See specific wallet documentation
```

---

## Version Switching Decision Guide

### When to Use Development Build

✅ **Good for:**
- Learning Bitcoin Core development
- Testing new features
- Contributing to Bitcoin Core
- Exploring cutting-edge functionality
- Running on testnet/regtest
- Small amounts of BTC (<$100)

❌ **Not recommended for:**
- Life savings
- Production systems
- Merchant applications
- Mining operations
- Services handling customer funds

### When to Use Stable Release

✅ **Good for:**
- Long-term Bitcoin storage
- Significant amounts
- Production environments
- Business use
- Peace of mind
- Maximum stability

### Hybrid Approach (Recommended for Learning)

1. **Mainnet node**: Use stable release (v28.0)
   - For real funds and security
   - Keep running 24/7

2. **Development node**: Use latest master
   - For testing and learning
   - Run on regtest/testnet
   - Explore new features

```powershell
# Mainnet (stable)
.\build\bin\Release\bitcoind.exe -datadir=D:\bitcoin-mainnet

# Testnet (development) - different directory
.\build-dev\bin\Release\bitcoind.exe -testnet -datadir=D:\bitcoin-testnet
```

---

## Summary

### Key Takeaways

1. **Your wallet is safe** ✅
   - Works across all Bitcoin Core versions
   - Based on unchanging cryptography
   - Can be exported to other wallets

2. **Switching versions is safe** ✅
   - No blockchain re-download needed
   - Wallet remains compatible
   - Configuration is preserved

3. **Backups are essential** 🚨
   - Regular automated backups
   - Test restore process
   - Multiple secure locations
   - Encrypt sensitive data

4. **Choose version based on use case** 💡
   - Development build: Learning, testing
   - Stable release: Production, real funds
   - Both: Best of both worlds

### Quick Commands Summary

```powershell
# Check current version
.\build\bin\Release\bitcoind.exe -version

# List stable releases
git tag -l "v*" | Select-String -Pattern "v\d+\.\d+$" | Select-Object -Last 5

# Switch to stable
git checkout v28.0
cmake --build build --config Release -j 12

# Backup wallet
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet backupwallet "D:\backup.dat"

# Verify wallet
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getwalletinfo
```

---

**Your Bitcoin is secure.** The wallet format is stable, your private keys are yours forever, and you can switch Bitcoin Core versions safely at any time. Focus on good backup practices and use stable releases for production funds.

**Happy HODLing!** 🚀

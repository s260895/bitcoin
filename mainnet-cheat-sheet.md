# Bitcoin Mainnet Node Cheat Sheet

## Directory Paths
```
Mainnet Data Directory: D:\bitcoin-mainnet
Built Binaries: C:\Users\s2608\Documents\bitcoin\build\bin\Release
```

---

## 🚀 Starting the Node

### Basic Start (Foreground)
```powershell
.\build\bin\Release\bitcoind.exe -datadir=D:\bitcoin-mainnet
```

### Start in New Window (Recommended)
```powershell
Start-Process powershell -ArgumentList "-NoExit", "-Command", "cd '$PWD'; .\build\bin\Release\bitcoind.exe -datadir=D:\bitcoin-mainnet"
```

### Start with Custom Settings
```powershell
# With increased connection limits
.\build\bin\Release\bitcoind.exe -datadir=D:\bitcoin-mainnet -maxconnections=125

# With specific debug logging
.\build\bin\Release\bitcoind.exe -datadir=D:\bitcoin-mainnet -debug=net -debug=mempool

# Prune mode (keeps only recent blocks, ~10GB instead of ~600GB)
.\build\bin\Release\bitcoind.exe -datadir=D:\bitcoin-mainnet -prune=10000
```

---

## 🛑 Stopping the Node

### Graceful Shutdown (Recommended)
```powershell
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet stop
```

### Force Stop (Use only if graceful fails)
```powershell
# Find the process ID first
Get-Process bitcoind

# Kill by process ID
Stop-Process -Id <PID> -Force
```

---

## 📊 Node Status & Info

### Blockchain Status
```powershell
# Overall blockchain info (height, difficulty, chain work)
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getblockchaininfo

# Network info (connections, protocol version)
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getnetworkinfo

# Peer connections
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getpeerinfo

# Connection count
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getconnectioncount
```

### Sync Progress
```powershell
# Check sync status (shows percentage)
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getblockchaininfo | Select-String "blocks","headers","verificationprogress"

# Estimated time to sync (based on current rate)
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getblockchaininfo | Select-String "verificationprogress"

# Monitor sync progress in real-time (updates every 30 seconds)
while ($true) {
    $info = .\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getblockchaininfo | ConvertFrom-Json
    $behind = $info.headers - $info.blocks
    $progress = [math]::Round($info.verificationprogress * 100, 4)
    $ibd = if ($info.initialblockdownload) { "SYNCING" } else { "SYNCED" }
    Write-Host "$(Get-Date -Format 'HH:mm:ss') | Status: $ibd | Blocks: $($info.blocks)/$($info.headers) | Behind: $behind | Progress: $progress%"
    Start-Sleep -Seconds 30
}
```

### Mempool Status
```powershell
# Mempool info (size, fees, memory usage)
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getmempoolinfo

# Raw mempool (list all unconfirmed transactions)
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getrawmempool

# Mempool with details
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getrawmempool true
```

---

## 🔍 Querying Blockchain Data

### Block Queries
```powershell
# Get best (latest) block hash
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getbestblockhash

# Get block by hash
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getblock <block_hash>

# Get block with full transaction details (verbosity=2)
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getblock <block_hash> 2

# Get block by height
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getblockhash <height>
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getblock $hash

# Get block header only
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getblockheader <block_hash>

# Get block count (current height)
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getblockcount
```

### Transaction Queries
```powershell
# Get raw transaction (requires txindex=1 in bitcoin.conf)
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getrawtransaction <txid>

# Get decoded transaction
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getrawtransaction <txid> true

# Decode raw transaction hex
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet decoderawtransaction <hex>

# Get transaction from mempool
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getmempoolentry <txid>
```

### UTXO Queries (requires txindex=1)
```powershell
# Get UTXO set info
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet gettxoutsetinfo

# Get specific transaction output info
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet gettxout <txid> <vout>
```

---

## 💾 Data Management

### Check Disk Usage
```powershell
# Check total size of blockchain data
(Get-ChildItem D:\bitcoin-mainnet -Recurse | Measure-Object -Property Length -Sum).Sum / 1GB

# Check blocks directory size
(Get-ChildItem D:\bitcoin-mainnet\blocks -Recurse | Measure-Object -Property Length -Sum).Sum / 1GB

# Check chainstate size (UTXO set)
(Get-ChildItem D:\bitcoin-mainnet\chainstate -Recurse | Measure-Object -Property Length -Sum).Sum / 1GB
```

### Prune Old Blocks
```powershell
# Enable pruning (keeps only ~10GB of blocks)
# Add to D:\bitcoin-mainnet\bitcoin.conf:
# prune=10000

# Manual prune to specific height (if pruning enabled)
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet pruneblockchain <height>
```

---

## 🔧 Configuration File

### Location
```
D:\bitcoin-mainnet\bitcoin.conf
```

### Recommended Settings
```ini
# Network
server=1
listen=1
maxconnections=125

# Performance
dbcache=4096                # 4GB RAM cache (adjust based on available RAM)
maxmempool=300              # 300MB mempool
maxorphantx=100

# Indexing (optional but useful)
txindex=1                   # Enable full transaction index

# Pruning (choose one or the other, not both with txindex)
# prune=10000               # Keep only 10GB of blocks (can't use with txindex=1)

# RPC (if you want remote access)
rpcuser=bitcoinrpc
rpcpassword=<your_strong_password>
rpcallowip=127.0.0.1
rpcport=8332

# Logging
debug=0                     # Disable debug logs in production
printtoconsole=1           # Show logs in console

# Fee estimation
blocksonly=0               # Download full blocks (0) or headers only (1)
```

### Edit Configuration
```powershell
# Open config in notepad
notepad D:\bitcoin-mainnet\bitcoin.conf

# View current config
Get-Content D:\bitcoin-mainnet\bitcoin.conf
```

---

## 📈 Monitoring & Logs

### View Debug Log
```powershell
# View last 50 lines
Get-Content D:\bitcoin-mainnet\debug.log -Tail 50

# Follow log in real-time
Get-Content D:\bitcoin-mainnet\debug.log -Wait -Tail 20

# Search for errors
Select-String -Path D:\bitcoin-mainnet\debug.log -Pattern "ERROR"

# Search for specific term
Select-String -Path D:\bitcoin-mainnet\debug.log -Pattern "block"
```

### Node Uptime
```powershell
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet uptime
```

---

## 🌐 Network Commands

### Peer Management
```powershell
# List all peers
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getpeerinfo

# Add a node manually
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet addnode <ip:port> add

# Remove a node
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet addnode <ip:port> remove

# Get added nodes
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getaddednodeinfo

# Disconnect peer
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet disconnectnode <ip:port>
```

### Network Traffic
```powershell
# Get network traffic statistics
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getnettotals

# Get network hash rate
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getnetworkhashps
```

---

## 🔐 Security & Backup

### Backup Configuration
```powershell
# Backup bitcoin.conf
Copy-Item D:\bitcoin-mainnet\bitcoin.conf D:\bitcoin-mainnet-backup-$(Get-Date -Format 'yyyy-MM-dd').conf

# Backup entire datadir (WARNING: Very large!)
# Not recommended - better to backup wallet.dat only if using wallet
```

### Check Chain Validity
```powershell
# Verify blockchain (takes hours!)
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet verifychain

# Quick verification (check level 0, 6 blocks)
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet verifychain 0 6
```

---

## 🧪 Testing & Troubleshooting

### Test RPC Connection
```powershell
# Simple ping
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getblockcount

# Check if node is responding
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet echo "test"
```

### Restart Node
```powershell
# Stop
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet stop

# Wait for shutdown (check process)
Get-Process bitcoind -ErrorAction SilentlyContinue

# Restart
.\build\bin\Release\bitcoind.exe -datadir=D:\bitcoin-mainnet
```

### Check for Issues
```powershell
# Get warnings
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getnetworkinfo | Select-String "warnings"

# Get blockchain warnings
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getblockchaininfo | Select-String "warnings"
```

---

## 📊 Fee Estimation

### Estimate Transaction Fees
```powershell
# Estimate fee for confirmation in 6 blocks
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet estimatesmartfee 6

# Estimate fee for confirmation in 1 block (next block)
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet estimatesmartfee 1

# Estimate fee for confirmation in 144 blocks (~1 day)
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet estimatesmartfee 144
```

---

## 🆘 Emergency Commands

### Node Stuck/Frozen
```powershell
# 1. Try graceful stop
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet stop

# 2. If no response, force kill
Get-Process bitcoind | Stop-Process -Force

# 3. Check for lock file (prevents startup if exists)
Remove-Item D:\bitcoin-mainnet\.lock -Force -ErrorAction SilentlyContinue

# 4. Restart
.\build\bin\Release\bitcoind.exe -datadir=D:\bitcoin-mainnet -reindex-chainstate
```

### Corrupted Database
```powershell
# Reindex chainstate (faster, rebuilds UTXO set)
.\build\bin\Release\bitcoind.exe -datadir=D:\bitcoin-mainnet -reindex-chainstate

# Full reindex (slow, re-validates all blocks)
.\build\bin\Release\bitcoind.exe -datadir=D:\bitcoin-mainnet -reindex
```

### Check Database Integrity
```powershell
# Stop node first
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet stop

# Start with verification
.\build\bin\Release\bitcoind.exe -datadir=D:\bitcoin-mainnet -checkblocks=100 -checklevel=3
```

---

## 📝 Useful Aliases (Optional)

Add to PowerShell profile (`$PROFILE`):

```powershell
# Bitcoin CLI shortcut
function btc { .\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet $args }

# Bitcoin daemon shortcut
function btcd { .\build\bin\Release\bitcoind.exe -datadir=D:\bitcoin-mainnet $args }

# Quick status
function btc-status {
    $info = .\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getblockchaininfo | ConvertFrom-Json
    Write-Host "Height: $($info.blocks)"
    Write-Host "Headers: $($info.headers)"
    Write-Host "Progress: $([math]::Round($info.verificationprogress * 100, 2))%"
}

# Quick sync check
function btc-sync {
    $info = .\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getblockchaininfo | ConvertFrom-Json
    $behind = $info.headers - $info.blocks
    Write-Host "Blocks behind: $behind"
    Write-Host "Sync: $([math]::Round($info.verificationprogress * 100, 2))%"
}
```

Usage after reload:
```powershell
btc getblockcount
btc-status
btc-sync
```

---

## 💰 Wallet Management

### Wallet Operations

```powershell
# List all loaded wallets
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet listwallets

# List all available wallets in wallet directory
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet listwalletdir

# Load a wallet
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet loadwallet "wallet-name"

# Unload a wallet
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet unloadwallet "wallet-name"

# Create a new wallet
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet createwallet "wallet-name"

# Create a new encrypted wallet
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet -named createwallet wallet_name="wallet-name" passphrase="your-passphrase"

# Get wallet info
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet -rpcwallet="wallet-name" getwalletinfo
```

### Address Management

```powershell
# Get a new receiving address
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet -rpcwallet="wallet-name" getnewaddress

# Get a new address with a label
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet -rpcwallet="wallet-name" getnewaddress "label-name"

# List all addresses (shows all addresses that have received funds)
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet -rpcwallet="wallet-name" listreceivedbyaddress 0 true

# Get all addresses by label
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet -rpcwallet="wallet-name" getaddressesbylabel ""

# List all address labels
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet -rpcwallet="wallet-name" listlabels

# Get address information
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet -rpcwallet="wallet-name" getaddressinfo "your-address"
```

### Wallet Security

```powershell
# Encrypt wallet (node will restart!)
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet -rpcwallet="wallet-name" encryptwallet "your-passphrase"

# Unlock wallet for 120 seconds
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet -rpcwallet="wallet-name" walletpassphrase "your-passphrase" 120

# Change wallet passphrase
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet -rpcwallet="wallet-name" walletpassphrasechange "old-passphrase" "new-passphrase"

# Lock wallet immediately
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet -rpcwallet="wallet-name" walletlock
```

### Balance & Transactions

```powershell
# Get wallet balance
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet -rpcwallet="wallet-name" getbalance

# Get unconfirmed balance
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet -rpcwallet="wallet-name" getunconfirmedbalance

# List all transactions (last 10)
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet -rpcwallet="wallet-name" listtransactions "*" 10

# List all transactions (last 100)
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet -rpcwallet="wallet-name" listtransactions "*" 100

# Get transaction details
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet -rpcwallet="wallet-name" gettransaction "txid"

# List unspent transaction outputs (UTXOs)
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet -rpcwallet="wallet-name" listunspent

# List received transactions by address
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet -rpcwallet="wallet-name" listreceivedbyaddress
```

### Monitoring Incoming Transactions

```powershell
# Watch for incoming transactions in real-time
while ($true) {
    $wallet = .\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet -rpcwallet="wallet-name" getwalletinfo | ConvertFrom-Json
    $mempool = .\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getmempoolinfo | ConvertFrom-Json
    Write-Host "$(Get-Date -Format 'HH:mm:ss') | Balance: $($wallet.balance) BTC | Unconfirmed: $($wallet.unconfirmed_balance) BTC | Mempool: $($mempool.size) txs"
    Start-Sleep -Seconds 10
}

# Check specific address for incoming transactions
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet -rpcwallet="wallet-name" listreceivedbyaddress 0 true | Select-String "your-address" -Context 5,5

# View recent wallet transactions
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet -rpcwallet="wallet-name" listtransactions "*" 5
```

### Sending Bitcoin

```powershell
# Send Bitcoin to an address
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet -rpcwallet="wallet-name" sendtoaddress "recipient-address" 0.001

# Send with custom fee rate (sat/vB)
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet -rpcwallet="wallet-name" sendtoaddress "recipient-address" 0.001 "" "" false true null "unset" null 20

# Send all funds from wallet
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet -rpcwallet="wallet-name" sendtoaddress "recipient-address" $(.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet -rpcwallet="wallet-name" getbalance) "" "" true
```

### Backup & Export

```powershell
# Backup wallet
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet -rpcwallet="wallet-name" backupwallet "D:\bitcoin-wallet-backups\backup-$(Get-Date -Format 'yyyy-MM-dd-HHmmss').dat"

# Export private key for specific address
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet -rpcwallet="wallet-name" dumpprivkey "your-address"

# Export all wallet private keys
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet -rpcwallet="wallet-name" dumpwallet "D:\wallet-export.txt"

# Restore wallet from backup
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet restorewallet "restored-wallet-name" "D:\bitcoin-wallet-backups\backup-file.dat"
```

---

## 🎯 Quick Reference Card

| Task | Command |
|------|---------|
| **Start node** | `.\build\bin\Release\bitcoind.exe -datadir=D:\bitcoin-mainnet` |
| **Stop node** | `.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet stop` |
| **Block height** | `.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getblockcount` |
| **Sync status** | `.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getblockchaininfo` |
| **Peer count** | `.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getconnectioncount` |
| **Mempool size** | `.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getmempoolinfo` |
| **Latest block** | `.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getbestblockhash` |
| **View logs** | `Get-Content D:\bitcoin-mainnet\debug.log -Tail 20` |
| **Disk usage** | `(Get-ChildItem D:\bitcoin-mainnet -Recurse \| Measure-Object -Property Length -Sum).Sum / 1GB` |

---

## 📚 Additional Resources

- **RPC API Reference**: `.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet help`
- **Specific command help**: `.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet help <command>`
- **All RPC commands**: `.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet help | more`

---

**Note**: This cheat sheet assumes you've completed Initial Block Download (IBD). If your node is still syncing, expect slower response times for queries.

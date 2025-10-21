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

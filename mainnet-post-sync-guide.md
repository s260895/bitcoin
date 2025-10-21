# Bitcoin Mainnet Node - Post-Sync Guide

## 🎉 Congratulations!

Your Bitcoin mainnet node is now fully synchronized with the network. You're running a complete, validating Bitcoin node that:
- ✅ Has verified the entire blockchain from the genesis block
- ✅ Is validating and relaying new blocks and transactions
- ✅ Is contributing to the Bitcoin network's decentralization
- ✅ Can serve as a trusted source of blockchain data

**Current Status:**
```
Blocks synced: 920,116+ blocks
Blockchain size: ~737 GB
Status: Full archival node (unpruned)
Network: Bitcoin Mainnet
```

---

## What Your Node Is Doing Now

### 1. Real-Time Block Validation
- Receiving new blocks every ~10 minutes
- Validating all transactions and consensus rules
- Adding valid blocks to your local blockchain
- Broadcasting valid blocks to peers

### 2. Transaction Relay
- Receiving unconfirmed transactions
- Validating transactions before adding to mempool
- Relaying valid transactions to network peers
- Helping propagate transactions across the network

### 3. Network Participation
- Maintaining connections with 8-125 peers
- Sharing blockchain data with other nodes
- Strengthening Bitcoin's decentralization
- Resisting censorship and central control

---

## What You Can Do Now

### 🔍 1. Explore the Live Blockchain

#### View Latest Block
```powershell
# Get the most recent block
$latest = .\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getbestblockhash
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getblock $latest 2

# This shows:
# - Block hash, height, timestamp
# - All transactions in the block
# - Miner information (coinbase transaction)
# - Block size, weight, difficulty
```

#### Explore Famous Blocks
```powershell
# Genesis Block (Block 0) - Satoshi's first block
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getblockhash 0
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getblock 000000000019d6689c085ae165831e934ff763ae46a2a6c172b3f1b60a8ce26f 2

# First Pizza Transaction Block (May 22, 2010) - 10,000 BTC for 2 pizzas
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getblockhash 57043

# First Halving Block (Block 210,000) - November 28, 2012
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getblockhash 210000
```

#### Watch Mempool Activity
```powershell
# See current unconfirmed transactions
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getmempoolinfo

# Watch mempool in real-time
while ($true) {
    $mp = .\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getmempoolinfo | ConvertFrom-Json
    Write-Host "$(Get-Date -Format 'HH:mm:ss') | Mempool: $($mp.size) txs | $([math]::Round($mp.bytes/1MB, 2)) MB | Min fee: $($mp.mempoolminfee) BTC/kB"
    Start-Sleep -Seconds 10
}
```

### 📊 2. Analyze Blockchain Statistics

#### Network Statistics
```powershell
# Current difficulty
$info = .\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getblockchaininfo | ConvertFrom-Json
Write-Host "Difficulty: $($info.difficulty)"
Write-Host "Chain work: $($info.chainwork)"

# Network hash rate (hashes per second)
$hashrate = .\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getnetworkhashps
Write-Host "Network hashrate: $([math]::Round($hashrate/1e18, 2)) EH/s (exahashes/second)"
```

#### UTXO Set Statistics
```powershell
# Analyze the entire UTXO set (WARNING: Takes 5-10 minutes!)
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet gettxoutsetinfo

# This shows:
# - Total number of UTXOs
# - Total amount of BTC in circulation
# - Database size
# - Best block hash
```

#### Block Analysis
```powershell
# Get statistics for a range of blocks
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getblockstats 920000

# Shows:
# - Average fee rate
# - Total fees collected
# - Number of transactions
# - Block size and weight
```

### 💰 3. Track Bitcoin Supply

```powershell
# Calculate total BTC mined (21M limit)
function Get-BitcoinSupply {
    $height = (.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getblockcount)

    # Calculate supply based on halving schedule
    $supply = 0
    $reward = 50.0
    $halvingInterval = 210000

    for ($h = 0; $h -lt $height; $h += $halvingInterval) {
        $blocksInEra = [math]::Min($halvingInterval, $height - $h)
        $supply += $blocksInEra * $reward
        $reward /= 2
    }

    Write-Host "Current block height: $height"
    Write-Host "Total BTC mined: $([math]::Round($supply, 8)) BTC"
    Write-Host "Remaining to mine: $([math]::Round(21000000 - $supply, 8)) BTC"
    Write-Host "Percentage mined: $([math]::Round($supply/21000000*100, 4))%"
}

Get-BitcoinSupply
```

### 🔍 4. Investigate Specific Transactions

```powershell
# Look up any transaction (requires txindex=1)
$txid = "your_transaction_id_here"
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getrawtransaction $txid true

# Analyze transaction fees
function Get-TransactionFee {
    param($txid)
    $tx = .\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getrawtransaction $txid true | ConvertFrom-Json

    # Calculate input value (requires full node with txindex)
    $inputValue = 0
    foreach ($vin in $tx.vin) {
        if ($vin.txid) {
            $prevTx = .\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getrawtransaction $vin.txid true | ConvertFrom-Json
            $inputValue += $prevTx.vout[$vin.vout].value
        }
    }

    # Calculate output value
    $outputValue = ($tx.vout | Measure-Object -Property value -Sum).Sum

    $fee = $inputValue - $outputValue
    Write-Host "Transaction Fee: $fee BTC"
}
```

---

## Node Maintenance

### Daily Operations

#### 1. Check Node Health
```powershell
# Quick health check script
function Test-NodeHealth {
    Write-Host "=== Bitcoin Node Health Check ===" -ForegroundColor Cyan

    # Check if running
    $process = Get-Process bitcoind -ErrorAction SilentlyContinue
    if ($process) {
        Write-Host "✓ Node is running (PID: $($process.Id))" -ForegroundColor Green
    } else {
        Write-Host "✗ Node is not running!" -ForegroundColor Red
        return
    }

    # Check sync status
    $info = .\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getblockchaininfo | ConvertFrom-Json
    $behind = $info.headers - $info.blocks
    if ($behind -eq 0) {
        Write-Host "✓ Fully synced (Height: $($info.blocks))" -ForegroundColor Green
    } else {
        Write-Host "⚠ Behind by $behind blocks" -ForegroundColor Yellow
    }

    # Check connections
    $netinfo = .\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getnetworkinfo | ConvertFrom-Json
    $connections = .\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getconnectioncount
    Write-Host "✓ Connected to $connections peers" -ForegroundColor Green

    # Check mempool
    $mempool = .\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getmempoolinfo | ConvertFrom-Json
    Write-Host "✓ Mempool: $($mempool.size) transactions ($([math]::Round($mempool.bytes/1MB, 2)) MB)" -ForegroundColor Green

    # Check disk space
    $drive = Get-PSDrive D
    $freeGB = [math]::Round($drive.Free / 1GB, 2)
    if ($freeGB -gt 50) {
        Write-Host "✓ Free disk space: $freeGB GB" -ForegroundColor Green
    } else {
        Write-Host "⚠ Low disk space: $freeGB GB" -ForegroundColor Yellow
    }

    # Check warnings
    if ($info.warnings) {
        Write-Host "⚠ Warnings: $($info.warnings)" -ForegroundColor Yellow
    }
}

Test-NodeHealth
```

#### 2. Monitor Resource Usage
```powershell
# CPU and Memory usage
$process = Get-Process bitcoind
Write-Host "CPU: $($process.CPU) seconds"
Write-Host "Memory: $([math]::Round($process.WorkingSet64/1GB, 2)) GB"

# Network traffic
$nettotals = .\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getnettotals | ConvertFrom-Json
Write-Host "Uploaded: $([math]::Round($nettotals.totalbytessent/1GB, 2)) GB"
Write-Host "Downloaded: $([math]::Round($nettotals.totalbytesrecv/1GB, 2)) GB"
```

#### 3. Log Monitoring
```powershell
# Check for errors in logs
Select-String -Path D:\bitcoin-mainnet\debug.log -Pattern "ERROR|Warning" -Tail 50

# Monitor new blocks being added
Get-Content D:\bitcoin-mainnet\debug.log -Wait | Select-String "UpdateTip"
```

### Weekly Maintenance

#### 1. Verify Chain Integrity
```powershell
# Quick verification (checks recent blocks)
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet verifychain 1 100

# Deep verification (monthly recommended)
# WARNING: Takes hours, high CPU usage
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet verifychain 4 1000
```

#### 2. Backup Configuration
```powershell
# Backup config file
Copy-Item D:\bitcoin-mainnet\bitcoin.conf "D:\backups\bitcoin.conf.$(Get-Date -Format 'yyyy-MM-dd').bak"

# Backup wallet (if using wallet functionality)
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet backupwallet "D:\backups\wallet-$(Get-Date -Format 'yyyy-MM-dd').dat"
```

#### 3. Check for Updates
```powershell
# Check current version
.\build\bin\Release\bitcoind.exe -version

# Check Bitcoin Core releases (manual)
# Visit: https://github.com/bitcoin/bitcoin/releases
```

---

## Advanced Usage

### 1. Enable Transaction Index (If Not Already)

**Why?** Allows you to query ANY transaction by txid, not just wallet transactions.

```powershell
# Stop node
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet stop

# Add to bitcoin.conf
Add-Content D:\bitcoin-mainnet\bitcoin.conf "txindex=1"

# Restart with reindex (takes hours!)
.\build\bin\Release\bitcoind.exe -datadir=D:\bitcoin-mainnet -reindex
```

### 2. Optimize Performance

#### Adjust Database Cache
```ini
# Edit D:\bitcoin-mainnet\bitcoin.conf
dbcache=4096  # Use 4GB RAM (default: 450MB)
              # Increase if you have RAM to spare
              # Reduces disk I/O significantly
```

#### Reduce Mempool Size (Save RAM)
```ini
maxmempool=100  # Limit to 100MB (default: 300MB)
```

#### Limit Connections (Save Bandwidth)
```ini
maxconnections=40  # Default: 125
                   # Reduce if bandwidth is limited
```

### 3. Set Up as a Local Block Explorer

```powershell
# Query any address balance (requires txindex + custom scripts)
# Query any block by height
# Analyze transaction patterns
# Track address activity

# Example: Get block by height
function Get-BlockByHeight {
    param($height)
    $hash = .\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getblockhash $height
    .\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getblock $hash 2
}

Get-BlockByHeight 800000
```

### 4. Monitor New Blocks in Real-Time

```powershell
# Watch for new blocks
$lastHeight = 0
while ($true) {
    $currentHeight = .\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getblockcount
    if ($currentHeight -ne $lastHeight) {
        $hash = .\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getbestblockhash
        $block = .\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getblock $hash | ConvertFrom-Json
        Write-Host "NEW BLOCK! Height: $currentHeight | Txs: $($block.nTx) | Size: $([math]::Round($block.size/1KB, 2)) KB" -ForegroundColor Green
        $lastHeight = $currentHeight
    }
    Start-Sleep -Seconds 10
}
```

---

## Security Best Practices

### 1. Firewall Configuration
```powershell
# Allow Bitcoin P2P port (if you want incoming connections)
# Port 8333 for mainnet

# Check current firewall rules
Get-NetFirewallRule | Where-Object {$_.DisplayName -like "*Bitcoin*"}

# Add rule (run as Administrator)
New-NetFirewallRule -DisplayName "Bitcoin Mainnet" -Direction Inbound -LocalPort 8333 -Protocol TCP -Action Allow
```

### 2. RPC Security
```ini
# In bitcoin.conf - restrict RPC access
rpcallowip=127.0.0.1  # Only localhost
rpcuser=yourusername  # Change default
rpcpassword=<strong_random_password>  # Use strong password
```

### 3. Regular Backups
```powershell
# Automated backup script
$backupDir = "D:\bitcoin-backups"
if (!(Test-Path $backupDir)) { New-Item -ItemType Directory -Path $backupDir }

Copy-Item D:\bitcoin-mainnet\bitcoin.conf "$backupDir\bitcoin.conf.$(Get-Date -Format 'yyyy-MM-dd').bak"

# If using wallet
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet backupwallet "$backupDir\wallet-$(Get-Date -Format 'yyyy-MM-dd').dat"
```

---

## Troubleshooting Common Issues

### Node Falls Behind Sync
```powershell
# Check peer quality
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getpeerinfo | ConvertFrom-Json |
    Select-Object addr, version, subver, synced_headers, synced_blocks

# Add faster peers manually (from bitnodes.io)
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet addnode "seed.bitcoin.sipa.be:8333" add
```

### High CPU Usage
```powershell
# Check if verifying old blocks
Get-Content D:\bitcoin-mainnet\debug.log -Tail 100 | Select-String "Verifying"

# Reduce validation threads (edit bitcoin.conf)
# par=2  # Use 2 threads instead of all cores
```

### High Disk I/O
```powershell
# Increase dbcache to reduce disk writes
# Edit bitcoin.conf:
# dbcache=8192  # Use 8GB if available
```

### Database Corruption
```powershell
# Rebuild chainstate only (faster, ~1-2 hours)
.\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet stop
.\build\bin\Release\bitcoind.exe -datadir=D:\bitcoin-mainnet -reindex-chainstate

# Full reindex (slower, ~24-48 hours)
.\build\bin\Release\bitcoind.exe -datadir=D:\bitcoin-mainnet -reindex
```

---

## Using Your Node for Development

### 1. As a Data Source
- Query blockchain data for analysis
- Build custom blockchain explorers
- Research Bitcoin economics
- Analyze transaction patterns

### 2. For Testing Applications
- Validate transactions before broadcasting
- Test wallet integrations
- Develop Bitcoin applications locally
- Simulate network conditions

### 3. For Privacy
- Validate your own transactions
- No reliance on third-party APIs
- Full control over your data
- Private blockchain queries

---

## Next Steps

### 🎓 Learning Path

1. **Study the codebase** - Use bitcoin-exploration-guide.md
2. **Analyze consensus rules** - Read validation.cpp
3. **Understand P2P protocol** - Watch network messages
4. **Explore script system** - Examine transaction scripts
5. **Build tools** - Create analysis scripts

### 🔧 Potential Projects

- Build a custom block explorer
- Create fee estimation tools
- Develop transaction monitoring
- Analyze blockchain statistics
- Build wallet applications
- Research network topology

### 📚 Resources

- **Developer Docs**: https://developer.bitcoin.org/
- **Bitcoin Core Docs**: https://github.com/bitcoin/bitcoin/tree/master/doc
- **RPC API**: `.\build\bin\Release\bitcoin-cli.exe help`
- **Bitcoin Stack Exchange**: https://bitcoin.stackexchange.com/

---

## Useful Monitoring Dashboard Script

Save as `monitor-node.ps1`:

```powershell
function Show-BitcoinDashboard {
    while ($true) {
        Clear-Host
        Write-Host "=== BITCOIN MAINNET NODE DASHBOARD ===" -ForegroundColor Cyan
        Write-Host "Refreshed: $(Get-Date -Format 'yyyy-MM-dd HH:mm:ss')" -ForegroundColor Gray
        Write-Host ""

        # Blockchain info
        $info = .\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getblockchaininfo | ConvertFrom-Json
        Write-Host "BLOCKCHAIN STATUS" -ForegroundColor Yellow
        Write-Host "  Height: $($info.blocks)"
        Write-Host "  Best block: $($info.bestblockhash.Substring(0,16))..."
        Write-Host "  Difficulty: $([math]::Round($info.difficulty/1e12, 2)) T"
        Write-Host "  Synced: $(if($info.initialblockdownload){'NO'}else{'YES'})"
        Write-Host ""

        # Network info
        $connections = .\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getconnectioncount
        Write-Host "NETWORK STATUS" -ForegroundColor Yellow
        Write-Host "  Connections: $connections"

        # Mempool info
        $mempool = .\build\bin\Release\bitcoin-cli.exe -datadir=D:\bitcoin-mainnet getmempoolinfo | ConvertFrom-Json
        Write-Host ""
        Write-Host "MEMPOOL STATUS" -ForegroundColor Yellow
        Write-Host "  Transactions: $($mempool.size)"
        Write-Host "  Size: $([math]::Round($mempool.bytes/1MB, 2)) MB"
        Write-Host "  Min fee: $($mempool.mempoolminfee) BTC/kB"
        Write-Host ""

        # System resources
        $process = Get-Process bitcoind -ErrorAction SilentlyContinue
        if ($process) {
            Write-Host "SYSTEM RESOURCES" -ForegroundColor Yellow
            Write-Host "  Memory: $([math]::Round($process.WorkingSet64/1GB, 2)) GB"
            Write-Host "  CPU Time: $([math]::Round($process.TotalProcessorTime.TotalHours, 2)) hours"
        }

        Write-Host ""
        Write-Host "Press Ctrl+C to exit" -ForegroundColor Gray
        Start-Sleep -Seconds 10
    }
}

Show-BitcoinDashboard
```

Run with: `.\monitor-node.ps1`

---

## Summary

Your Bitcoin mainnet node is now:
- ✅ **Fully operational** - Synced with the network
- ✅ **Validating** - Checking all new blocks and transactions
- ✅ **Contributing** - Strengthening network decentralization
- ✅ **Ready for use** - Available for development and analysis

**Keep it running!** The longer it runs, the more valuable it becomes to the network. Your node helps ensure Bitcoin remains decentralized, censorship-resistant, and trustless.

**Happy exploring!** 🚀

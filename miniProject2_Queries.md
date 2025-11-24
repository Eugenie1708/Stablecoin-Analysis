# **📊 USDT Competitive Intelligence Dashboard**

# **Overview** #
## A cross-token intelligence dashboard comparing USDT against USDC, DAI, and PYUSD across volume trends, sector penetration, user profiles, and retention behavior.
## This dashboard answers one question:
## **_Where is USDT winning, and where is it losing ground?_**

## All charts are interactive, ensuring dynamic and real-time competitive insights.

# USDT 60-Day Headlines
## Use this for: Counters (Volume, Tx Count, Active Users)
## This query calculates USDT’s headline activity metrics over the past 60 days, including total volume, transaction count, and the number of unique active wallets.
```
SELECT 
    -- Counter 1: Volume in Billions 
    -- Logic: (Value / 6 decimals) / 1,000,000,000
    SUM(value / 1e6) / 1e9 as total_volume_billions,
    
    -- Counter 2: Total Number of Transfers
    COUNT(*) as total_transfers,
    
    -- Counter 3: Unique Active Wallets (People)
    COUNT(DISTINCT "from") as unique_active_users

FROM erc20_ethereum.evt_Transfer
WHERE contract_address = 0xdac17f958d2ee523a2206206994597c13d831ec7 -- USDT Only
AND evt_block_time > NOW() - INTERVAL '60' DAY
```

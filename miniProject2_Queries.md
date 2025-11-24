# **📊 USDT Competitive Intelligence Dashboard**

# **Overview** #
A cross-token intelligence dashboard comparing USDT against USDC, DAI, and PYUSD across volume trends, sector penetration, user profiles, and retention behavior. 

This dashboard answers one question: **_Where is USDT winning, and where is it losing ground?_**
All charts are interactive, ensuring dynamic and real-time competitive insights.

## USDT 60-Day Headlines
Use this for: Counters (Volume, Tx Count, Active Users)
This query calculates USDT’s headline activity metrics over the past 60 days, including total volume, transaction count, and the number of unique active wallets.
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
## USDT Long-Term Retention Counter
Logic: Calculates % of users who hold > 30 Days.
Generates long-term retention rates (% of 30+ day active users) for major stablecoins based on historical transfer activity.
```
WITH raw AS (
    SELECT
        contract_address,
        "from" AS wallet,
        evt_block_time
    FROM erc20_ethereum.evt_Transfer
    WHERE contract_address IN (
        0xdac17f958d2ee523a2206206994597c13d831ec7, -- USDT
        0xa0b86991c6218b36c1d19d4a2e9eb0ce3606eb48, -- USDC
        0x6b175474e89094c44da98b954eedeac495271d0f, -- DAI
        0x6c3ea9036406852006290770bedfcaba0e23a0e8  -- PYUSD
    )
    AND evt_block_time > NOW() - INTERVAL '180' DAY
),

lifecycle AS (
    SELECT
        CASE 
            WHEN contract_address = 0xdac17f958d2ee523a2206206994597c13d831ec7 THEN 'USDT'
            WHEN contract_address = 0xa0b86991c6218b36c1d19d4a2e9eb0ce3606eb48 THEN 'USDC'
            WHEN contract_address = 0x6b175474e89094c44da98b954eedeac495271d0f THEN 'DAI'
            ELSE 'PYUSD'
        END AS token,
        DATE_DIFF('day', MIN(evt_block_time), MAX(evt_block_time)) AS retention_days
    FROM raw
    GROUP BY 1, wallet
)

SELECT
    -- This calculates the % of users with >30 days retention for each token
    COUNT(CASE WHEN token = 'USDT' AND retention_days > 30 THEN 1 END) * 100.0 / NULLIF(COUNT(CASE WHEN token = 'USDT' THEN 1 END), 0) as usdt_loyal_pct,
    
    COUNT(CASE WHEN token = 'USDC' AND retention_days > 30 THEN 1 END) * 100.0 / NULLIF(COUNT(CASE WHEN token = 'USDC' THEN 1 END), 0) as usdc_loyal_pct,
    
    COUNT(CASE WHEN token = 'DAI' AND retention_days > 30 THEN 1 END) * 100.0 / NULLIF(COUNT(CASE WHEN token = 'DAI' THEN 1 END), 0) as dai_loyal_pct
    FROM lifecycle
```
## The "Market Evolution" Chart (Volume Trends)
This query computes daily transfer volume (USD-normalized) for USDT, USDC, DAI, and PYUSD over the past 60 days.
```
WITH raw_data AS (
    SELECT
        date_trunc('day', evt_block_time) as time,
        CASE
            WHEN contract_address = 0xdac17f958d2ee523a2206206994597c13d831ec7 THEN 'USDT'
            WHEN contract_address = 0xa0b86991c6218b36c1d19d4a2e9eb0ce3606eb48 THEN 'USDC'
            WHEN contract_address = 0x6b175474e89094c44da98b954eedeac495271d0f THEN 'DAI'
            WHEN contract_address = 0x6c3ea9036406852006290770bedfcaba0e23a0e8 THEN 'PYUSD'
        END as symbol,
        CASE
            WHEN contract_address = 0x6b175474e89094c44da98b954eedeac495271d0f THEN value / 1e18
            ELSE value / 1e6
        END as amount_usd
    FROM erc20_ethereum.evt_Transfer
    WHERE contract_address IN (
        0xdac17f958d2ee523a2206206994597c13d831ec7, -- USDT
        0xa0b86991c6218b36c1d19d4a2e9eb0ce3606eb48, -- USDC
        0x6b175474e89094c44da98b954eedeac495271d0f, -- DAI
        0x6c3ea9036406852006290770bedfcaba0e23a0e8  -- PYUSD
    )
    AND evt_block_time > NOW() - INTERVAL '60' day
)
SELECT
    time,
    symbol,
    SUM(amount_usd) as daily_volume
FROM raw_data
WHERE symbol IS NOT NULL
GROUP BY 1, 2
ORDER BY 1 DESC;
```
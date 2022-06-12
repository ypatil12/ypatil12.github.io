---
title: 'Yearn Overview'
date: 2022-02-22
permalink: /posts/Yearn/
tags:
  - DeFi
  - Protocol Overview
---

A brief overview of vault mechanics in Yearn. 

### Vaults/Strategies
Each vault corresponds to a single deposit token (ETH, USDC, DAI, etc). For each vault, there can exist up to 20 strategies. Strategies report profit/loss to the vault, which will then issue debt back to the strategy to reinvest with. 

### Deposits/Withdrawals
Every deposit on Yearn maintains the [invariant](https://docs.yearn.finance/getting-started/guides/how-to-understand-yvault-roi#example)  `S = P * D` where S is the number of share tokens, P is the price, and D is the number of deposit tokens. Let’s assume that Alice has 100 ETH that she wants to invest in a yETH vault. At that time, there exists 10450 ETH and 10000 yETH. Thus, Alice will receive 100 * share price yETH, or 100 * (10000/10450) = 95.7yETH. Within the contract itself this is given by: 

- `SharesToMint = amount * totalSupply / freeFunds`
    - `totalSupply = supply of wrapped (yTokens)`
    - `freeFunds = totalAssets - lockedProfit`
        - `totalAssets = tokenBalance + totalDebt`

Where tokenBalance is the amount of ETH that is deposited in a vault and totalDebt is the amount of ETH that is invested across al strategies for the vault. Every withdrawal on a Yearn vault also maintains the above variant. If we assume that the vault has yielded an additional 100 ETH, then the value of yETH held by Alice = `95.7*10650/10095.7 = 100.9 ETH`. Essentially, `shareValue = shares * freeFunds/totalSupply`.

**If we have a profitable vault, the price per share will always go up.** Maintaining this invariant allows anytime deposit. Dripping the profit over time and minting shares without taking into account locked profit allows for anytime withdrawal and prevents both yield stripping attacks and sandwich attacks. 

### Oracles
Yearn does not use oracles in any of their vaults or strategies. The only exception is in the flash loan mint strategy, where oracle price of underlying is used. Not using an oracle implies that **every strategy is invested in a farm that has a corresponding token or derivative token.** For example, ETH can only be invested in an ETH LP, stETH, or ETH lending market. One asset cannot be swapped for another and invested accordingly (ie. USDC -> Frax -> Frax Curve LP -> Convex). 

### Keepers
Keepers can only call two functions on a strategy: tend and harvest. Tend will reinvest the returns based on the debt of the strategy (Tend is not commonly called and is more important on strategies that use leverage and need to be weary of collateralization ratios). Harvest not only reinvests returns but also reports profit/loss and receives credit from the vault. Keepers are incentivized with the K3PR token. 

The time between harvests varies based on the strategy - can be done every couple of hours or once a week. The lower and upper bounds for this are minReportDelay and maxReportDelay. 

### Harvests/Reports
Vaults will be harvested if there is a loss, if debt needs to be owed back to the vault, if we have passed the maxReportDelay, or if there is a profit and it is worth to harvest (in terms of gas cost). The Harvest mechanics are:
1. Get the debt outstanding from the vault (ie. how many tokens need to be withdrawn from strategy to the vault)
2. Report profit, loss, and debt payment to the vault based on debt outstanding
    - Vault management fee calculated 
    - If loss, lower the debt_ratio, totalDebt, and update loss
    - Take debt payment from strategy and issue additional credit (debt)
        - Credit issued is bounded by min and max value set by governance
3. Adjust position if we are in the minInvest/maxInvest time bounds
    - Swap, Stake, etc

Profits from the harvest are dripped linearly over 6 hours. Thus, the price per share will also increase linearly over this 6 hour period. A reported loss will result in the decrease in the price per share. 

### Withdrawal
Users can withdraw at any time. Funds are first taken from the vault if there are any other uninvested funds in the strategy. If there is more to withdraw, funds are taken from the strategy from a withdrawal queue. The queue is set by governance and is ordered based on which strategies will have the lowest impact on vault gains. Users can set a tolerance for how much they are willing to lose during the withdraw (eg. external fees for liquidation strategy funds from farms). **There is no withdrawal fee.**








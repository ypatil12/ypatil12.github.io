---
title: 'GRO Overview'
date: 2022-02-22
permalink: /posts/GRO/
tags:
  - DeFi
  - Protocol Overview
  - Stablecoins
---

[GRO](https://www.gro.xyz) is a protocol tranches stablecoin yields for DeFi users. 

### Overall Design
Gro is a fork of Yearn. As of February 2022, they have 4 yearn vaults (USDC, DAI, USDT, and 3Crv) with each vault having a primary and secondary strategy. 

Gro removed the share token (yv-Token) functionality from Yearn. Instead, users can deposit USDC, DAI, and USDT and choose to receive one of two yield-bearing tokens: PWRD and GVT. PWRD is a rebasing stablecoin and is the senior tranche. GVT (Gro Vault Token) is the junior tranche (similar pricing mechanism as yvTokens) that absorbs losses from PWRD strategies but also has higher yields from profit sharing on the senior tranche strategies. 

### Oracles
The amount of PWRD or GVT that is minted is based on the dollar amount of stablecoins deposited. The prices for each are checked via the 3crv oracle and sanity tested against the chainlink price for each token. The contract checks the amount of LP that would have been added into a 3crv pool and converts that into a USD amount through `get_virtual_price`. 


### PWRD: Rebasing Stablecoin
Rebases of PWRD occur anytime an event impacts the value of the underlying tokens backing PWRD, such as harvests or withdrawals. Rebases are generally not negative since losses are absorbed by GVT. The mechanics of rebasing are similar to most rebasing tokens. Here is how balances increase:
1. Alice calls balanceOf(address)
2. Contract checks the factor, or total amount of assets that back the PWRD token
    - This is based on totalAssets (total assets backing PWRD), which can be updated on deposits, withdraws, harvests, and price change updates
    - `Factor = totalSupply of PWRD/totalAssets`
3. Alice's balance is initialUserBalance / factor
    - As totalAssets increases, the Alice's balance increases too

### GVT Token
The GVT token behaves much like a standard Yearn Vault Token, except with oracle checks done on deposits and withdrawals. Below is an example flow for deposits

1. User deposits USDC/DAI/USDT
    - Curve prices are checked to not deviate some set percentage off the price of Chainlink Oracle
    - The dollar amount added is calcualted as:
        - `DollarAmt = 3Pool_LP * 3Pool_VirtualPrice`, where `3Pool_LP` is the amount of LP returned if the user had just deposited usdc/dai/usdt
2. Contract checks the factor, or total amount of assets that back the GVT token
    - Just as for PWRD, this is based on totalAssets. 
3. Amount of GVT Sent to User = `DollarAmt*Factor = DollarAmt * Total GVT Supply / totalAssets`
    - The above formula is similar to Yearn’s, except `DollarAmt` is multiplied by `totalSupply/totalAssets` instead of the input token amount

### Withdraw Example
Assume we are withdrawing GVT and want a balanced amount of DAI/USDC/USDT returned

1. Get the value of GVT Owned. `GVT_Owned * pricePerShare`
2. `returnUSD = GVT_owned - withdrawFee`
3. `lpAmount = GVT_owned/virtual_price` (of 3CrvPool)
4. Get fractions for how much USDC/DAI/USDT to withdraw (based on returnUSD), multiply LP amount, and convert from LP amount to Stable coin. Conversion is done by calling `calc_withdraw_one_coin` on Curve. Although this function on a curve pool is not resistant to manipulation, GRO cross checks it against 


### Distribution of Yields
The distribution of yields between PWRD and GVT depends on the `Utilization Ratio = Assets in PWRD/Assets in GVT`. If the utilization ratio is 25%, then for a single vault, 75% of assets are in the primary strategy and the remaining assets are in the secondary strategy. 

If the utilization ratio (UR) is less than 80%, then the vault will earn yields from the primary strategy and also a rate of 30% + ⅜*UR of the yields from the secondary strategy. If the UR is more than 80%, then the vault will earn a rate of 60% + 2(UR-80) of the yields from the secondary strategy. [More info](https://docs.gro.xyz/gro-docs/pwrd_vault/vault-less-than-greater-than-pwrd-interaction/profit-sharing-mechanism)


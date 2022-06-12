---
title: 'Adding Liquidity to Curve'
date: 2022-05-17
permalink: /posts/CRV-Liquidity/
tags:
  - DeFi
  - Protocol Thoughts
---

At Ondo, we're working on strategies that use Curve Pools. This got me thinking - what is the safest way to add liquidity to a Curve Stableswap Pool given very [clever sandwich strategies](https://twitter.com/bertcmiller/status/1527757146716348416)

For background, in Curve, In curve, `virtual_price` is essentially an accounting mechanism for the price of LP tokens. Virtual_price for an LP token should always increase in stableswap pairs due to accumulation of fees (this is not the case in the tri-crypto pool or in non-correlated asset pools in curveV2). `Calc_token_amount` returns the amount of LP tokens received given a list of tokens to LP into a pool. `Calc_token_amount` can be manipulated and is not safe to use as-is when being called from a smart contract. 

When adding liquidity to a curve pool, there are two forms of slippage tolerance that we need to watch out for:
1. Slippage caused by being sandwiched 
2. Slippage caused by adding an imbalanced ratio of tokens 

For 1, generalized frontrunning bots can monitor the mempool for transactions that add liquidity. Protocols must be even more careful when they allow keepers to permssionlessly add pooled liquidity as they can easily be frontrun - see [FEI Postmortem](https://medium.com/immunefi/fei-protocol-flashloan-vulnerability-postmortem-7c5dc001affb). To prevent being frontrun, users can do the following:
- Use flashbots protect
- Add liquidity only if the slippage does not cause an LP returned amount to deviate from a set oracle price - call `get_virtual_price` to return the expected LP and multiply by the oracle price. See this [chainlink article](https://blog.chain.link/using-chainlink-oracles-to-securely-utilize-curve-lp-pools/) for more info. 

Doing the above two together is the most secure. 

For the second form of slippage, note that Curve also allows users to add an imbalanced ratio of tokens when adding liquidity. The catch is that a user pays extra fees for any resulting pool imbalance. To prevent too large a price impact, we can set a slippage tolerance based on expected number of tokens to be received. Simply divide balance of pool by `virtual_price` and then subtract by expected slippage amount. A discussion on adding liquidity can be found in this [discord convo](https://discord.com/channels/729808684359876718/729812922649542758/897814906794029096).

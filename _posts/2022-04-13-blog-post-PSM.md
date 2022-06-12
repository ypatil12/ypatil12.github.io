---
title: 'PSMs in DeFi'
date: 2022-04-13
permalink: /posts/PSM-Overview/
tags:
  - DeFi
  - Stablecoins
---

1:1 redeemability is the primary mechanism for a crypto-collateralized stablecoin to maintain peg. More specifically, such a system creates a risk free arbitrage opportunity for arbitrageurs to buy or sell stablecoins when they are off peg. A couple notable PSMs include:

- Swaps via Rate-Limited PSM-Contract (FEI-Like)
- Swaps over a CDP-Wrapper (Maker-Like)
- 1-way swaps from stablecoin -> collateral asset (Liquity-Like)

One can think of PSMs as a simple Decentralized Exchange (DEX) contract, but unlike protocols such as Uniswap or Curve, these DEXs are not automated market makers (AMMs) that govern the exchange of tokens based on a curve. They simply allow conversion between assets at a static price. In this overview we assume knowledge of Maker, CDPs, and Vaults. 

### FEI PSM
FEI’s rate limited PSM contracts allow 1:1 redeemability of FEI into a ETH, DAI, or LUSD, the latter of which can be activated during market downturns. Each PSM may have fees on mint/redeem; for example, on the DAI PSM, users pay a 0 bps fee on minting FEI and a 10 bps fee on redeeming FEI for DAI. In general, every unit of FEI in circulation is redeemable for $1 of assets in the PCV, keeping the stablecoin overcollateralized and creating an arbitrage opportunity to keep FEI at peg. 

The ETH PSM contract checks the price of ETH against a chainlink oracle to calculate the FEI given/received. In contrast, the DAI psm simply just checks if the price of DAI given by an oracle is outside a price floor/ceiling. If so, redemptions are paused. Each PSM contract's reserves are not directly redeemed from the PCV. Instead, the PCV will either drip funds to replenish tokens or an admin will take an action to transfer into the PSM contract. Any surplus token transferred is invested into money markets, such as DAI into Compound. 
	
### Maker PSM
For background, Maker originally used Stability Fees and the Dai Savings Rate to keep DAI at peg. The former is an interest rate charged on CDP borrowers and the latter is interest earned for staking DAI. During DeFI summer, circular COMP farming increased the demand of DAI and drove up price. Furthermore, the demand for DAI being greater than supply made it difficult for people to close out borrow positions as not enough DAI was in circulation. As demand for DAI increased, Maker adjusted the CR of USDC-A vaults from 125% CR down to 101% CR with a 4% stability fee. Because of a stability fee on these vaults, some ended up becoming undercollateralized. Although these liquidations could generate revenue for the protocol, Maker’s collateral auction mechanism isn’t good at pricing low-risk assets. Maker eventually shut liquidations off and transitioned to a new mechanism. 

Maker’s PSM today is a special wrapper around a CDP vault with a 100% CR and 0% Borrow Rate. When an arbitrageur trades USDC for DAI, the USDC is added to a CDP position and DAI is minted at a 100% CR. When a user trades DAI for USDC, the DAI is used to pay down the debt in a CDP and free up USDC in the vault. Thus, one can consider Maker’s PSM as doing swaps over a CDP. 

There are currently no mint or redeem fees to make this trade, resulting in an attractive opportunity of risk-free arbitrage. Maker started out having fees, which did generate revenue for the protocol, but lowered to be able to improve the UX of DAI and increase its utility as being pegged to USD for institutions and algo stablecoins, among other reasons. One point of contention among many folks is the inefficiency of the PSM. Unlike FEI, which invests excess capital into DeFi, idle USDC in Maker earns no revenue for the protocol. 

### Liquity PSM
For background, [Liquity](https://www.liquity.org) is a fork of Maker that only allows ETH as collateral in its vaults. Vaults have a 110% CR (collateralization ratio) and 0% SF (stability fee). Liquity's stablecoin is LUSD and their governance token is LQTY.

Any LUSD holder can redeem their LUSD for ETH 1:1, plus a small redemption fee. The protocol takes the ETH from the riskiest CDP vaults (ie. those with the lowest CR) and uses the returned LUSD to pay off debts of said vaults. Thus, the LUSD is burned by the protocol. The redemption fee is dynamic. As more redemptions occur, the fee increases as a function of supply. Conversely, as redemptions cool, the redemption fee decreases. To disincentivize users from creating more CDPs after a redemption, which would bring the LUSD price back <$1, LUSD vaults have issuance fees. These fees are paid when LUSD is borrowed and, critically, adjust along with the redemption fee. 

Liquity has a couple of mechanisms to bring the price LUSD down when it's above peg. The ceiling for the price of LUSD is at $1.10, given the 110% CR. If the price of LUSD goes above $1.10, an arbitrageur can create a CDP, mint LUSD, and profit from the delta between the two prices. In addition, the LUSD staked in a stability pool, which is used to pay off liquidated CDP vaults, helps bring the LUSD back to peg since depositors could end up losing money off of liquidations as the price approaches $1.10 and may instead decide to move their LUSD in DeFi money markets, decreasing the price. 



---
title: 'Maker Overview'
date: 2022-03-01
permalink: /posts/Maker/
tags:
  - DeFi
  - Protocol Overview
  - Stablecoins
---

An overview of some mechanisms behind Maker. Much of this post has been adapted from Maker's amazing docs [here](https://docs.makerdao.com) and [here](https://makerdao.world/en/learn/MakerDAO).

## Auctions
There are three types of auctions in the system: collateral liquidations auctions, debt auctions, and surplus auctions.
### Collateral Liquidation Auctions (FLIP)
When the value of a user’s collateral goes below the set collateralization ratio (for ETH this is 150%), the user’s collateral is auctioned off. For example, assume I deposit $100 of ETH into a Maker vault and borrow $66 worth of DAI. If the value of my ETH goes below $100, I will be liquidated. The collateralization ratio (CR) of a particular vault is defined as `CR = (Collateral Amount * Collateral Price) / DAI Debt * 100`. The price at which a user’s collateral can be triggered to auction is the `liquidation price = (Dai Debt * CR) / (Collateral Amount)`. Any user can trigger a collateral auction and is incentivized to do so with a fixed + proportional fee incentive, which is a percentage based on the value of the debt being liquidated. This incentive should be less than the liquidation penalty (described below) to prevent keepers from making a profit by opening many vaults and farming liquidation fee incentives. 

The liquidation auction is a dutch auction defined by an auction price function. The price of the collateral auctioned off decreases by a predefined amount (cuts) every set interval of time (step). The initial price is defined by an auction price multiplier - if the multiplier is 1.2, and the value of collateral is $800, then the bidding starts at $960 and decreases over time. Note that the value of the debt that the auction tries to raise includes a liquidation penalty that is paid to the protocol. 

Bids for an auction do not have to buy up all the collateral - the auction allows partial purchases. For example, if the protocol aims to get rid of $60,000 of bad debt, Alice can come in to buy $50,000 of collateral when the price is 1000 DAI per unit of collateral. Later on, as the price drops over time, Bob can come in and buy $10,000 of the remaining debt when the price is 800 DAI per unit of collateral. Note that keepers are not setting the price at which the collateral will be bought at. Instead, they specify the amount of collateral they wish to buy at a price defined by the auction price function. 

If too much time has elapsed before the auction has completed or if the collateral price goes below a set floor (max auction drawdown), the auction is restarted. Auctions will also not be initiated if the global amount of DAI to be raised across all auctions is greater than the global liquidation limit. 

### Debt Auctions (FLOP)
Debt auctions are triggered when not all the collateral in a liquidation auction is auctioned off. For example, this can occur when the debt to be raised for the collateral is greater than the ceiling on the maximum amount of DAI that can be raised in an auction. Excess collateral can also be present if there is a small amount of collateral remaining at the end of a collateral auction (auction may not be reset to avoid cluttering auction queue). In these cases, bad debt accrues to the protocol’s balance sheet. 

The debt auction is a dutch auction in which bidders will bid to accept decreasing amounts of MKR, Maker's governance token, for paying a fixed amount of DAI. The DAI is then burned to cover the system debt. The auction is triggered when the protocol has a debt in DAI that is below the system limit. Each bid in the auction must be decremented from the previous bid by a set amount. The auction completes when the auction duration has been reached or when there hasn’t been a bid for a set period of time. Debt auctions naturally results in downward price pressure on MKR. 

### Surplus Buffer Auctions (FLAP)
Surplus buffer is a reserve of DAI that protects protocol when there is bad debt. The buffer accrues from stability fee revenue. Once the buffer becomes too high, a FLAP auction is triggered and DAI is auctioned for MKR. This MKR is burned, creating upward pressure on the price of the governance token. 

The surplus buffer auction is a standard English auction where bidders bid increased amounts of MKR for a set amount of DAI. Each bid must be increased from the previous maximum bid by a set amount. 

In general, the greater the debt in the system, the larger the surplus buffer should be to balance risks. However, the larger the surplus buffer, there is less amount of MKR being burned and also less DAI in the market, creating upward pressure on peg. Conversely, with a low surplus buffer, there may not be enough DAI to cover bad debt, increasing the potential for a debt auction and resulting inflation of the MKR supply. Governance must balance between these two to set the optimal surplus buffer. 

## Oracle
The price of a collateral type is defined by the medianizer and Oracle Security Module (OSM). The medianizer takes the median of 20 price feeds to find the market price of an asset. The OSM delays the price feed of the medianizer by one hour, which gives the protocol time to react in the case of an attack. This delay also allows users to top-up their debt in the event that they expect to be liquidated. 

The downside of the one hour delay is the potential vector of an OSM timing attack, where if there is a steep price drop of collateral, vault owners can shift loss to the protocol by minting DAI and leaving bad debt. In the case where the deviation between the previous oracle price and the next oracle price is too large, liquidation auctions will pause. The percent difference between these two prices is defined as the breaker price tolerance, which is a parameter tunable by governance. 

## Peg Stability
### Peg Stability Module (PSM)
The PSM allows users to swap stablecoins for DAI at a fixed rate. It is essentially a vault with a stability fee of 0% and a CR of 100%. When DAI demand is greater than supply, causing an increase in price of DAI, the PSM can be used to keep DAI at peg by allowing users to swap USDC in. The reverse holds when the peg is below $1. These create incentives for arbitrageurs to maintain peg. 

### Stability Fee (JUG)
Stability fees, or borrow rates, are the fees paid by vault owner’s on the amount of DAI they mint. The fee varies depending on the type of collateral used to mint DAI. In general, the riskier the collateral, the higher the stability fee will be. A single type of collateral can have multiple stability fees, meaning there will be multiple vaults for that collateral. For example, Maker has ETH-A and ETH-B, where the former has a higher CR but a lower stability fee and the latter has a lower CR but a higher stability fee. The fee can also be adjusted by governance such that the protocol can increase its revenue or be compensated for a riskier collateral type. 

The stability fee is the largest source of revenue for Maker. 

### Debt Ceiling
The debt ceiling is the maximum amount of DAI that can be minted for a single collateral type. Specifically, the debt ceiling is the current maximum debt that can be adjusted by anyone and the debt ceiling limit is a hard cap, adjustable by governance. Debt ceiling governance is minimized by allowing users to adjust debt ceiling up to the debt ceiling limit or down up to the defensive debt ceiling, which is the debt floor. 

### Dai Savigs Rate
Dai locked in the DSR contract accrues interest at the same global rate (irregardless of collateral used to mint DAI). The DSR is effectively a monetary policy engine for the protocol. 

When DAI is below peg, governance can increase the DSR, which will increase the demand for DAI and increase the price. Conversely, when DAI is above peg, decreasing the DSR lowers the demand for DAI and thereby decreases the price of DAI. 

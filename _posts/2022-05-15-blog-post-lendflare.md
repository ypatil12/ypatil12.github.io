---
title: 'Thoughts on Lendflare'
date: 2022-05-15
permalink: /posts/Lendflare-thoughts/
tags:
  - DeFi
  - Protocol Thoughts
---

[Lendflare](https://lendflare.finance) is a protocol that allows users to lever up on their Convex positions, whilst still earning CRV, CVX, and native pool rewards. 

One part of the protocol that caught my eye was that they offer a *fixed* borrow rate for the duration of the loan. How can that be? There must be some sort of compromise... let's dig in.

Lendflare's borrow rate is defined as `Borrow Rate = Lendflare Multiplier * Comp Borrowing Rate`. This gives lenders to the protocol a yield that is at least as good as lending on Compound. The Lendflare Multiplier is defined by a [utilization curve](https://en-docs.lendflare.finance/lend-flare-v1/borrow-asset#li-lv-ji-suan) with a single kink. Thus, given the linked graph, if the current utilization rate is 20% and the COMP borrow rate is 3.10%, then I will receive a `3.875%=3.10*1.25` interest rate for the duration of my loan, even if I drive utilization up above the kink and towards 100%. The *very next* person who opens up a position will receive a much higher borrow rate if they borrow the remaining amount (roughtly `12.4%=4*3.1). Thus, the rate at which you borrow depends on the size of the loan of those who opened up a position before you. 

Upon review of the [code](https://github.com/LendFlare/lendflare_finance_contracts/blob/main/contracts/LendingMarket.sol#L218), I found a design decision that was quite alarming. Those who open up a position to borrow have no opportunity to set the max rate at which they would like to borrow. Let me state that in different terms: if you and another individual open a borrow position in the same block, the person whose transaction was included second has no protection against paying a subsantially higher borrow rate. Thus, for users of Lendflare, I recommend using flashbots protect to hide your transactions from others who try to frontrun your borrow to get a cheaper loan. Although this is overkill given the size of the protocol today, it may be necessary if Curve/Convex yields come back and there is demand for levering up positions on Lendflare. In the long run, hopefully the protocol implements a simple check to allow the user to specify a max borrow rate when opening up a position. I've [suggested they do so](https://discord.com/channels/909992314024390706/956934066559660082/979781106582818876) in their v2. 
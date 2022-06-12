---
title: 'Real-world NFT Verification'
date: 2022-06-01
permalink: /posts/NFT-Verification/
tags:
  - DeFi
  - Ideas
---

On a recent Bankless episode, Kevin Rose discusses how he needs solutions to allows proof collective members to be able to verify themselves in the real world ([Starts at minute 48](https://www.youtube.com/watch?v=6RAFSFowZSk)). The 

Currently, there is no *simple* you can walk to any location or place to say that you own an NFT. 

One may ask, why can't we just use people's hot wallets to verify ownership, like credit/debit cards. Unlike a self-custodied NFT, if someone steals a debit/credit card, you can go to your bank or provider and immediatley cancel that card. Even better is the piece of mind knowing that if there are fradulent transactions, your bank will void them. If someone steals a wallet there are a couple of scenarios:
1. Stealer gets a wallet, but can't spend assets
  - If a user has safely backed up her secret, she can safely rescue assets
  - If a user does not keep her secret, NFTs are bricked
2. Worst case, the stealer gets a wallet and is somehow able to unlock metamask on a user's phone 

Not everyone will practice proper opsec even with wallets that have valuable NFTs where the incentive to steal is high. Furthermore, given the security concerns of holding an NFT that's worth 5-6 figures, one does not want to be walking around with their wallet for verification. 

So given that we don't want NFT holders to be carrying around their wallets, how do we prove that they own an NFT? In the real world, how do people prove that they own an asset without having that hard asset? For money to spend at a PoS or online service, a major card issuer has a network that a gatewa can tap into to cross-check bank balances or credit available. For non-fungible items, such as houses or cars, a centralized record service like a state's DMV or housing record can be cross-checked. 

For verifying ownership of an NFT, we would want the experience to be just as easy as scanning a card at a PoS but with the security that banks provide their users. In a broad sense, this card (or QR code, digital wallet card, etc) should not be able to spend your funds and is essentially read-only for one's assets. If someone steals your card, nobody can spend assets because it does not store your private key. Furthermore, one should be able to use their private key (or a secure login or backup mechanism) to disable this card to revoke read access for their NFTs so that it cannot be reused.

Open Problems here:
- How do we plug in with a PoS that's at these venues? Square app marketplace?
- Would BAYC or other blue-chip NFT holders want some sort of "personalized" card that verifies membership?
- How centralized/decentralized can this service be? Perhaps use IPFS for storage and the Graph for indexing transfer events on the NFTs. Or, manage everything in a centralized db?

---
layout: post
title: "Atem"
permalink: /atem/
---
[Atem](https://app.atem.earth/) is a carbon credit aggregation protocol that integrates with tokenized carbon credits on Polygon and Celo. 

## Integrating with Toucan and C3 (then SolidWorld)
The first major project with Atem was to build adapter smart contracts to interact with the different tokenisation protocols that existed. Toucan was the largest of these, and thus the first one we worked on. Our adapter contract had to make it possible to both buy and retire tokenised carbon credits from our app. This involved swapping USDC for one of the Toucan pool tokens on Sushiswap, and then redeeming that pool token for the relevant project token.
I built first a Toucan adapter and then a C3 adapter, and deployed both into production on the polygon chain. The contracts were built in VS Code in Solidity, with unit tests built using Hardhat and Typescript, using a forked node. Deployment was handled by hardhat tasks that automated contract verification, deployment logging, as well as adding the contracts to OZ Defender and Sentinel.
### Challenges
One of the most challenging things was to get the C3 adapter to execute two swap across two different DEX's - the first one on Sushiswap was not an issue, but the second one on Sushiswap Trident presented a bit more of a challenge because the Trident code is substantially different to the Uniswap approach that most DEX's have followed. 

Unfortunately the Atem monorepo is private and therefore can't be shared here.

## NEXT SECTION COMING SOON


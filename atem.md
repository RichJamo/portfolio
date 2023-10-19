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
One of the most challenging things was to get the C3 adapter to execute two swap across two different DEX's - the first one on Sushiswap was not an issue, but the second one on Sushiswap Trident presented a bit more of a challenge because the Trident code is substantially different to the Uniswap approach that most DEX's have followed. I was able to get this working by reading up on the Trident docs and finding code examples of how to implement swaps on Trident.

Unfortunately the Atem monorepo is private and therefore can't be shared here.

## Subgraph Creation and Deployment
There were a number of subgraphs required for our protocol - the first was to get live prices of the Toucan bundle tokens from Sushiswap. This meant have a subgraph that would give us the reserves of the relevant pools in Sushiswap, so that we could calculate an up-to-date price. For this we adapted an existing Sushiswap subgraph, but brought it up-to-date (in terms of its dependencies) and radically simplified the code to remove data fields that we didn't need.
The second subgraph was one to track events coming out of our own adapter contracts, specifically purchases and retirements of tokenised carbon credits. These events were essentially transaction confirmations, telling us that a blockchain transaction had completed successfully, and allowing us to update our backend to reflect past purchases as well as changes in inventory for specific users. 

### Challenges
The initial challenge for me in starting to work with subgraphs was just to get my head around the different moving parts - the manifest, the mappings, and the template. After that there were definitely some challenges in figuring out how to debug the code without having to redeploy the subgraph every time, and there were some challenges in refactoring our existing Sushiswap subgraph, especially when all of the dependencies had to be updated, which broke some parts of the code which had been deprecated in the newer dependencies.

## Threat Modelling - COMING SOON

### Challenges

## Migrating from Tangany to OZ Defender Relayer - COMING SOON

### Challenges


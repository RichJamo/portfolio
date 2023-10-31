---
layout: post
title: "Atem"
permalink: /atem/
---
[Atem](https://app.atem.earth/) is a carbon credit aggregation protocol that integrates with tokenized carbon credits on Polygon and Celo. 

## Integrating with Toucan and C3 (then SolidWorld)
The first major project with Atem was to build adapter smart contracts to interact with the different tokenisation protocols that existed. Toucan was the largest of these, and thus the first one we worked on. Our adapter contract had to make it possible to both buy and retire tokenised carbon credits from our app. This meant automating the swapping of USDC for one of the Toucan pool tokens on Sushiswap, and then redeeming that pool token for the relevant project token via the Toucan smart contract.
I built first a Toucan adapter and then a C3 adapter (C3 is a second carbon credit protocol on polygon), and deployed both into production on the polygon chain. The contracts were built in Solidity, with unit tests built using Hardhat and Typescript, using a forked node. Deployment was handled by hardhat tasks that automated contract verification, deployment logging, as well as adding the contracts to OZ Defender and Sentinel.
### Challenges
One of the most challenging things was to get the C3 adapter to execute two swap across two different DEX's - the first one on Sushiswap was not an issue, but the second one on Sushiswap Trident presented a bit more of a challenge because the Trident code is substantially different to the Uniswap approach that most DEX's have followed. I was able to get this working by reading up on the Trident docs and finding code examples of how to implement swaps on Trident.

Unfortunately the Atem monorepo is private and therefore can't be shared here.

## Subgraph Creation and Deployment
There were a number of subgraphs required for our protocol - the first was to get live prices of the Toucan bundle tokens from Sushiswap. This meant we needed a subgraph that would give us the reserves of the relevant pools in Sushiswap, so that we could calculate an up-to-date price. For this I adapted an existing Sushiswap subgraph, but brought it up-to-date (in terms of its dependencies) and radically simplified the code to remove data fields that we didn't need.
The second subgraph was one to track events coming out of our own adapter contracts, specifically purchases and retirements of tokenised carbon credits. These events were essentially transaction confirmations, telling us that a blockchain transaction had completed successfully, and allowing us to update our backend to reflect past purchases as well as changes in inventory for specific users. 

### Challenges
The initial challenge for me in starting to work with subgraphs was just to get my head around the different moving parts - the manifest, the mappings, and the template. After that there were definitely some challenges in figuring out how to debug the code without having to redeploy the subgraph every time, and there were some challenges in refactoring our existing Sushiswap subgraph, especially when all of the dependencies had to be updated, which broke some parts of the code which had been deprecated in the newer dependencies.

## Threat Modelling - COMING SOON
Our goal at Atem was to create a product that traditional companies would use, so we had to be extremely careful to ensure that our smart contracts were not vulnerable to attacks, and particularly that client funds (in the form of USDC or carbon credits) would not be at risk. In order to do this we created an overview of all of the systems that we use and how they are connected (databases in AWS, smart contracts, front end code, hosting service, multisigs, oracle contracts, Coinbase). We then created a series of detailed tables describing any and all possible threats that we could think of, also detailing the size of the threat. Finally we created a table of security measures to cover all of these potential threats - detailing both what we could do to prevent (or minimize) the threat, as well as how we could make sure we could monitor the threat and how we would respond if it did come to pass.

### Challenges
The major challenge with this threat modelling work is knowing that vulnerabilities could exist where you least expect them to. The key then is to be as thorough as possible, looking at every component of the system and taking the time to imagine as many different scenarios as possible. What helped was to think like an attacker, and try to identify from that perspective where the vulnerabilities might be. It also helped to document all of these details in an organised way (we used tables with sub-pages) in a secure area of our notion, so that we could add to it over several weeks.

Finally, we also shared our work (both our threat modelling and our smart contracts) with a security audit firm to get an outside perspective and check that we weren't missing any obvious vulnerabilities. 

## Migrating from Tangany to OZ Defender Relayer - COMING SOON
When we initially built Atem, we built it with the idea that each of our customers' carbon credit tokens would be stored in their own separate wallet, and that these wallets would be custodial wallets, hosted by a service called Tangany. This had a number of implications for how we built both our smart contracts and our back end. The back end would mainly interact with Tangany, via the Tangany API, and then Tangany in turn would make calls to our adapter smart contracts. Tokens could then be transferred from the Tangany custodial wallets to our adapter contracts, and vice versa.

Based on several strategic decisions made within the business, it became clear that it wasn't necessary for us to have individual custodial wallets for each of our customers. Tokens could be stored as securely, if not more securely, in a single wallet, with watertight logic at the level of the smart contract to track who's tokens were whose.

However, we still wanted to work through some kind of service that would sit between our backend and our adapter smart contracts. We could have built this ourselves, but when we looked at the OZ Defender API, we realised that it would do exactly what we needed it to do. We could create a Relayer smart contract wallet within OZ Defender, and then we could make calls via the OZ Defender API to initiate the calls to our smart contracts that would be needed to buy tokens or retire tokens. The benefit of this is the OZ Defender would take care of making sure that our smart contract calls went through even if there were delays or problems at the node level.

This move from Tangany to OZ Defender needed to be pulled off seamlessly, with no risk to our customers funds and no interruption to our offering. I spend several months first researching alternatives to Tangany (before settling on OZ Defender), and then also speccing out the architecture for how our backend would interact with OZ Defender and how that in turn would interact with our smart contracts. In the end we were able to switch across without any problems whatsoever, and on schedule.

### Challenges
The major challenge here was to balance convenience with security, without leaning too far one way or the other. If we put all of our customers carbon credit tokens into a multisig wallet, that would be the most secure solution, but it would also be the least convenient - we would have to execute approval calls requiring multiple signatures every time a client wanted to retire a credit. On the other hand, if we stored all of our clients credits on a hot wallet, then they might well be targeted by hackers, especially as the value of credits stored started to rise. The compromise solution that we settled on was to store small amounts of credits on the hot wallet, but to move larger amounts across to the multisig. We set a threshold amount for this, and set up alerts so that we would know when we had to move credits one way or the other.

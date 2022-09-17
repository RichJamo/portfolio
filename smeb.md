---
layout: default
title: "SMEB AutoBalancer"
permalink: /smeb/
---

[SMEB](https://spheron.infura-ipfs.io/ipfs/QmUjegH2uJPHjYceU4SRtm6qrdQbommrfcLnE6jJbAro72/) is a non-custodial, automatically rebalanced, equal weighted index of SUSHI, MATIC, ETH and BTC, on the Polygon Mainnet, and using SushiSwap's BentoBox as a base. Invest USDC in order to participate in the index. Withdraw your funds at any time. (source code can be found [here](https://github.com/RichJamo/BentoBoxBalancer))

I built this Dapp originally for the Encode Hack Africa Hackathon back in 2021, based on a personal desire that I had to create an automatically rebalancing portfolio of crypto tokens for myself. You can see the video of my hackathon submission below:  
<iframe width="560" height="315" src="https://www.youtube.com/embed/FGsvNk8yzNo?start=1686" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

The Dapp placed 2nd in that hackathon.

I have since continued to work on small improvements to the dapp when time has allowed, and entered it into two more hackathons this year - see [here](https://devpost.com/software/automated-rebalancer) for my submission to the Polygon BuidlIt Hackathon 2022.

## Challenges
One of the challenges on the first version of the dapp was to integrate it onto the SushiSwap BentoBox platform - any token flows in / out of the dapp had to go via the bentobox interface, and there were some ways in which this was slightly counter-intuitive. I met this challenge by going back to the technical videos that the SushiSwap team had shared with us, and by communicating with some of their devs over discord.

The next big challenge was to create the logic for the actual rebalancing of the tokens. This had to be done via a series of swaps, where the number of swaps is n-1 if the number of tokens is n. One of the key design decisions that I had to make was how much of this to do in solidity versus javascript. In the end I created a struct representing a token, which contained variables for, among other things, the USD equivalent of the dapp's token balance and the difference between that balance and what it needed to be for the portfolio to be balanced. This made it possible to calculate the amounts that needed to be swapped in order to rebalance the portfolio.

In order to get the USD exchange rate of each token, I used Chainlink price data feeds, integrated directly into the smart contract.

Another challenge was automating the rebalancing process, so that it wouldn't require someone manually calling the smart contract whenever it needed doing. This I did via Chainlink Keepers, which had the added benefit of allowing me to take a lot of the rebalancing logic off-chain.

The repo for this project can be seen [here](https://github.com/RichJamo/autoBalancer-v2).  
The contract has been deployed and verified on polygon and can be seen [here](https://polygonscan.com/address/0xbb1472776aabdFaB4c26531363C44d407BAA699A)

I am continuing to work on this dapp, and one of the next features will be turning the smart contract into an ERC20 so that users get a receipt token when they deposit into the portfolio.


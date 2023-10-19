---
layout: post
title: "OptyFi Vaults"
permalink: /opty/
---
[Opty.Fi](https://app.opty.fi/) is a DeFi yield aggregation protocol on Ethereum and Polygon. 

## Building a Beefy Adapter
I originally responded to a bounty from the team back in 2021, where I built and tested a smart contract adapter for the Beefy protocol on Polygon. The challenge here was initially to figure out the codebase that Opty had already built, and how the adapter was designed to work. Then I had to adapt it to the specific way that the Beefy protocol had been built, so that the Opty vaults would be able to deposit to Beefy, withdraw, and perform other vital functions.

### Challenges
One of the most challenging things was to test the adapter across all 170+ pools that Beefy was vaulting at that time. This meant adapting the TypeScript unit test in such a way that it could run efficiently on so many pools, and also meant figuring out how to get the adapter to work across a range of edge cases.

The adapter in it's final form can be seen [here](https://github.com/Opty-Fi/defi-adapters/blob/main/contracts/2_matic/beefy.finance/BeefyFinanceAdapter.sol).
The repo where I did most of the development work, and where you can see the unit tests and other details, is [here](https://github.com/RichJamo/beefy-adapter-kit).

## Building logic for switching and compounding
From December 2021 onwards I started working with the core team at OptyFi, where I joined the Optimisation Engine team. Our challenge here was to build, mostly in Python, an engine that could pull in data from protocols and liquidity pools right across ethereum, assess them on a risk/return basis, and then generate signals indicating when each of our active vaults should switch strategies, or when we should compound the rewards those vaults were generating.

My subtasks within the team were to code up the following:
- a way to generate estimates of the strategy switching costs, both fixed and proportional, and combine these with the estimated apr on different strategies to generate a yes/no switching signal for each vault
- a way to execute this strategy switch from a button on an internal dashboard website
- a way to estimate the gain versus cost that would come from compounding the rewards in any of our vaults, and a yes/no signal for whether to compound at any given time
- a way to execute the compound from a button on an internal dashboard website

### Challenges
Working with the Curve-Convex ecosystem was one thing that made these tasks fairly challenging. For any developer who has had to build integrations into this ecosystem will know, there are some idiosyncrasies that make things a bit complicated. Some of the Curve contracts are written in Vyper, and moreover, there is not just one contract standard that the protocol has stuck to over time. This meant trying to adapt our code for a variety of edge cases in calculating costs and rewards, and in executing reward compounding in the cases where there were two or even three rewards tokens linked to a particular strategy.

Initially I tried to solve this by getting an in-depth understanding of how Curve calculate proportional costs in particular, and looking to replicate this logic in python. This proved a long and ultimately frustrating process, with lots of edge cases that were making our code pretty unwieldy. I eventually decided to switch wholesale to a very different approach - simulating the potential switches that a vault could make by running them on a hardhat forked node, and getting the gas costs from the transaction receipts, and the proportional costs from looking at the balance of the vault before and after the transaction.

One of the other major challenges was that our optimisation engine code was mostly written in python, and needed to stay in python, whereas most of the code interfacing between our different front-ends and our solidity contracts was written in javascript. JavaScript certainly is better suited to interacting with smart contracts, as the web3 and ethers libraries in JS are more mature and have better support. Initially I tried to solve this by creating my cost and reward estimation code in python, using web3.py. However, this became increasingly unwieldy as it became clear that web3.py lacked some of the wrapper functions that one could use in ethers.js to interface with smart contracts, particularly when doing batched calls, or using tools like Gnossis Safe for multisig transactions. The other problem that became very apparent was that we were duplicating code and creating potential future problems if the code had to be changed in any way.

The eventual solution that I arrived at, with help from my team mates, was to use a subprocess.run call from python to actually run the JS code, which would generate the codes for a given transaction, pass those codes back to python, and then simulate the transaction in python. This gave us the best of both worlds - we were able to re-use code that had already been written and tested in JavaScript, we wouldn't run into future problems with code mismatch, and we were able to generate the estimates we needed in python for our optimisation engine.

A couple of tools that were particularly helpful in this process were hardhat, which we used to generate a hardhat forked node where we could simulate transactions and work out costs and/or gains. Also useful was tenderly, particularly from a smart contract debugging point of view, to see exactly what was going wrong, for example when coding up the buttons to execute switches and compounds. I used an Alchemy archive node in the background, and a combination of VS Code and Spyder as IDE's.

Most of the code for this work is in private OptyFi repos, so I unfortunately can't share it here.

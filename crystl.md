---
layout: post
title: "Crystl Finance"
permalink: /crystl/
---
[Crystl Finance](https://polygon.crystl.finance/vaultsV3) is an award-winning, multi-chain vaulting platform. I was the lead on a team of 3 solidity developers, working in collaboration with the front-end team, the CEO and product manager. There were three major pieces of work we completed together:
1 - Quickly and efficiently deploying new vaults, accomodating the quirks of new and different yield farms
2 - Building, testing and deploying an improved version of the protocol's vaults
3 - Deploying our contracts to Cronos, a brand new chain, for the first time

## Efficiently deploying vaults
At the height of DeFi summer, there were new yield farms being spun up on an almost daily basis. One of our tasks on the smart contract team was to get new vaults spun up for these new yield farms as quickly and efficiently as possibly - efficiency here meaning gas efficiency as well as human resource time efficiency.  
  
We did this by semi-automating parts of the process - creating config files within our repo, and deployment scripts that pulled the configs directly from those files. We used Truffle to deploy and verify our contracts.  
  
Along the way we sometimes had to tweak our Strategy contracts to accomodate the quirks of different yield farm designs. Most yield farms were forks of two or three basic designs (the MasterChef and StakingRewards designs being the most popular), so we never had to completely overhaul our contracts. However, there were things like dual reward farms, or Quickswap introducing the dQuick token, which required us to quickly tweak our contracts whilst changing as little as possible and being sure not to introduce any security vulnerabilities or other bugs.  
  
One of the things that I was responsible for creating was a set of unit tests to test our contracts after all changes. This was built originally in Javascript, using hardhat and chai. The tests would deploy contracts on to a hardhat forked node (run off an Alchemy archive node), and test deposits, withdrawals and other functionality.  

## Building V3 Vaults
A strategic decision was made to overhaul our vaulting contracts to make possible a raft of innovations, specifically:  
1. Maximizer vaults (or Ultra Farms, as we later called them)  
2. Boosted vaults (vaults that pay additional rewards on top of the compounding yield)  
3. Zap in/out functionality (a UX improvement so that users could deposit in a range of tokens)  

You can learn more about the protocol and the features we built by checking out the following devpost [submission](https://devpost.com/software/crystl-finance). This short youtube video is also great for demonstrating how our Ultra Farms differ from standard yield farm vaulting solutions:
<iframe width="560" height="315" src="https://www.youtube.com/embed/oKEYdlj0jpw" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

And this image shows our standard vault design from our V2 vaults:  
<img src="../images/crystl_vaults-V2 compounding.drawio.png" alt="V2 compounding" width="400"/>  
Whereas this image shows what we were aiming to do with Ultra Farms:  
<img src="../images/crystl_vaults-V3 Maximizer idea.drawio.png" alt="V3 compounding" width="400"/>

For this work, we won a 2nd runner-up prize in the TRON Grand Hackathon 2022, and a runner-up prize in the Polygon BuidlIt Hackathon 2022.  
![crystl winner badge](./images/crystl_winner_badge.png)
### Challenges
The overarching challenge at the start of this process was to decide how much to change our existing code and contract structure, and how much to try and keep it the same. Obviously in smart contract development, it is always comforting from a security point of view to be able to retain as much battle-tested code as possible. However, for the new features we were being asked to build, some big changes were going to be needed. We eventually did decide to go the route of overhauling the majority of our contracts, but still keeping core pieces of logic.

This first image shows a high level overview of the basic contract structure of our V2 vaults:  
<img src="../images/crystl_vaults-V2 high level.drawio.png" alt="V2 high level" width="500"/>   

And this second image shows a high level overview of our V3 vaults, showing some of the key changes:  
<img src="../images/crystl_vaults-V3 high level.drawio.png" alt="V3 high level" width="500"/>  

The first specific challenge was to figure out how to do multiple levels of vaulting, whilst still accurately tracking user shares of each compounded vault. This might seem trivial at first, but when you start to factor in that users could deposit and withdraw at different times, and in unpredictable ways, it becomes quite complex quite quickly. The way we solved this was with a lot of elbow grease, collaboration between myself and one other very smart solidity dev on the team, and going back to first principles in mathematics to figure out how it could be done.

A second challenge was to keep our main vault smart contract from becoming too big to deploy. Once we had everything working, we then had to do further work to reduce the size of our smart contract. We tackled this in a number of different ways - combing through the code to remove redundant variables or code, then extracting certain functions into externally deployed libraries, and finally using the optimiser to eventually bring the code down to the size that we needed it to be.

Another major challenge was finding a way to bring down the cost of deployment per vault. We did this by creating a factory contract, which would in turn deploy the individual instances of the strategy contract for each vault.

Finally, we had to ensure that this new set of smart contracts was secure and ready for production. This was not just a fork of battle-tested smart contract code, there was a lot of novel code, and we first had to satisfy ourselves that it was secure, and then get it through an audit process. 

We built our own suite of unit tests, first in javascript, and then converted them into typescript. This was an area that I spent quite a lot of time on - we used a hardhat forked node (of polygon mainly), running off an Alchemy archive node. This proved to be a really useful way to keep testing our smart contracts right through the development process without having to constantly redeploy them to polygon mainnet.  

We then experimented with Mythril to test the code and finally we submitted it to Hashex for the audit - the preliminary audit report can be seen [here](./documents/Crystl-Vaults_preliminary-audit-report_1648559243967.pdf), and the final audit report can be seen [here](./documents/Crystl-Vaults_audit-report_1651814621605.pdf).

The repo for these vaults is unfortunately private, but on request I would be happy to share portions of the code with interested parties.

## Deploying on Cronos
As DeFi summer started to wane on Polygon, we were looking around for new opportunities. Cronos chain, which was about to be launched by Crypto.com, seemed a promising place to launch a vaulting platform, as the chain was brand new and we were able to get some support from the Cronos leadership team.  
  
We decided to initially launch our V2 vaults on Cronos, as V3 was still under development. However, the process was not without challenges. We probably underestimated how different it would be deploying contracts on a brand new chain. Initially there was only one block explorer, provided by the Cronos team themselves, and it was very buggy. This made it very difficult for us to verify our contracts, which in turn made it very difficult to do any manual testing on them.  
  
In addition, parts of the truffle suite didn't work properly on Cronos. We found that we couldn't actually deploy our contracts using Truffle. This meant that we had to come up with a workaround, and ended up deploying manually via Remix.  
  
It was also not possible to run our unit tests, because there wasn't yet an archive node available through any of the major node providers. Again, we had to come up with a workaround and find manual ways of testing. One of the things we did was test major functionality on Polygon, and then transfer our contracts to Cronos and only test the specific differences there.  
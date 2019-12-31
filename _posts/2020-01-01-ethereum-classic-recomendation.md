---
layout: post
title:  "Ethereum Classic Recomendation to Reconfigure Miner Settings"
date:   2018-01-15 12:00:00 -0600
categories: EthereumClassic
---

As predicted several months ago, the gas usage on the Ethereum Classic network has recently begun to fill blocks at a rate similar to that of Ethereum. 

There have been a few proposals to mitigate this including [ECIP-1047](https://github.com/ethereumclassic/ECIPs/blob/master/_specs/ecip-1047.md). Many supporters of Ethereum Classic have hoped to avoid this because of the problems that it has caused Ethereum. To name a few, ETH full-nodes are nearly impossible to run on a laptop, due to a 2 TB of storage requirement (for a fully validating archive node) and purported [month-long sync times](https://twitter.com/ercwl/status/1159940020331040770). All this has had a severe centralizing affect on ETH, most users and even developers opting not to run full-nodes, instead connecting to trusted servers like Infura.

For Ethereum Classic the problem is much less severe. Until recently a full-node client could easily be run from a laptop using under 200 GB and syncing in just a few days - a key differentiator for Ethereum Classic. This is mostly because less people used the network, as opposed to any technical reason.

However, it was discussed since early in 2019, that if ETC did not take proactive steps, it would likely see its network suffer a similar fate. That day has come, as ETC blocks are now seeing near 90% capacity. Luckily, the sum total of data is still much less than ETH, but if this continues another 6 - 12 months, ETC will have the same permanent problem as ETH. 

Fortunately, the problem can still be mitigated, and without a heavy-handed approach. After much discussion of a wide variety of solutions, it is the opinion of ETC Coop and this author that the following is the best approach overall. Mostly because it is 

 - Easy and fast to implement
 - Light-handed (does not require a hard or even soft fork)
 - Very low-risk from the standpoint of security

The main recomendation is that miners run their nodes using a lower `block-gas-limit` configuration.

#### Geth
```
--targetgaslimit 1000000 --gasprice 1000000000
```

#### Parity
```
--gas-floor-target 1000000 --gas-cap 0 --min-gas-price 1000000000
```

This will not create a fork, but will cause the `block-gas-limit` (currently ~8M), to slowly decrease.

### Background
The `block-gas-limit` determines how many txs can be fit into a block (more accurately how much total computation and storage). It is somewhat analogous to the `block-size` from Bitcoin. When Ethereum was first being architected, it could not be agreed on what the block-gas-limit should be. It uses an _affective_ voting system, where miners choose a preferred `block-gas-limit`, and are permitted to nudge the limit up or down (toward said preference) whenever they solve a block. The result is that the limit drifts into a sort-of _average_ between miner preferences and stays there indefinitely. 

In 2016 the `block-gas-limit` hovered around 3 million. It was only _after_ Ethereum hard forked away from Ethereum Classic, that Ethereum's limit was raised. For a long while ETH sat at 8 million. At some point Parity set this number as its default, and incidentally it became used for ETC as well, even though the ETC network had (and still has) significantly less traffic.

### Current Scenario
Statistics show that just 6 months ago, blocks were only using an average of less than 200,000 gas. Recently however, a Gas Token Miner has been deployed on ETC that constantly fills blocks to their maximum capacity (currently 8 million). Gas Token acts as a futures market for gas prices, but does so by exploiting a loophole in the EVM (discovered during a [security audit](https://github.com/LeastAuthority/ethereum-analyses/blob/master/GasEcon.md#abusing-the-storage-refund-mechanism) from 2015). It fills the blockchain with meaningless storage data, that it later erases for a refund higher than it originally paid (in many cases they are [paying nothing](https://etcblockexplorer.com/tx/0x3932d548454ec6876dcad6e56ae08617eae41e8bbda594b84ad7379a191e6a80)).

Although it may be possible to close some of the loopholes used by gastoken, these solutions are likely to take longer, require a hard fork, and be rightfully more controversial than the proposal here:

### How Miners can Help
We are recommending that miners choose a block-gas-limit between 1 - 2 Million.

 - If you are a miner this is your choice. Our calculations show this will have no significant affect on profits, because [fees currently account for < 0.01% of miner revenue](https://blockscout.com/etc/mainnet/blocks/9498285/transactions). 
 - There is not risk of forking by do so. 
 - If most miners implement the change, it cut blockchain growth by at least 3/4

We are also seeing miners are accepting sub-optimal `gasPrices`. By accepting transactions, miners increase their risk of having their block turn into an uncle, and losing its reward. It is advised they accept `gasPrice`s only greater than `1 GWei` to offset that cost. Doing so may help to reduce bloat as well.



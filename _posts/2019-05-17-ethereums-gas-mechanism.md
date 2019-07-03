---
layout: post
title:  "Ethereum’s Gas Mechanism: Details and Rationale from First Principles"
date:   2019-05-17 10:50:00 -0400
---
Ethereum is essentially Bitcoin with a gas mechanism added in order to enable it to run a turning complete virtual machine.

In that sense, this gas mechanism was Ethereum’s biggest advancement. There. I said it. It is fairly complex because there are multiple different values to think about. These values are especially confusing when you don’t know their rational. This paper outlines the thought process that logically arrived at it’s design (which is exactly as complex as it needed to be, and no more).

There are 4 variables involved in understanding gas:

* gas-cost
* gas-price
* gas-limit
* block-gas-limit

## Design Rational

First let’s understand Bitcoin’s problem and solution, then observe why Ethereum introduces new issues, then see how the gas-based solution solves them.

### Bitcoin

<span style="color:#850303;">Problem</span>: Public blockchains are open networks. Therefore, anyone can DOS attack the whole network by sending millions of transactions at once.

<span style="color:#0c8201;">Solution</span>: To mitigate this lets require the transaction sender to attach a fee to the transaction.

<span style="color:#850303;">Problem</span>: Each block only has limited space. With a fixed fee, a block can still become “full” and there is no more room for more transactions. The rest of the pending transactions will have to wait an arbitrarily long amount of time.

<span style="color:#0c8201;">Solution</span>: What we really want is a market, so the user can offer a competitive fee, and the miner will prioritize by highest payment. Users can attach larger fees if they don’t want to wait in line.

<span style="color:#850303;">Problem</span>: The transaction data unfortunately can vary in size. So even with a fee, someone can still DOS attack the network by sending a couple of _huge_ transactions.

<span style="color:#0c8201;">Solution</span>: To solve this, the miner first looks at the tx inputs and outputs, and subtracts them to find its fee, then it divides by the total length of the tx data to prioritize the transaction based on its _price-per-byte_ of data. An attacker now must pay a price roughly proportional to the amount of computation the miner will be performing, and the miner can verify that fact. There is also a limit on the amount of bytes that can be included in a single block, so the miner is interested mostly in maximizing this price-per-byte unit specifically.

### Ethereum

<span style="color:#850303;">Problem</span>: The scripting language is Turing complete. This means that a transaction script can have jumps and loops. Therefore the _amount of computation_ done on it, is no longer related to the _size of the code_ in bytes. So we can no-longer rely on the _price-per-byte_ metric to be fair payment. An attacker can create a very small piece of code that pays a significant _price-per-byte_ but causes the virtual machine to loop a million times before completing! There is no way for a miner to know this until it actually executes the million loops.

<span style="color:#0c8201;">Solution</span>: Enter – _Gas_. The concept that _as_ the virtual machine executes a program, it tallies how much computation it is doing. Therefore a million loops would “use up” a million times more gas than one loop. Specifically each _operation_ of the virtual machine uses a certain amount of gas whenever it is run: ADD costs 3 gas, MUL costs 5 gas, JUMPI costs 10 gas etc. These values are referred to as _gas-cost_ and are generally static global values defined in a table in the yellow paper.

Instead of specifying a fee amount (as in Bitcoin), the sender specifies a _gas-price_ with his transaction. As the transaction is processed, he will be charged this gas-price for each unit of gas used. This amount goes to the miner. Without both _gas-cost_ and _gas-price_ we can not have a market between miners and senders. We would have the same problem described for bitcoin where a block becomes full and there is nothing to do but wait in line.

And this is about how much people _usually_ know about gas. However we can’t stop there, because there are still issues. let’s see what they are

<span style="color:#850303;">Problem</span>: We also need to have a limit on the _amount of computation done per block._ This is because blocks need to be able to be processed in a timely manner. Bitcoin had solved this by capping the amount of combined bytes of all the transactions in a block (so-called block-size), but this would not be sufficient in a turing complete environment for the same reasons described above.

<span style="color:#0c8201;">Solution</span>: And for those same reasons, we limit the block computation using gas, and defining another value, the “block-gas-limit”. This is a cap on the cumulative gas used by all the transactions of a single block. This value is not tied to a specific transaction, it is a global value associated with the whole network (as an aside, Ethereum’s block-gas-limit is somewhat dynamic as opposed to Bitcoin’s block-size which is hard-coded).

<span style="color:#850303;">Problem</span>: We have unfortunately just created another issue. This is the part that people rarely understand. As the miner assembles transactions into a block, the cumulative gas counter will approach the block-gas-limit. As they pick each transaction to include (prioritized by highest gas-price), they will have less room left before reaching the limit. They don’t actually know however, how much gas the next transaction will use until they process it. The sender of the transaction had no fault in this either. Here no one is to blame, but unusable computation was executed.

<span style="color:#0c8201;">Solution</span>: Another value is defined by the sender, “_gas-limit_” (confusingly referred to as simply “gas” from the RPC interface). This value is a hard cap that the transaction sender is willing to use before it should halt. This protects the sender from spending more on the transaction than he expected, however its main purpose is to give the miner an upper bound on the gas that the transaction will consume _before_ processing it. This way, when miner first prioritizes by gas-price, processes transactions one-by-one skipping those whose gas-limit is less then the remaining available space in the current block.

The miner will also check that the sender has enough Ether to pay for the gas-limit that they specified before processing.

If the transaction does reach the gas-limit, everything in the VM is reverted but the payment is still made from the sender to the miner. This is important because the miner could not have known the transaction would halt, and must be compensated for processing it. The sender however can run the transaction locally beforehand and hopefully (based on the contract) ensure it would execute as desired. He must take special care however: Some contracts calls may halt only depending on a state which may be updated by someone else in the meantime because the function is public.

## Summary

The sender of the transaction creates it, and decides _how much they are willing to pay per unit of computation_ (gas-price). They also specify an upper bound on _how much computation the transaction should do before halting_ (gas-limit) – which limits their possible cost ether cost as the product of the two. They usually can pre-calculate exactly how much computation is needed, and will send “a little extra”. Only gas that is used, gets charged (so the gas-limit only affects the sender as an upper cap).

The miner must assemble blocks within the block-gas-limit. They maximize profits by prioritizing their mempool by highest gas-price, process those transactions one by one, first checking that each one’s gas-limit is less then the remaining available before the block-gas-limit is reached. Also verifying that the sender has sufficient funds for the gas-limit*gas-price that they themselves specified.

All of this combines to create an ethics system where users and miners are able to participate in a market and where promises are made up front, before work has begun. It may seem complicated, but I hope I’ve proven that it’s exactly what is required (and nothing more) to go from Bitcoin Script, to the more versatile Ethereum Virtual Machine.

Ethereum is simply Bitcoin with a gas mechanism added in order to enable it to run a turning complete virtual machine.
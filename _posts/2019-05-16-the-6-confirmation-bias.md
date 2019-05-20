---
layout: post
title:  "The 6-Confirmation Bias"
date:   2019-05-16 10:50:00 -0400
---
The more confirmations, the lower the risk of a transaction getting re-ordered, and specifically, re-ordered such that it produces a different result. In Bitcoin this different result is that you don’t receive the money because the sender balance lacks necessary funds. With smart contracts the effect could be anything really – just a different outcome of the transaction.

I’ve been researching layer 2 solutions to PoW blockchains and I find this need for finality systemic. Lightning networks, state channels, sidechains: they all have issues with finality and they basically all “solve” the problem by defining values for timelocks and block amounts needed before the next stage can move forward. This creates multiple problems of its own (slows things down, may be insecure during outages).

I believe there are more **fundamental** or **natural** ways to approach these issues, and I will try to enumerate some here.

First prerequisite is to convince yourself of this FACT: _An objective chronological ordering of two events that took place in different inertial reference frames is not possible._

This statement is not domain specific. It’s a simple consequence of Einstein’s Relativity. We therefore cannot strictly _solve_ the double spend (bitcoin), or the ordering of state transitions (ethereum). The idea of what came first is impossible to solve, so as engineers often do, Satoshi relaxed the constraints. Instead of proving which of two events happened first, we merely aim to achieve a _consensus_ as to which did.

Ruminate on that for a moment.

As it turns out, the relaxed constraint and the Nakamoto Consensus use to achieve it, has been mostly sufficient for humans to engage in commerce. I don’t care which of 2 transactions came first as long as I can have a definitive answer to that question within a reasonable wait period. The longer I wait, the more confidently I am that the matter is settled.

But let’s not fool ourselves. There is nothing magic about 6 confirmations. There is no inherent finality to this system, and as we engineer constructions atop bitcoin, imposing finality assumptions tend to break their structural integrity.

There is another way to force ordering as needed for these constructions that is much stronger than PoW consensus however that is often overlooked:

hash-pointers

If my transaction data contains a hash-pointer to a previous piece of data, we _know_ which came first (with the only assumption being the cryptographic integrity of the hash). Subjective ordering is then _practically_ impossible and we can reconcile Einstein’s Relativity by observing that we now have a natural constraint; That “information” (in the einsteinian sense) must first travel from the first event’s reference frame to the second’s, in order for the hash to become embedded in the second transaction. This imposes a speed between Tx1 and Tx2, that is precisely long enough such that _all_ inertial reference frames observing the events, universally agree as to which event came first (even absent this hash proof). But enough physics for now. There are real uses that could/should exist for layer 2\. The Bitcoin lightning network can and does use them, Ethereum currently can not.

Why? because Ethereum deals with errors at the VM level, and Bitcoin deals with them locally. Stated differently – in Ethereum script errors are propagated on-chain, whereas on bitcoin, they are not propagated past a bitcoin client node.

Coming soon: How to use these properties, and how to fix Ethereum so it can use them too.
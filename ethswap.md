Trustless ETH <-> ETC Token Bridge
Features
Basic atomic swapping of token
No light clients or header relays needed
Bids and Offers

## Concept

  1. Ethereum contract
     - Player 1
        - locks ETH into an Ethereum contract
        - commits to `player2` as ETH recipient
        - specifies the value of ETC desired for the trade
        - commits to a secret `S` by specifying its hash `S’`
  2. Ethereum Classic contract
     - Player 2
        - locks ETC into an Ethereum Classic contract
        - commits to releasing ETC to `player1` if `S` is provided
     - Player 1
        - provides `S` to release ETC into `player1`'s account
  3. Ethereum contract
     - Any party
        - provides `S` (which is publicly available) to release ETH into `player2`’s account

### Considerations

The design above represents the “Happy Path”, or the sequence of events when each party participates willingly toward an honest exchange. However, because there are 3 separate transactional steps, either player could attempt to cheat by completing one of the transactional steps, and not the others. In this case, we need ways to roll back the agreement safely. 



## Secure MVP

Below is a more secure version of the concept, through the amendment of some "timelocking" features.

  1. Ethereum contract
     - Player 1
        - locks ETH into an Ethereum contract for time `xt`
        - commits to `player2` as ETH recipient
        - specifies the value of ETC desired for the trade
        - commits to a secret `S` by specifying its hash `S’`
  2. Ethereum Classic contract
     - Player 2
        - observes the `now + 2t` is less than the expiration from step 1
        - locks ETC into an Ethereum Classic contract for time `t`
        - commits to releasing ETC to `player1` if `S` is provided
     - Player 1
        - observes `now` is less than expiration from step 2
        - reveals `S` to release ETC into `player1`'s (her own) account
  3. Ethereum contract
     - Any party
        - provides `S` (which is publicly available) to release ETH into `player2`’s account




### Interaction Paths and Their Response

Below is a tree of each attack vector as well as how they are mitigated.

  - Player 1 is dishonest
     -  do a thing
  - Player 2 is dishonest
     -  do a thing

There are of course many different actions an adversarial player can attempt in order to cheat the system.

Any possible attack vector taken by the adversarial player must be futile, the honest player must have recourse. To achieve this, we introduce a few more rules into the contracts, and specify time-dependent functions during which, the honest player can either (a) Cancel the transaction, or (b) Forward it to completion.





### Design Rationale

## Advanced MVP





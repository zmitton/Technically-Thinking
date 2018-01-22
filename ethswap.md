Trustless ETH <-> ETC Token Bridge
Features
Basic atomic swapping of token
No light clients or header relays needed
Bids and Offers

## Concept

  1. Ethereum contract
     - Player 1
        - locks ETH into an Ethereum contract
        - commits to a secret S by specifying its hash S’
        - specifies the value of ETC desired for the trade
     - Player 2
        - commits to fulfilling the trade within time
  2. Ethereum Classic contract
     - Player 2
        - commits to releasing ETC to - Player 2 if S is provided
     - Player 1
        - provides S to release ETC into player 1’s account
  3. Ethereum contract
     - Any party
        - provides S (which is publicly available) to release ETH into player 2’s account

### Considerations
The design above represents the “Happy Path”, or the sequence of events when each party participates willingly toward an honest exchange. However, because there are 3 separate transactional steps, either player could attempt to cheat by completing one of the transactions, and not the others. In this case, we need ways to roll back the agreement safely. 

Below is an outline of some features that solve these issues, and an outline of each path of attack and how it is mitigated.


## All Interaction Paths and Their Response
  - Player 1 is dishonest
     - 
  - Player 2 is dishonest

There are of course many different actions an adversarial player can attempt in order to cheat the system.

Any possible attack vector taken by the adversarial player must be futile, the honest player must have recourse. To achieve this, we introduce a few more rules into the contracts, and specify time-dependent functions during which, the honest player can either (a) Cancel the transaction, or (b) Forward it to completion.

## Secure MVP




### Design Rationale

## Advanced MVP





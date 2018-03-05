## Trustless ETH <-> ETC Token Bridge

### Features
  - No constant monitoring or relaying of headers required
  - Basic atomic swapping of tokens
  - Bids and Asks

### MVP

  1. Player 1: Ethereum contract
     - locks ETH into an Ethereum contract for (at least) time `3t`
     - commits to `player2` as ETH recipient
     - specifies the value of ETC desired for the trade
     - commits to a secret `S` by specifying its hash `S’`
  2. Player 2: EthereumClassic contract
     - observes the `now + 2t` is less than the expiration from step 1
     - locks ETC into an Ethereum Classic contract for time `t`
     - commits to releasing ETC to `player1` if `S` is provided
  3. Player 1: EthereumClassic contract
     - observes `now` is less than expiration from step 2
     - reveals `S` to release ETC into `player1`'s (her own) account
  4. Anyone: Ethereum contract (`player2` expected)
     - provides `S` (which is publicly available) to release ETH into `player2`’s account

### Considerations

The design above represents the “Happy Path”, or the sequence of events when each party participates willingly toward an honest exchange. However, because there are 3 separate transactional steps, either player could attempt to cheat by completing one of the transactional steps, and not the others. In this case, we need ways to roll back the agreement safely.


### Unhappy Paths and Their Responses

Below are hypothetical attack vectors as well as how they are mitigated. Each player is expected to run a client *while* her own funds are locked in an active trade. However others can help by running a server to help ensure integrity.

  * `player1` completes `tx1`. `player2` doesn't respond.
     - `player1` withdraws her ETH after waiting `3t`
  * `player1` completes `tx1`. `player2` completes `tx2`. `player1` doesn't respond
     - `player2` withdraws her ETC after waiting `t`



### Advanced MVP

It should be possible to extend this concept from singular swaps, to ongoing order books on each side. This could approximate a more liquid market.

### Architecture

Besides the contracts, there are many more pieces in order to deliver a fluid UX. A stateless webpage can be constructed. It will use either centralized APIs like (infura and epool), or it could require the user has metamask(ETH) and the ETC version of it. It would be more robust to create the tools for running 2 light clients together in a console app.



### Smart Contracts
```solidity
pragma solidity ^0.4.0;

contract Swap{
    uint constant TIMELOCK = 600; //10 min
    uint expirationOfOffer;
    uint expirationOfFulfillment;
    address offerer = 0x1F4E7Db8514Ec4E99467a8d2ee3a63094a904e7A;
    address filler = 0x393B2103Ee8016F763efe4e1A681ccd4A6E3393B;
    bytes32 hashedSecret;
    
    modifier only(address party){ if(msg.sender == party) _; }
    modifier onlyAfter(uint expiration){ if(now > expiration) _; }
    modifier onlyBefore(uint expiration){ if(now <= expiration) _; }
    modifier onlyWithSecret(bytes32 secret){ if(keccak256(secret) == hashedSecret) _; }

    function offer(bytes32 _hashedSecret) public payable only(offerer){
        if(hashedSecret == 0){ hashedSecret = _hashedSecret; }
        expirationOfOffer = now + 3*TIMELOCK;
    }
    function collectOffer(bytes32 secret) public onlyBefore(expirationOfOffer) onlyWithSecret(secret){
        filler.transfer(this.balance);
    }
    function withdrawOffer() public only(offerer) onlyAfter(expirationOfOffer){
        offerer.transfer(this.balance);
    }

    function fulfill(bytes32 _hashedSecret) public payable only(filler){
        if(hashedSecret == 0){ hashedSecret = _hashedSecret; }
        expirationOfFulfillment = now + 2*TIMELOCK;
    }
    function collectFulfillment(bytes32 secret) public only(offerer) onlyBefore(expirationOfFulfillment) onlyWithSecret(secret){
        offerer.transfer(this.balance);
    }
    function withdrawFulfillment() only(filler) public onlyAfter(expirationOfFulfillment){
        filler.transfer(this.balance);
    }
}
```

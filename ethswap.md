## Trustless ETH <-> ETC Token Bridge

### Features
  - No constant monitoring or relaying of headers required
  - Basic atomic swapping of tokens
  - Bids and Offers

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

???

### Smart Contracts
```
pragma solidity ^0.4.0;

//deployed on ETH
contract Maker{
    uint constant TIMELOCK = 600;
    uint expiration;
    uint bid; //the price willing to pay (non binding)
    address player1;
    address player2;
    bytes32 hashedSecret;
    
    modifier onlyPlayer1(){ if(msg.sender == player1) _; }
    modifier onlyAfterExpiration(){ if(now < expiration) _; }
    modifier onlyBeforeExiration(){ if(now < expiration) _; }
    modifier onlyWithSecret(bytes32 secret){ if(sha3(secret) == hashedSecret) _; }
    
    function offer(bytes32 hashedSecret, uint bid) payable onlyPlayer1{
        expiration = now + 3*TIMELOCK;
    }
    
    function withdrawOffer() onlyPlayer1 onlyAfterExpiration{
        player1.send(this.balance);
    }
    
    function collect(bytes32 secret) onlyBeforeExiration onlyWithSecret(secret){
        player2.send(this.balance);
    }
}

//deployed on ETC
contract Taker{
    uint constant TIMELOCK = 600;
    uint expiration;
    address player2;
    bytes32 hashedSecret;
    
    modifier onlyPlayer2(){ if(msg.sender == player2) _; }
    modifier onlyAfterExpiration(){ if(now < expiration) _; }
    modifier onlyBeforeExiration(){ if(now < expiration) _; }
    modifier onlyWithSecret(bytes32 secret){ if(sha3(secret) == hashedSecret) _; }
    
    function offer(bytes32 hashedSecret) payable onlyPlayer2{
        expiration = now + 2*TIMELOCK;
    }
    
    function withdrawOffer() onlyPlayer2 onlyAfterExpiration{
        player2.send(this.balance);
    }
    
    function collect(bytes32 secret) onlyBeforeExiration onlyWithSecret(secret){
        msg.sender.send(this.balance);
    }
}
```

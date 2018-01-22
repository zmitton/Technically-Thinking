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

Besides the contracts, there are many more pieces in order to deliver a fluid UX. I stateless webpage can be constructed. It will use either centralized APIs like (infura and epool), or it could require the user has metamask(ETH) and the ETC version of it.

Ideally you could upload your key into a program that can take care of the coordination stuff for you. Ok I need to figure out the best UX. If you use hardware, you simply cant delegate control to a program. So you'd have to sign some money over into a new wallet (that the program has full access to). Then the program can take it from there. For stability it might be better to not have it running in a browser page, and instead use a terminal server application. Even in this case, you want a nice interface, so you can serve from a local webpage back to it. At that point its a bit easier to just give them a native desktop application. Then its a bigger undertaking to build. Honestly people can just use shapeshift. In the end its going to be cheaper (less TXs) and they dont have to download a thing. So what is the use case anyway if people have shapeshift? I guess its a bit of an academic thing. In the case where its an academic thing, what do I want out of it? Who will use it and what experience is "good enough" for them. I guess the differentiator is trustlessness, so I should at least maintain that into the final experience layer or I really have nothing but a shitty shape shift... Ok so if its trustless then it shouldnt use infura stuff... I guess I have to just expect the person to install both metatmasks and both geths. Thne they point their metamasks to geth ports and boom! trustless conections to both chains. But then the webpage I build should poll both and pull up metamask whenever its time to sign something... The user has to sit there and sign the diologe as it comes up, or risk losing things. Of course all this software needs to be bulletproof to even come close to the security of using a trusted service like shapeshift. I dont know man. All my ideas turn to shit at the final step somehow... 

Ideal multimillion$ solution? We create a full client thats working as a desktop application that follows both chains. They run together. You send money to a controlled key, and it does its thing. Would be cool if Ethereum classic should actually run ethereum and itself both. Id have it as my default then. 

### TODO
  - provide smart contracts

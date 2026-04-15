# Combinatorial Auction with Value Density Heuristic – Specification

> [!IMPORTANT]
> **Please make sure to review the authoritative assignment requirements at the [assignments wiki](https://github.com/ftsrg-bta/assignments/wiki).**


## Overview

In a [combinatorial auction](https://en.wikipedia.org/wiki/Combinatorial_auction), participants can place bids not only on individual items but also item _bundles._
Determining the winning bids at the end of the auction is a complex ([NP-hard](https://en.wikipedia.org/wiki/NP-hardness)) problem.
In this simplified version we employ a relatively simple heuristic algorithm to determine winners based on _value densities_ (described later).

Formally, we define the combinatorial auction model as follows:

* Given a set of items $M = \lbrace 1, 2, \ldots \rbrace$
* There are a set of bids $B = \lbrace b_1, b_2, \ldots, b_n \rbrace$
  * Where each bid is a pair $b_i = (S_i, v_i)$
    * $S_i \subseteq M$ is the subset of items being bid on (the _bundle_)
    * $v_i$ is the value offered for $S_i$ in the bid

The goal is to find a set of non-overlapping bids that maximizes revenue.


## Greedy Value-Density-Heuristic Algorithm

In this assignment, you must solve the winner determination problem using a slightly enhanced greedy algorithm.
Its steps are the folowing:

1. For each bid $b_i$, determine its **value density:** $d_i = v_i / |S_i|$
2. Iterate through all bids in descending order of value density:
   * If the current bid does not conflict (overlap) with any already allocated items, accept it (mark it as a winning bid) and mark its items as allocated
   * Otherwise, continue to the next bid
3. Return the set of bids marked winning


## Details

There are a couple more aspects to the auction contract you must implement that are specified below.

> [!WARNING]
> Besides the detailed description below, make sure to read the NatSpec documentation comments in [`ICombinatorialAuction.sol`](contracts/ICombinatorialAuction.sol).

> [!NOTE]
> Unless stated otherwise, failing any requirement must result in the transaction **reverting.**

#### 1. Auction Items

In a full implementation, the items being auctioned would likely be represented by ERC-721 tokens (NFTs) or similar.
However, for simplicity, you do not have to integrate NFTs in this assignment.
Instead, we represent auction items by abstract entities that have the following attributes:

* `id`: a unique identifier of the item (`uint256`)
* `description`: an arbitrary textual description of the item meant for humans
* `minBid`: the minimum bid value for the item individually (in a bid for an item bundle, the bid value must be at least the sum of the `minBid`s of the included items or greater)
* `owner`: represents the current owner of this abstract item (an Ethereum account (`address`) – **if currently under auction and not allocated, this must be the contract itself!**)

**The items for the auction are given in the `initialize` function of the contract, as an array of `AuctionItem` structs.**

* There must be at least one item in the auction.
* Initialization must revert if the passed items array includes items with duplicate `id`s.
* The intial `owner` of all items passed to `initialize` must be the set to the contract’s address.


#### 2. Sealed-Bid Auction via Hash Commitments

In this assignment the auction must also be blind: the bids of other participants must not be known to any bidder.
In other words, whenever a participant places a bid, they must be doing so oblivious to what bids others placed on what items.
This can be slightly tricky to achieve in public blockchain environments (such as Ethereum) where the world state is known to everybody.

Fortunately, a design pattern known as **hash commitment** has emerged that solves this problem and is already widely used.
In a nutshell, participants can submit bid **commitments** for a limited time – these _commitments_ are simply `keccak256` hashes of their bid data.
Nobody can tell what items the bid is about or what the bid value is based on its commitment.
(You may be wondering how they pay: they deposit some funds together with their commitment that can exceed the bid amount and are after refunded if needed – this way, the bid amount remains secret in the first phase.)
Then, once the **commitment phase** has ended, participants are required to **reveal** their bids, meaning they send a transaction with the original bid data they used to calculate the hashes in the commitment phase (the hash preimage).
It is indeed a _commitment_ because the smart contract can check whether the hash of the revealed data matches the previously submitted hash value, so bidders are forced to submit the same data they used in the commitment phase, otherwise their bid will not be accepted.
After a predetermined **reveal phase,** winners can be determined.

Concretely, in this assignment:

* **The lengths of the commitment and reveal phases are set in the `initialize` function of the contract, in seconds.**
* **Bid commitments are made using the `commitBid` function,** which takes a `keccak256` hash of the following:
  * The bidder’s address
  * The list of item IDs included in the bid bundle
  * The bid amount
  * A nonce value chosen by the bidder _(this is against brute-force attacks)_
  * You must use `abi.encode` to encode the parameters to `keccak256` in the order stated above.
  * Bids can only be committed in the commitment phase
  * You can only commit to your own bids
  * There must not be duplicate items in the bid bundle.
  * There must not be any ‘unknown’ item in the bid bundle (an item that has need been given to `initialize`).
* **Bids are revealed using the `revealBid` function,** which takes as parameters the same data that was used to create the commitment hash
  * Except the bidder address as that can be determined from `msg.sender`
  * Bids can only be revealed in the reveal phase
  * You can only reveal your own bids
  * Only committed and non-withdrawn bids can be revealed 
  * The bid amount revealed must be covered by the deposit sent in the commitment phase; otherwise, the function reverts
  * The sum of the minimum bids of the items included must be less than or equal to the bid value
* **There can be only one bid per address**

#### 3. Withdrawal of Bids

A participant may decide that they wish to withdraw their bid.
This is only possible in the commitment phase.

**A bidder can call the `withdrawBid` function to withdraw their bid.**

* You can only withdraw your own bid
* Bids can only be withdrawn in the commitment phase
* You can only withdraw committed bids
* You are immediately refunded upon successful withdrawal
* Withdrawn bids do not take part in the rest of the auction
* You cannot place another bid after withdrawing

#### 4. Winner determination

After the reveal phase ends, the winners can be determined.
Winner determination must be performed using the greedy value-density-heuristic algorithm near the top of this document.
After the winners have been determined, the auction is over and the resulting state is that auction items have their owners set to either the contract (if they were not allocated to any bidder) or to the bidder who won them.
After the winners have been determined, losing bidders can refund their bids (see later).

**The winner determination algorithm runs when the `solveWinnerDetermination` function is called.**

* This function may only be called once (by anybody)
* The function implements the greedy value-density-heuristic algorithm
* Committed, but not revealed bids cannot be considered
* Withdrawn bids also cannot be considered
* No eth transfer occurs in this function
* In the formula to determine value densities ($d = v / |S|$), `/` means _integer division_; ie truncating the result toward zero
* When allocating items based on the value density order, if two bids happen to have equal value density, the bid with the greater total value $v$ should be chosen first.
  * If even the total values of these bids are equal, then use the bidder address to break the tie: process the bid with the _lower_ address first. Addresses are guaranteed to be unique since we only accept a single bid from a given address.

#### 5. Refunding losing bids

At winner determination, some bids will be determined as winners.
The remaining bids are considered losing bids (this includes non-revealed bids) and their deposits can be refunded.

**Losing bids can be refunded using the `refundLosingBid` function that takes an address parameter.**

* Curiously, anybody can initiate a refund for anybody’s address
* However, a single address can only be refunded once
* All non-winning bids can be refunded this way, including non-revealed bids
* Withdrawn bids cannot be refunded
* Refunding means transferring all deposited eth back to the bidder
* Losing bids can only be refunded after the auction has ended and winners were determined
* Refunding must revert if there is nothing to refund

#### 6. Additional Operations

**There are several getter-like query functions that can give information about the auction contract’s state:**

* `getAuctionInfo` returns an `AuctionInfo` object
* `getItems` returns the list of items being auctioned so bidders can choose
* `getItem` takes an item ID parameter and returns its data (reverts if it does not exist)
* `getBid` takes an address parameter and returns the bid of that address (reverts if it does not exist)
* `getResult` returns the auction results once the auction has ended (reverts if not ended yet)
* `getWinningBidOfItem` takes an item ID parameter and returns the bid that won that item (reverts if it does not exist)

#### 7. Events

**Some of the functions must emit events:**

* `initialize` → `AuctionStarted`
* `commitBid` → `BidCommitted`
* `revealBid` → `BidRevealed`
* `withdrawBid` → `BidWithdrawn`

#### 8. Technical Details

**For technical reasons, the auction must not be intialized in a constructor but using the `intialize` function.**

#### 9. Other

* **There must be at least one item in each bid’s bundle**


## Assignment Owner

| Year | Owner                                                                          |
|:----:|:------------------------------------------------------------------------------:|
| 2026 | Bertalan Zoltán Péter `<bpeter@edu.bme.hu>` [@bzp99](https://github.com/bzp99) |
| 2025 | Bertalan Zoltán Péter `<bpeter@edu.bme.hu>` [@bzp99](https://github.com/bzp99) |

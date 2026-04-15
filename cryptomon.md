# Cryptómon Game Specification

> [!IMPORTANT]
> Your assignment is only complete if you have all three:
>
> 1. Implementation of the specification below (a skeleton has been prepared for you in [`contracts/`](contracts/)
> 2. Unit tests for the specification below (in [`tests/`](tests/), using Hardhat)
> 3. Documentation of your solution in [`README.md`](README.md)
>
> More information: https://github.com/ftsrg-bta/assignments/wiki


## Overview

Cryptómon is a decentralized blockchain-based game where players collect, trade, and battle unique digital monsters (‘cryptómons’).
Each monster exists as a non-fungible token (NFT) with distinct attributes, abilities, and rarity levels.
Players can participate in battles and tournaments to earn experience points (XP) and rewards for their monsters.


## Technical Requirements

### 1. Monster Ownership
- Each cryptómon must be implemented as a unique non-fungible token (NFT) following the [ERC-721](https://eips.ethereum.org/EIPS/eip-721) standard
- Tokens must represent ownership, attributes, and abilities of each digital creature
- Standard token operations (transfer, approval, etc.) must be supported

### 2. Contract Administration
- The contract must implement the [ERC-173](https://eips.ethereum.org/EIPS/eip-173) standard for ownership
- The designated admin must have exclusive rights to create new monsters
- The ownership shall be transferable as specified in the ERC-173 standard

### 3. Player Challenge System
- Players must be able to challenge others to battles using specific monster IDs
- Challenge validation must include:
  - Verification that the player owns the challenging monster
  - Verification that the opponent monster exists and is owned by another player
- The opponent must explicitly accept challenges before battles commence

### 4. Tournament System
- Implement a single-elimination tournament system where:
  - Players enter their monsters into competition brackets
  - Winners of each round advance to subsequent rounds
  - Tournament continues until only one undefeated monster remains
- Tournament configurations must include:
  - Maximum number of participants
  - Level restrictions for participating monsters
  - Tournament progress tracking

### 5. Battle Mechanics
- Implement a turn-based battle system with the following characteristics:
  - Each monster has a defined set of moves (attack, defend, heal, etc.)
  - Moves affect opponent Health Points (HP) in varying ways
  - Players alternate turns, using one move per turn
  - Battle concludes when one monster's HP reaches zero


## Contract Structure

The main Cryptómon contract must implement the following class structure:

```mermaid
classDiagram

class ERC721 {
  +balanceOf(_owner: address) uint256
  +ownerOf(_tokenId: uint256) address
  +safeTransferFrom(_from: address, _to: address, _tokenId: uint256, data: bytes) payable
  +safeTransferFrom(_from: address, _to: address, _tokenId: uint256) payable
  +transferFrom(_from: address, _to: address, _tokenId: uint256) payable
  +approve(_approved: address, _tokenId: uint256) payable
  +setApprovalForAll(_operator: address, _approved: bool)
  +getApproved(_tokenId: uint256) address
  +isApprovedForAll(_owner: address, _operator: address) bool
}

class ERC173 {
  +owner() address
  +transferOwnership(_newOwner: address)
}

class Cryptomon {
  +createMonster(name) uint
  +getMonster(monsterId) Monster
  +challangeMonster(myMonsterId, opponentMonsterId) Battle
  +acceptChallange(battleId)
  +claimBattleReward(monsterId)
  -gainXP(monsterId, amount)
  -evolve(monsterId)
  +createTournament(maxParticipants, levelLimit) uint
  +joinTournament(tournamentId, monsterId)
  +executeTournamentRound(tournamentId)
  +claimTournamentPrize(tournamentId, monsterId)
  +moves(monsterId)
  +useMove(battleId, moveId)
}

class Monster {
  +name: string
  +level: uint256
  +xp: uint256
  +evolutionStage: uint256
}

class Tournament {
  +id: uint256
  +creator: address
  +maxParticipants: uint256
  +levelLimit: uint256
  +participants: uint256[]
  +active: bool
}

class Battle {
  +id: uint256
  +started: bool
  +finished: bool
  +player1Turn: bool
  +player1MonsterId: uint256
  +player2MonsterId: uint256
  +monster1Hp: uint256
  +monster2Hp: uint256
}

Cryptomon --> Battle
Cryptomon --> Monster
Cryptomon --> Tournament
Cryptomon ..|> ERC721
Cryptomon ..|> ERC173
```


## System Interactions

### Battle Sequence
1. Player1 initiates a challenge by calling `challengeMonster()`
2. Player2 accepts the challenge via `acceptChallenge()`
3. Players alternate turns using `useMove()` until one monster's HP reaches zero
4. Winner calls `claimBattleReward()` to receive XP
5. If sufficient XP is accumulated, monster evolution may occur

```mermaid
sequenceDiagram
    actor Player1
    actor Player2
    participant C as Cryptomon
    

    %% Player1 initiates a challenge
    Player1 ->> C: challengeMonster(myMonsterId, opponentMonsterId)
    activate C
    create participant B as Battle
    C ->> B: Create new Battle struct
    C ->> B: initialize(battleId, player1MonsterId, player2MonsterId)
    B -->> C: battleId
    deactivate C

    %% Player2 accepts the challenge
    Player2 ->> C: acceptChallenge(battleId)
    activate C
    C ->> B: started = true
    deactivate C

    loop Until one monster reaches 0 HP
        Player1 ->> C: useMove(battleId, moveId)
        activate C
        C ->> B: monster2Hp -= 1
        Note over B: Update HPs based on the move\nSwitch turn
        opt monsterHP == 0
            B ->> C: finished = true
            B -->> Player1: ""
        end
        deactivate C

        Player2 ->> C: useMove(battleId, moveId)
        activate C
        C ->> B: monster1Hp -= 1
        Note over B: Update HPs based on the move\nSwitch turn
        opt monsterHP == 0
            B ->> C: finished = true
            B -->> Player2: ""
        end
        deactivate C
    end

    %% Claim rewards after the battle ends
    Player1 ->> C: claimBattleReward(myMonsterId)
    activate C
    C ->> C: gainXP(myMonsterId)
    C ->> C: evolve(myMonsterId) [optional]
    deactivate C

```

### Tournament Sequence
1. Organizer creates tournament with `createTournament()`
2. Players join tournament with `joinTournament()`
3. Organizer executes tournament rounds via `executeTournamentRound()`
4. System automatically pairs participants and creates battles
5. Winners advance to subsequent rounds
6. Final winner claims prize with `claimTournamentPrize()`

```mermaid
sequenceDiagram
    actor Organizer
    actor Player1
    actor Player2
    actor Player3
    actor Player4

    participant C as Cryptomon
    
    participant B as Battle

    %% Organizer creates a new tournament
    Organizer ->> C: createTournament(maxParticipants=4, levelLimit)
    activate C
    create participant T as Tournament
    C ->> T: Create new Tournament struct
    C ->> T: initialize(tournamentId, maxParticipants, levelLimit)
    T -->> C: tournamentId
    deactivate C

    %% Players join the tournament
    Player1 ->> C: joinTournament(tournamentId, monsterId1)
    activate C
    C ->> T: addParticipant(monsterId1)
    deactivate C

    Player2 ->> C: joinTournament(tournamentId, monsterId2)
    activate C
    C ->> T: addParticipant(monsterId2)
    deactivate C

    Player3 ->> C: joinTournament(tournamentId, monsterId3)
    activate C
    C ->> T: addParticipant(monsterId3)
    deactivate C

    Player4 ->> C: joinTournament(tournamentId, monsterId4)
    activate C
    C ->> T: addParticipant(monsterId4)
    deactivate C

    %% Organizer starts the first round
    Organizer ->> C: executeTournamentRound(tournamentId)
    activate C
    C ->> T: retrieveParticipants()
    Note over C: Pair up participants\n(e.g., 1 vs 2, 3 vs 4)

    C ->> C: create Battle structs (battle1, battle2)
    C ->> B: battle1.start()
    C ->> B: battle2.start()
    Note over B: Battle logic runs, winners are determined

    C ->> T: updateTournamentWinners(winner1, winner2)
    deactivate C

    %% Next round if needed
    Organizer ->> C: executeTournamentRound(tournamentId)
    Note over T,B: ... possibly more rounds ...
    opt Only one winner remains
        Note over T: tournament finished = true
    end

    %% Winner claims prize
    Player1 ->> C: claimTournamentPrize(tournamentId, winnerMonsterId)
    activate C
    C -->> Player1: release prize
    deactivate C

```


## Additional Requirements

> [!IMPORTANT]
> Implementers must also create:
> 
> 1. A reward system that distributes appropriate XP and other incentives
> 2. An evolution system that allows monsters to advance in stages based on XP thresholds


## Additional Tasks

* **Test the smart contract** by writing unit tests
* **Document your solution** in the repository-level [`README.md`](../README.md) file.


## Assignment Owner

| Year | Owner                                                                                          |
|:----:|:----------------------------------------------------------------------------------------------:|
| 2025 | Toldi Balázs Ádám `<balazs.toldi@edu.bme.hu>` [@Bazsalanszky](https://github.com/Bazsalanszky) |

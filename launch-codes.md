# L(a)unch Codes – Specification

> [!IMPORTANT]
> **Please make sure to review the authoritative assignment requirements at the [assignments wiki](https://github.com/ftsrg-bta/assignments/wiki).**


## Overview

A high-security facility always houses a shift of two soldiers.  It must also provide regular access to low-security clearance staff (food delivery, cleaning, etc).  All entries and exits are tracked and authorized by a distributed ledger (a tamper-proof electronic lock on the main entrance continuously monitors the ledger and decides whether it should open or close; requests and authorizations are supported by smart cards and electronic terminals). 

1. Entry is requested from the outside and must be authorized by both soldiers on duty. 
   * Each of the two soldiers on duty must provide exactly one authorization.  A soldier who has already authorized a pending requiest must not be permitted to authorize it a second time.
2. Successful entry must be logged inside by the entering party (after the door is closed). 
3. The protocol for exits is the same.
4. Shift changes happen in two phases (first, `soldier1’` replaces `soldier1` in a complete entry-exit cycle, followed by `soldier2’` replacing `soldier2` in the same manner).  In each phase, the incoming soldier (eg `soldier1`) must first enter the facility as a visitor through the normal entry protocol, while no other visitor is present.  Only after the incoming soldier is physically present inside (ie recorded as the current `visitorID`) may initiate a shift change to begin the handover.  The outgoing soldier then acknowledges the transfer.
5. Guard duty is transferred inside by a mutual ‘acknowledgment’ of the two involved soldiers. 
6. There must not be more than three persons in the facility at any time. 
7. Shift change must take place when the facility is empty, and no entry is allowed until it is over.
   * More precisely: the shift change may only be initiated when the facility’s sole visitor is the income replacement soldier (who entered via the normal entry protocol as per rule 4).  No request entry by any other card is permitted while a shift change is in progress.
8. Soldiers cannot enter or exit the facility while on guard duty. 

Design and implement a smart contract supporting the above access management protocol. 


## Additional Details (Java implementation)

> [!NOTE]
> Unless stated otherwise, failing any requirement must result in the transaction **reverting;** ie, throwing a `ChaincodeException`.

* All asset classes have already been prepared for you in the [`hu.bme.mit.ftsrg.chaincode.launchcodes.assets`](src/main/java/hu/bme/mit/ftsrg/chaincode/launchcodes/assets) package
* Similarly, events are prepared in [`hu.bme.mit.ftsrg.chaincode.launchcodes.events`](src/main/java/hu/bme/mit/ftsrg/chaincode/launchcodes/events)
* **Your task is to implement the contract in [`LaunchCodes.java`](src/main/java/hu/bme/mit/ftsrg/chaincode/launchcodes/contract/LaunchCodes.java)** by filling in the method bodies

> [!IMPORTANT]
> You must follow these rules when preparing your assignment:
>
> 1. Every **SUBMIT** transaction must emit either a `CloseDoorEvent` or an `OpenDoorEvent` in case of successful execution, for example:
>  
>     ```java
>     ctx.getStub().setEvent(CloseDoorEvent.class.getName(), serialize(closeDoorEvent));
>     ```
> 2. Every **SUBMIT** transaction must set the relevant entities and their relation to a consistent state (see their documentation).
> 3. You cannot change the declaration of asset or event classes or add/use new classes that would be persisted on the ledger!
> 4. You can declare **additional** utility methods in the contract class (or additional helper classes), but you cannot change the declaration of the existing contract methods (**except** for the type of the `Context` parameter when using a custom implementation)!
> 5. Request ID parameters are always passed in composite key form, as returned by `CreateCompositeKey`.
> 6. Other simpler IDs are passed in simple "business ID" form (that was used during their registration).
> 7. The `AssetBase` interface implementations of asset classes (the `getTypeForCompositeKey` and `getAttributesForCompositeKey` methods) indicate how their composite key is composed. You must conform to this key format every time.
> 8. All timestamps must be derived from the transaction timestamps provided by the Fabric stub (`ctx.getStub().getTxTimestamp()`), and must be formatted as an RFC3339 string of the form `YYYY-MM-DDTHH:mm:ssZ`, where Z denotes UTC.  Valid example: `2025-01-01T12:00:00Z`.
> 9. Every tranasction must validate its string parameters before use.  If a required string parameter (eg `cardID`, `cardHolderName`, etc) is `null`, empty (`""`), or blank (whitespace only), the transaction must throw a `ChaincodeException`.
> 10. Unless explicitly stated otherwise, if a transaction includes an ID parameter and the ID lookup on the ledger yields no results (eg a given `cardID` does not exist), the transaction must throw a `ChaincodeException`.
> 11. When registering a secure facility with two initial soldiers, the two soldiers must not already be assigned to any secure facility (ie the `secureFailityID` field of their `Card`s must be `null`).
> 12. Transactions that accept soldier card ID parameters must read the corresponding `Card` asset from the ledger and verify that (a) it exists, (b) it is a _soldier’s_ card, (c) the secure facility ID associated with it matches the facility being operated on.
> 13. You must use composite keys to store data on the ledger.  Use `getName()` on the given asset’s class to derive the FQCN (fully-quailified class name) to use as the ‘type’; eg `Card.class.getName()` for storing soldier card assets.


## Assignment Owner

| Year | Owner                                                                             |
|:----:|:---------------------------------------------------------------------------------:|
| 2026 | Attila Klenik `<attila.klenik@vik.bme.hu>` [@aklenik](https://github.com/aklenik) |
| 2025 | Attila Klenik `<attila.klenik@vik.bme.hu>` [@aklenik](https://github.com/aklenik) |

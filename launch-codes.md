# L(a)unch Codes

A high-security facility always houses a shift of two soldiers.
It must also provide regular access to low-security clearance staff (food delivery, cleaning, etc.).
All entries and exits are tracked and authorized by a distributed ledger (a tamper-proof electronic lock on the main entrance continuously monitors the ledger and decides whether it should open or close; requests and authorizations are supported by smart cards and electronic terminals). 

1. Entry is requested from the outside and must be authorized by both soldiers on duty. 
2. Successful entry must be logged inside by the entering party (after the door is closed). 
3. The protocol for exits is the same.
4. Shift changes happen in two phases (first, soldier1’ replaces soldier1 in a complete entry-exit cycle, followed by soldier2’ replacing soldier2 in the same manner). 
5. Guard duty is transferred inside by a mutual ‘acknowledgment’ of the two involved soldiers. 
6. There must not be more than three persons in the facility at any time. 
7. Shift change must take place when the facility is empty, and no entry is allowed until it is over. 
8. Soldiers cannot enter or exit the facility while on guard duty. 

Design and implement a smart contract supporting the above access management protocol. 


## Additional Details (Java implementation)

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

## Additional Tasks

* **Test the smart contract** by writing unit tests (using mocks)
* **Document your solution** in the repository-level [`README.md`](../README.md) file.

> [!IMPORTANT]
> Your assignment is only complete if you have all three:
>
> 1. Implementation in [`LaunchCodes.java`](src/main/java/hu/bme/mit/ftsrg/chaincode/launchcodes/contract/LaunchCodes.java)
> 2. Unit tests
> 3. Documentation in [`README.md`](../README.md) file.
>
> More information: https://github.com/ftsrg-bta/assignments/wiki


## Assignment Owner

| Year | Owner                                                                             |
|:----:|:---------------------------------------------------------------------------------:|
| 2025 | Attila Klenik `<attila.klenik@vik.bme.hu>` [@aklenik](https://github.com/aklenik) |

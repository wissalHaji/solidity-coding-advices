# Smart contracts best practices

In Solidity, there are many global variables and constructs available for developers to use. However, there are some limitations you must know about, otherwise, it could harm your contracts in production.

If your contract has bugs or incorrect architecture, which allows an attacker to gain unauthorized control over your contracts, an attacker can steal funds from your contracts, or they can perform unintended operations on contracts, which, in turn, can harm you or your users economically. There have been many attacks in past that have allowed an attacker to gain full control over a contract and steal millions of dollars worth of ether. One such example is the Parity MultiSig wallet hack, in which more than 150,000 ETH was stolen in July 2017.

The Ethereum blockchain is slow in terms of transaction execution. This slowness also creates some problems related to the transaction reordering. Also, the Ethereum blockchain is public, meaning that any transaction data that you passed in is visible to everyone. Hence, you will have to keep those things in mind when you design your contracts.

While writing the smart contracts in Solidity as a developer, you will have to be aware of a few things and constructs. Let's discuss the best practices that you will have to be aware of.

- [Avoiding floating pragma](avoiding-floating-pragma.md)
- [Avoid sharing a secret on-chain](avoid-sharing-secret-onchain.md)
- [Be careful while using loops](be-careful-with-loops.md)
- [Avoid using tx.origin for authorization](avoid-txorigin-for-auth.md)
- [The timestamp can be manipulated by miners](timestamp-can-be-manipulated.md)
- [Carefully making external function calls](careful-with-external-calls.md)
- [Rounding errors with division](rounding-errors-with-division.md)
- [Using assert(), require() and revert() properly](assert-require-revert.md)
- [Gas consumption](gas-consumption.md)

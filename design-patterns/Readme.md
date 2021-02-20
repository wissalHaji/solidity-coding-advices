# Solidity Design Patterns

The Solidity language is a contract-oriented programming language. There are many constructs required for contract interaction. Also, due to the limitations of the language, few data structure-specific design patterns are used.

In this chapter, you will learn about the different design patterns, that are grouped into different categories based on their usage. These categories are as follows:

- Security design patterns
- Creational patterns
- Behavioral patterns
- Economic patterns
- Life cycle patterns

Out of these categories, some of the design patterns that are mostly used in Solidity contracts are listed here:

- The withdrawal pattern to withdraw ether from the contract
- The factory contract pattern to create new contracts
- The state machine pattern to transition a contract from different states
- The tight variable packing to reduce gas consumption when using structs

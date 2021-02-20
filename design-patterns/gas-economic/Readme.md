# Gas economic patterns

Ether has an economic value and is being traded on exchanges. Ether is used as a crypto fuel to execute transactions on the Ethereum blockchain. In Solidity, each function execution consumes gas. The gas consumed is always paid in ether from the transaction initiator to the block miner. Higher gas consumption by a contract would need more ether; similarly, lower gas consumption would incur a lower amount of ether, and thus lowers the execution cost. Hence, it is always preferred to write the contract in such a way that it can consume the least amount of gas possible for the processing of each transaction.

One possible way to reduce gas consumption is to always deploy the contract with an enabled optimization flag. This process optimizes the EVM bytecode, which then consumes less gas.

We will discuss some of the patterns that can be used to reduce the gas consumption of the contract while carrying out certain types of operations.

## String equality comparison pattern

In Solidity, there is no native function to compare the strings. Hence, it is not possible to compare the equality of the two strings. There are ways to do this by checking byte-by-byte data, but it would be a costly operation in terms of gas consumption. Solidity is not ideal for this, so you should think carefully when using this.

To check for string equality, we can generate the keccak256 hash of both strings and compare the hashes with each other. The generated hash of the two same strings will always give the same hash value. If the strings are not equal, their hashes will also differ.

The following code should be used to compare two strings for their equality:

```
function compare(string memory a, string memory b) internal returns (bool) {
    if(bytes(a).length != bytes(b).length) {
        return false;
    } else {
        return keccak256(a) == keccak256(b);
    }
}
```

As you can see, in the compare() function, first, it checks that the length of the strings are equal, then generates the hash, and compares them. If the length of the strings are not equal, it returns false. If the length of both of the strings are equal, then generate the keccak256 hash of both strings and check. The result of the comparison is returned.

### Applicability

The string equality comparison design pattern should be used in the following cases, when:

- You want to compare two different strings for equality.
- The string length is larger than the two characters.
- There could be multiple sizes of strings passed to a function and we want to have the gas-optimized solution.

## Tight variable packing pattern

Tight variable packing should be utilized when using structs in Solidity. The Solidity language allows structs in the contract, that is used to define an abstract data type. When the storage is allotted to a struct type variable in EVM storage, it is allotted in slots. Each storage slot is 32 bytes long. When statically sized data types are used in the struct (for example, uintX, intX, and bytesX), these variables are allotted storage slots starting from 0 index. Storing and reading data from these storage slots consumes gas based on the number of storage slots written or accessed. Hence, when variables are not tightly packed in a struct, it could consume more storage slots, which would result in more gas consumption during each function call.

Consider the following example code in which we have a Record struct consisting of four fields:

```
contract StructPacking {

    struct Record {
        uint param1; // 1st storage slot
        bool valid1; // 2nd storage slot

        uint param2; // 3rd storage slot
        bool valid2; // 4th storage slot
    }

    Record[] records;

    //Params can be passed to function
    function addRecord() public {
        Record memory record = Record(
            1, true,
            2, true);

        records.push(record);
    }
}
```

When the preceding contract is deployed and the addRecord() function is executed, it consumes 122,414 gas for the transaction cost and 101,142 gas for the execution cost. The EVM allotted four storage slots for the record variable to store. Let's understand how the storage slots are allotted for each of the variables present in the Record struct:

- param1: This variable is of the uint type (that is, uint256), using 256 bits, meaning 32 bytes. Hence, it would consume a full storage slot.
- valid1: This variable is of the bool, type, which consumes 1 byte. As the first slot is fully consumed in storing param1, EVM would allocate the second storage slot and store the valid1 variable in it.
- param2: This variable is of the uint type, which would also require 32 bytes of storage. As the valid1 variable does not consume a full storage slot, 31 bytes are still free; however, 32 bytes (256 bit) of data cannot be accommodated in 31 bytes storage. Hence, EVM would allocate a new third storage slot for the param2 variable and store it.
- valid2: This variable is of the bool type, which consumes 1 byte. However, the third storage slot is fully occupied by param2, hence valid2 would be stored in a new fourth storage slot.

In total, four storage slots are allotted to store a Record struct variable.

You can optimize the struct by reorganizing it and consuming fewer storage slots. Solidity does not perform automatic reorganization to tightly pack struct variables. Hence, this has to be done manually.

In the following code, we have reorganized the variables present in the Record struct:

```
contract StructPacking {

    struct Record {
        uint param1; // 1st storage slot
        uint param2; // 2nd storage slot

        bool valid1; // 3rd storage slot
        bool valid2; // 3rd storage slot
    }

    Record[] records;

    //Params can be passed to function
    function addRecord() public {
        Record memory record = Record(
            1, 2, true, true);

        records.push(record);
    }
}
```

The Solidity optimizer does not optimize structs.

When the preceding contract is deployed, an addRecord() function is called; it consumes 102,219 gas as the transaction cost and 80,947 gas for the execution cost. If you compare the gas cost difference from a previous execution, this new code with a tightly packed struct consumes less gas by 20,195 gas-per-function call. This is a considerably high gas saving per function call. If this function is called multiple times with the preceding tightly packed struct, you would be able to save a lot of gas units.

Let's understand how storage slots are allotted by EVM using optimized code:

- param1: This variable is of the uint type (that is, uint256), using 256, bits meaning 32 bytes. Hence, it would consume a full slot.
- param2: This variable is of the uint type, which would also require 32 bytes of storage. There is no empty space left in the previous storage slot. Hence, EVM would allocate a new second storage slot for the param2 variable and store it.
- valid1: This variable is of the bool type, which consumes 1 byte. As the second storage slot is fully consumed in storing param2, EVM would allocate the third slot and store the valid1 variable in it.
- valid2: This variable is of the bool type, which consumes 1 byte. However, the third storage slot is not fully occupied, hence, the valid2 variable will be stored in the third storage slot.

This way, the new optimized code would consume only three storage slots in storing a variable of the Record type struct.

Let's understand when the tight variable packing pattern should be used for a struct.
A bool variable in Solidity takes 1 byte to store its value.

### Applicability

A tight variable packing pattern is only applicable to struct types. It is the developer's responsibility to check the structs used in each contract and ensure that they are tightly packed. Tight packing should be used only when you are certain that a lot of gas could be saved by optimizing the struct. Otherwise, if it is not making any significant difference, you can avoid it.

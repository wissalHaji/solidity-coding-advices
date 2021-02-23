# The timestamp can be manipulated by miners

In the Solidity language, there are two globally available variables: block.timestamp and now. Both are aliases and are used to get the timestamp when the block is added to the blockchain. The timestamp is in Unix epoch time in seconds. This timestamp is set by the miner when solving the Proof-of-Work (PoW) puzzle. The timestamp of the new block must be greater than the timestamp of the previous block. However, the miner can manipulate the timestamp.

If your contracts need random numbers, you should not use block.timestamp or now, as these can be manipulated by miners. Consider the following example:

```
// Bad Practice
function random() public view returns (uint){
    bytes32 hash = keccak256(
        abi.encode(blockhash(block.number), block.timestamp));
    return uint(hash);
}
```

As you can see, the random number generation is using blockhash and block.timestamp. As block.timestamp can be manipulated by miners, they can set it in a certain way so that they can benefit from it. The random number generation from the contract is still not very robust; hence, you should not generate random numbers in contracts.

## The 15-second blocktime rule

According to the yellow paper of Ethereum, the timestamp for the new block must be greater than the previous block; otherwise, the block will be rejected. The timestamp can be manipulated by the miners using the client software they are running to mine the blocks. There are two Ethereum client software used mostly, and these are Geth and Parity clients. Using their software code and algorithms, these two clients maintain the 15-second rule such that the timestamp difference between the two blocks should not be more than 15 seconds. Otherwise, the client software rejects the block.

Hence, if you are using block.timestamp or now in your contract code, you need to ensure that you are not performing any processes that would require less than 15 seconds.

# Ether can be sent forcibly to a contract

While writing a contract, you can define a payable fallback function to accept ether in your contract, as follows:

```
function() external payable {
}
```

If this fallback function is not present in a contract and it does not have the payable modifier for any function, then your contract is not meant to receive ether.

However, there is still a possible way to send ether to a contract that does not accept ether. This is possible via a selfdestruct function call:

- Let's assume there is a contract x that has some ether present in it.
- Also, there is a contract y.
- Contract x calls the selfdestruct(address_Of_ContractY) function.
- This process sends all ether present in contract x to contract y, even if contract y neither has a fallback function nor a payable function.

The code snippet from contract x is as follows:

```
function kill(address _contractY_addr) external onlyOwner {
    selfdestruct(_contractY_addr);
}
```

The preceding is the code present in contract x; once the kill() function is called by the owner of the contract x, all the ether present in this contract will be sent to contract y. Even If contract y has a payable fallback function defined; in this case of forcible sending of ether, it will not be executed, however, the ether balance of contract y would increase silently.

By using the previous approach, an attacker could affect the behavior of your contract in certain situations: if your contracts are only accepting ether from authorized sources and via either fallback or payable functions, and if your contract code contains some decision logic using address(this).balance (this gives the current ether balance of the contract). Then, an attacker can influence the decision logic, as he would be able to manipulate the contract's ether balance using an unauthorized method.

## Prevention and precaution

At the moment, there is no possible way to prevent forceful ether sending from happening.

However, the developer should caution that you should not assume any specific amount of ether in the contract and write the contract logic. If you do so, an attacker can attack and might lock your contract by sending some ether to your contract forcefully. For example, you should not use address(this).balance in your code to check the contract ether balance and compare it with any specific value.

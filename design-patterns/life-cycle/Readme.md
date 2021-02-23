# Life cycle patterns

A Solidity contract is created either by deploying a new contract or by creating from within a contract. Apart from these, a contract and its functions can follow a different life cycle. A Solidity contract can also be destroyed by calling the selfdestruct function. Once a contract is destroyed, it cannot be recreated on the same address.

Similarly, using the timestamp's global variables and time units, contract states can also be moved. It is also possible to allow the contract functions to be called based on these timestamps. Let's look into the life cycle design patterns.

## Mortal pattern

The mortal pattern allows a contract to be destroyed from the Ethereum blockchain. As you know, a Solidity contract can be destroyed using a selfdestruct() function call in the contract. You can have this function call in your contract to allow it to be destroyed once the contract job is over. Once a contract is destroyed, the contract states would not remain on the blockchain, and if there are any ethers present in the contract, it would be sent to an address passed to the selfdestruct() function as argument.

As you can see in the following sample code, there is a kill() function. Only the owner of the contract can call this function:

import "openzeppelin-solidity/contracts/ownership/Ownable.sol";

```
contract Mortal is Ownable {

    //...
    //Contract code here
    //...

    function kill() external onlyOwner {
        selfdestruct(owner());
    }
}
```

Once the owner of the contract calls the kill() function, the contract is destroyed from the Ethereum blockchain and any ether present in the contract is sent to the owner.
The mortal pattern is very dangerous when used in the contract. You should not use this pattern in the first place.

Let's understand when and where the mortal pattern should be used.

### Applicability

The mortal pattern should be used in the following cases, when:

- You do not want a contract to be present on the blockchain once its job is finished
- You want the ether held on the contract to be sent to the owner and the contract is not required further
- You do not need the contract state data after the contract reaches a specific state

## Auto deprecate pattern

The auto deprecate pattern allows time-based access to certain function calls. In Solidity, using the block.timestamp and now calls, you can get the time of the block when it is mined. Using this time, you can allow or restrict certain function calls in Solidity. There are also some globally available time units present in the Solidity language that you can use. These time units are seconds, minutes, hours, days, weeks, and years. You can use these time units to calculate the Unix epoch time in the past or in the future.

In the following code, the contribution() function should only be called in a specified duration of time:

```
contract AutoDeprecate {

    uint startTime;
    uint endTime;

    modifier whenOpen() {
        require(now > startTime && now <= endTime);
        _;
    }

    constructor(uint _startTime, uint _endTime) public {
        require(_startTime > now);
        require(_endTime > _startTime);
        require(_endTime > _startTime + 1 weeks);

        startTime = _startTime;
        endTime = _endTime;
    }

    function contribute() public payable whenOpen {
        //Contribution code here
    }

    function isContributionOpen() public view returns(bool) {
        return now > startTime && now <= endTime;
    }
}
```

The contribution() function uses the whenOpen modifier. The whenOpen modifier only allows the function call between the startTime and endTime variables. Both of these times are set from the constructor of the contract and the duration between these two times must be equal to or greater than a week.

Let's look at when and where auto deprecate patterns should be used.

### Applicability

The auto deprecate design pattern should be used in the following cases, when:

- You want to allow or restrict a function call before or after a specified time
- You want to allow or restrict a function call for a specified duration of time
- Auto-expire a contract, which would not allow any function calls after the expiry time

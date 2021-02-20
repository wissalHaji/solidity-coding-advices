# Security patterns

There are a few security patterns that ensure the security of funds in a contract. Using these patterns, you can improve the security of the contract. Using these patterns, you could eradicate common security issues in the contract. In the year 2018, there were more than $1 billion smart contract-related hacks. The patterns that we are discussing are built to avoid developers mistakes while developing smart contracts.

Let's look at some of the security design patterns.

## Withdrawal pattern

The withdrawal pattern is also known as a pull-over-push pattern. In this pattern, ether or token transfer (push) from the contract is avoided; rather, the user is allowed to pull ether or token from the contract.

There can be many contracts in which you want to send ether or token to multiple addresses or to a group of addresses. Sending ether from a contract via iterative or non-iterative methods is always going to cause problems, so this should be avoided. Consider the following DividendContract sample code of the contract. Using this contract, you can distribute the dividend to your investors:

```
contract DividendContract is Ownable {

    address[] public investors;

    function registerInvestor(address _investor) public onlyOwner {
        require(_investor != address(0));
        investors.push(_investor);
    }

    //Bad Practice
    function distributeDividend() public onlyOwner {
        for(uint i = 0; i < investors.length; i++) {
            uint amount = calculateDividend(investors[i]);
            investors[i].transfer(amount); //Push ether to user
        }
    }

    function calculateDividend(address _investor) internal returns(uint) {
        //Dividend calculation here
    }
}
```

The contract maintains the list of investors in the investors array. Only the owner of the contract can add investors who are eligible to receive the dividend. The ether present in the contract will be distributed as dividends. As you can see in the distributeDividend() function, there is an unbounded loop and it will iterate the number of times equals the investors array length. There could be multiple issues in the preceding code, such as the following:

- In the future, there could be an out-of-gas exception due to this loop as the amount of gas consumption would increase linearly according to the increase in the investor list, hence it's dangerous to do this iteratively. There is no way to remove an investor from an array; hence, once the loop starts consuming more gas units than a block gas limit, the transaction to this function call will always fail. It is dangerous to write loops like this way. This will lead to the locking of ether in the contract, assuming that there is no other way to take funds out of the contract.
- It is possible that the address of an investor is a contract address, which has a continually failing fallback function. This leads to whole transaction failure for the distributeDividend() function call each time. This also leads to the locking of ether in the contract.

To avoid the issues described in the preceding text, the contract should be designed in a way that a user should be able to claim their dividend from the contract. This way, it's the onus of the user to pull the funds. As you can see in the following code, we have the claimDividend() function, that can be called by anyone. However, only the user who has valid balances present in the contract can claim dividends, as shown in the following:

```
//Good Practice - Pull ether
function claimDividend() public {
    uint amount = balances[msg.sender];
    require(amount > 0);
    //Ensure to update balance before transfer
    //to avoid reentrancy attack
    balances[msg.sender] = 0;
    msg.sender.transfer(amount);
}
```

The contract should maintain or update the balances of each user or investor accordingly, so that they can withdraw the balance amount from the contract, as, in the preceding code, the balances mapping must be updated via other functions.

Let's take a look at when the withdrawal pattern should be applied.

### Applicability

We should use the withdrawal pattern or pull-over-push pattern in the following cases, when:

- You want to send ether/token to multiple addresses from the contract.
- You want to avoid the risk associated with transferring ether/token from the contract to the users.
- You want to avoid paying transaction fees as we know, the transaction initiator has to pay the transaction fee to get their transaction included in the blockchain. Hence, you may want to avoid paying transaction fees for transferring ether or token (push transaction) from the contract to your users. Instead, you want your users to pay transaction fees (pull transaction) and get their share of ether/token withdrawn from the contract.

## Access restriction pattern

As the name suggests, the access restriction design pattern restricts access to the functions of the contract based on roles. The Ethereum blockchain is public and all the addresses and transaction data is public to everyone. However, we can define the contract state variables as private; doing this it only restricts reading the state variable from the contract. Still, anyone can follow the transactions and can find the values of the private state variables present in the contract. To allow anyone to call a function present in the contract, it should be defined as public or external. However, when you need restricted access to a function, you should use modifiers to check for the access rights.

In the following example code, we have two address state variables—owner and admin. The access restriction requirements are as follows:

- The owner variable stores the address of the contract owner. Only the owner of the contract is allowed to change the admin address using the changeAdmin() function.
- The admin variables store the address of the contract admin.
- Both owner and admin are authorized to pause the contract via the pause() function:

Let's have a look at the AccessControl contract:

```
contract AccessControl {
    address public owner;
    address public admin;

    bool public paused;
    modifier whenPaused() {
        require(paused);
        _;
    }

    modifier whenNotPaused() {
        require(!paused);
        _;
    }

    modifier onlyOwner() {
        require(msg.sender == owner);
        _;
    }

    modifier onlyAdmin() {
        require(msg.sender == admin);
        _;
    }

    modifier onlyAuthorized() {
        require(msg.sender == owner || msg.sender == admin);
        _;
    }

    function changeAdmin(address _newAdmin) public onlyOwner {
        require(_newAdmin != address(0));
        admin = _newAdmin;
    }

    function pause() public onlyAuthorized whenNotPaused {
        paused = true;
    }
}
```

As you can see in the preceding code, the following is true:

- The changeAdmin() function is only allowed to be executed from the owner address as it is restricted with an onlyOwner modifier. This function is always allowed to be called from owner, as it is not protected with either the whenPaused or whenNotPaused modifiers.
- The pause() function is allowed to be executed by an authorized person as it is restricted with an onlyAuthorized modifier. Both the admin and owner addresses are authorized to call this function. Also, the precondition to call this function is that the state must not have been previously paused, as it is protected with the whenNotPaused modifier.

Let's see when the access restriction pattern should be used.

### Applicability

The access restriction pattern should be used in the following cases, when:

- Some functions should only be allowed to be executed from certain roles
- Similar kinds of roles and access are needed for one or more functions or actions
- You want to improve the security of the contracts from unauthorized function calls

> Note : This pattern is available in the openzeppelin contracts library in AccessControl.sol contract.

## Emergency stop pattern

The emergency stop design pattern allows the contract to pause and stop the function calls that could harm the contract state or funds present in the contract. As we know that Ethereum smart contracts are immutable once deployed on the blockchain, it might be possible that the contract can have bugs after deployment and could be found by an attacker to gain the control over the contract or funds. To handle these emergency situations, this design pattern could be helpful to reduce the damage to the contract.

It is important to note that when an emergency stop is activated on a contract, all its stakeholders must be able to see the current state of the contract. This should be done to ensure that the stakeholders get the correct status of the contract as they trust the owner of the contracts (maybe a company in this case).

The emergency stop or pause should only be in control of the owner or authorized person. Only they are allowed to call these functions. However, pausing or stopping the contract could potentially add trust issues to the contracts. You should avoid using this pattern; however, in cases when centralized control is needed, these functions can be added to improve the security of the funds and contracts. For example, the project called Wrapped BTC (WBTC) has WBTC tokens on Ethereum blockchain. Each WBTC token is pegged to one bitcoin (BTC). This project has used pausing functionality to pause the token transfer. The owner of the contract is allowed to pause the contract in extreme circumstances. You can see its code present on GitHub at https://github.com/WrappedBTC/bitcoin-token-smart-contracts/blob/master/contracts/token/WBTC.sol.

In the following code sample, we have a contract called Deposit. This contract holds the funds deposited to this contract:

```
contract Deposit {

    address public owner;
    bool paused = false;

    modifier onlyOwner() {
        require(msg.sender == owner);
        _;
    }

    modifier whenPaused() {
        require(paused);
        _;
    }

    modifier whenNotPaused() {
        require(! paused);
        _;
    }

    constructor() public {
        owner = msg.sender;
    }

    function pause() public onlyOwner whenNotPaused {
        paused = true;
    }

    function unpause() public onlyOwner whenPaused {
        paused = false;
    }

    function deposit() public payable whenNotPaused {
        //Ether deposit logic here
    }

    function emergencyWithdraw() public onlyOwner whenPaused {
        owner.transfer(address(this).balance);
    }
}
```

As you can see, the owner of the contract is allowed to pause or unpause the contract:

- The owner can call the pause() function to pause the contract, which, in turn, stops the deposit of the funds and allows onlyOwner to call the emergencyWithdraw() function.
- The owner can call the unpause() function to unpause the contract. This puts the contract back to normal and should allow the deposit of funds again.

Let's see when and where the emergency stop pattern should be used.

### Applicability

The emergency stop design pattern should be used in the following cases, when:

- You want your contract to be handled differently in case of any emergency situations
- You want the ability to pause the contract functions in unwanted situations
- In case of any failure, you want to stop contract failure or state corruption

> Note : This pattern is available in the openzeppelin contracts library in Pausable.sol contract.

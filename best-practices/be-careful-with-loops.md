# Be careful while using loops

You can use loops in two ways as bounded or unbounded loops. If you are performing some operations in contract or just calculating some results, you can use either of the loops.

You can have loops for any function in the Solidity language. However, if the loop is updating some state variables of a contract, it should be bounded; otherwise, your contract could get stuck if the loop iteration is hitting the block's gas limit. If a loop is consuming more gas than the block's gas limit, that transaction will not be added to the blockchain; in turn, the transaction would revert. Hence, there is a transaction failure. Always consider having bounded loops in which you are updating contract state variables for each iteration. Try to avoid using loops that change contract state variables in the first place.

You can have unbounded loops for view and pure functions, as these functions are not added into the block; they just read the state from the blockchain when message calls are made to these functions.

However, if these view or pure functions (containing loops) you are using in other public/external functions, it could block your contract operation because the view or pure functions would consume gas when they are being called from non-pure / non-view functions. Let's look at the following code:

```
contract DividendCalculator is Ownable {
    struct Account {
        address payable investor;
        uint dividend;
    }
    Account[] investors;
    mapping(address => Account) investorsMap;

    modifier onlyInvestor() {
        require(msg.sender == investorsMap[msg.sender].investor);
        _;
    }

    function calculateDividend() public onlyOwner {
        //Bad Practice
        for(uint i = 0; i < investors.length; i++) {
            uint dividendAmt = calcDividend(investors[i].investor);
            investors[i].dividend = dividendAmt;
        }
    }

    function withdraw() public onlyInvestor {
        Account memory account = investorsMap[msg.sender];
        uint dividendAmount = account.dividend;
        account.dividend = 0;
        account.investor.transfer(dividendAmount);
    }

    function calcDividend(address investor) internal returns
    (uint){
        //Logic to calculate dividend
    }
}
```

As shown in the preceding code, the owner can calculate the dividend amount for each investor. However, in the calculateDividend() function, there is an unbounded loop, which could start failing the transaction because of insufficient gas, once it starts consuming more gas units than the block gas limit.

To avoid these issues, you must pass in the number of iterations a loop can execute:

```
//Good Practice
function calculateDividend(uint from, uint to) public onlyOwner {
    require(from < investors.length);
    require(to <= investors.length);
    for(uint i = from; i < to; i++) {
        uint dividendAmt = calcDividend(investors[i].investor);
        investors[i].dividend = dividendAmt;
    }
}
```

The preceding code would help the owner of the contract create multiple batches to calculate dividend. This avoids the transaction failure issues related to insufficient gas.

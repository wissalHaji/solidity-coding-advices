# Behavioral patterns

The behavioral design patterns are used when a contract behaves differently according to its current state. In Solidity, we can have a contract in different states based on the values of its variables. These behaviors should be defined in the contract definition. According to the behavior defined in a contract, a contract transitions between different states. Let's look at some of the behavioral design patterns.

## State machine pattern

In a state machine, the initial state and the final state are known for the machine/process. Between these two states, there could be more intermediate states. In each state, different behaviors or functions are allowed on the machine. For example, a vending machine is a state machine, in which it first asks for the user to select the item number, and, once selected, it moves to the collect amount state. Once the amount is paid by the user, it dispenses the item requested by the user and, at last, the process ends.

The state machine pattern allows a contract to transition from different states and enables certain functions to be executed in each state.

A Solidity contract can be designed in such a way that it represents the state in which it transitioned. Based on the state of the contract, you can allow or disallow functions present in the contract. Using the state machine pattern, you can define different states according to your needs and transition the contract from its initial state to its final state. During this transition, the processing of the contract can allow or disallow its functions, so it can successfully transition to its next state.

In the following code, we have a LoanContract example. As you can see, it has an enum LoanState, which represents different states in which a contract can transition. Let's understand the meaning of the different states:

- NONE: No state represented
- INITIATED: When a loan is initiated by borrower
- COLLATERAL_RCVD: When collateral is received from borrower in the contract
- FUNDED: When the loan received the required collateral from borrower and lender funded the loan
- REPAYMENT: When borrower accepted the funding and starts repaying the loan
- FINISHED: When the loan has finished, and borrower has paid off the full loan

The following is the code for the LoanContract contract:

```
contract LoanContract {
    enum LoanState { NONE, INITIATED, COLLATERAL_RCVD, FUNDED,
        REPAYMENT, FINISHED }

    address public borrower;
    address public lender;
    IERC20 token;
    uint collateralAmount;
    uint loanAmount;
    LoanState public currentState;

    modifier onlyBorrower() {
        require(msg.sender == borrower);
        _;
    }

    modifier atState(LoanState loanState) {
        require(currentState == loanState);
        _;
    }

    modifier transitionToState(LoanState nextState) {
        _;
        currentState = nextState;
    }

    constructor(IERC20 _token, uint _collateralAmount, uint _loanAmount)
        public transitionToState(LoanState.INITIATED) {
        borrower = msg.sender;
        token = _token;
        collateralAmount = _collateralAmount;
        loanAmount = _loanAmount;
    }

    function putCollateral()
        public
        onlyBorrower
        atState(LoanState.INITIATED)
        transitionToState(LoanState.COLLATERAL_RCVD)
    {
        require(IERC20(token)
            .transferFrom(borrower, address(this), collateralAmount));
    }
    //Rest of the code
}
```

In the preceding code, we have an atState() modifier, which is used to check the current state of the loan. The transitionToState modifier is used to transition the contract state from one state to another. As you can see in the putCollateral() function, we used the atState(LoanState.INITIATED) modifier. This allows the function call only in the INITIATED state. Once the function call is executed successfully, it transitions from the INITIATED state to the COLLATERAL_RCVD (collateral received) state.

Let's understand when and where the state machine pattern can be applied.

### Applicability

The state machine pattern should be used in the following cases, when:

- A contract needs to transition from different states
- A contract needs to allow different functions and to behave differently during each of the intermediate states

## Iterable map pattern

The iterable map pattern allows you to iterate over the mapping entries.

In Solidity, you can define a mapping. The mapping holds the key-value pairs. However, there is no way to iterate over the mapping entries in Solidity. There are some cases for which you would need to iterate over the mapping entries; for these situations, you can use this pattern.

Note that the iteration over the mapping entries should not cause an out-of-gas exception. To avoid these situations, use the iteration only in the view function. This way, you would execute a function via message calls only, without generating a transaction.

In the following code, we have DepositContract, in which anyone can deposit ether. When ether is deposited, its entry is updated in the balances mapping to track how much ether is deposited. It also adds an entry into the holders address array, which is used to maintain the unique list of addresses that have been deposited in the contract:

```
contract DepositContract is Ownable {

    mapping(address => uint) public balances;
    address[] public holders;

    function deposit() public payable {
        require(msg.value > 0);
        bool exists = balances[msg.sender] != 0;
        if (!exists) {
            holders.push(msg.sender);
        }
        balances[msg.sender] += msg.value;
    }

    function getHoldersCount() public view returns (uint) {
        return holders.length;
    }
}
```

As you can see in the preceding code; in the deposit() function a new depositor address is added in holders array. Using this holders array you can iterate over the balances mapping

Using the holders address array, you can find the count of depositors in the contract by calling the getHoldersCount() function. Similarly, if you need to filter out some data mapping, you can do it by writing a view function.

### Applicability

The iterable map pattern should be used in the following cases, when:

- You need iterable behaviors over the Solidity mappings
- There would be fewer mapping entries that would require iterable behavior
- You would need to filter some data out of the mapping and get the result via a view function

## Indexed map pattern

The indexed map pattern allows you to read an entry from a map using an index. The pattern also allows you to remove an item from a map and an array.

As we have seen in the previous section, in an iterable map pattern, an item is added to a map as well as to an array. Using the iterable map, we can fetch records using an index. However, the iterable map pattern is useful when you just want to keep on adding items in a map and in an array; it does not support the removal of an item. Hence, when you want to remove an item from the map and array, things get a little tricky. Here, in the IndexedMapping library contract, we can maintain the list of items and remove them from the list as well:

```
library IndexedMapping {
    struct Data {
        mapping(address=>bool) valueExists;
        mapping(address=>uint) valueIndex;
        address[] valueList;
    }

    function add(Data storage self, address val) internal returns (bool) {
        if (exists(self, val)) return false;

        self.valueExists[val] = true;
        self.valueIndex[val] = self.valueList.push(val) - 1;
        return true;
    }

    function remove(Data storage self, address val) internal returns
    (bool) {
        uint index;
        address lastVal;

        if (!exists(self, val)) return false;

        index = self.valueIndex[val];
        lastVal = self.valueList[self.valueList.length - 1];

        // replace value with last value
        self.valueList[index] = lastVal;
        self.valueIndex[lastVal] = index;
        self.valueList.length--;

        // remove value
        delete self.valueExists[val];
        delete self.valueIndex[val];

        return true;
    }

    function exists(Data storage self, address val) internal view
    returns (bool) {
        return self.valueExists[val];
    }

    function getValue(Data storage self, uint index) internal view
    returns (address) {
        return self.valueList[index];
    }

    function getValueList(Data storage self) internal view returns
    (address[]) {
        return self.valueList;
    }
}
```

We have defined the IndexedMapping contract as a library so that it can be used in any contract. As you can see, we have a Data struct to maintain the data for this library. In the Data struct, we have the following variables:

- valueExists: This is a map from the address type to the bool type. This map entry tells us whether a particular address exists in the list or not. When an address exists, its bool value will be set to true, otherwise, it will be set to false.
- valueIndex: This is a map from the address type to the uint type. This map entry stores the address to array index mapping.
- valueList: This is an array of addresses. This stores the addresses in the list.

As you can see, there are two functions (add() and remove()) to add and remove an item from the address list. The getValue() function is used to ensure an item is present at the specified index in an array.

Let's see where the indexed map pattern can be applied.

### Applicability

The indexed map pattern can be used in the following cases, when:

- You want indexed access of an element in a single operation (order of 1 O(1) operation), instead of iterating an array of elements. This is only when the iterable map feature is also required.
- You also want indexed access for elements along with support to remove elements from the list.

## Address list pattern

The address list pattern is used to maintain a curated list of addresses by the owner.

In contracts, there are some situations in which you would need a curated list of addresses. For example, you would need a list of whitelisted addresses that are allowed to call a certain function of your contract. Another example is when you want to maintain a list of supported tokens addresses to allow your contracts to interact with these selected tokens only.

In the address list pattern, adding and removing addresses from the list can only be done by the owner of the contract. In the following code, we have an AddressList contract:

```
contract AddressList is Ownable {

    mapping(address => bool) internal map;

    function add(address _address) public onlyOwner {
        map[_address] = true;
    }

    function remove(address _address) public onlyOwner {
        map[_address] = false;
    }

    function isExists(address _address) public view returns (bool) {
        return map[_address];
    }
}
```

As you can see in the code, only the owner can add or remove addresses in the contract by calling the add() or remove() functions. The isExists() function is a view function and is open for anyone to call. Another contract can even call this function to check whether an address is present in this list or not.

Let's understand where the address list pattern should be applied.

### Applicability

The address list pattern should be used in the following cases, when:

- You want to maintain a curated list of addresses.
- You want to maintain the whitelisted address, which is allowed/disallowed to perform a certain task.
- You want to maintain a list of contract addresses that are allowed. For example, an address list of ERC20 token contract addresses.

## Subscription pattern

The subscription pattern is used when you need to provide a periodic subscription fee for any kind of service.

A contract can provide different kinds of features or a premium service. Any subscriber can subscribe for services that your contract provides. To enable this, you can charge a subscription fee for a period of time from the subscriber. To implement this service, you can create a Subscription contract, as we have developed in the following code example:

```
contract Subscription is Ownable {
    using SafeMath for uint;
    //subscriber address => expiry
    mapping(address => uint) public subscribed;
    address[] public subscriptions;
    uint subscriptionFeePerDay = 1 ether;

    modifier whenExpired() {
        require(isSubscriptionExpired(msg.sender));
        _;
    }

    modifier whenNotExpired() {
        require( ! isSubscriptionExpired(msg.sender));
        _;
    }

    constructor(uint _subscriptionFeePerDay) public {
        subscriptionFeePerDay = _subscriptionFeePerDay;
    }

    function isSubscriptionExpired(address _addr) public view returns
    (bool) {
        uint expireTime = subscribed[_addr];
        return expireTime == 0 || now > expireTime;
    }

    function subscribe(uint _days) public payable whenExpired {
        require(_days > 0);
        require(msg.value == subscriptionFeePerDay.mul(_days));
        subscribed[msg.sender] = now.add((_days.mul(1 days)));
        subscriptions.push(msg.sender);
    }

    function withdraw() public onlyOwner {
        owner().transfer(address(this).balance);
    }

    function useService() public whenNotExpired {
        //User allowed to use service
    }
}
```

As you can see, in this Subscription contract, at the time of deployment, we have configured subscription-per-day fees. This fee is charged per day to the subscriber. The subscriber will call the subscribe() function to subscribe for the service for a given number of days. To do this, they have to send the ether along with the function call. After a successful subscription, they will be able to call the useService() function to use the service until it expires. Once the service of a subscriber expires, they can renew their subscription by calling the subscribe() function again. The owner of the contract can withdraw subscription payments from the contract at any time.

Let's discuss when this subscription pattern can be used.

### Applicability

The subscription pattern can be used in the following cases, when:

- You have a periodic paid service-based model
- A user has an existing request and they want to purchase a premium status for their request for a limited period of time
- An existing contract can purchase a premium status for a limited duration

# Creational patterns

Creational patterns are used to create different contracts as and when needed. There are some situations when a new contract creation is required from within a contract. The factory contract pattern is the most-used design pattern for creating new contracts. The creation pattern also helps a contract to only create a predefined contract. Let's look at the factory contract pattern.

## Factory contract pattern

The factory contract pattern is used when you want to create a new child contract from a parent contract. The benefit of this pattern is that the contract definition is predefined in the main contract and the new contract is deployed with this predefined code. This protects the new contract code from an attacker, as the code of the contract is already defined.

In the following example code, we have a LoanMaster contract. This contract has a createLoanRequest() function that can be called by anyone. This further calls the createInstance() function of the LoanFactory contract to deploy a new instance of a Loan contract using new Loan(). Here, the new solidity keyword is used to create and deploy a new instance of the Loan contract and call its constructor with the given arguments. The creator of the Loan contract is assigned as a borrower in the Loan contract, as shown in the following sample code:

```
contract LoanMaster {
    address[] public loans;
    address public loanFactoryAddress;

    function createLoanRequest(address _token, uint _loanAmount) public {
        address loan = LoanFactory(loanFactoryAddress)
            .createInstance(msg.sender, _token, _loanAmount);
        loans.push(loan);
    }
}

contract LoanFactory {
    function createInstance(
        address _borrower,
        address _token,
        uint _loanAmount
    ) public returns (address) {
        return new Loan(_borrower, _token, _loanAmount);
    }
}

contract Loan {
    address token;
    address borrower;
    uint loanAmount;

    constructor(address _borrower, address _token, uint _loanAmount) public
    {
        borrower = _borrower;
        token = _token;
        loanAmount = _loanAmount;
    }

    //Other logic of Loan contract
}
```

The LoanFactory contract is deployed separately from the LoanMaster contract. This way, the LoanFactory contract can be used by any of the contracts to create any number of new Loan contracts. In the preceding example, a Loan contract is used between a single borrower and a single lender.

Let's understand when and where factory contract patterns should be used.

### Applicability

The factory contract pattern should be used in the following cases, when:

- A new contract is required for each request to be processed. For example, in the case of creating a new loan term between two parties, a master contract can create a new child contract called Loan. This new Loan contract has logic to handle contract terms and conditions along with the funds as well.
- You would need to keep the funds separate in a different contract.
- A separate contract logic should be deployed per request and one or more entities are to be tied together using this newly deployed contract.

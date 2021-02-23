# Carefully making external function calls

The Solidity smart contracts do have a size limit for deployment. A contract that consumes less than 8 million gas at the time of deployment can be deployed on the Ethereum blockchain at the moment. In the future, this limit of 8 million gas could be increased. When your contracts are big enough and cannot be deployed in a single block, they need to be further divided into multiple small contracts. You can break a big contract into multiple small contracts and libraries and interconnect these contracts via function calls with the required authorization.

Not only that, if your contracts needed to communicate with some external service, you would need to make external function calls to external contracts. For example, the Oraclize service is a third-party service that provides APIs to fetch data from the internet and lets you use it in the blockchain. The Oraclize service has exposed some external function calls, which your contracts can call and use.

When you need to call external functions, the developer must know under which category the target contract belongs to, such as the following:

- Trusted contracts: A contract that is deployed and managed by you is known as a trusted contract. External function calls to trusted contracts often not create any issues, as they are known.
- Untrusted contracts: A contract that is deployed and managed by another entity, for example, Oraclize contracts. These are known as untrusted contracts. External function calls to these untrusted contracts might have some security implications in future if their contracts are attacked.

However, the definition of an untrusted contract is up to the developers and the project; if they believe that certain external service contracts can be treated as trusted contracts, they can use it. If they are not sure about the contract code or the third-party contract, then they can classify that contract as an untrusted contract.

To give an example, if your contracts require integration with external contracts, such as KyberNetwork's decentralized exchange, you can use them as KyberNetwork contracts are security audited and have been used by many people for a considerable amount of time. You can take KyberNetwork contracts as trusted. If you are unsure about some external contract and the code is not open, then you can treat these contracts as untrusted contracts. However, once again this is your decision when architecting the design of your contracts and integration with external services.

Let's discuss some of the things that should be avoided when making external function calls from a contract.

## Avoid dependency on untrusted external calls

As we have discussed, there are some contracts or services that are managed by an external third party. The risk in calling an external function on these services is very high. Our external function calls are dependent on their contract and code, and hence it might be possible for these external services to inject malicious code; if executed, your contracts would behave unexpectedly. It is always recommended to have fewer untrusted external function calls.

Ensure that enough due diligence is done while choosing an external contract for integration with your contracts.

## Avoid using delegatecall to untrusted contract code

By using the delegatecall function present in the contract, you can refer some code from another contract and execute it on the current contract context. Library functions are delegate-called to the current contract execution.

When the target contract address is untrusted, and/or when you are not sure about the code, you should not make delegatecall to these untrusted contracts. These untrusted contracts can perform malicious operations on your contracts as follows:

```
function _delegate(address _target) internal {
    // Bad Practice
    _target.delegatecall(bytes4(keccak256("externalCall()")));
}
```

As you can see from the preceding code, although the \_delegate function is internal, it still takes the \_target argument. If the \_target contract address is an untrusted contract, it can perform any arbitrary code execution on your contract. If the target contract is killed via selfdestruct, the external call to the function will always fail, and if there is any dependency of your contract on that target contract, your contract would stuck forever.

To avoid such untrusted calls, only use trusted contract addresses to perform the delegatecall operation. Also, do not allow users to pass in the \_target contract address.

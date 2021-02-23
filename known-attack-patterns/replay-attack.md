# Replay attack

This is a type of attack in which an attacker is allowed to recall the function of the contract, allowing them to update the state variables. Using these attacks, an attacker can update the state variables or perform some unintended operations multiple times when they should not be allowed. The signature replay attacks are mostly prone to replay attacks. You should ensure that signatures are handled correctly in contracts. Let's discuss the signature replay attack.

## Signature replay attacks

There are some cases when a user signs some data off-chain and the data is given to some other authorized user who will submit the signed data on the contract. This process allows users to perform transactions even when off-chain and later, the confirmation or trade is updated on-chain. For example, in projects such as 0xProject, where trades are matched off-chain by signing the order data and later on, actual trade is updated on-chain.

Let's look at an example:

```
import "openzeppelin-solidity/contracts/cryptography/ECDSA.sol";

contract ReplayAttack {
    using ECDSA for bytes32;

    //Bad Practice
    function submitRequest(
        address _signer,
        address _target,
        uint _param1,
        uint _param2,
        bytes memory _signature
    )
        public onlyAuthorized
    {

        bytes memory input = abi.encode(_target, _param1, _param2);
        bytes32 inputHash = keccak256(abi.encodePacked(input));
        inputHash = inputHash.toEthSignedMessageHash();
        address recoveredAddress = inputHash.recover(_signature);

        require(recoveredAddress == _signer);

        //Further action on target address
        _target.submitRequest(_param1, _param2);
    }
}
```

In the preceding code, a user signs the data related to the submitRequest() function and sends it to an authorized user off-chain; later, the authorized person submits the signature along with the signed data to the submitRequest() function. The function checks the inputs signed by the user themselves; otherwise, the transaction will fail.

However, the preceding code is prone to signature replay attack because the same signed data can be sent again by the authorized person. Sending this data again, they can perform unintended operations that are expected to be performed only once.

## Preventing a signature replay attack

To prevent a replay attack, you should use the nonce in the signed data. The user should sign the data along with a unique nonce value each time. Also, the nonce should be stored on-chain, to show that the user has previously sent the signature with that nonce:

```
    mapping (address => mapping(uint => bool)) nonceUsedMap;

    function submitRequest(
        address _signer,
        address _target,
        uint _param1,
        uint _param2,
        uint _nonce,
        bytes memory _signature
    )
    public onlyAuthorized {

        bytes memory input = abi.encode(_target, _param1, _param2, _nonce);
        bytes32 inputHash = keccak256(abi.encodePacked(input));
        inputHash = inputHash.toEthSignedMessageHash();
        address recoveredAddress = inputHash.recover(_signature);

        require(recoveredAddress == _signer);
        require(nonceUsedMap[_signer][_nonce] == false);

        nonceUsedMap[_signer][_nonce] = true;

        //Further action on target address
        _target.submitRequest(_param1, _param2);
    }
}
```

As you can see in the preceding code, we have introduced a mapping that takes the address of the signer and the nonce used by the signer to sign the data. The call is executed when the nonce is not used previously. The transaction fails when a nonce is used previously.

There are other prevention methods you can also use to prevent a signature replay attack, as follows:

- If there are multiple functions in your contract that accept the exact same type of signed data, in that case, a user signs the data thinking that function A() will be called by an authorized person. However, an authorized person can also call function B(), as both of the functions needed the same type of parameters to be signed. To prevent these types of attacks, you can include the function name in the signed data. Having this, even an authorized person cannot call incorrect functions. To include a function name in the signature data, you could also use msg.sig (this gives us the first four bytes of the function called).
- You can also maintain the incremented sequence of the nonce on-chain in the contract itself. This ensures that the function execution from an authorized person can only be performed in sequence according to the nonce increment. This is a good solution when you need only the sequential execution of the transactions. This way, you do not need to track the nonce off-chain on the client side. Also, the nonce would not go off-sync as it starts from 0 and keeps increasing by 1 with every signed transaction. This also lets the user know what is the last nonce they have used, all the other nonce numbers after that are still not used or valid. For example, if a user has sent 50 transactions and the last nonce used is 49, they know that from 50 onwards, all the nonce values are still valid and have not been used.

In the upcoming Ethereum hard fork named Istanbul, there would be a new instruction to get the chain ID of the network in the contract. The chain ID is the fixed and unique ID associated with each testnet and mainnet out there. Adding this chain ID to your signature data would ensure that no one can reply to your signatures, which were previously used on testnet and played on mainnet.

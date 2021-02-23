# Front-running attacks

The Ethereum blockchain is slow, and it is a public blockchain. Because it is a public blockchain, all the transaction data is open and can be seen by others. Even when a transaction is in the pending state, its data can be seen by others.

A transaction can be processed slowly or quickly, depending on the quantity of the gas fees each transaction is going to give to the miner to execute and add to the blockchain. A transaction paying higher gas fees are picked up by the miners first and added to the blockchain. On the other hand, if a transaction is paying lower gas fees, they are picked up later when miners are free and not many higher gas fees transactions are in the pending state. In other words, your transaction is chosen to be executed based on the higher gas fees a miner can earn for executing it. When a transaction is not processed due to low gas fees, it will remain in the pool and be in the pending transaction state.

Once a transaction is in the pending state, all the blockchain clients sync up their pending transactions as well; hence, a pending transaction is also known to each and every client node. You can write a program or script that would listen for any transactions initiated from a specific wallet or with specific parameters. One can also find all pending transactions and see its transaction data.

In a front-running attack, an attacker sees transaction A (which is in pending state), and they immediately send transaction B with a higher gas price to get economic benefit.

Let's discuss the front-running attack that is possible with a specific implementation of ERC20's approve() function.

## Example of an ERC20 approve function

In the ERC20 token standard, there is a function called approve(), following is the implementation code for it. You can also refer Chapter 7, ERC20 Token Standard, The approve function, for more details on the working of the code:

```
function approve(address spender, uint tokens)
public returns (bool success)
{
    allowed[msg.sender][spender] = tokens;
    Approval(msg.sender, spender, tokens);
    return true;
}
```

This function is always prone to a front-running attack if not handled correctly. Let's see how a front-running attack happens on the approve() function, with the help of the following diagram, showing a transaction flow step by step:
[!image](tkn-front-running-attack.png)

Front-running attack transaction flow

In the preceding diagram we have the following:

- There are two people named Alice (left) and Bob (right)
- There is an ERC20 token contract (symbol: TKN) on which both Alice and Bob interact and initiate transactions
- Initially, Alice holds 3,000 TKN and Bob holds 0 TKN
- We are assuming that both have sufficient ether present in their wallets, in order to initiate transactions
- The sequence of actions are numbered from 1 to 6 in the preceding diagram

Let's go through each numbered action and transactions happened between both Alice and Bob:

1. Alice initiates a transaction (Tx-1) and calls the approve(Bob, 1000) function on the TKN contract to give approval of 1,000 TKN tokens to Bob. This transaction is executed, confirmed, and Bob is approved for 1,000 TKN.
2. Bob starts listening for events on the blockchain. He keeps listening for an event when any transaction from Alice is initiated. He also gets Alice in confidence that she approved fewer tokens; originally, he wanted to have 1,500 TKN approved. It could also be that Alice and Bob communicated and agreed—before Tx-1 happened—that Bob actually needs 1,500 TKN; however, by mistake, Alice approved only 1,000 TKN.
3. Now, Alice realized the mistake, initiated a transaction (Tx-2), and calls the approve(Bob, 1500) function to approve 1,500 TKN tokens to Bob.
4. Bob immediately got the notification from his event listener that Alice initiated the new Tx-2 transaction (the transaction is still in the pending state) to give him approval for 1,500 TKN. However, before Tx-2 gets confirmed, Bob initiated transaction Tx-3 (to front-run Tx-2), and calls the transferFrom(Alice, Bob, 1000) function to transfer 1,000 TKN from Alice's account to Bob's account as he already has 1,000 TKN approved. Bob initiate this transaction with the higher gas price to get his Tx-3 transaction confirmed and executed before transaction Tx-2. For example, transaction Tx-2 is initiated with a gas price of 21 gwei. Bob will initiate his Tx-3 transaction with a gas price of more than 21 gwei, for example, 40 gwei. We assume that Bob's transaction Tx-3 gets confirmed and executed before Alice's transaction, Tx-2. Hence, transaction Tx-3 would transfer 1,000 TKN from Alice's account to Bob's. After this transaction, Alice has 2,000 TKN and Bob has 1,000 TKN in their respective wallets. Also, Bob's approved balance would be 0 TKN after transaction Tx-3, as he has used the approval and transferred 1,000 TKN tokens.
5. Now, Alice's Tx-2 transaction got confirmed and executed. This transaction gave Bob a fresh approval of 1,500 TKN.
6. As Bob is already listening for events, he gets a notification that transaction Tx-2 is also confirmed and executed. He immediately initiates another transaction, Tx-4, and calls the transferFrom(Alice, Bob, 1500) function. Once this Tx-4 transaction is confirmed, Alice would have 500 TKN left in her wallet, and Bob would have managed to get 2,500 TKN by performing a front-running attack.

As we have seen in the preceding example; originally Alice wanted to approve 1,500 TKN tokens only, however Bob end up getting 2,500 TKN tokens.

We discussed how a front-running attack can be used by an attacker to transfer more tokens than intended. Let's discuss the front-running prevention techniques.

## Preventing an attack on the approve function

To overcome an attack on the approve function, there are different techniques.

For the previously discussed problem of the approve function, one solution is to set the allowance to 0 (zero) before setting it again with a new value:

```
function approve(address spender, uint tokens)
    public returns (bool success)
{
    require((tokens == 0) || (allowed[msg.sender][spender] == 0));
    allowed[msg.sender][spender] = tokens;
    Approval(msg.sender, spender, tokens);
    return true;
}
```

The preceding code would prevent the front-running attack and would enforce that the approver would always set the allowance to 0 (zero) before setting it again with a new non-zero value. However, this technique requires two transactions in case the approver wants to change the allowance.

Here is another technique in which two extra functions are provided by the contract. These functions would allow the approver to increase or decrease the allowance when required. The code for these functions is as follows:

```
function increaseAllowance(address spender, uint256 addedValue) public returns (bool) {
    require(spender != address(0));

    _allowed[msg.sender][spender] =
        _allowed[msg.sender][spender].add(addedValue);
    emit Approval(msg.sender, spender, _allowed[msg.sender][spender]);
    return true;
}

function decreaseAllowance(address spender, uint256 subtractedValue) public returns (bool) {
    require(spender != address(0));

    _allowed[msg.sender][spender] =
        _allowed[msg.sender][spender].sub(subtractedValue);
    emit Approval(msg.sender, spender, _allowed[msg.sender][spender]);
    return true;
}
```

The increaseAllowance() function would allow the approver to increase the allowance by the provided number of tokens. However, the decreaseAllowance() function would allow the approver to decrease the allowance. Using these functions, the front-running attack is prevented. You can also refer Chapter 7, ERC20 Token Standard, Advance functions, for more details on these functions.

## Other front-running attacks

We only discussed the approve() function-specific front-running attack. However, other kinds of front-running attacks can also happen. For example, when a user is registering a unique value, once this is registered, no one is allowed to register it again on the same contract. Like the domain name registration, once it is registered with a user, another person cannot register it again, as the first person has became the owner of that.

An attacker can watch for the transactions on that contract and can send the high gas-price transaction to front run the user's transaction.

To prevent this type of front-running attack, you should use the commit-and-reveal scheme. We discussed this technique in this chapter when we were discussing about not sharing confidential information on chain in the Avoid sharing a secret on-chain section.

This type of attack is mostly dependent upon how you write your contract code. If your contract is vulnerable to front-running attacks and have some ether fund movements linked to it, an attacker could attack your contracts more often to gain more ether.

Anyone is allowed to initiate a transaction with a high gas price specified in the transaction. For this, they can watch the current network's high gas price and send a transaction with a higher gas price than that. For example, the current networks' high gas price is X wei. They can send a transaction with X + Y wei to get their transaction added to the block as soon as possible.

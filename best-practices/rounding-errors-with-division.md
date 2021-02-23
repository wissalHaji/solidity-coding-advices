# Rounding errors with division

In the Solidity language, there is no official support for floating-point numbers. In the future, there will be support for floating-point numbers. Developers have to use unsigned uintX or signed intX integers only for their calculations. Hence, if any division operation is performed, the result of that calculation might have rounding errors. The result is always rounded down to the nearest integer.

Here is an example in the following code:

```
//Bad Practice
uint result = 5 / 2;
```

The preceding code would set value 2 in the result variable, as 2.5 rounded down to the nearest integer.

These rounding errors could cause problems when you are calculating some values that affect tokens, bonuses, or dividends. Hence, a developer must know that the rounding errors would be possible in the calculations and must ensure the best way possible to mitigate these issues:

```
uint bonusTokens = balances[msg.sender] * dividendAmount / totalSupply;
```

As you can see from the previous code, there could be some rounding errors present in the bonusTokens calculation.

To prevent rounding errors to a certain extent, you should do the multiplication (if multiplication exists) of the values first and then only perform the division operation, as this would have less rounding errors. Otherwise, performing calculations on variables that have some rounding errors already could cause more issues in the final result:

```
//Bad Practice
uint intermediateResult = 5 / 2;
uint finalResult = 10 * intermediateResult;

//Good Practice
uint finalResult = (10 * 5) / 2;
```

As you can see, the first finalResult variable will be assigned with a value of 20. However, the second finalResult variable will be assigned with a value of 25 as we performed multiplication before the division operation.

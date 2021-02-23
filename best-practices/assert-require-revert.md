# Using assert(), require(), and revert() properly

In Solidity, three functions are provided by the language to check for the invariant, validations, and to fail the transaction. These functions are as follows:

- assert(): The assert() function should be used when you want to check for invariants in the code. When any invariant is incorrect, the code execution stops, transaction fails, and contract state changes are reverted. This function should only be used for invariant checking. It should not be used for input validation or pre-condition checking.
- require(): The require() function should be used when you want to validate the arguments provided to the function. It is also used to check for the valid conditions and variable values to be in an expected state. If the validation fails, the transaction also fails, and the contract state changes are reverted.
- revert(): The revert() function should be used to simply fail the transaction. Ensure that the revert() function is called under some certain conditions. Once this function is called, the transaction fails, and the contract state changes are reverted. This should be used when you cannot use the require() function.

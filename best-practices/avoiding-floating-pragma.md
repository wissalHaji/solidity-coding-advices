# Avoiding floating pragma

The first line of a Solidity contract always starts with defining the pragma solidity version. This indicates the Solidity compiler version, using which the code should be compiled. The pragma is defined as follows:

```
// Bad Practice
pragma solidity ^0.5.0;
```

The preceding code signifies that the code should be compiled with any Solidity version starting from 0.5.0 to 0.5.x because the version starts with a ^ (caret sign). For example, if the latest compiler version of Solidity available is 0.5.8, then the contract can be compiled with any compiler version between 0.5.0 and 0.5.8.

It is always recommended that pragma should be fixed to the version that you are intending to deploy your contracts with. The contracts should be deployed with the version that they have been tested with. Using floating pragma does not ensure that the contracts will be deployed with the same version.

It is possible that if you used floating pragma while deploying your contracts, then the most recent compiler version will be picked. The most recent version of compiler has higher chances of having bugs in it. Hence, it is better to keep your contract to a fixed compiler version.

Fixed pragma is defined as follows:

```
// Good Practice
pragma solidity 0.5.0;
```

Using the preceding line, you are instructing the compiler to compile the contract only with the 0.5.0 Solidity compiler. The fixed pragma also prevent contracts from not using the very latest version of the compiler. As the latest version of the compiler might have some unknown bugs.

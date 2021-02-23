# Taking Advantage of Code Quality Tools

Code quality is one of the important aspects of writing applications. Good quality code always tends to have fewer bugs and problems when deployed in production. To maintain and improve code quality, there are always some tools specific to the language you are coding with. Similarly, there are some tools for Solidity as well.

In this chapter, we are going to learn about some of the tools that are used while developing Solidity contracts. There are many open source tools available for the Solidity language, and these include code quality tools such as contract graph generators, linters, and code coverage tools. Using these contract graph generators, you can view how the contracts are linked together; using the linters, you can fix possible bugs, errors, and stylistic errors; and using the code coverage tools, you can discover which part of the code is well tested and which part is not covered by the test cases.

## Using the surya tool

The surya tool is an open source command-line utility tool that is used to generate a number of graphical and other reports. The tool works by scanning through the Solidity smart contract files and can generate inheritance and function call graphs. It also generates a function-specific report in a tabular format. Using all of these generated graphs and reports, a developer can understand smart contract architecture and dependencies by doing a manual inspection.

Let's start by installing the surya tool on your machine.

Installing surya

To install the surya utility tool on your machine, run the following command:

`npm install -g surya`

## Understanding Solidity linters

Linters are the utility tools that analyze the given source code and report programming errors, bugs, and stylistic errors. For the Solidity language, there are some linter tools available that a developer can use to improve the quality of their Solidity contracts. These tools report the known pattern of errors or bugs and also check any security flaws that could be checked by the developers to ensure the safety of the contract.

However, these linter tools should be used along with the compiler's reported warnings. Because the compiler itself reports many warnings and informs the developer about the best language guidelines, it also suggests using the improved language syntax to reduce the security bug. You should bear in mind that the compiler warnings are not sufficient to have good quality code. You can also automate and write a script to compile the code and run linters on it after successful compilation.

For the Solidity language, there are two commonly used linter tools available; these are solhint and solium (also known as ethlint).

## The solidity-coverage tool

Code coverage tools are used to determine which part of the code is covered and tested by the different test cases and which part is not tested. These tools offer an insightful view of the code and its related test cases. Once developers write the test cases for their Solidity project, they can use the coverage tools to find their code coverage. The more the code is covered with test cases, the lower the probability that you will find any bugs in the code in the future.

For Solidity, there is an open source tool called solidity-coverage.

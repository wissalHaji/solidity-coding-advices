# Integer overflow and underflow attacks

In Solidity, there are data types that represent fixed-size integers. For example, all uint8 datatype from uint8, uint16, and moving up to uint256, increasing the bit value by 8 each time. Each type has a different limit to store an integer. For example, a variable of type uint8 can store values from 0 to 28-1 (0 to 255).

Similarly, int8 up to int256 (increasing the bit value by 8 each time) are also prone to integer overflow or underflow attacks.

When a value of the variable reaches the upper limit and further increases, it will cause integer overflow and the value goes back to zero. Also, when the value of the variable reaches the lower limit and further decreases, it will cause integer underflow, and the value goes back to a maximum value of the data type.

For example, you have an int8 variable in the contract. The value assigned to it is 255 (the maximum value an int8 variable can hold). Now, you increase the value of this variable just by 1, either using arithmetic operators such as +, +=, or ++. The new value of the variable would be 0 (the lowest value an int8 variable can hold) as it has caused integer overflow.

The values of these variables are increased or decreased using some operators. These operators are as follows:

- Arithmetic operators: +, -, and \*
- Arithmetic and assignment operators: +=, -=, and \*=
- Pre and post, increment, and decrement operators: ++ and --

You need to be cautious when using the preceding operators for arithmetic operations:

```
// Bad Practice
function transfer(address _to, uint256 _value) public {
    balanceOf[msg.sender] -= _value;
    balanceOf[_to] += _value;
}
```

The preceding transfer() function is prone to both integer overflow and integer underflow attacks. Because there is no check present that ensures that the \_value variable can have a valid value that would not cause integer overflow or integer underflow. An attacker can pass a high value for the \_value argument so that the balance of msg.sender is increased to the maximum of uint256; that way, they can perform an integer underflow attack. Similarly, they can decrease the balance of \_to to zero.

One way to avoid integer overflow or underflow attacks in Solidity code is to check for the boundaries of the data type before assigning new values; however, this can be dangerous if any condition is missed. Doing this requires extra care and attention while writing code:

```
// Good Practice, but not the best as more code is required to prevent
// from integer overflow and underflow attacks
function transfer(address _to, uint256 _value) public {
    require(balanceOf[msg.sender] >= _value);

    balanceOf[msg.sender] -= _value;
    balanceOf[_to] += _value;
}
```

Instead, you can use the SafeMath library provided by the OpenZeppelin libraries. This library reverts the transaction when integer overflow or underflow happens:

```
import "openzeppelin-solidity/contracts/math/SafeMath.sol";

contract ERC20 {
    using SafeMath for uint;
    mapping(address => uint) balanceOf;

    // Good Practice
    function transfer(address _to, uint256 _value) public {
        balanceOf[msg.sender] = balanceOf[msg.sender].sub(_value);
        balanceOf[_to] = balanceOf[_to].add(_value);
    }
}
```

In the preceding code, we are using the SafeMath library; when using the sub() or add() library functions, we do not even need to check for other conditions as the transaction would revert automatically when the integer overflow or underflow happens.

Note that the SafeMath library is for the uint256 data type only. For the int256 data type, you can use the SignedSafeMath library present in the OpenZeppelin libraries. You can refer Chapter 9, Deep Dive Into the OpenZeppelin Library, Math-related libraries, for more details on these library files.

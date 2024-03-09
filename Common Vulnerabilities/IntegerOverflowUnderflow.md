<h1 align="center"><b>Integer Overflow/Underflow</b></h1> 

Integer overflow and underflow bugs in Solidity occur when the result of an arithmetic operation exceeds the maximum or minimum value representable by the data type used. These bugs typically arise when developers perform arithmetic operations on unsigned integers (`uint`) without proper bounds checking, leading to unexpected behavior or vulnerabilities in the contract.

>**Note**: In version `^0.8.0` the solidity provides a default checks for Integer Overflow and Underflow.

Here's a more detailed explanation of integer overflow and underflow bugs:

1. **Integer Overflow**:
   - Integer overflow occurs when the result of an arithmetic operation exceeds the maximum representable value of the data type.
   - In Solidity, unsigned integers (`uint`) wrap around upon overflow, meaning that if the result exceeds the maximum value, it "wraps" back to zero.
   - For example, adding 1 to the maximum value of a `uint8` (255) results in 0 due to overflow (`1 + 255 = 256` but it results in `0` due to overflow of `uint8`).

2. **Integer Underflow**:
   - Integer underflow occurs when the result of an arithmetic operation falls below the minimum representable value of the data type.
   - In Solidity, underflow typically results in unexpected behavior, such as producing a large positive value instead of the expected negative value.
   - For example, subtracting 1 from zero in an unsigned integer (`uint8`) results in the maximum value of the data type due to underflow (`0 - 1 = -1` but it results in `0` due to underflow of `uint8`).

3. **Consequences of Integer Overflow/Underflow**:
   - Integer overflow/underflow bugs can lead to various security vulnerabilities, including:
     - Loss of funds or tokens: If arithmetic operations involve account balances or token balances, integer overflow/underflow can result in incorrect balance calculations, leading to loss of funds.
     - Unauthorized access or manipulation: Attackers can exploit integer overflow/underflow vulnerabilities to bypass security checks, gain unauthorized access, or manipulate contract behavior.
     - Denial of service: Integer overflow/underflow bugs can disrupt contract functionality or cause unexpected behavior, leading to denial of service attacks.

4. **Mitigation Strategies**:
   - Use SafeMath library: Implement arithmetic operations using safe math libraries, such as OpenZeppelin's SafeMath, which provide functions to perform arithmetic operations with overflow/underflow checks.
   - Bounds checking: Perform explicit bounds checking before executing arithmetic operations to ensure that the result does not exceed the maximum or minimum representable values.
   - Use larger data types: Consider using larger data types (`uint256`) to minimize the risk of overflow/underflow, especially for operations involving large numbers or cumulative values.

## Examples

Here are examples illustrating integer overflow and underflow bugs in Solidity:

1. **Integer Overflow Example**:

```javascript
pragma solidity ^0.8.0;

contract IntegerOverflow {
    uint8 public maxUint8 = 255;

    function overflow() public view returns (uint8) {
        return maxUint8 + 1; // Integer overflow occurs here
    }
}
```

Explanation:
- In this example, `maxUint8` is initialized to the maximum value representable by a `uint8`, which is 255.
- The `overflow` function attempts to add 1 to `maxUint8`, resulting in an integer overflow because the result (256) exceeds the maximum value representable by `uint8`.
- Due to overflow, the result wraps around to 0, producing unexpected behavior.

2. **Integer Underflow Example**:

```javascript
pragma solidity ^0.8.0;

contract IntegerUnderflow {
    uint8 public minUint8 = 0;

    function underflow() public view returns (uint8) {
        return minUint8 - 1; // Integer underflow occurs here
    }
}
```

Explanation:
- In this example, `minUint8` is initialized to the minimum value representable by a `uint8`, which is 0.
- The `underflow` function attempts to subtract 1 from `minUint8`, resulting in an integer underflow because the result (-1) falls below the minimum value representable by `uint8`.
- Solidity does not have signed integer types, so underflow typically results in unexpected behavior, such as producing a large positive value instead of the expected negative value.

**Summary**:
These examples demonstrate how integer overflow and underflow bugs can lead to unexpected behavior and vulnerabilities in Solidity contracts. It's essential for developers to implement proper bounds checking and use safe arithmetic operations to prevent these types of bugs and ensure the correctness and security of their smart contracts.


## Mitigation Steps

Here are mitigation steps for the contracts provided earlier to address integer overflow and underflow vulnerabilities:

1. **Mitigation for Integer Overflow**:

```javascript
pragma solidity ^0.8.0;

library SafeMath {
    function add(uint256 a, uint256 b) internal pure returns (uint256) {
        uint256 c = a + b;
        require(c >= a, "SafeMath: addition overflow");
        return c;
    }
}

contract IntegerOverflow {
    using SafeMath for uint8;

    uint8 public maxUint8 = 255;

    function overflow() public view returns (uint8) {
        return maxUint8.add(1); // Safe addition with overflow check
    }
}
```

Mitigation Steps:
- Implement safe arithmetic operations using a library like SafeMath.
- Replace arithmetic operations (`+`, `-`, `*`, `/`, etc.) with equivalent functions from the SafeMath library.
- Use the `add` function from SafeMath, which includes an overflow check to ensure that the result does not exceed the maximum value representable by the data type and it returns `1` instead of `0`.

2. **Mitigation for Integer Underflow**:

```javascript
pragma solidity ^0.8.0;

library SafeMath {
    function sub(uint256 a, uint256 b) internal pure returns (uint256) {
        require(b <= a, "SafeMath: subtraction overflow");
        return a - b;
    }
}

contract IntegerUnderflow {
    using SafeMath for uint8;

    uint8 public minUint8 = 0;

    function underflow() public view returns (uint8) {
        return minUint8.sub(1); // Safe subtraction with underflow check
    }
}
```

Mitigation Steps:
- Implement safe arithmetic operations using a library like SafeMath.
- Replace arithmetic operations (`+`, `-`, `*`, `/`, etc.) with equivalent functions from the SafeMath library.
- Use the `sub` function from SafeMath, which includes an underflow check to ensure that the result does not fall below the minimum value representable by the data type and it returns `1` instead of `-1`.
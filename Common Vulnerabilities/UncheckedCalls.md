<h1 align="center"><b>Unchecked External Call</b></h1>

The "Unchecked External Calls" vulnerability refers to situations in Solidity smart contracts where external calls are made to other contracts or external addresses without proper validation of the return values or handling of potential errors. This vulnerability can lead to various security issues, including loss of funds, unexpected behavior, and contract compromise. Let's break down this vulnerability in more detail:

1. **External Calls in Solidity**:
   Solidity allows contracts to interact with other contracts or external addresses using functions like `call`, `delegatecall`, and `send`. These functions are commonly used to invoke functions on other contracts, transfer Ether, or perform other actions outside the contract's scope.

2. **Unchecked Return Values**:
   When making an external call, it's crucial to validate the return values to ensure that the call was successful and the expected behavior occurred. Ignoring or not properly handling the return values can lead to vulnerabilities. For example, if an external call fails (due to out-of-gas exceptions, reverted transactions, or other errors), it may leave the contract in an inconsistent state.

3. **Reentrancy Vulnerabilities**:
   Unchecked external calls are often associated with reentrancy vulnerabilities. If an external call is made before the contract's state changes are finalized, an attacker may exploit this window of opportunity to reenter the contract and manipulate its state. This can lead to scenarios where the contract's funds are drained, unauthorized operations are performed, or contract logic is bypassed.

4. **Examples of Unchecked External Calls**:
   - **Sending Ether**: When using `send` or `transfer` to send Ether to an external address, the contract should handle potential failures by checking the return value (`bool success`) and reverting if the transfer fails.
   - **Calling External Functions**: When calling functions on other contracts using `call` or `delegatecall`, it's essential to validate the return value to ensure that the called function executed as expected. Failure to do so may allow attackers to exploit unexpected behavior or vulnerabilities in the called contract.

5. **Mitigation Strategies**:
   To mitigate the risks associated with unchecked external calls, developers should:
   - Always validate return values and handle errors when making external calls.
   - Follow the Checks-Effects-Interactions pattern to ensure that state changes are finalized before interacting with external contracts.
   - Use the latest Solidity version and follow best practices recommended by the Ethereum community.
   - Consider using higher-level abstractions or security libraries that provide safer methods for interacting with external contracts.

## Example 1 :
The Unchecked External Calls bug in Solidity with a simple example involving two contracts: a token contract and a contract that interacts with the token contract. We'll use the names Alice and Bob to represent two different users interacting with these contracts.

### Scenario Setup :

1. **Alice** is the owner of a token contract. This contract has a function `transfer` that allows transferring tokens to another address.
2. **Bob** is a user who wants to transfer tokens from Alice's contract to his own address.

### The Token Contract :

Alice's token contract:

```javascript
pragma solidity ^0.8.0;

contract AliceToken {
    mapping(address => uint256) public balances;

    constructor(uint256 initialSupply) {
        balances[msg.sender] = initialSupply;
    }

    function transfer(address to, uint256 amount) external {
        require(balances[msg.sender] >= amount, "Insufficient balance");
        balances[msg.sender] -= amount;
        balances[to] += amount;
    }
}
```

### The Contract Interacting with Alice's Token Contract :

Bob's contract, which interacts with Alice's token contract, might look like this:

```javascript
pragma solidity ^0.8.0;

interface IAliceToken {
    function transfer(address to, uint256 amount) external;
}

contract BobContract {
    IAliceToken public aliceToken;

    constructor(address _aliceToken) {
        aliceToken = IAliceToken(_aliceToken);
    }

    function transferTokens(address to, uint256 amount) external {
        // Unchecked external call
        aliceToken.transfer(to, amount);
    }
}
```

### The Unchecked External Call Bug :

In the `transferTokens` function of Bob's contract, there's an unchecked external call to the `transfer` function of Alice's token contract. This means that if the `transfer` function call fails (for example, if Bob does not have enough tokens to transfer), Bob's contract will not be aware of this failure. It will continue executing as if the transfer was successful, which could lead to incorrect state updates or other unexpected behavior.

### Mitigation Example :

To mitigate the risk of the Unchecked External Calls bug, we can modify Bob's contract to check the return value of the external call. However, since Solidity 0.6.0, the `call` function is low-level and does not return a boolean indicating success or failure. Instead, we can use the `transfer` function directly, which reverts on failure, or use the `call` function with a try-catch pattern to handle failures gracefully.

Here's how Bob's contract could be modified to use the `transfer` function directly:

```javascript
pragma solidity ^0.8.0;

interface IAliceToken {
    function transfer(address to, uint256 amount) external;
}

contract BobContract {
    IAliceToken public aliceToken;

    constructor(address _aliceToken) {
        aliceToken = IAliceToken(_aliceToken);
    }

    function transferTokens(address to, uint256 amount) external {
        // Direct call to transfer function with revert
        (bool success, ) = aliceToken.transfer(to, amount);
        require(success, "Transfer failed");

    }
}
```
By adding the revert(`require`) statement the contract can make updates of the fsilure transactions and Bob will aware of that failure.

## Example 2 :

```javascript
    function withdraw(uint256 amount) external nonReentrant {
@>      weth.transferFrom(msg.sender, address(this), amount); // Unchecked External Call
        weth.withdraw(amount);
        Address.sendValue(payable(msg.sender), amount);
    }
```

### Description of the Example 2:

1. **Unsafe `transferFrom` Call**:
   - The contract has a function called `withdraw`, which calls an unsafe `transferFrom` function from an ERC20 token contract.
   - The `transferFrom` function is used to transfer tokens from one address to another, but if insufficient tokens are present, it may return `false` instead of reverting.
   - In this case, if the `transferFrom` function returns `false`, indicating that the transfer failed due to insufficient tokens, the `withdraw` function continues execution without reverting.

2. **Potential Exploitation**:
   - If the `transferFrom` function returns `false`, indicating a failed transfer due to insufficient tokens, the `withdraw` function continues execution.
   - As a result, the caller of the `withdraw` function (e.g., Alice) receives tokens for free, without actually having the required balance to cover the withdrawal.

### The Unchecked External Call Bug :

1. Alice calls the `withdraw` function with a certain amount of tokens (e.g., 100).
2. The `transferFrom` function inside the `withdraw` function fails due to insufficient tokens and returns `false`, but it does not revert the transaction.
3. Despite the failed transfer, the `withdraw` function continues execution and calls `weth.withdraw()`.
4. Alice receives tokens (e.g., WETH) for free, as the contract fails to properly handle the failed transfer and continues execution.

### Mitigation Steps:

1. **Check the Result of `transferFrom` and `transfer`**:
   - Before proceeding with any further actions, the contract should validate the result of the `transferFrom` function call. If it returns `false`, indicating a failed transfer, the contract should revert the transaction to prevent further execution.

```javascript   
    function withdraw(uint256 amount) external nonReentrant {
        // Check the return value of transferFrom
        require(weth.transferFrom(msg.sender, address(this), amount), "TransferFrom failed");

        // Withdraw WETH
        weth.withdraw(amount);

        // Send Ether to the caller
        Address.sendValue(payable(msg.sender), amount);
    }
```
2. **Use SafeERC20 Library**:
   - Alternatively, the contract can make use of the SafeERC20 library, which provides safe versions of the `transfer` and `transferFrom` functions. These safe functions revert the transaction if the transfer fails, ensuring that funds are not transferred erroneously.


## Example 3 :

Let's consider another example involving a decentralized exchange (DEX) contract and two users, Alice and Bob. In this scenario, Alice wants to sell some tokens on the DEX, and Bob wants to exploit a vulnerability in the DEX contract through an unchecked external call.

Here's a simplified version of the contracts:

```javascript
// Decentralized Exchange (DEX) contract
contract DEX {
    mapping(address => uint256) public balances;

    function deposit() public payable {
        // Deposit Ether into the DEX
        balances[msg.sender] += msg.value;
    }

    function withdraw(uint256 amount) public {
        // Withdraw Ether from the DEX
        require(balances[msg.sender] >= amount, "Insufficient balance");
        balances[msg.sender] -= amount;
        msg.sender.transfer(amount);
    }

    function sellTokens(uint256 amount) public {
        // Buy tokens from the DEX
        amount <= balances[msg.sender] 
        balances[msg.sender] -= amount;
       // Vulnerability: No validation of token transfer success
        (bool success, ) = msg.sender.call.value(amount)("");
        require(success, "Transfer failed");
    }
}

// User contract representing Alice
contract Alice {
    DEX public dex;

    constructor(DEX _dex) public {
        dex = _dex;
    }

    function sell(uint256 amount) public {
        // Alice sells tokens on the DEX
        // Assume token transfer logic here
        dex.sellTokens(amount);
    }
}

// User contract representing Bob (attacker)
contract Bob {
    DEX public dex;

    constructor(DEX _dex) public {
        dex = _dex;
    }

    function attack() public payable {
        // Bob initiates an attack on the DEX
        // Bob deposits some Ether into the DEX
        dex.deposit.value(msg.value)();

        // Bob calls Alice's function to trigger token sale
        Alice alice = Alice(address(this));
        alice.sell(msg.value);
    }

    // Function to receive Ether (fallback function)
    receive() external payable {}
}
```

In this example:

- The `DEX` contract represents a decentralized exchange where users can deposit Ether, withdraw Ether, and buy tokens.
- The `Alice` contract represents Alice, who wants to sell tokens on the DEX.
- The `Bob` contract represents Bob, the attacker, who wants to exploit a vulnerability in the DEX contract.

Now, let's walk through the attack scenario:

1. Bob initiates an attack by depositing some Ether into the DEX and then calls Alice's function to trigger a token sale.
2. Inside the `sell` function of the DEX, an external call is made to transfer Ether to Alice as payment for the tokens.
3. Before the `sell` function completes, the fallback function of Bob's `Bob` contract is triggered due to the external call, allowing Bob to reenter the DEX contract.
4. Since the DEX contract does not properly validate the return value of the external call, Bob's `Bob` contract can exploit this vulnerability to perform a reentrancy attack, potentially draining funds from the DEX contract or causing other unexpected behavior.

By this Unchecked External Call all the tokens sold by Alice is transfered to Bob.

### Mitigation Steps:

To mitigate the vulnerability in the DEX contract and prevent reentrancy attacks, we can implement several security measures:

1. **Checks-Effects-Interactions Pattern**:
   - Follow the Checks-Effects-Interactions (CEI) pattern to ensure that state changes are made before any external interactions take place.
   - Ensure that all state changes are finalized and validated before making external calls.

2. **Use of Withdrawal Pattern**:
   - Separate the transfer of funds from the execution of other contract logic to prevent reentrancy attacks.
   - Implement a withdrawal pattern where users can withdraw their funds in a separate transaction after the trade is completed.

3. **Validation of External Call Results**:
   - Validate the return value of external calls to ensure that the intended action was executed successfully.
   - Revert the transaction if the external call fails or if the return value indicates an unexpected outcome.

4. **Limitations on Ether Transfer**:
   - Consider limiting the amount of Ether that can be transferred in a single transaction to reduce the impact of potential attacks.
   - Implement mechanisms such as withdrawal limits or cooldown periods to mitigate the risk of large-scale fund withdrawals.

Here's an updated version of the DEX contract with these mitigations:

```solidity
contract DEX {
    mapping(address => uint256) public balances;

    function deposit() public payable {
        // Deposit Ether into the DEX
        balances[msg.sender] += msg.value;
    }

    function withdraw(uint256 amount) public {
        // Withdraw Ether from the DEX
        require(balances[msg.sender] >= amount, "Insufficient balance");
        balances[msg.sender] -= amount;
        msg.sender.transfer(amount);
    }

    function buyTokens(uint256 amount) public {
        // Buy tokens from the DEX
        // Assume token transfer logic here
        require(amount <= balances[msg.sender], "Insufficient balance");
        balances[msg.sender] -= amount;

        // External call to transfer Ether
        (bool success, ) = msg.sender.call.value(amount)("");
        require(success, "Transfer failed");
    }
}
```

In this updated version:

- The `buyTokens` function now verifies that the user has sufficient balance before deducting the token amount.
- External calls are made after state changes are finalized, following the CEI pattern.
- The return value of the external call is checked, and the transaction is reverted if the transfer fails.

> *Note*: Always the external call should revert when the transactions are failed. Always use a `revert` or `require` or an `if` statements to check whether the transactions are failed and should not use a bool has the return value.

```javascript                
                            (bool success, ) = msg.sender.call.value(amount)("");
                            require(success, "Transfer failed");
```
By implementing these mitigations, we can significantly reduce the risk of Unchecked External Calls issues and Reentrancy attacks and improve the security of the contract.


### Conclusion :

By always checking the return value of external calls or using the direct function call which inherently reverts on failure, developers can prevent the Unchecked External Calls bug in Solidity contracts. This practice is crucial for ensuring the correct behavior and security of contracts that interact with external contracts.
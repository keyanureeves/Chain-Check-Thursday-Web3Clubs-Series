## 1. Reentrancy

### 1.1 What it is
Reentrancy occurs when a contract makes an external call to another untrusted contract before it has finalized internal state changes. The untrusted contract can re-enter the calling function and perform operations in an inconsistent state.
 
### 1.2 Vulnerable Code Example
```solidity
mapping(address => uint256) public balances;

function withdraw() public {
    uint256 amount = balances[msg.sender];
    require(amount > 0);

    (bool sent, ) = msg.sender.call{value: amount}("");
    require(sent, "Failed to send Ether");

    balances[msg.sender] = 0; // State update happens too late
}
```

### 1.3 Fix
Use the Checks-Effects-Interactions pattern:
```solidity
function withdraw() public {
    uint256 amount = balances[msg.sender];
    require(amount > 0);

    balances[msg.sender] = 0; // Update state before external call

    (bool sent, ) = msg.sender.call{value: amount}("");
    require(sent, "Failed to send Ether");
}
```
Alternatively, use `ReentrancyGuard` from OpenZeppelin:
```solidity
contract Vault is ReentrancyGuard {
    function withdraw() public nonReentrant {
        uint256 amount = balances[msg.sender];
        require(amount > 0);

        balances[msg.sender] = 0;
        (bool sent, ) = msg.sender.call{value: amount}("");
        require(sent, "Failed to send Ether");
    }
}
```

### 1.4 Real-World Hack
- [The DAO Hack (2016)](https://blog.chain.link/reentrancy-attacks-and-the-dao-hack/)
- [The Akropolis Hack(2021)](https://slowmist.medium.com/a-brief-analysis-of-the-akropolis-attack-7e979bd23831)

### 1.5 Learn More
- SWC Registry: [Kaysel: Reentrancy](https://www.kayssel.com/post/web3-4/)
- Solidity Docs: [Security Considerations](https://docs.soliditylang.org/en/latest/security-considerations.html#re-entrancy)

## 2. Unprotected `selfdestruct()`

### 2.1 What it is
When `selfdestruct()` is exposed without proper access control, any user can call it and destroy the contract, permanently removing code and balance.

### 2.2 Vulnerable Code Example
```solidity
function destroy() public {
    selfdestruct(payable(msg.sender)); // No access control
}
```

### 2.3 Fix
Add access control using `onlyOwner` or similar:
```solidity
address public owner;

constructor() {
    owner = msg.sender;
}

function destroy() public {
    require(msg.sender == owner, "Not authorized");
    selfdestruct(payable(owner));
}
```

### 2.4 Real-World Hack
- [Rubixi (DynamicPyramid) exploit](https://hackernoon.com/how-to-not-destroy-millions-in-smart-contracts-pt-2-85c4d8edd0cf)

### 2.5 Learn More
- SWC Registry: [SWC-106: Unprotected Selfdestruct](https://swcregistry.io/docs/SWC-106)

  ## 3. Incorrect State Transitions

### 3.1 What it is
Logic bugs that mishandle state updates, rewards, or conditions. Often subtle, leading to unfair behavior or bricking contracts.

### 3.2 Vulnerable Code Example

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

interface IBadERC20 {
    function transferFrom(address from, address to, uint256 amount) external returns (bool);
}

contract BrokenSubscription {
    IBadERC20 public token;
    mapping(address => uint256) public subscriptions;

    constructor(address tokenAddress) {
        token = IBadERC20(tokenAddress);
    }

    function subscribe(uint256 amount) external {
        // Business logic bug: ignoring transfer result
        token.transferFrom(msg.sender, address(this), amount);

        // State is updated even if transfer failed
        subscriptions[msg.sender] += amount;
    }
}
```

This contract updates internal state without confirming if the external token transfer actually succeeded. If the token’s `transferFrom()` silently fails (returns `false` without reverting), the contract still increases the `subscriptions` mapping — leading to a mismatch between payments and state.

### 3.3 Fix

Always check the return value of external calls, especially for tokens that may fail silently:

```solidity
function subscribe(uint256 amount) external {
    require(token.transferFrom(msg.sender, address(this), amount), "Transfer failed");

    subscriptions[msg.sender] += amount;
}
```

This fix ensures that the internal state is only updated if the token transfer succeeds. If the token call fails, the transaction reverts and no state is changed, maintaining consistency and security.


### 3.4 Real-World Hack
- [Cover Protocol exploit – logic flaw in claims](https://rekt.news/cover-rekt/)

### 3.5 Learn More
- [SWC Registry:](https://swcregistry.io/docs/SWC-135/)

## 4. Front-running Vulnerability

### 4.1 What it is
Attackers monitor mempool and front-run profitable transactions (e.g., buying before a user in DEX).

### 4.2 Vulnerable Code Example
```solidity
function placeBid(uint amount) public {
    require(amount > highestBid);
    highestBid = amount;
    winner = msg.sender;
}
```

### 4.3 Fix
Use commit-reveal or minimal slippage constraints:
```solidity
mapping(address => bytes32) public commits;

function commit(bytes32 hash) public {
    commits[msg.sender] = hash;
}

function reveal(uint value, string memory salt) public {
    require(keccak256(abi.encode(value, salt)) == commits[msg.sender], "Invalid reveal");

    // proceed to count bid or vote after revealing
}

```

### 4.4 Learn More
- [SWC-114: Front-running](https://swcregistry.io/docs/SWC-114)


## 5. Incorrect Precision Handling 
### 5.1 Truncation Errors

#### 5.1.1 What it is
Truncation occurs when Solidity performs integer division and silently rounds down any decimal result. Since Solidity does not support floating-point numbers, calculations like percentages, ratios, or shares must be carefully scaled to avoid precision loss or unintended behavior.

A common mistake is using raw integer division without fixed-point scaling, especially in share accounting like:

```solidity
userShare = assetsDeposited / totalAssets;
```
This truncates anything below 1, meaning small depositors may get 0 shares even if they deposited something non-zero.

#### 5.1.2 Vulnerable Code Example
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Vault {
    uint256 public totalAssets;
    uint256 public totalShares;
    mapping(address => uint256) public shares;

    function deposit(uint256 amount) external {
        uint256 share;

        if (totalAssets == 0 || totalShares == 0) {
            // First depositor gets 1:1 shares
            share = amount;
        } else {
            // Vulnerable: truncation occurs here
            share = (amount * totalShares) / totalAssets;
        }

        require(share > 0, "Truncated to zero shares"); // Will fail for small deposits
        shares[msg.sender] += share;

        totalAssets += amount;
        totalShares += share;
    }
}
```

What’s wrong?
If `amount * totalShares < totalAssets`, then the result rounds down to zero.

Small deposits get no shares, and their funds are essentially donated to the pool.

#### 5.1.3 Fix
Use a fixed-point math library like `PRBMath` or `WadRayMath`, or apply consistent scaling using 1e18 to preserve precision.

Safe version using `PRBMath`:
```solidity
import { ud, UD60x18 } from "@prb/math/UD60x18.sol";

function deposit(uint256 amount) external {
    uint256 share;

    if (totalAssets == 0 || totalShares == 0) {
        share = amount;
    } else {
        UD60x18 amt = ud(amount);
        UD60x18 ts = ud(totalShares);
        UD60x18 ta = ud(totalAssets);
        share = amt.mul(ts).div(ta).unwrap(); // High-precision math
    }

    require(share > 0, "No shares minted");
    shares[msg.sender] += share;
    totalAssets += amount;
    totalShares += share;
}
```
### 5.2 Incorrect token decimal handling
#### 5.2.1 What it is
Tokens have varying decimal values (e.g., USDT: 6, ETH: 18). Misunderstanding this causes incorrect math or transfers.

#### 5.2.2 Fix
Normalize based on token decimals:
```solidity
uint8 decimals = IERC20Metadata(usdt).decimals();
uint amount = 10 ** uint(decimals); // 1 USDT
```
e.g
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/extensions/IERC20Metadata.sol";

contract TokenSender {
    function transferToken(address token, address to, uint amountWholeTokens) external {
        uint8 decimals = IERC20Metadata(token).decimals();
        uint scaledAmount = amountWholeTokens * (10 ** uint(decimals));

        require(IERC20(token).transfer(to, scaledAmount), "Transfer failed");
    }
}
```

### 5.5 Learn More
- [PRBMath – Resource Guide](https://github.com/PaulRBerg/prb-math)
- [OpenZeppelin ERC20 Metadata](https://docs.openzeppelin.com/contracts/5.x/api/token/erc20#IERC20Metadata)

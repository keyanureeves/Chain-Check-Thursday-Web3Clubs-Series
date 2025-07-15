# TokenMarket

[//]: # (contest-details-open)

### Details

- nSLOC: 298

## About the Project

- **TokenMarket.sol:**

  The `TokenMarket` contract is a peer-to-peer trading system designed for `ERC20` tokens like `wETH`, `wBTC`, and `wSOL`. Sellers can list tokens at their desired price in `USDC`, and buyers can fill them directly on-chain.

  The flow is simple:

  - Sellers lock their tokens and list an order with a price and deadline
  - Buyers purchase tokens by paying the listed `USDc` amount
  - if the order isn't filled before the deadline, sellers can cancel and retrieve their tokens

  All orders are tracked using a unique `orderId`, and sellers retain full control over their listings until filled or expired.

  Token transfers use `SafeERC20` to ensure secure movement of funds, and the system enforces a strict set of violation rules to prevent misuse.

  The contract also supports:

  - Amending orders (e.g. changing price or amount)
  - Canceling active or expired orders
  - Emergency withdrawals by the owner (for non-core tokens only)
  - human-readable order infor using `getOrderDetailsString`

- **Features:**

  - Fixed-price order creation for selected `ERC20` tokens
  - Secure and gas-efficient architecture
  - Deadline enforcement to prevent stale listings

[//]: # (contest-details-close)
[//]: # (scope-open)

## Scope (contracts)

- **Scope:**

```
├── src
│   └── TradeMarket.sol
```

- **Compatibility:**

  - Chain: Ethereum
  - Tokens: `wETH`, `wBTC`, `wSOL`, `USDC`

[//]: # (scope-close)
[//]: # (getting-started-open)

## Set-up

```bash
git clone https://github.com/CodeHawks-Contests/2025-06-orderbook.git

forge install --no-git --no-commit Openzeppelin/openzeppelin-contracts

forge build

forge test
```

[//]: # (getting-started-close)
[//]: # (known-issues-open)

## Known Issues

None!

[//]: # (known-issues-close)

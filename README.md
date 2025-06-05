# Bitcoin Oracle Prediction Market

A decentralized prediction market smart contract built on the Stacks blockchain that enables users to stake STX tokens on Bitcoin price movement predictions. Users can bet on whether Bitcoin's price will go "up" or "down" over specified time periods, with winnings distributed proportionally among correct predictors.

## Overview

This smart contract implements a trustless prediction market system where:

- Markets are created with specific time windows and starting Bitcoin prices
- Users stake STX tokens on price direction predictions ("up" or "down")
- An authorized oracle resolves markets with final Bitcoin prices
- Winners receive proportional payouts based on their stake and prediction accuracy
- The platform takes a configurable fee from winnings

## Features

- **Decentralized Prediction Markets**: Create time-bound markets for Bitcoin price predictions
- **Proportional Payouts**: Winnings distributed based on stake size and total winning pool
- **Oracle Integration**: Secure price resolution through authorized oracle addresses
- **Fee Management**: Configurable platform fees with owner withdrawals
- **Stake Protection**: Minimum stake requirements and balance validation
- **Market Lifecycle**: Complete market creation, participation, resolution, and claim process

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    Bitcoin Oracle Prediction Market             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────────┐    ┌──────────────────┐    ┌─────────────┐ │
│  │   Market        │    │   User           │    │   Oracle    │ │
│  │   Creation      │    │   Predictions    │    │   Resolution│ │
│  │                 │    │                  │    │             │ │
│  │ • Start Price   │    │ • Stake STX      │    │ • End Price │ │
│  │ • Time Window   │    │ • Up/Down Vote   │    │ • Market    │ │
│  │ • Market ID     │    │ • Claim Rewards  │    │   Settlement│ │
│  └─────────────────┘    └──────────────────┘    └─────────────┘ │
│                                                                 │
├─────────────────────────────────────────────────────────────────┤
│                        Core Components                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │                    Data Storage                             │ │
│  │                                                             │ │
│  │  Markets Map:                                               │ │
│  │  • market-id → {start-price, end-price, stakes, blocks}    │ │
│  │                                                             │ │
│  │  User Predictions Map:                                      │ │
│  │  • {market-id, user} → {prediction, stake, claimed}        │ │
│  └─────────────────────────────────────────────────────────────┘ │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │                  Access Control                             │ │
│  │                                                             │ │
│  │  • Contract Owner: Market creation, admin functions        │ │
│  │  • Oracle Address: Market resolution authority             │ │
│  │  • Users: Predictions and claim operations                 │ │
│  └─────────────────────────────────────────────────────────────┘ │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## Contract Functions

### Public Functions

#### Market Management

- `create-market(start-price, start-block, end-block)` - Creates a new prediction market
- `resolve-market(market-id, end-price)` - Resolves market with final Bitcoin price (Oracle only)

#### User Operations

- `make-prediction(market-id, prediction, stake)` - Places a prediction with STX stake
- `claim-winnings(market-id)` - Claims proportional winnings from resolved markets

#### Administrative

- `set-oracle-address(new-address)` - Updates authorized oracle address
- `set-minimum-stake(new-minimum)` - Updates minimum stake requirement
- `set-fee-percentage(new-fee)` - Updates platform fee percentage
- `withdraw-fees(amount)` - Withdraws accumulated platform fees

### Read-Only Functions

- `get-market(market-id)` - Returns market details and status
- `get-user-prediction(market-id, user)` - Returns user's prediction for a market
- `get-contract-balance()` - Returns total STX balance held by contract

## Usage Flow

### 1. Market Creation

```clarity
(create-market u50000000 u1000 u2000) ;; $50k start price, blocks 1000-2000
```

### 2. Making Predictions

```clarity
(make-prediction u0 "up" u10000000) ;; Bet 10 STX that price goes up
```

### 3. Market Resolution

```clarity
(resolve-market u0 u55000000) ;; Oracle sets final price to $55k
```

### 4. Claiming Winnings

```clarity
(claim-winnings u0) ;; Claim proportional winnings if prediction was correct
```

## Payout Calculation

Winnings are calculated proportionally based on:

- **User Stake**: Amount of STX the user staked
- **Total Pool**: Combined stakes from all participants
- **Winning Pool**: Combined stakes from correct predictors

```
Winnings = (User Stake × Total Pool) ÷ Winning Pool
Payout = Winnings - Platform Fee
```

## Configuration Parameters

| Parameter | Default Value | Description |
|-----------|---------------|-------------|
| `minimum-stake` | 1,000,000 μSTX (1 STX) | Minimum stake required per prediction |
| `fee-percentage` | 2% | Platform fee taken from winnings |
| `oracle-address` | ST1PQHQKV0... | Authorized address for price resolution |

## Error Codes

| Code | Constant | Description |
|------|----------|-------------|
| u100 | `err-owner-only` | Operation restricted to contract owner |
| u101 | `err-not-found` | Market or prediction not found |
| u102 | `err-invalid-prediction` | Invalid prediction format or parameters |
| u103 | `err-market-closed` | Market is not accepting predictions |
| u104 | `err-already-claimed` | Winnings have already been claimed |
| u105 | `err-insufficient-balance` | Insufficient STX balance for operation |
| u106 | `err-invalid-parameter` | Invalid function parameter provided |

## Security Features

- **Access Control**: Owner-only administrative functions and oracle-only resolution
- **Balance Validation**: Ensures users have sufficient STX before accepting stakes
- **Double-Claim Protection**: Prevents multiple reward claims for the same prediction
- **Market State Validation**: Enforces proper market lifecycle and timing constraints
- **Parameter Validation**: Validates all inputs to prevent invalid states

## Deployment Requirements

- Stacks blockchain environment
- Clarity smart contract runtime
- Authorized oracle service for Bitcoin price feeds
- Initial STX balance for contract owner operations

## Integration Examples

### Frontend Integration

```javascript
// Connect to Stacks wallet and make prediction
const makePredicition = async (marketId, direction, stakeAmount) => {
  const functionArgs = [
    uintCV(marketId),
    stringAsciiCV(direction),
    uintCV(stakeAmount)
  ];
  
  return await openContractCall({
    contractAddress: CONTRACT_ADDRESS,
    contractName: CONTRACT_NAME,
    functionName: 'make-prediction',
    functionArgs: functionArgs
  });
};
```

### Oracle Integration

```javascript
// Oracle service resolving market with Bitcoin price
const resolveMarket = async (marketId, finalPrice) => {
  return await contractCall({
    contractAddress: CONTRACT_ADDRESS,
    contractName: CONTRACT_NAME,
    functionName: 'resolve-market',
    functionArgs: [uintCV(marketId), uintCV(finalPrice)]
  });
};
```

## Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

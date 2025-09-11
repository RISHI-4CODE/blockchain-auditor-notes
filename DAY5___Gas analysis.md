# Day 5 — Integration + Gas Cost Testing

## Gas Usage Analysis

| Action        | Gas Used | Explanation |
|---------------|----------|-------------|
| Deposit #1    | XXXX     | First storage write → `SSTORE` (20k gas) + event log (`LOG2`) |
| Deposit #2    | YYYY     | Update existing storage slot → cheaper `SSTORE` (5k gas) + event log |
| Withdraw      | ZZZZ     | Storage update + event log + ETH transfer (extra cost for external call) |

### Key Insights
- **Storage writes dominate** gas consumption. First writes are far more expensive than updates.  
- **Events (LOGx opcodes)** add a base cost of 375 gas + 8 gas/byte of data.  
- **Withdrawals** are costlier than deposits since they both update storage *and* transfer ETH.  
- **View functions** (like `getBalance`) are free off-chain (0 gas when called locally).  

### Notes
- Always benchmark gas in your environment — costs can shift with Solidity/EVM version.  
- Designing contracts to **minimize storage writes** is the #1 way to optimize gas.

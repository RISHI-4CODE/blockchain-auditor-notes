# Day 4 — Events & Deployment Mastery

## 🔹 Solidity Events Basics

- **Events** are a logging mechanism for contracts.
- They don’t touch storage; instead, they’re stored in transaction receipts.
- Useful for off-chain indexing, UI updates, and analytics.
- Cannot be read back by contracts.

**Declaration:**
```solidity
event Transfer(address indexed from, address indexed to, uint amount);
```

- **Indexed params** → searchable (up to 3).
- **Non-indexed params** → stored in data section.
- Always includes event signature hash as `topic[0]`.

---

## 🔹 EVM Internals

- `emit` compiles into `LOGx` opcodes (`LOG0` to `LOG4`, depending on number of indexed params).

**Log structure:**
- `address`: contract that emitted it.
- `topics[]`: event signature + indexed params.
- `data`: ABI-encoded non-indexed params.

**Bloom Filters:**
- Each block header has a 2048-bit Bloom filter.
- Used to quickly find if logs (addresses/topics) are inside a block.
- False positives possible, never false negatives.

---

## 🔹 Gas Costs

- Gas cost = base + topics + data.
    - **Base cost:** 375 gas.
    - **Per topic:** 375 gas.
    - **Per byte of data:** 8 gas.

**Example:**
```solidity
emit Transfer(msg.sender, receiver, 100);
```
- `LOG3` → base 375 + 3×375 (signature + 2 indexed params).
- Data = 32 bytes (`uint256`) → 32×8 = 256 gas.
- **Total:** ~1,806 gas.

**Compare:**
- Storage write (`SSTORE`) → 20,000 gas.
- Emitting event → < 2,000 gas.

👉 **Events = much cheaper when persistence on-chain isn’t required.**

---

## 🔹 Practice Recap (Remix)

- Built Bank contract with Deposit & Withdrawal events.
- Deployed in Remix → emitted events visible in transaction receipt.

**Logs structure seen in Remix:**
- Indexed address → topics.
- Amount → data section.

---

## ✅ Key Takeaways

- Events = efficient, off-chain-readable logs.
- Never rely on events for contract logic (can’t read them back).
- Use events for state-change transparency, debugging, and dApp/frontend sync.
- Gas-wise: events are far cheaper than storage, but
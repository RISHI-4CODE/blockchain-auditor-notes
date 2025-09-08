# Day 2 — Storage vs Memory Deep Dive (Auditor Notes)

---

## Storage vs Memory vs Calldata

### Storage
- Permanent, lives on the blockchain.  
- Every **write** is costly in gas (20k gas for a fresh slot, ~5k for overwrite).  
- Vulnerability surface: **unbounded writes** (user-controlled `push`, `pop`, `delete`) can lead to gas griefing.  
- **Audit check:** 
  - Is storage being updated unnecessarily inside loops?  
  - Can writes be moved to memory first, then persisted once?  

```solidity
uint[] public numbers;

function addToStorage(uint _num) public {
    numbers.push(_num); // writes to storage
}
```
- 💰 Observation: Heavy storage writes should be minimized. If user input drives storage growth without limits → possible DoS vector.

### Memory
- Temporary, function-scoped.
- Cheaper than storage but more expensive than calldata.
- Data disappears after execution.

### Audit check:

- Is the developer copying large arrays to memory unnecessarily?
- Memory allocation cost grows linearly with array size.

```solidity
function copyToMemory() public view returns (uint[] memory) {
    uint[] memory copyOfNumbers = numbers; // temporary copy
    if (copyOfNumbers.length > 0) {
        copyOfNumbers[0] = 13; // doesn’t affect storage
    }
    return copyOfNumbers;
}
```

- 💡 Best practice: Favor memory for intermediate calculations, but avoid copying large state arrays unless required.

### Calldata
- Cheapest, read-only input location.
- Ideal for external/public functions receiving large arrays.

### Audit check:

- Ensure developers use calldata for function parameters when mutation isn’t needed.
- Using memory instead of calldata can waste 10x+ gas.

```solidity
function sumFromCalldata(uint[] calldata input) public pure returns (uint total) {
    for (uint i = 0; i < input.length; i++) {
        total += input[i]; // read only
    }
}
```

- ⚡ Gas: Most efficient. Not using calldata is a common audit finding.

### EVM Internals: Storage Slots

- Each state variable sits in a 32-byte slot.
- For dynamic arrays:
- The slot stores only the array length.
- The actual data begins at:

```scss
keccak256(slot_number)
Example: if numbers is at slot 0, element i lives at keccak256(0) + i.
```

- 🔑 Audit perspective:
- This layout is deterministic, not encrypted.
- Attackers can read raw storage using tools (e.g., eth_getStorageAt).
- Never store secrets in contract storage (passwords, private keys).

### Gas Comparison (from Remix tests)
- Storage writes: highest cost.
- Memory: moderate cost.
- Calldata: ~10x cheaper in observed tests.

### Audit implication:

- Inefficient use of storage is one of the top causes of excessive gas fees.
- Gas inefficiency can lock users out if tx costs spike (→ economic DoS).

### Key Takeaways for Auditors
- Flag unnecessary storage writes, especially inside loops.
- Recommend calldata for external function parameters whenever possible.
- Watch for unbounded storage growth (potential DoS via gas).
- Verify sensitive data is not stored in raw storage (since all storage is publicly readable).
# Day 2 — Storage vs Memory Deep Dive

---

## Solidity Basics: Storage vs Memory vs Calldata

### Storage
- Permanent, lives on the blockchain.  
- Every **write** is expensive in gas.  
- Use only when data must persist across transactions.  

```solidity
uint[] public numbers;

function addToStorage(uint _num) public {
    numbers.push(_num); // writes to storage
}
```
- 💰 Gas: Storage writes are the most expensive.

### Memory

- Temporary, exists only for the duration of a function call.
- Cheaper than storage.
- Data in memory does not persist after the function ends.

```solidity
function copyToMemory() public view returns (uint[] memory) {
    uint[] memory copyOfNumbers = numbers; // creates temp copy
    if (copyOfNumbers.length > 0) {
        copyOfNumbers[0] = 13; // doesn’t affect storage
    }
    return copyOfNumbers;
}
```
- 💡 Use case: **temporary** calculations, avoid modifying persistent state.

### Calldata

- Read-only, cheapest option.
- Typically used for external function inputs.
- Cannot be modified inside the function.

```solidity
function sumFromCalldata(uint[] calldata input) public pure returns (uint total) {
    for (uint i = 0; i < input.length; i++) {
        total += input[i]; // read only
    }
}
```

- ⚡ Gas: Most efficient when dealing with input arrays/structs.

### EVM Internals: Storage Slots

- Each state variable is stored in a storage slot (32 bytes).
- For dynamic arrays like numbers, the slot stores only the length.
-Actual array data starts at:
```solidity
keccak256(slot_number)
```
- If numbers is at slot 0, its data starts at keccak256(0).
- Index i lives at keccak256(0) + i.

- 🔑 This isn’t encryption — it’s deterministic placement to avoid collisions.


### Gas Comparison (from Remix tests)

- Storage write: most expensive.
- Memory: moderately expensive.
- Calldata: ~10x cheaper than memory/storage in observed tests.

- **Key Takeaways**

- Default to calldata for external inputs whenever possible.
- Use memory for temporary variables.
- Reserve storage for data that must persist between transactions.
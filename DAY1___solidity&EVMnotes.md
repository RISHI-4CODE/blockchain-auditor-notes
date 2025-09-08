# Solidity & EVM Notes (Auditor’s Lens)

---

## 🔹 Variables & Types

### Integers (`uint256`, `int256`)
- **What:** Signed vs unsigned integer values.  
- **Security Risks:**  
  - Pre-Solidity 0.8: overflow/underflow bugs.  
  - Type truncation (`uint8` ↔ `uint256`).  
- **Auditor Checklist:**  
  - Confirm explicit type sizes.  
  - Ensure safe math where backward compatibility is required.  

---

### Boolean (`bool`)
- **What:** True/false values, often control logic.  
- **Security Risks:**  
  - Defaults to `false` → could unintentionally grant access.  
- **Auditor Checklist:**  
  - Check initialization.  
  - Verify booleans guard critical logic correctly.  

---

### Address (`address`, `address payable`)
- **What:** Identifies accounts/contracts.  
- **Security Risks:**  
  - Misuse of `tx.origin` (phishing).  
  - Incorrect handling of Ether transfers.  
- **Auditor Checklist:**  
  - Ensure `msg.sender` vs `tx.origin` is correct.  
  - Check payable functions don’t allow unintended withdrawals.  

---

### String & Bytes
- **What:** Arbitrary-length data.  
- **Security Risks:**  
  - Expensive to store → gas griefing.  
  - Unbounded input → DoS risks.  
- **Auditor Checklist:**  
  - Check for input size limits.  
  - Flag unnecessary on-chain string storage.  

---

### Visibility Modifiers (`public`, `private`, `internal`, `external`)
- **What:** Controls who can call a function.  
- **Security Risks:**  
  - Wrong visibility = unintended access.  
- **Auditor Checklist:**  
  - Ensure sensitive functions are not `public`.  
  - Confirm internal/external consistency.  

---

## 🔹 EVM Overview

### Gas
- **What:** Execution cost measure.  
- **Security Risks:**  
  - High gas → DoS.  
- **Auditor Checklist:**  
  - Check for unbounded loops.  
  - Verify expensive operations are minimized.  

---

### Stack (1024 slots, 32 bytes each)
- **What:** Temporary computation layer.  
- **Security Risks:**  
  - Stack-depth bombs (rare today).  
- **Auditor Checklist:**  
  - Review inline assembly/Yul carefully.  

---

### Memory
- **What:** Temporary storage cleared after function call.  
- **Security Risks:**  
  - Large unbounded arrays → gas exhaustion.  
- **Auditor Checklist:**  
  - Verify array sizes are bounded.  

---

### Storage
- **What:** Permanent, replicated across nodes.  
- **Security Risks:**  
  - Expensive.  
  - State corruption/collisions in upgradeable contracts.  
- **Auditor Checklist:**  
  - Check for storage collisions.  
  - Confirm state cleanup when required.  

---

### Mappings
- **What:** Key-value storage. Keys hashed with `keccak256(key, slot)`.  
- **Security Risks:**  
  - Deleted mappings leave ghost values.  
- **Auditor Checklist:**  
  - Ensure delete/reset logic is implemented.  
  - Check for key reuse issues.  

---

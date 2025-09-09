# Day 3 — Solidity Deep Dive: Structs, Inheritance, Modifiers & ABI

## 1️⃣ Structs

Structs let you create **custom data types** by grouping multiple variables.

```solidity
struct Account {
    string holder;
    uint balance;
}

Account internal account;

```

- Stored in storage or memory
- Useful for modeling users, transactions, or any complex entity
- Key takeaway: Organizes data and makes contracts readable and maintainable.

## 2️⃣ Function Modifiers
- Modifiers are reusable checks that run before or after a function executes.

```solidity
modifier onlyOwner() {
    require(msg.sender == owner, "Not the owner!");
    _;
}
```

- _ = placeholder for the original function code
- Can enforce access control, balance checks, or other reusable logic

**Example usage:**

```solidity
function withdrawAmount(uint _amount) public onlyOwner sufficientBalance(_amount) returns (uint) {
    account.balance -= _amount;
    emit Withdraw(msg.sender, _amount, account.balance);
    return account.balance;
}
```

- Key takeaway: Essential for security-critical functions.

## 3️⃣ Inheritance
- Child contracts can reuse and extend parent contract logic.

```solidity
contract BankContract is BankAccount {
    constructor(string memory _name, uint _initialBalance)
        BankAccount(_name, _initialBalance) {}
}
```

- BankContract inherits BankAccount’s storage, events, and modifiers
- Enables modular, maintainable contract architecture

## 4️⃣ ABI Encoding & Calldata
- ABI = Application Binary Interface, how function calls + arguments are encoded as bytes
- First 4 bytes = function selector (hash of function signature)
- Rest = padded arguments

```solidity
function peekCalldata() public view returns (bytes memory) {
    return msg.data;
}
```
- msg.data contains raw calldata
- Useful for auditing, debugging, or low-level proxy calls

- Key takeaway: Knowing ABI encoding helps detect unexpected calls or delegatecall issues.

## 5️⃣ Events
- Events are logs on the blockchain, useful for frontends or monitoring.

```solidity
event Deposit(address indexed user, uint amount, uint newBalance);

emit Deposit(msg.sender, _amount, account.balance);
```
- Indexed fields allow efficient searches
- Do not change storage, cheaper than storing state variables
- Can track deposits, withdrawals, ownership changes, or function calls

## 6️⃣ Example: BankContract
```solidity
contract BankContract is BankAccount {
    function depositAmount(uint _amount) public onlyOwner returns (uint) {
        account.balance += _amount;
        emit Deposit(msg.sender, _amount, account.balance);
        emit CalldataPeek(msg.data);
        return account.balance;
    }

    function withdrawAmount(uint _amount) public onlyOwner sufficientBalance(_amount) returns (uint) {
        account.balance -= _amount;
        emit Withdraw(msg.sender, _amount, account.balance);
        emit CalldataPeek(msg.data);
        return account.balance;
    }
}
```

**Key points:**
- Combines structs, inheritance, and modifiers
- Emits events for transactions and raw calldata
- Only the owner can interact with sensitive functions

## ✅ Summary Takeaways
- Structs = clean data modeling
- Modifiers = reusable checks for security
- Inheritance = modular & maintainable contracts
- ABI & msg.data = low-level understanding for auditing
- Events = front-end integration & transaction logging
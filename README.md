# L1 Boss Bridge – Security Audit Findings 🛡️

This repository documents my security review and vulnerability findings for the **L1 Boss Bridge** system.  
The audit focuses on identifying **critical attack vectors, design flaws, and protocol-level risks** that could lead to loss of funds, replay attacks, infinite minting, or denial of service.

---

## 👨‍💻 Auditor

**Vivek Sharma**

- 🐦 X (Twitter): https://x.com/viveksh0062  
- 💼 LinkedIn: https://www.linkedin.com/in/vivek-sharma-679606360/  

---

## 📌 Scope

The review primarily covers the following contracts:

- `L1BossBridge.sol`
- `L1Vault.sol`
- `TokenFactory.sol`

---

## 🚨 High Severity Findings

### [H-1] Users who approve tokens to `L1BossBridge` may have assets stolen

The `depositTokensToL2` function allows anyone to specify an arbitrary `from` address.  
Any account that has approved tokens to the bridge can have its tokens transferred without consent.

**Impact:**  
An attacker can drain tokens from any approved user and mint them to an attacker-controlled L2 address.

**Recommended Fix:**  
Restrict deposits so tokens are transferred only from `msg.sender`.

---

### [H-2] Infinite minting by transferring tokens from Vault to Vault

Because the vault grants infinite approval to the bridge, attackers can self-transfer tokens from the vault to itself and emit unlimited `Deposit` events.

**Impact:**  
Unbacked tokens can be minted infinitely on L2.

**Recommended Fix:**  
Prevent arbitrary `from` address usage in `depositTokensToL2`.

---

### [H-3] Lack of replay protection enables withdrawal replay attacks

Withdrawal signatures lack nonces or unique identifiers.

**Impact:**  
A single valid operator signature can be replayed to drain the entire vault.

**Recommended Fix:**  
Introduce nonces or one-time message tracking for withdrawals.

---

### [H-4] Arbitrary calls via `sendToL1` allow vault takeover

`sendToL1` allows arbitrary low-level calls with no target or calldata restrictions.

**Impact:**  
Attackers can call `L1Vault::approveTo` and grant themselves infinite allowance, draining the vault.

**Recommended Fix:**  
Disallow arbitrary calls to sensitive contracts like `L1Vault`.

---

### [H-5] `CREATE` opcode incompatibility with zkSync Era

**Impact:**  
Token deployment may fail or behave unexpectedly on zkSync Era.

---

### [H-6] Deposit limit check can cause permanent DoS

Improper `DEPOSIT_LIMIT` logic may permanently block deposits.

---

### [H-7] Withdrawal amount not validated against deposit amount

Attackers may withdraw more tokens than they deposited.

---

### [H-8] Tokens deployed via `TokenFactory::deployToken` are permanently locked

Tokens become inaccessible after deployment.

---

## ⚠️ Medium Severity Findings

### [M-1] Withdrawals vulnerable to return-bomb gas griefing

Low-level calls forward all gas and copy returndata.

**Impact:**  
Malicious targets can force excessive gas usage.

**Recommended Fix:**  
Use safe-call patterns such as `ExcessivelySafeCall`.

---

## 🟡 Low Severity Findings

### [L-1] Missing withdrawal event emissions

No events are emitted for withdrawals.

**Impact:**  
Off-chain monitoring and alerting becomes difficult.

---

### [L-2] Duplicate token symbols allowed

Multiple tokens can be deployed with the same symbol.

---

### [L-3] Unsupported opcode `PUSH0`

Potential incompatibility with certain EVM environments.

---



### Conclusion
The L1 Boss Bridge contains multiple critical vulnerabilities that could lead to:

Complete vault drainage

Infinite minting of unbacked tokens

Replay-based fund extraction

Privilege escalation via arbitrary calls

These issues should be addressed before any production deployment.

🧠 If you’re interested in Web3 security reviews, protocol auditing, or exploit research — feel free to connect.

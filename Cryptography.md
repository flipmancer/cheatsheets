# Cryptography for Web3 Developers

> Practical guide to cryptography in blockchain development. Real code, real patterns, real security mistakes to avoid.

## Table of Contents

- [Cryptography for Web3 Developers](#cryptography-for-web3-developers)
	- [Table of Contents](#table-of-contents)
	- [Overview](#overview)
	- [Getting Started](#getting-started)
		- ["I know nothing about cryptography" — Start here](#i-know-nothing-about-cryptography--start-here)
	- [Core Cryptographic Primitives](#core-cryptographic-primitives)
		- [1. Core Mental Model](#1-core-mental-model)
		- [2. Hashing (SHA-256, Keccak-256)](#2-hashing-sha-256-keccak-256)
		- [3. Signatures (ECDSA over secp256k1)](#3-signatures-ecdsa-over-secp256k1)
		- [4. Key Derivation \& Mnemonics (BIP-39, BIP-32, BIP-44)](#4-key-derivation--mnemonics-bip-39-bip-32-bip-44)
		- [5. Merkle Trees](#5-merkle-trees)
		- [6. Symmetric Encryption (AES-GCM)](#6-symmetric-encryption-aes-gcm)
		- [7. Randomness](#7-randomness)
	- [Advanced Concepts](#advanced-concepts)
		- [8. Zero-Knowledge Proofs](#8-zero-knowledge-proofs)
		- [9. Signature Recovery \& ecrecover](#9-signature-recovery--ecrecover)
		- [10. Address Derivation (Public Key → Address)](#10-address-derivation-public-key--address)
		- [11. Replay Protection \& Cross-Chain Signatures](#11-replay-protection--cross-chain-signatures)
		- [12. Nonce Management \& Meta-Transactions](#12-nonce-management--meta-transactions)
		- [13. Multi-Sig \& Threshold Cryptography](#13-multi-sig--threshold-cryptography)
	- [Security \& Best Practices](#security--best-practices)
		- [14. Gas Optimization (Crypto-Relevant)](#14-gas-optimization-crypto-relevant)
		- [15. Contract Upgrades \& Key Rotation](#15-contract-upgrades--key-rotation)
		- [16. Common EIPs You Must Know](#16-common-eips-you-must-know)
		- [17. Hardware Wallet Integration](#17-hardware-wallet-integration)
		- [18. Side-Channel Attacks \& Timing](#18-side-channel-attacks--timing)
		- [19. Key Stretching \& Password Security](#19-key-stretching--password-security)
	- [Security Checklist](#security-checklist)
		- [20. Pre-Deployment Auditing](#20-pre-deployment-auditing)
		- [21. Common Attacks](#21-common-attacks)
		- [22. Best Practices Summary](#22-best-practices-summary)
	- [Quick Reference](#quick-reference)
		- [Handy Snippets](#handy-snippets)
		- [Glossary](#glossary)
		- [Configuration Defaults](#configuration-defaults)
		- [What NOT to Do](#what-not-to-do)
	- [Resources](#resources)

## Overview

- Purpose: Quick reference for cryptographic primitives used in Web3 development (signatures, hashing, key management, zero-knowledge proofs)
- Audience: Advanced (assumes JavaScript/Solidity knowledge, no crypto math background required)
- Scope: Ethereum/EVM-focused cryptography. Covers ECDSA, Keccak-256, Merkle trees, EIPs, hardware wallets, common attacks. Excludes general networking/TLS, Bitcoin-specific tech (Taproot, Lightning), post-quantum cryptography
- Updated: 2025-11-09

---

## Getting Started

### "I know nothing about cryptography" — Start here

Short explanation:

- What it is: Cryptography is the math that keeps your crypto safe
- Why it matters: Every wallet interaction uses crypto—signing transactions, deriving addresses, proving ownership
- Mental model: Three core primitives: **Hashing** (one-way fingerprints), **Signatures** (prove you control a key), **Encryption** (lock data with a password)

**You already use cryptography every day without realizing it:**

- When you log into MetaMask → you decrypt your private key (AES encryption)
- When you send ETH → you sign the transaction (ECDSA signature)
- When you mint an NFT from an allowlist → you provide a Merkle proof (hash tree)
- When you see your wallet address → it's derived from your public key (Keccak-256 hash)

**The golden rule:** Private keys are EVERYTHING. Lose them = lose your funds. Leak them = someone steals your funds. All of cryptography exists to protect and use private keys safely.

**How to use this cheatsheet:** Read sections 1-7 first (fundamentals). Then jump to whatever you need: signatures (3), Merkle trees (5), hardware wallets (17), etc. Code examples use Solidity + TypeScript (ethers v5).

---

## Core Cryptographic Primitives

### 1. Core Mental Model

Short explanation:

- What it is: Three cryptographic superpowers that power all of Web3
- Mental model: Hashing = fingerprints, Signatures = prove identity, Encryption = lock secrets

Key concepts:

- **Hashing:** One-way fingerprinting. Data → fixed-size hash. Can't reverse it.
  - Example: Your address is a hash of your public key
- **Signatures:** Prove you control a private key without revealing it.
  - Example: Every transaction you send is signed with your private key
- **Encryption:** Lock data with a password/key, unlock with the same password/key.
  - Example: Your keystore file is your private key encrypted with your MetaMask password

**Public-key cryptography:**

- Keypair = private key (secret) + public key (shareable)
- Blockchains mostly use: **hashing + signatures**
- Encryption is mostly off-chain (keystores, secrets)

---

### 2. Hashing (SHA-256, Keccak-256)

Short explanation:

- What it is: Think of hashing like a fingerprint for data—put in any data, get back a fixed-size unique string
- Why it matters: Your wallet address, transaction IDs, block hashes—all are hashes. It's how we verify data hasn't been tampered with
- Mental model: Like a paper shredder—you can't un-shred documents to get the original back

**Properties:**

- **Deterministic:** same input = same hash (always)
- **One-way:** hash → data is impossible (can't reverse it)
- **Collision-resistant:** nearly impossible for two different inputs to give the same hash
- **Avalanche effect:** change one bit in input, completely different hash

**Uses:**

- Addresses (derive from public key via hashing)
- Transaction IDs, block headers
- Merkle trees
- Commit-reveal schemes

**Practical tips:**

- Ethereum uses Keccak-256 (not NIST SHA3-256). In ethers: `ethers.utils.keccak256(data)`.
- Hash structured data carefully; prefer EIP-712 for typed structured messages.

Example (TypeScript — ethers v5):

```ts
import { ethers } from "ethers";

// Hash a string
const hash = ethers.utils.keccak256(ethers.utils.toUtf8Bytes("hello"));
console.log(hash);  // 0x1c8aff950685c2ed4bc3174f3472287b56d9517b9c948127319a09a7a36deac8

// Hash structured data (simple)
const data = ethers.utils.defaultAbiCoder.encode(['address', 'uint256'], [addr, amount]);
const hash2 = ethers.utils.keccak256(data);
```

Common pitfalls:

- Using `keccak256` vs `sha256`—Ethereum uses Keccak
- Not using EIP-712 for off-chain signatures (hash collisions possible)
- Assuming hash length = security (truncating hashes weakens security)

---

### 3. Signatures (ECDSA over secp256k1)

Short explanation:

- What it does: Digital signatures are like a handwritten signature, but mathematically secure
- Why it matters: This is how transactions work! Sign with private key, anyone can verify with public key
- Mental model: Like a wax seal on a letter—only you have the stamp (private key), but anyone can verify the seal

**How it works:**

- **Proves ownership** of the private key without revealing it
- **Verify** = anyone with the public key can confirm signature matches message
- **Deterministic signing** (RFC 6979) avoids nonce leaks (nonce reuse = private key leaked!)

**Ethereum specifics:**

- **Curve:** secp256k1 (same as Bitcoin)
- **Signature format:** r, s, v (r and s are the signature, v helps recover the public key)
- **Message prefixes:** personal_sign adds prefix vs EIP-712 typed data (domain-separated)

Example sign/verify:

```ts
import { ethers } from "ethers";

const wallet = ethers.Wallet.createRandom();
const msg = "hello";
const sig = await wallet.signMessage(msg);

// Verify signature
const signer = ethers.utils.verifyMessage(msg, sig);
console.log(signer === wallet.address);  // true (checksum address that signed)
```

Common pitfalls:

- Using `personal_sign` instead of EIP-712 (no domain separation = cross-protocol replay attacks)
- Not checking `ecrecover` return value (returns `address(0)` on invalid signature)
- Signature malleability (s can have two valid values—use OpenZeppelin ECDSA library)

---

### 4. Key Derivation & Mnemonics (BIP-39, BIP-32, BIP-44)

Short explanation:

- What it does: ONE seed phrase (12 or 24 words) generates unlimited wallets
- Why it matters: Lose those words = lose everything. But with them, you can recover ALL wallets on any device
- Mental model: Like a tree—seed phrase is the root, every wallet address is a branch. Same root = same branches, always

**Standards:**

- **BIP-39:** Converts random bytes → 12/24 words → seed → master key
- **BIP-32:** HD (Hierarchical Deterministic) wallets - tree of keys from one master seed
- **BIP-44:** Standard path format `m / 44' / coin' / account' / change / index`
  - **Ethereum:** `m/44'/60'/0'/0/0` (60 = ETH coin type)
  - **Bitcoin:** `m/44'/0'/0'/0/0` (0 = BTC coin type)

Example derive addresses (ethers v5):

```ts
import { ethers } from "ethers";

const mnemonic = "test test test test test test test test test test test junk";

// Derive first 3 addresses
for (let i = 0; i < 3; i++) {
  const path = `m/44'/60'/0'/0/${i}`;
  const wallet = ethers.Wallet.fromMnemonic(mnemonic, path);
  console.log(`Address ${i}: ${wallet.address}`);
}
```

Security tips:

- Mnemonic = private key of private keys. Never expose or paste in logs
- Use a password + keystore (PBKDF2/scrypt + AES-GCM) for storage
- Write down seed phrase on paper, store in safe (never digital storage)
- 12 words = 128-bit security, 24 words = 256-bit (both secure against brute-force)

---

### 5. Merkle Trees

Short explanation:

- What it does: Compress a huge list into a single hash, but still prove any item was in that list
- Why it matters: Airdrops! Instead of storing 10,000 addresses on-chain (expensive), store ONE hash (Merkle root). Users prove membership with a few hashes
- Mental model: Tournament bracket—to prove a team made it to the finals, show the path from that team to the championship (not every single game)

**How it works:**

1. Hash all the leaves (addresses, data, etc.)
2. Pair them up and hash the pairs (creates parent nodes)
3. Keep pairing and hashing until you get ONE root hash
4. To prove a leaf is in the tree, you only need the sibling hashes along the path to the root

**Uses:**

- **Airdrops:** whitelist millions of addresses cheaply
- **Allowlists:** mint passes, presale lists
- **Rollups:** prove data is available without storing it all on-chain

Building & verifying (Solidity):

```solidity
function verify(bytes32[] memory proof, bytes32 root, bytes32 leaf) public pure returns (bool) {
    bytes32 computedHash = leaf;
    for (uint256 i = 0; i < proof.length; i++) {
        computedHash = computedHash < proof[i]
            ? keccak256(abi.encodePacked(computedHash, proof[i]))
            : keccak256(abi.encodePacked(proof[i], computedHash));
    }
    return computedHash == root;
}
```

Off-chain (TypeScript with merkletreejs):

```ts
import { MerkleTree } from "merkletreejs";
import { ethers } from "ethers";

const addresses = ['0x123...', '0x456...', '0x789...'];
const leaves = addresses.map(addr => ethers.utils.keccak256(addr));
const tree = new MerkleTree(leaves, ethers.utils.keccak256, { sortPairs: true });

// Get root (store on-chain)
const root = tree.getHexRoot();

// Get proof for address (give to user)
const proof = tree.getHexProof(leaves[0]);

// Verify off-chain
const isValid = tree.verify(proof, leaves[0], root);
```

Common pitfalls:

- Not sorting pairs (makes tree construction non-deterministic)
- Using wrong hash function (must match on-chain verification)
- Off-by-one errors in proof indices

---

### 6. Symmetric Encryption (AES-GCM)

Short explanation:

- What it does: Symmetric encryption uses ONE password/key to both lock and unlock data
- Why it matters: Your MetaMask password encrypts your private key. Without the password, the encrypted file (keystore) is useless gibberish
- Mental model: Like a padlock—same key locks and unlocks it. If someone steals the padlock but doesn't have the key, they can't open it

**Key points:**

- **AES-256-GCM:** Industry standard (256-bit key, Galois/Counter Mode)
- **Confidentiality + Integrity:** GCM mode not only encrypts but also detects tampering (auth tag)
- **Password → Key:** Use scrypt or Argon2 to turn a weak password into a strong encryption key
- **Random IV:** Must use a new random IV (initialization vector) every time you encrypt

Example (Node.js):

```ts
import crypto from 'crypto';

// Encrypt
function encrypt(plaintext: string, password: string): { iv: string, ciphertext: string, tag: string } {
  const salt = crypto.randomBytes(32);
  const key = crypto.scryptSync(password, salt, 32);
  const iv = crypto.randomBytes(12);

  const cipher = crypto.createCipheriv('aes-256-gcm', key, iv);
  let ciphertext = cipher.update(plaintext, 'utf8', 'hex');
  ciphertext += cipher.final('hex');
  const tag = cipher.getAuthTag();

  return {
    iv: iv.toString('hex'),
    ciphertext,
    tag: tag.toString('hex')
  };
}

// Decrypt
function decrypt(encrypted: any, password: string): string {
  const salt = /* retrieve salt */;
  const key = crypto.scryptSync(password, salt, 32);

  const decipher = crypto.createDecipheriv('aes-256-gcm', key, Buffer.from(encrypted.iv, 'hex'));
  decipher.setAuthTag(Buffer.from(encrypted.tag, 'hex'));

  let plaintext = decipher.update(encrypted.ciphertext, 'hex', 'utf8');
  plaintext += decipher.final('utf8');
  return plaintext;
}
```

Best practices:

- Random 96-bit IV per encrypt; store {alg, kdf, salt, iv, tag, ct}
- Don't roll your own formats—use well-adopted keystore formats
- Use strong KDF parameters (see section 19)

---

### 7. Randomness

Short explanation:

- What it does: Randomness is the foundation of security—keys, nonces, IVs all need to be unpredictable
- Why it matters: If someone can predict your "random" numbers, they can steal your keys
- Mental model: Like shuffling cards—you need truly random shuffling, not a predictable pattern

**The trap:** Developers try to use `block.timestamp` or `blockhash` for randomness in smart contracts. This is DANGEROUS because miners can manipulate these values.

**Solutions:**

- **Off-chain (safe):** Use Node's `crypto.randomBytes()` or Web Crypto API (CSPRNG = Cryptographically Secure Pseudo-Random Number Generator)
- **On-chain (hard):** Use Chainlink VRF (Verifiable Random Function) or commit–reveal schemes
- **Never use:** `blockhash` alone, `block.timestamp`, `msg.sender` hashing for anything with value

Example (off-chain):

```ts
import crypto from 'crypto';

// Generate random bytes (CSPRNG)
const randomBytes = crypto.randomBytes(32);

// Generate random wallet
import { ethers } from 'ethers';
const wallet = ethers.Wallet.createRandom();  // Uses secure RNG
```

Example (on-chain with Chainlink VRF):

```solidity
import "@chainlink/contracts/src/v0.8/VRFConsumerBase.sol";

contract RandomNumber is VRFConsumerBase {
    bytes32 internal keyHash;
    uint256 internal fee;
    uint256 public randomResult;

    function getRandomNumber() public returns (bytes32 requestId) {
        return requestRandomness(keyHash, fee);
    }

    function fulfillRandomness(bytes32 requestId, uint256 randomness) internal override {
        randomResult = randomness;
    }
}
```

---

## Advanced Concepts

### 8. Zero-Knowledge Proofs

Short explanation:

- What it does: Prove something is true WITHOUT revealing the actual information
- Why it matters: Privacy coins hide transaction details while proving validity. zkRollups prove thousands of transactions are correct without recalculating everything
- Mental model: Prove you're over 21 without showing your ID—hand the bouncer a black box that lights green if true, red if false. They learn you're 21+ but nothing else

**Key concepts:**

- **Prover:** "I know X is true" (e.g., I have enough balance)
- **Verifier:** "Prove it, but don't tell me the details" (don't reveal balance)
- **Zero knowledge:** Verifier learns ONLY that the statement is true, nothing more

**Types:**

- **zk-SNARKs:** Small proofs, fast verification, requires trusted setup (risky if setup is compromised)
- **zk-STARKs:** Larger proofs, no trusted setup, quantum-resistant
- **PLONK, Groth16:** Different proof systems with tradeoffs (setup, speed, size)

**Practical ZK use cases:**

- Privacy: Tornado Cash (shielded pool), Zcash
- Scaling: zkSync, StarkNet (validity rollups)
- Identity: Proof of humanity without doxxing
- Gaming: Provable RNG, hidden information

---

### 9. Signature Recovery & ecrecover

Short explanation:

- What it does: Ethereum RECOVERS the public key (and thus address) FROM the signature itself
- Why it matters: Powers permit functions (gasless approvals)—users sign off-chain, contract recovers signer's address and executes
- Mental model: Like reverse-engineering a wax seal to figure out which stamp was used

**How it works:**

The signature contains r, s (the actual signature) and v (recovery id). The `ecrecover` function uses these three values plus the message hash to calculate which address signed it.

**The trap:** `ecrecover` returns `address(0)` (0x0000...0000) if the signature is invalid. If you don't check for this, anyone can pass garbage signature and it'll "succeed" as address zero.

Solidity example:

```solidity
function recoverSigner(bytes32 hash, bytes memory signature) public pure returns (address) {
    require(signature.length == 65, "Invalid signature length");

    bytes32 r;
    bytes32 s;
    uint8 v;

    assembly {
        r := mload(add(signature, 32))
        s := mload(add(signature, 64))
        v := byte(0, mload(add(signature, 96)))
    }

    address signer = ecrecover(hash, v, r, s);
    require(signer != address(0), "Invalid signature");
    return signer;
}
```

Common pitfalls:

- `ecrecover` returns `address(0)` on failure (ALWAYS check it!)
- Signature malleability: s can have two valid values; use OpenZeppelin's ECDSA library
- Always hash the message with domain separator (EIP-712)

---

### 10. Address Derivation (Public Key → Address)

Short explanation:

- What it does: Your Ethereum address is mathematically derived from your public key using hashing
- Why it matters: Explains why you can have the same address across different EVM chains (same private key = same address on ETH, Polygon, BSC, etc.)
- Mental model: Private key → public key → address (one-way, deterministic—like a funnel that only goes one direction)

**How it works:**

1. Start with your public key (64 bytes, uncompressed)
2. Hash it with Keccak-256 (gives you 32 bytes)
3. Take the last 20 bytes (rightmost 40 hex characters)
4. Add `0x` prefix = your Ethereum address

**Note:** Bitcoin does it differently (uses hash160 + base58check encoding). Solana uses ed25519 public keys directly as addresses.

Example (TypeScript):

```ts
import { ethers } from "ethers";

const wallet = ethers.Wallet.createRandom();

// Public key = 64 bytes (uncompressed, no 0x04 prefix in ethers)
const pubKeyHash = ethers.utils.keccak256(wallet.publicKey);

// Address = last 20 bytes
const address = "0x" + pubKeyHash.slice(-40);

console.log(address === wallet.address.toLowerCase());  // true
```

---

### 11. Replay Protection & Cross-Chain Signatures

Short explanation:

- What it does: Prevents a signature meant for one chain (e.g., Ethereum) from being reused on another chain (e.g., Polygon)
- Why it matters: After Ethereum Classic split, transactions could be "replayed" on both chains. EIP-155 fixed this by including chainId
- Mental model: Like writing "Bank of America only" on a check—can't cash it at Wells Fargo

**The problem:** Without chainId, a dApp on testnet could trick you into signing a message, then use that exact signature on mainnet to drain your funds.

**How it works:**

- **EIP-155:** Transactions include chainId in the signature (prevents cross-chain replay)
- **EIP-712:** Off-chain messages include chainId + contract address in domain separator
- **Always verify:** Contract should check `block.chainid` matches the signed chainId

EIP-712 domain with chainId:

```ts
const domain = {
    name: "MyDapp",
    version: "1",
    chainId: await provider.getNetwork().then(n => n.chainId),
    verifyingContract: contractAddress
};
```

Contract-side verification:

```solidity
bytes32 domainSeparator = keccak256(abi.encode(
    keccak256("EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)"),
    keccak256(bytes("MyDapp")),
    keccak256(bytes("1")),
    block.chainid,  // Verify chainId matches
    address(this)
));
```

---

### 12. Nonce Management & Meta-Transactions

Short explanation:

- What it does: A nonce (number used once) is a counter that increments with each transaction, preventing replay attacks
- Why it matters: Without nonces, someone could replay your "send 1 ETH" transaction 100 times and drain your account
- Mental model: Like check numbers—bank knows check #101 should be cashed before #102. Can't cash #101 twice

**Two types:**

1. **On-chain nonces:** Every address has a transaction count (nonce). Each transaction must use the next sequential nonce. Wallets track this automatically.
2. **Off-chain nonces:** Smart contracts track nonces per user for permit/meta-tx functions. Prevents replaying signed approvals.

**Meta-transactions:** Users sign a message off-chain (no gas), a relayer submits it on-chain (pays gas), contract verifies signature and increments user's nonce. This enables gasless UX.

Permit (EIP-2612) example:

```solidity
mapping(address => uint256) public nonces;

function permit(
    address owner,
    address spender,
    uint256 value,
    uint256 deadline,
    uint8 v,
    bytes32 r,
    bytes32 s
) external {
    require(block.timestamp <= deadline, "Expired");

    bytes32 structHash = keccak256(abi.encode(
        PERMIT_TYPEHASH,
        owner,
        spender,
        value,
        nonces[owner]++,  // Increment nonce
        deadline
    ));

    bytes32 digest = keccak256(abi.encodePacked("\x19\x01", DOMAIN_SEPARATOR, structHash));
    address signer = ecrecover(digest, v, r, s);
    require(signer == owner, "Invalid signature");

    _approve(owner, spender, value);
}
```

---

### 13. Multi-Sig & Threshold Cryptography

Short explanation:

- What it does: Multi-signature wallets require multiple people to approve a transaction before it executes (M-of-N threshold)
- Why it matters: DAOs, treasuries, and protocols use multi-sigs for security. One compromised key can't steal funds
- Mental model: Like launching nuclear missiles—two officers must turn their keys simultaneously. One rogue officer can't do anything alone

**Types:**

- **On-chain multi-sig:** Smart contract holds the logic (Gnosis Safe, multisig.sol) - all signatures verified on-chain
- **Off-chain multi-sig (threshold):** Wallets collaborate off-chain to produce ONE signature (MPC wallets, Fireblocks) - cheaper, more private

**Gnosis Safe concepts:**

- Owners vote on transactions
- Threshold (e.g., 3-of-5) required for execution
- Nonce prevents replay
- Can execute arbitrary calls (upgrade contracts, send ETH/tokens)

Common patterns:

- Treasury: 3-of-5 multi-sig
- Protocol upgrades: 2-of-3 multi-sig + 48-hour timelock
- Emergency actions: 5-of-9 multi-sig

---

## Security & Best Practices

### 14. Gas Optimization (Crypto-Relevant)

Short explanation:

- What it does: Write smart contracts that use less computational resources, reducing transaction costs
- Why it matters: Every hash, signature verification, and storage operation costs gas. Poorly optimized contracts can cost users 10x more
- Mental model: Like optimizing a factory assembly line—eliminate unnecessary steps

**The basics:**

- **Storage is expensive:** Reading/writing to blockchain storage costs ~20,000 gas per slot
- **Hashing is cheap:** Keccak256 costs ~30-60 gas (use it liberally)
- **ecrecover is medium:** ~3,000 gas per signature verification (batch if possible)

**Using hash efficiently:**

- `keccak256(abi.encodePacked(...))` cheaper than `abi.encode` (no padding)
- Store merkle roots, verify proofs off-chain when possible
- Cache domain separator (compute once in constructor)

**Signature verification:**

- Batch verify signatures if possible (amortize ecrecover cost)
- Use EIP-1271 for contract signature validation (avoid per-user nonce storage)

---

### 15. Contract Upgrades & Key Rotation

Short explanation:

- What it does: Proxy patterns let you upgrade smart contract logic without losing data or changing the contract address
- Why it matters: Smart contracts are immutable by default—proxies let you fix bugs while keeping the same address and data
- Mental model: Your house (proxy) and the furniture inside (implementation logic). Replace furniture without moving to a new address

**The trade-off:** Upgradeable contracts are more complex and introduce centralization risks. Who controls the upgrade key controls the entire protocol.

**Proxy patterns:**

- **Transparent proxy:** admin can't call implementation
- **UUPS:** implementation holds upgrade logic
- **Beacon:** multiple proxies point to one implementation

**Key rotation strategies:**

- Owner multisig: rotate signers, not contract
- Timelock: delay on sensitive operations (48h)
- Revoke + replace: old keys marked invalid, new keys added

Security checklist:

- Never store private keys in contract storage (even encrypted)
- Use events for audit trail of key changes
- Test upgrade paths in staging before mainnet
- Use multi-sig + timelock for production upgrades

---

### 16. Common EIPs You Must Know

Short explanation:

- What it does: EIPs (Ethereum Improvement Proposals) are standards that define how features work across the Ethereum ecosystem
- Why it matters: Following EIP standards ensures your dApp is compatible with wallets, other contracts, and tools
- Mental model: Like USB-C standard—everyone implements it the same way, so things just work together

**Key EIPs every dev should know:**

- **EIP-155:** Replay protection (chainId in tx signature)
- **EIP-191:** Signed data standard (`\x19Ethereum Signed Message:\n`)
- **EIP-712:** Typed structured data hashing and signing
- **EIP-1271:** Contract signature validation (`isValidSignature`)
- **EIP-2612:** Permit (approve via signature, no tx)
- **EIP-4337:** Account abstraction (smart contract wallets)
- **EIP-1559:** Fee market (baseFee + priorityFee)

Quick reference:

```ts
// EIP-191 (personal_sign)
const hash = ethers.utils.hashMessage(message);
// Adds prefix: "\x19Ethereum Signed Message:\n" + len(message) + message

// EIP-712 (typed data)
const hash = ethers.utils._TypedDataEncoder.hash(domain, types, value);
```

---

### 17. Hardware Wallet Integration

Short explanation:

- What it does: Hardware wallets (Ledger, Trezor) store private keys in a secure chip. The key NEVER leaves the device
- Why it matters: Software wallets store encrypted keys on your computer. Malware can steal them. Hardware wallets protect against this
- Mental model: Your computer is a glass box. Software wallet = keys in glass box with a lock. Hardware wallet = keys in a metal safe outside the glass box. Even if attacker smashes the glass, they can't reach the safe

**The blind signing problem:** Older devices show raw hex data when signing. You can't verify what you're actually signing. Modern wallets support EIP-712, which shows human-readable typed data ("Transfer 100 DAI to vitalik.eth").

**Key points:**

- Ledger, Trezor use secure element to store keys
- Signing happens on device; private key never leaves
- Derivation paths must match (m/44'/60'/0'/0/x for ETH)

Integration (ethers v5):

```ts
import { ethers } from "ethers";
// Use LedgerSigner or TrezorSigner from @ethersproject/hardware-wallets

// Example (conceptual)
const signer = new LedgerSigner(provider, "m/44'/60'/0'/0/0");
const tx = await signer.sendTransaction({ to: recipient, value: amount });
```

---

### 18. Side-Channel Attacks & Timing

Short explanation:

- What it does: Side-channel attacks exploit how long operations take, rather than breaking the math
- Why it matters: If signature verification leaks timing info, attackers can recover private keys. Always use constant-time comparisons for secrets
- Mental model: Like a safe where each correct dial number makes a click. Attacker hears "click... click... click... no click" and knows the first 3 numbers are right

**The JavaScript problem:** JavaScript engines (V8, SpiderMonkey) optimize code unpredictably. JIT compilation, garbage collection, and branch prediction make timing non-constant. For high-security crypto, use native libraries or hardware modules.

Timing attack example:

```ts
// BAD: early return leaks info
function compareSecret(a: string, b: string): boolean {
    if (a.length !== b.length) return false;
    for (let i = 0; i < a.length; i++) {
        if (a[i] !== b[i]) return false; // leaks position of mismatch
    }
    return true;
}

// BETTER: constant-time (Node.js crypto)
import crypto from "crypto";
const isEqual = crypto.timingSafeEqual(Buffer.from(a), Buffer.from(b));
```

Best practices:

- Use constant-time operations for secret comparison
- JavaScript/TypeScript NOT constant-time (JIT, GC)
- For high-stakes crypto, use vetted libraries or hardware modules

---

### 19. Key Stretching & Password Security

Short explanation:

- What it does: Key Derivation Functions (KDFs) make weak passwords harder to crack by intentionally making computation slow and memory-intensive
- Why it matters: Users pick terrible passwords ("password123"). KDFs like scrypt/Argon2 are designed to be SLOW, making brute-force attacks impractical
- Mental model: Cheap lock (SHA-256) takes 1 second to try a combo. Bank vault (scrypt) takes 10 minutes per combo. Same security for legitimate users (open once), but 600x harder for attackers trying billions of combos

**How it works:** Instead of `key = hash(password)`, you do `key = expensive_hash(password, salt, cost_params)`. The "expensive" part forces attackers to spend significant CPU/RAM per guess.

**Which to use:**

- **scrypt:** Memory-hard, GPU-resistant (Ethereum keystore default)
- **Argon2:** Modern, tunable (CPU + memory), recommended for new systems (won the Password Hashing Competition)
- **PBKDF2:** Older, less resistant to ASICs/GPUs (avoid unless for compatibility)

Recommended params (2025):

- scrypt: N=2^18 (262144), r=8, p=1 (desktop); N=2^14 for mobile
- Argon2id: m=64MB, t=3 iterations, p=4 threads

Example (Node.js):

```ts
import crypto from "crypto";

// Derive key from password
const salt = crypto.randomBytes(32);
const key = crypto.scryptSync(password, salt, 32, { N: 262144, r: 8, p: 1 });

// Use key for AES encryption
const iv = crypto.randomBytes(12);
const cipher = crypto.createCipheriv('aes-256-gcm', key, iv);
// ... encrypt data
```

---

## Security Checklist

### 20. Pre-Deployment Auditing

Short explanation:

- What it does: A checklist to catch common crypto mistakes before they cost you millions
- Why it matters: Code might compile and pass tests, but still have catastrophic crypto bugs. One missed check = protocol gets drained
- Mental model: Pre-flight checklist for pilots—every item matters, skipping one can be fatal

**Real-world failures:**

- Poly Network hack ($600M): Missing signature validation
- Wormhole hack ($320M): Signature verification bypassed
- Nomad bridge ($190M): Hash collision in Merkle proof check

**Pre-deployment checklist:**

- [ ] All private keys stored encrypted (keystore/hardware wallet)
- [ ] No hardcoded secrets in source/env files committed to git
- [ ] EIP-712 used for off-chain signatures (not personal_sign)
- [ ] Domain separator includes chainId + contract address
- [ ] `ecrecover` return value checked (not zero address)
- [ ] Signature malleability prevented (OpenZeppelin ECDSA or s-value check)
- [ ] Nonce management prevents replay (on-chain or in signature)
- [ ] Merkle proofs verified correctly (sorted pairs, keccak256)
- [ ] No `tx.origin` for authentication (use `msg.sender`)
- [ ] Multi-sig for admin functions (Gnosis Safe or timelock)
- [ ] Upgrade proxy audited (UUPS/Transparent pattern)
- [ ] Events emitted for all state changes (audit trail)
- [ ] Fuzz tests for signature/hash edge cases
- [ ] Testnet deployment with monitoring before mainnet

---

### 21. Common Attacks

**Key exfiltration:**

- Malware, clipboard hijacking, logging private keys
- Mitigation: Hardware wallets, air-gapped signing

**Reused nonces in ECDSA:**

- Catastrophic—reveals private key
- Mitigation: Deterministic signing (RFC 6979), don't hand-roll

**Length-extension & malleability:**

- Hash collisions in signatures
- Mitigation: Domain separation, EIP-712, MACs/GCM

**Poor KDF params:**

- Weak passwords crackable
- Mitigation: Strong scrypt/Argon2 settings

**RNG failures:**

- Predictable "random" numbers
- Mitigation: Use platform CSPRNG only

**Phishing of signing prompts:**

- Users blindly sign malicious data
- Mitigation: EIP-712 typed data (human-readable)

---

### 22. Best Practices Summary

**Keys:**

- Hardware wallets for meaningful funds
- Separate hot/cold wallets; rotate hot keys; restrict allowances
- Never reuse nonces

**Storage:**

- Keystore (AES-GCM + scrypt/Argon2). Unique random salt & IV
- Don't log secrets; scrub crash dumps; encrypt backups
- Write seed phrases on paper, store in safe (never digital)

**Signing:**

- Prefer EIP-712 for off-chain auth (not personal_sign)
- Verify chainId, domain, version
- Always check `ecrecover` return value

**Contracts:**

- Use OpenZeppelin libraries; add reentrancy guards
- Permissioned functions behind multi-sig timelock
- Emit events for all state changes

**Deploy:**

- Use allowlist Merkle proofs when possible
- Monitor events, set alerts for anomaly behavior
- Testnet first, always

---

## Quick Reference

### Handy Snippets

**Hashing (Solidity):**

```solidity
bytes32 h = keccak256(abi.encodePacked(user, amount));
```

**EIP-712 (TypeScript):**

```ts
const domain = { name: "MyDapp", version: "1", chainId: 1, verifyingContract: addr };
const types = { Permit: [{ name: "owner", type: "address" }, { name: "value", type: "uint256" }] };
const value = { owner, value };
const sig = await wallet._signTypedData(domain, types, value);
```

**Derive wallets from mnemonic:**

```ts
import { ethers } from "ethers";
const mnemonic = "test test test test test test test test test test test junk";
for (let i = 0; i < 3; i++) {
  const path = `m/44'/60'/0'/0/${i}`;
  console.log(ethers.Wallet.fromMnemonic(mnemonic, path).address);
}
```

**Encrypt private key to keystore:**

```ts
const keystore = await wallet.encrypt(password); // JSON keystore (scrypt + AES-128/256-CTR/GCM)
```

---

### Glossary

- **CSPRNG:** Cryptographically Secure Random Number Generator
- **KDF:** Key Derivation Function (PBKDF2, scrypt, Argon2)
- **MAC/AEAD:** Message Authentication Code / Authenticated Encryption with Associated Data (GCM, Poly1305)
- **Curve:** secp256k1 (ETH, BTC), ed25519 (SOL, some L2 tools)
- **Domain separation:** Context-specific prefixing to avoid cross-protocol replay

---

### Configuration Defaults

**Recommended sane defaults:**

- **scrypt:** N=16384, r=8, p=1; 32-byte salt; 32-byte key
- **AES-GCM:** 256-bit key; 96-bit IV; store auth tag
- **Mnemonics:** 12 words (16 bytes entropy) for UX, 24 words (32 bytes) for resilience

---

### What NOT to Do

- Don't invent your own crypto scheme if a standard exists
- Don't reuse nonces/IVs
- Don't store plaintext mnemonics or keys in .env or logs
- Don't sign blind strings in wallets—use typed data (EIP-712)
- Don't use `tx.origin` for authentication
- Don't use `block.timestamp` or `blockhash` for randomness with value

---

## Resources

- **Ethereum:** [EIP-712](https://eips.ethereum.org/EIPS/eip-712), [EIP-55](https://eips.ethereum.org/EIPS/eip-55), [Yellow Paper](https://ethereum.github.io/yellowpaper/paper.pdf)
- **Bitcoin:** [BIP-32/39/44](https://github.com/bitcoin/bips) (HD wallets), [BIP-340](https://github.com/bitcoin/bips/blob/master/bip-0340.mediawiki) (Schnorr)
- **Security:** [OWASP Cryptographic Storage Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html)
- **Standards:** [NIST SP 800-56A](https://csrc.nist.gov/publications/detail/sp/800-56a/rev-3/final) (key agreement), [800-38D](https://csrc.nist.gov/publications/detail/sp/800-38d/final) (GCM)
- **Libraries:** [OpenZeppelin Contracts](https://docs.openzeppelin.com/contracts), [ethers.js](https://docs.ethers.io/v5/)
- **Tools:** [MerkleTree.js](https://github.com/miguelmota/merkletreejs), [Chainlink VRF](https://docs.chain.link/vrf/v2/introduction)

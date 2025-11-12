# Solidity Smart Contract Development

> Complete guide to Solidity development with Foundry and Hardhat. Covers syntax, patterns, testing, deployment, and security vulnerabilities.

## Table of Contents

- [Solidity Smart Contract Development](#solidity-smart-contract-development)
	- [Table of Contents](#table-of-contents)
	- [Overview](#overview)
	- [Development Environment Setup](#development-environment-setup)
		- [1. Foundry Setup](#1-foundry-setup)
		- [2. Hardhat Setup](#2-hardhat-setup)
	- [Solidity Language Fundamentals](#solidity-language-fundamentals)
		- [3. Basic Syntax](#3-basic-syntax)
		- [4. Functions \& Visibility](#4-functions--visibility)
		- [5. Events \& Errors](#5-events--errors)
		- [6. Modifiers](#6-modifiers)
		- [7. Inheritance \& Interfaces](#7-inheritance--interfaces)
	- [Common Patterns](#common-patterns)
		- [8. Access Control Patterns](#8-access-control-patterns)
		- [9. Checks-Effects-Interactions Pattern](#9-checks-effects-interactions-pattern)
		- [10. Pull Over Push Pattern](#10-pull-over-push-pattern)
		- [11. Factory Pattern](#11-factory-pattern)
	- [Testing with Foundry](#testing-with-foundry)
		- [12. Foundry Test Basics](#12-foundry-test-basics)
		- [13. Fuzz Testing](#13-fuzz-testing)
		- [14. Invariant Testing](#14-invariant-testing)
	- [Testing with Hardhat](#testing-with-hardhat)
		- [15. Hardhat Test Basics](#15-hardhat-test-basics)
	- [Security: Threats \& Vulnerabilities](#security-threats--vulnerabilities)
		- [16. Reentrancy Attacks](#16-reentrancy-attacks)
		- [17. Access Control Vulnerabilities](#17-access-control-vulnerabilities)
		- [18. Integer Overflow/Underflow](#18-integer-overflowunderflow)
		- [19. Front-Running (MEV)](#19-front-running-mev)
		- [20. Oracle Manipulation](#20-oracle-manipulation)
		- [21. Denial of Service (DoS)](#21-denial-of-service-dos)
		- [22. Signature Replay Attacks](#22-signature-replay-attacks)
		- [23. Unchecked External Calls](#23-unchecked-external-calls)
		- [24. Delegatecall Vulnerabilities](#24-delegatecall-vulnerabilities)
	- [Deployment](#deployment)
		- [25. Foundry Deployment](#25-foundry-deployment)
		- [26. Hardhat Deployment](#26-hardhat-deployment)
	- [Gas Optimization Tips](#gas-optimization-tips)
		- [27. Common Optimizations](#27-common-optimizations)
	- [Quick Reference](#quick-reference)
		- [Security Checklist](#security-checklist)
	- [Resources](#resources)

## Overview

- Purpose: Quick reference for Solidity smart contract development, testing frameworks, and security best practices
- Audience: Intermediate to Advanced (assumes basic programming knowledge)
- Scope: Solidity 0.8.x syntax, Foundry and Hardhat workflows, common patterns, security vulnerabilities, testing strategies. Excludes Solidity < 0.8 (has built-in overflow protection now), non-EVM chains
- Updated: 2025-11-09

---

## Development Environment Setup

### 1. Foundry Setup

Short explanation:

- What it is: Blazing fast Ethereum development framework written in Rust
- Why it matters: Faster tests (10-100x vs Hardhat), better debugging, native Solidity testing (no JavaScript)
- Mental model: Like Cargo for Rust—all-in-one toolkit for smart contracts

**Installation:**

```shell
# Install Foundry
curl -L https://foundry.paradigm.xyz | bash
foundryup

# Verify installation
forge --version
cast --version
anvil --version
```

**Create new project:**

```shell
# Create project
forge init my-project
cd my-project

# Project structure
# src/          - Smart contracts
# test/         - Solidity tests
# script/       - Deployment scripts
# lib/          - Dependencies (git submodules)
# foundry.toml  - Configuration
```

**Install dependencies:**

```shell
# Install OpenZeppelin contracts
forge install OpenZeppelin/openzeppelin-contracts

# Install with remapping
forge install OpenZeppelin/openzeppelin-contracts --no-commit
forge remappings > remappings.txt
```

**foundry.toml configuration:**

```toml
[profile.default]
src = "src"
out = "out"
libs = ["lib"]
solc_version = "0.8.20"
optimizer = true
optimizer_runs = 200
via_ir = false

[profile.ci]
fuzz = { runs = 10000 }
invariant = { runs = 1000 }

[rpc_endpoints]
mainnet = "${MAINNET_RPC_URL}"
sepolia = "${SEPOLIA_RPC_URL}"
```

**Common Forge commands:**

```shell
# Build contracts
forge build

# Run tests
forge test
forge test -vvvv                    # Verbose (show stack traces)
forge test --match-test testFuzz   # Run specific test
forge test --match-contract Counter # Run tests in specific contract

# Gas reports
forge test --gas-report

# Coverage
forge coverage

# Format code
forge fmt

# Generate documentation
forge doc
```

---

### 2. Hardhat Setup

Short explanation:

- What it is: JavaScript/TypeScript Ethereum development environment
- Why it matters: Mature ecosystem, excellent plugins, TypeScript support, better for complex scripting
- Mental model: Like npm scripts on steroids—task runner with built-in EVM

**Installation:**

```shell
# Create project
mkdir my-project && cd my-project
npm init -y

# Install Hardhat
npm install --save-dev hardhat

# Initialize project
npx hardhat init
# Choose: Create a TypeScript project
```

**Project structure:**

```txt
contracts/       - Smart contracts
test/            - TypeScript/JavaScript tests
scripts/         - Deployment scripts
hardhat.config.ts - Configuration
artifacts/       - Compiled contracts (auto-generated)
cache/           - Build cache
```

**hardhat.config.ts:**

```ts
import { HardhatUserConfig } from "hardhat/config";
import "@nomicfoundation/hardhat-toolbox";
import "dotenv/config";

const config: HardhatUserConfig = {
  solidity: {
    version: "0.8.20",
    settings: {
      optimizer: {
        enabled: true,
        runs: 200,
      },
    },
  },
  networks: {
    hardhat: {
      chainId: 31337,
    },
    sepolia: {
      url: process.env.SEPOLIA_RPC_URL || "",
      accounts: process.env.PRIVATE_KEY ? [process.env.PRIVATE_KEY] : [],
    },
    mainnet: {
      url: process.env.MAINNET_RPC_URL || "",
      accounts: process.env.PRIVATE_KEY ? [process.env.PRIVATE_KEY] : [],
    },
  },
  etherscan: {
    apiKey: process.env.ETHERSCAN_API_KEY,
  },
};

export default config;
```

**Common Hardhat commands:**

```shell
# Compile contracts
npx hardhat compile

# Run tests
npx hardhat test
npx hardhat test --grep "transfer"  # Run specific test

# Coverage
npx hardhat coverage

# Start local node
npx hardhat node

# Deploy
npx hardhat run scripts/deploy.ts --network sepolia

# Verify on Etherscan
npx hardhat verify --network sepolia <CONTRACT_ADDRESS> <CONSTRUCTOR_ARGS>

# Console (REPL)
npx hardhat console --network localhost
```

**Essential plugins:**

```shell
npm install --save-dev @nomicfoundation/hardhat-toolbox
# Includes: hardhat-ethers, hardhat-chai-matchers, hardhat-gas-reporter,
#           hardhat-network-helpers, hardhat-verify, typechain, coverage
```

---

## Solidity Language Fundamentals

### 3. Basic Syntax

Short explanation:

- What it is: Solidity is a statically-typed language for writing smart contracts
- Mental model: Like JavaScript + TypeScript + Rust—C-style syntax with strong typing

**Contract structure:**

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

/// @title My Token
/// @notice This is a simple ERC20 token
contract MyToken is ERC20 {
    // State variables
    address public owner;
    uint256 public totalMinted;

    // Events
    event Minted(address indexed to, uint256 amount);

    // Errors (gas-efficient)
    error NotOwner();
    error InvalidAmount();

    // Modifiers
    modifier onlyOwner() {
        if (msg.sender != owner) revert NotOwner();
        _;
    }

    // Constructor
    constructor() ERC20("MyToken", "MTK") {
        owner = msg.sender;
    }

    // Functions
    function mint(address to, uint256 amount) external onlyOwner {
        if (amount == 0) revert InvalidAmount();
        _mint(to, amount);
        totalMinted += amount;
        emit Minted(to, amount);
    }
}
```

**Data types:**

```solidity
// Value types
bool public isActive = true;
uint256 public count = 0;          // uint = uint256
int256 public balance = -100;
address public owner;
bytes32 public hash;

// Fixed-size arrays
uint256[5] public fixedArray;
address[10] public addresses;

// Dynamic arrays
uint256[] public dynamicArray;
address[] public users;

// Mappings
mapping(address => uint256) public balances;
mapping(address => mapping(address => uint256)) public allowances;

// Structs
struct User {
    string name;
    uint256 balance;
    bool isActive;
}
mapping(address => User) public users;

// Enums
enum Status { Pending, Active, Completed, Cancelled }
Status public currentStatus;
```

---

### 4. Functions & Visibility

Short explanation:

- What it is: Functions are the executable units of code in smart contracts
- Why it matters: Visibility controls who can call functions; state mutability controls gas costs
- Mental model: Like API endpoints—some are public, some are private, some read-only

**Function visibility:**

```solidity
contract Visibility {
    // public: callable from anywhere, auto-generates getter
    function publicFunc() public pure returns (string memory) {
        return "public";
    }

    // external: only callable from outside (not from within contract)
    function externalFunc() external pure returns (string memory) {
        return "external";
    }

    // internal: only callable from within this contract and derived contracts
    function internalFunc() internal pure returns (string memory) {
        return "internal";
    }

    // private: only callable from within this contract
    function privateFunc() private pure returns (string memory) {
        return "private";
    }
}
```

**State mutability:**

```solidity
contract StateMutability {
    uint256 public count;

    // view: reads state but doesn't modify (no gas when called externally)
    function getCount() public view returns (uint256) {
        return count;
    }

    // pure: doesn't read or modify state (no gas when called externally)
    function add(uint256 a, uint256 b) public pure returns (uint256) {
        return a + b;
    }

    // payable: can receive ETH
    function deposit() public payable {
        // msg.value is the ETH sent
    }

    // (default): can modify state
    function increment() public {
        count++;
    }
}
```

---

### 5. Events & Errors

Short explanation:

- What it is: Events are logs stored on blockchain; Errors are gas-efficient failure messages
- Why it matters: Events enable off-chain indexing; Custom errors save gas vs require strings
- Mental model: Events = append-only log file; Errors = early exit with reason

**Events (best practices):**

```solidity
contract EventExamples {
    // Indexed parameters (up to 3) enable filtering
    event Transfer(address indexed from, address indexed to, uint256 amount);
    event Approval(address indexed owner, address indexed spender, uint256 value);

    function transfer(address to, uint256 amount) public {
        // ... transfer logic
        emit Transfer(msg.sender, to, amount);
    }
}
```

**Custom errors (gas-efficient):**

```solidity
contract ErrorExamples {
    // Custom errors (cheaper than require strings)
    error InsufficientBalance(uint256 available, uint256 required);
    error Unauthorized(address caller);
    error InvalidAddress();

    mapping(address => uint256) public balances;

    function withdraw(uint256 amount) public {
        uint256 balance = balances[msg.sender];
        if (balance < amount) {
            revert InsufficientBalance(balance, amount);
        }
        balances[msg.sender] -= amount;
        payable(msg.sender).transfer(amount);
    }

    // Old way (more expensive)
    function oldWithdraw(uint256 amount) public {
        require(balances[msg.sender] >= amount, "Insufficient balance");
        // ...
    }
}
```

---

### 6. Modifiers

Short explanation:

- What it is: Reusable code that runs before/after function execution
- Why it matters: DRY principle for access control and validation
- Mental model: Like middleware in Express—runs before the main function

```solidity
contract ModifierExamples {
    address public owner;
    bool public paused;

    constructor() {
        owner = msg.sender;
    }

    // Access control modifier
    modifier onlyOwner() {
        require(msg.sender == owner, "Not owner");
        _;  // _ is where the function body executes
    }

    // State check modifier
    modifier whenNotPaused() {
        require(!paused, "Contract is paused");
        _;
    }

    // Input validation modifier
    modifier validAddress(address _addr) {
        require(_addr != address(0), "Invalid address");
        _;
    }

    // Multiple modifiers (execute left to right)
    function mint(address to, uint256 amount)
        public
        onlyOwner
        whenNotPaused
        validAddress(to)
    {
        // Function body
    }

    function pause() public onlyOwner {
        paused = true;
    }
}
```

---

### 7. Inheritance & Interfaces

Short explanation:

- What it is: Contracts can inherit from other contracts; Interfaces define contract APIs
- Why it matters: Code reuse, standardization (ERC20, ERC721), modularity
- Mental model: Like OOP inheritance—child contracts inherit parent functionality

**Inheritance:**

```solidity
// Base contract
contract Ownable {
    address public owner;

    constructor() {
        owner = msg.sender;
    }

    modifier onlyOwner() {
        require(msg.sender == owner, "Not owner");
        _;
    }

    function transferOwnership(address newOwner) public virtual onlyOwner {
        owner = newOwner;
    }
}

// Derived contract
contract MyContract is Ownable {
    uint256 public value;

    // Override parent function
    function transferOwnership(address newOwner) public override onlyOwner {
        require(newOwner != address(0), "Invalid address");
        super.transferOwnership(newOwner);  // Call parent implementation
    }

    function setValue(uint256 _value) public onlyOwner {
        value = _value;
    }
}

// Multiple inheritance (left to right)
contract MultiInherit is Ownable, ReentrancyGuard {
    // Inherits from both contracts
}
```

**Interfaces:**

```solidity
// Interface definition
interface IERC20 {
    function totalSupply() external view returns (uint256);
    function balanceOf(address account) external view returns (uint256);
    function transfer(address to, uint256 amount) external returns (bool);

    event Transfer(address indexed from, address indexed to, uint256 value);
}

// Using interface
contract TokenSwap {
    function swapTokens(address tokenAddress, uint256 amount) public {
        IERC20 token = IERC20(tokenAddress);
        require(token.balanceOf(msg.sender) >= amount, "Insufficient balance");
        token.transfer(address(this), amount);
    }
}
```

---

## Common Patterns

### 8. Access Control Patterns

Short explanation:

- What it is: Restrict who can call certain functions
- Why it matters: Prevent unauthorized users from executing privileged operations
- Mental model: Like permissions in an operating system—admin, user, guest roles

```solidity
import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/access/AccessControl.sol";

// Simple owner pattern
contract SimpleOwner is Ownable {
    function privilegedFunction() public onlyOwner {
        // Only owner can call
    }
}

// Role-based access control (RBAC)
contract RoleBasedAccess is AccessControl {
    bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
    bytes32 public constant BURNER_ROLE = keccak256("BURNER_ROLE");

    constructor() {
        _grantRole(DEFAULT_ADMIN_ROLE, msg.sender);
        _grantRole(MINTER_ROLE, msg.sender);
    }

    function mint(address to, uint256 amount) public onlyRole(MINTER_ROLE) {
        // Only minters can mint
    }

    function burn(uint256 amount) public onlyRole(BURNER_ROLE) {
        // Only burners can burn
    }
}
```

---

### 9. Checks-Effects-Interactions Pattern

Short explanation:

- What it is: Design pattern to prevent reentrancy attacks
- Why it matters: Most common vulnerability in smart contracts (see DAO hack)
- Mental model: 1) Check conditions, 2) Update state, 3) Interact with external contracts (in that order)

```solidity
contract SafeWithdrawal {
    mapping(address => uint256) public balances;

    function withdraw(uint256 amount) public {
        // 1. CHECKS: Validate conditions
        require(balances[msg.sender] >= amount, "Insufficient balance");

        // 2. EFFECTS: Update state BEFORE external call
        balances[msg.sender] -= amount;

        // 3. INTERACTIONS: External calls last
        (bool success, ) = msg.sender.call{value: amount}("");
        require(success, "Transfer failed");
    }
}

// Alternative: Use ReentrancyGuard
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";

contract SafeWithdrawal2 is ReentrancyGuard {
    mapping(address => uint256) public balances;

    function withdraw(uint256 amount) public nonReentrant {
        require(balances[msg.sender] >= amount, "Insufficient balance");
        balances[msg.sender] -= amount;
        (bool success, ) = msg.sender.call{value: amount}("");
        require(success, "Transfer failed");
    }
}
```

---

### 10. Pull Over Push Pattern

Short explanation:

- What it is: Let users withdraw funds themselves instead of pushing funds to them
- Why it matters: Prevents failures from blocking entire system; prevents griefing attacks
- Mental model: ATM (pull) vs mailing checks (push)—users withdraw when they want

```solidity
// BAD: Push pattern (vulnerable to griefing)
contract PushPayment {
    address[] public winners;

    function distributeRewards() public {
        for (uint256 i = 0; i < winners.length; i++) {
            // If one transfer fails, entire function fails
            payable(winners[i]).transfer(1 ether);
        }
    }
}

// GOOD: Pull pattern (griefing-resistant)
contract PullPayment {
    mapping(address => uint256) public pendingWithdrawals;

    function creditWinners(address[] calldata winners) public {
        for (uint256 i = 0; i < winners.length; i++) {
            pendingWithdrawals[winners[i]] += 1 ether;
        }
    }

    function withdraw() public {
        uint256 amount = pendingWithdrawals[msg.sender];
        require(amount > 0, "No funds");
        pendingWithdrawals[msg.sender] = 0;
        (bool success, ) = msg.sender.call{value: amount}("");
        require(success, "Transfer failed");
    }
}
```

---

### 11. Factory Pattern

Short explanation:

- What it is: Contract that deploys other contracts
- Why it matters: Standardized deployment, tracking deployed contracts, upgradeable patterns
- Mental model: Like a production line—factory creates identical products (contracts)

```solidity
contract Token {
    string public name;
    address public owner;

    constructor(string memory _name, address _owner) {
        name = _name;
        owner = _owner;
    }
}

contract TokenFactory {
    Token[] public deployedTokens;

    event TokenCreated(address indexed tokenAddress, string name, address indexed owner);

    function createToken(string memory name) public returns (address) {
        Token newToken = new Token(name, msg.sender);
        deployedTokens.push(newToken);
        emit TokenCreated(address(newToken), name, msg.sender);
        return address(newToken);
    }

    function getDeployedTokens() public view returns (Token[] memory) {
        return deployedTokens;
    }
}
```

---

## Testing with Foundry

### 12. Foundry Test Basics

Short explanation:

- What it is: Write tests in Solidity (not JavaScript)
- Why it matters: Faster execution, no context switching between languages
- Mental model: Like Go tests—test code in the same language as implementation

**Basic test structure:**

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "forge-std/Test.sol";
import "../src/Counter.sol";

contract CounterTest is Test {
    Counter public counter;

    // Setup runs before each test
    function setUp() public {
        counter = new Counter();
    }

    // Test functions must start with "test"
    function testIncrement() public {
        counter.increment();
        assertEq(counter.count(), 1);
    }

    function testDecrement() public {
        counter.increment();
        counter.increment();
        counter.decrement();
        assertEq(counter.count(), 1);
    }

    // Test expected reverts
    function testFailDecrementBelowZero() public {
        counter.decrement();  // Should revert
    }

    // Alternative: expectRevert
    function testDecrementBelowZeroReverts() public {
        vm.expectRevert();
        counter.decrement();
    }
}
```

**Cheatcodes (vm):**

```solidity
contract CheatcodeTest is Test {
    function testPrank() public {
        address alice = address(0x1);

        // Next call comes from alice
        vm.prank(alice);
        // ... call function as alice

        // All subsequent calls come from alice (until stopPrank)
        vm.startPrank(alice);
        // ... multiple calls as alice
        vm.stopPrank();
    }

    function testDeal() public {
        address alice = address(0x1);

        // Give alice 100 ETH
        vm.deal(alice, 100 ether);
        assertEq(alice.balance, 100 ether);
    }

    function testWarp() public {
        // Fast forward time to timestamp
        vm.warp(block.timestamp + 1 days);

        // Advance block number
        vm.roll(block.number + 100);
    }

    function testExpectEmit() public {
        // Check event emission
        vm.expectEmit(true, true, false, true);
        emit Transfer(address(this), alice, 100);
        token.transfer(alice, 100);
    }
}
```

---

### 13. Fuzz Testing

Short explanation:

- What it is: Foundry automatically generates random inputs to test edge cases
- Why it matters: Finds bugs you didn't think to test for
- Mental model: Like a fuzzer for normal software—throws random inputs to find crashes

```solidity
contract FuzzTest is Test {
    // Fuzz test: Foundry generates random uint256 values
    function testFuzzAdd(uint256 a, uint256 b) public {
        // Assume: filter out invalid inputs
        vm.assume(a <= type(uint256).max - b);  // Prevent overflow

        uint256 result = a + b;
        assertGe(result, a);
        assertGe(result, b);
    }

    // Fuzz with multiple parameters
    function testFuzzTransfer(address to, uint256 amount) public {
        vm.assume(to != address(0));
        vm.assume(amount <= 1000 ether);

        // Test transfer logic
    }

    // Bounded fuzz: restrict input range
    function testBoundedFuzz(uint256 amount) public {
        amount = bound(amount, 1, 1000);  // amount between 1 and 1000
        // Test with bounded amount
    }
}
```

**Run fuzz tests:**

```shell
# Default: 256 runs
forge test

# More runs (better coverage)
forge test --fuzz-runs 10000
```

---

### 14. Invariant Testing

Short explanation:

- What it is: Define properties that should ALWAYS be true, Foundry tries to break them
- Why it matters: Catches complex bugs in state transitions
- Mental model: Like property-based testing—define rules that should never be violated

```solidity
contract TokenInvariantTest is Test {
    Token public token;
    address[] public users;

    function setUp() public {
        token = new Token();

        // Setup multiple users
        for (uint160 i = 1; i <= 10; i++) {
            users.push(address(i));
            vm.deal(address(i), 100 ether);
        }

        // Target contract for invariant testing
        targetContract(address(token));
    }

    // Invariant: total supply = sum of all balances
    function invariant_totalSupplyEqualsBalances() public {
        uint256 sumBalances;
        for (uint256 i = 0; i < users.length; i++) {
            sumBalances += token.balanceOf(users[i]);
        }
        assertEq(token.totalSupply(), sumBalances);
    }

    // Invariant: balance never exceeds total supply
    function invariant_balanceNeverExceedsTotalSupply() public {
        for (uint256 i = 0; i < users.length; i++) {
            assertLe(token.balanceOf(users[i]), token.totalSupply());
        }
    }
}
```

---

## Testing with Hardhat

### 15. Hardhat Test Basics

Short explanation:

- What it is: Write tests in TypeScript/JavaScript using Mocha + Chai
- Why it matters: Better for complex scenarios, async testing, mocking
- Mental model: Like Jest for smart contracts

**Basic test structure:**

```ts
import { expect } from "chai";
import { ethers } from "hardhat";
import { Contract } from "ethers";
import { SignerWithAddress } from "@nomicfoundation/hardhat-ethers/signers";

describe("Counter", function () {
  let counter: Contract;
  let owner: SignerWithAddress;
  let addr1: SignerWithAddress;

  beforeEach(async function () {
    // Get signers
    [owner, addr1] = await ethers.getSigners();

    // Deploy contract
    const Counter = await ethers.getContractFactory("Counter");
    counter = await Counter.deploy();
    await counter.waitForDeployment();
  });

  it("Should increment counter", async function () {
    await counter.increment();
    expect(await counter.count()).to.equal(1);
  });

  it("Should revert on decrement below zero", async function () {
    await expect(counter.decrement()).to.be.reverted;
  });

  it("Should emit event on increment", async function () {
    await expect(counter.increment())
      .to.emit(counter, "Incremented")
      .withArgs(1);
  });
});
```

**Advanced matchers:**

```ts
describe("Token", function () {
  it("Should transfer tokens", async function () {
    // Test balance change
    await expect(token.transfer(addr1.address, 100))
      .to.changeTokenBalances(token, [owner, addr1], [-100, 100]);

    // Test ETH balance change
    await expect(contract.withdraw())
      .to.changeEtherBalance(owner, ethers.parseEther("1.0"));

    // Test revert with specific error
    await expect(token.transfer(addr1.address, 1000))
      .to.be.revertedWithCustomError(token, "InsufficientBalance");

    // Test revert with error parameters
    await expect(token.transfer(addr1.address, 1000))
      .to.be.revertedWithCustomError(token, "InsufficientBalance")
      .withArgs(100, 1000);
  });
});
```

**Time manipulation:**

```ts
import { time } from "@nomicfoundation/hardhat-network-helpers";

describe("Timelock", function () {
  it("Should unlock after time passes", async function () {
    const unlockTime = (await time.latest()) + 86400; // +1 day

    // Fast forward time
    await time.increaseTo(unlockTime);

    await expect(contract.withdraw()).to.not.be.reverted;
  });
});
```

---

## Security: Threats & Vulnerabilities

### 16. Reentrancy Attacks

Short explanation:

- What it is: Attacker calls back into your contract before first call finishes
- Why it matters: The DAO hack ($60M stolen in 2016)
- Mental model: Like a recursive phone call—before you hang up, they call you back and exploit state

**Vulnerable contract:**

```solidity
// VULNERABLE: Reentrancy attack
contract VulnerableBank {
    mapping(address => uint256) public balances;

    function withdraw(uint256 amount) public {
        require(balances[msg.sender] >= amount, "Insufficient balance");

        // DANGER: External call before state update
        (bool success, ) = msg.sender.call{value: amount}("");
        require(success);

        balances[msg.sender] -= amount;  // State updated too late!
    }
}

// ATTACKER CONTRACT
contract Attacker {
    VulnerableBank public bank;

    constructor(address _bank) {
        bank = VulnerableBank(_bank);
    }

    // Receive ETH and call withdraw again (reentrancy!)
    receive() external payable {
        if (address(bank).balance >= 1 ether) {
            bank.withdraw(1 ether);
        }
    }

    function attack() public payable {
        bank.withdraw(1 ether);  // Starts the reentrancy loop
    }
}
```

**Fixes:**

```solidity
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";

// FIX 1: Checks-Effects-Interactions pattern
contract SafeBank {
    mapping(address => uint256) public balances;

    function withdraw(uint256 amount) public {
        require(balances[msg.sender] >= amount);

        // Update state BEFORE external call
        balances[msg.sender] -= amount;

        (bool success, ) = msg.sender.call{value: amount}("");
        require(success);
    }
}

// FIX 2: ReentrancyGuard
contract SafeBank2 is ReentrancyGuard {
    mapping(address => uint256) public balances;

    function withdraw(uint256 amount) public nonReentrant {
        require(balances[msg.sender] >= amount);
        balances[msg.sender] -= amount;
        (bool success, ) = msg.sender.call{value: amount}("");
        require(success);
    }
}
```

---

### 17. Access Control Vulnerabilities

Short explanation:

- What it is: Missing or incorrect access control lets anyone call privileged functions
- Why it matters: Attackers can mint tokens, drain funds, self-destruct contracts
- Mental model: Like leaving your admin panel public—anyone can become admin

**Vulnerable:**

```solidity
// VULNERABLE: No access control
contract VulnerableToken {
    mapping(address => uint256) public balances;

    // Anyone can mint!
    function mint(address to, uint256 amount) public {
        balances[to] += amount;
    }
}

// VULNERABLE: Wrong visibility
contract VulnerableContract {
    address public owner;

    // Should be external, but it's public (can be called internally)
    function initialize(address _owner) public {
        owner = _owner;  // Attacker can call this repeatedly!
    }
}
```

**Fixes:**

```solidity
import "@openzeppelin/contracts/access/Ownable.sol";

// FIX: Proper access control
contract SafeToken is Ownable {
    mapping(address => uint256) public balances;

    function mint(address to, uint256 amount) public onlyOwner {
        balances[to] += amount;
    }
}

// FIX: Initialization protection
contract SafeContract {
    address public owner;
    bool private initialized;

    function initialize(address _owner) external {
        require(!initialized, "Already initialized");
        owner = _owner;
        initialized = true;
    }
}
```

---

### 18. Integer Overflow/Underflow

Short explanation:

- What it is: Numbers wrap around when exceeding max/min values
- Why it matters: Solidity 0.8+ has built-in protection, but still possible with `unchecked`
- Mental model: Like an odometer rolling over—99999 + 1 = 00000

```solidity
// Solidity < 0.8: VULNERABLE
contract OldVulnerable {
    uint8 public count = 255;

    function increment() public {
        count++;  // Wraps to 0 in Solidity < 0.8
    }
}

// Solidity 0.8+: SAFE by default
contract SafeByDefault {
    uint8 public count = 255;

    function increment() public {
        count++;  // Reverts on overflow in 0.8+
    }
}

// Using unchecked (still vulnerable)
contract StillVulnerable {
    uint256 public count;

    function increment() public {
        unchecked {
            count++;  // Can overflow! Use carefully
        }
    }
}

// SAFE: Use SafeMath (for Solidity < 0.8)
import "@openzeppelin/contracts/utils/math/SafeMath.sol";

contract SafeOldVersion {
    using SafeMath for uint256;
    uint256 public count;

    function increment() public {
        count = count.add(1);  // Reverts on overflow
    }
}
```

---

### 19. Front-Running (MEV)

Short explanation:

- What it is: Bots see your transaction in mempool and submit a higher gas price transaction first
- Why it matters: Sandwich attacks on DEXs, frontrunning NFT mints
- Mental model: Like jumping ahead in line by paying more—miner picks highest bidder first

**Vulnerable:**

```solidity
// VULNERABLE: Price can be frontrun
contract VulnerableDEX {
    function swap(uint256 amountIn) public {
        uint256 amountOut = getPrice();  // Price at time of execution
        // Attacker sees your tx, submits higher gas to manipulate price
    }
}
```

**Mitigations:**

```solidity
// MITIGATION 1: Slippage protection
contract SafeDEX {
    function swap(uint256 amountIn, uint256 minAmountOut) public {
        uint256 amountOut = getPrice();
        require(amountOut >= minAmountOut, "Slippage too high");
        // User sets acceptable min output
    }
}

// MITIGATION 2: Commit-Reveal scheme
contract CommitReveal {
    mapping(address => bytes32) public commits;

    // Step 1: Commit to action (hash of secret + action)
    function commit(bytes32 hash) public {
        commits[msg.sender] = hash;
    }

    // Step 2: Reveal action after delay
    function reveal(string memory secret, uint256 action) public {
        require(
            commits[msg.sender] == keccak256(abi.encodePacked(secret, action)),
            "Invalid reveal"
        );
        // Execute action
    }
}

// MITIGATION 3: Flashbots (private mempool)
// Submit transactions directly to miners, bypassing public mempool
```

---

### 20. Oracle Manipulation

Short explanation:

- What it is: Attacker manipulates price feeds to exploit protocols
- Why it matters: DeFi protocols rely on oracles for prices—bad data = bad outcomes
- Mental model: Like rigging a thermometer to trigger insurance payout

**Vulnerable:**

```solidity
// VULNERABLE: Single DEX as price source
contract VulnerableLending {
    function getPrice() public view returns (uint256) {
        // Gets price from single Uniswap pool (can be manipulated!)
        return uniswapPool.getPrice();
    }

    function borrow(uint256 amount) public {
        uint256 collateralValue = getPrice() * userCollateral;
        require(collateralValue >= amount * 1.5, "Insufficient collateral");
        // Attacker manipulates pool price, borrows more than they should
    }
}
```

**Mitigations:**

```solidity
import "@chainlink/contracts/src/v0.8/interfaces/AggregatorV3Interface.sol";

// FIX 1: Use Chainlink price feeds (decentralized oracles)
contract SafeLending {
    AggregatorV3Interface public priceFeed;

    constructor(address _priceFeed) {
        priceFeed = AggregatorV3Interface(_priceFeed);
    }

    function getPrice() public view returns (uint256) {
        (, int256 price, , uint256 updatedAt, ) = priceFeed.latestRoundData();
        require(block.timestamp - updatedAt < 3600, "Stale price");
        require(price > 0, "Invalid price");
        return uint256(price);
    }
}

// FIX 2: TWAP (Time-Weighted Average Price)
contract TWAPOracle {
    uint256 public constant PERIOD = 1 hours;

    function getTWAP() public view returns (uint256) {
        // Average price over time period (harder to manipulate)
        return uniswapPool.getTWAP(PERIOD);
    }
}

// FIX 3: Multi-oracle with median
contract MultiOracle {
    address[] public oracles;

    function getMedianPrice() public view returns (uint256) {
        uint256[] memory prices = new uint256[](oracles.length);
        for (uint256 i = 0; i < oracles.length; i++) {
            prices[i] = IOracle(oracles[i]).getPrice();
        }
        return median(prices);
    }
}
```

---

### 21. Denial of Service (DoS)

Short explanation:

- What it is: Attacker makes contract unusable by all users
- Why it matters: Contract becomes permanently bricked
- Mental model: Like jamming a door so nobody can enter

**DoS patterns:**

```solidity
// VULNERABLE: DoS via revert
contract VulnerableAuction {
    address public highestBidder;
    uint256 public highestBid;

    function bid() public payable {
        require(msg.value > highestBid, "Bid too low");

        // Refund previous bidder
        payable(highestBidder).transfer(highestBid);  // Can fail!

        highestBidder = msg.sender;
        highestBid = msg.value;
    }
}

// ATTACK: Attacker makes bid() revert
contract Attacker {
    // Reject all ETH transfers
    receive() external payable {
        revert();  // Now nobody can outbid!
    }
}

// FIX: Pull over push pattern
contract SafeAuction {
    address public highestBidder;
    uint256 public highestBid;
    mapping(address => uint256) public pendingReturns;

    function bid() public payable {
        require(msg.value > highestBid);

        if (highestBidder != address(0)) {
            // Don't send, just track pending withdrawal
            pendingReturns[highestBidder] += highestBid;
        }

        highestBidder = msg.sender;
        highestBid = msg.value;
    }

    function withdraw() public {
        uint256 amount = pendingReturns[msg.sender];
        pendingReturns[msg.sender] = 0;
        payable(msg.sender).transfer(amount);
    }
}

// VULNERABLE: DoS via gas limits
contract VulnerableDistribution {
    address[] public recipients;

    function distribute() public {
        for (uint256 i = 0; i < recipients.length; i++) {
            payable(recipients[i]).transfer(1 ether);
        }
        // If recipients array is too large, runs out of gas!
    }
}

// FIX: Batch processing or pull pattern
contract SafeDistribution {
    mapping(address => uint256) public claimable;

    function claim() public {
        uint256 amount = claimable[msg.sender];
        claimable[msg.sender] = 0;
        payable(msg.sender).transfer(amount);
    }
}
```

---

### 22. Signature Replay Attacks

Short explanation:

- What it is: Attacker reuses a valid signature multiple times
- Why it matters: Can drain contracts that rely on signatures for auth
- Mental model: Like photocopying a signed check and cashing it multiple times

**Vulnerable:**

```solidity
// VULNERABLE: No nonce or expiry
contract VulnerablePermit {
    mapping(address => uint256) public balances;

    function withdraw(uint256 amount, bytes memory signature) public {
        bytes32 hash = keccak256(abi.encodePacked(msg.sender, amount));
        address signer = recoverSigner(hash, signature);

        require(signer == owner, "Invalid signature");
        balances[msg.sender] -= amount;
        // Attacker can reuse this signature forever!
    }
}
```

**Fixes:**

```solidity
// FIX: Nonce + Expiry + EIP-712
contract SafePermit {
    mapping(address => uint256) public nonces;

    function withdraw(
        uint256 amount,
        uint256 deadline,
        bytes memory signature
    ) public {
        require(block.timestamp <= deadline, "Signature expired");

        bytes32 structHash = keccak256(abi.encode(
            keccak256("Withdraw(address user,uint256 amount,uint256 nonce,uint256 deadline)"),
            msg.sender,
            amount,
            nonces[msg.sender]++,  // Increment nonce
            deadline
        ));

        bytes32 hash = keccak256(abi.encodePacked("\x19\x01", DOMAIN_SEPARATOR, structHash));
        address signer = recoverSigner(hash, signature);

        require(signer == owner, "Invalid signature");
        balances[msg.sender] -= amount;
    }
}
```

---

### 23. Unchecked External Calls

Short explanation:

- What it is: Not checking return values of external calls
- Why it matters: Silent failures can lead to state inconsistencies
- Mental model: Like assuming a network request succeeded without checking the status code

```solidity
// VULNERABLE: Unchecked call
contract Vulnerable {
    function sendEther(address to, uint256 amount) public {
        to.call{value: amount}("");  // Ignores return value!
        // If call fails, contract thinks it succeeded
    }
}

// FIX 1: Check return value
contract Safe1 {
    function sendEther(address to, uint256 amount) public {
        (bool success, ) = to.call{value: amount}("");
        require(success, "Transfer failed");
    }
}

// FIX 2: Use transfer (reverts on failure, but be careful with gas limits)
contract Safe2 {
    function sendEther(address to, uint256 amount) public {
        payable(to).transfer(amount);  // Reverts on failure
    }
}

// VULNERABLE: Unchecked token transfer
contract VulnerableToken {
    function transferTokens(address token, address to, uint256 amount) public {
        IERC20(token).transfer(to, amount);  // Some tokens return false instead of reverting!
    }
}

// FIX: SafeERC20
import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";

contract SafeToken {
    using SafeERC20 for IERC20;

    function transferTokens(address token, address to, uint256 amount) public {
        IERC20(token).safeTransfer(to, amount);  // Reverts on failure
    }
}
```

---

### 24. Delegatecall Vulnerabilities

Short explanation:

- What it is: `delegatecall` executes code in the context of calling contract
- Why it matters: Can overwrite storage slots, leading to complete contract takeover
- Mental model: Like running someone else's code with your permissions—dangerous if not careful

**Vulnerable:**

```solidity
// VULNERABLE: Unsafe delegatecall
contract Vulnerable {
    address public owner;  // Storage slot 0

    function callContract(address target, bytes memory data) public {
        // DANGER: target can modify this contract's storage!
        target.delegatecall(data);
    }
}

// ATTACKER CONTRACT
contract Attacker {
    address public owner;  // Also slot 0!

    function becomeOwner() public {
        owner = msg.sender;  // Overwrites Vulnerable.owner!
    }
}

// Attack: vulnerable.callContract(attackerAddress, abi.encodeWithSignature("becomeOwner()"))
```

**Fixes:**

```solidity
// FIX 1: Whitelist allowed contracts
contract Safe1 {
    address public owner;
    mapping(address => bool) public allowedContracts;

    function callContract(address target, bytes memory data) public {
        require(allowedContracts[target], "Contract not allowed");
        target.delegatecall(data);
    }
}

// FIX 2: Use library pattern (libraries are stateless)
library Math {
    function add(uint256 a, uint256 b) internal pure returns (uint256) {
        return a + b;
    }
}

contract Safe2 {
    using Math for uint256;

    function calculate(uint256 a, uint256 b) public pure returns (uint256) {
        return a.add(b);  // Safe delegatecall to library
    }
}

// FIX 3: Proxy pattern with storage gap
contract Proxy {
    address public implementation;

    fallback() external payable {
        address impl = implementation;
        assembly {
            calldatacopy(0, 0, calldatasize())
            let result := delegatecall(gas(), impl, 0, calldatasize(), 0, 0)
            returndatacopy(0, 0, returndatasize())
            switch result
            case 0 { revert(0, returndatasize()) }
            default { return(0, returndatasize()) }
        }
    }
}
```

---

## Deployment

### 25. Foundry Deployment

```shell
# Load environment variables
source .env

# Deploy to local network (anvil)
forge create src/MyContract.sol:MyContract --private-key $PRIVATE_KEY

# Deploy to testnet
forge create src/MyContract.sol:MyContract \
  --rpc-url $SEPOLIA_RPC_URL \
  --private-key $PRIVATE_KEY \
  --verify \
  --etherscan-api-key $ETHERSCAN_API_KEY

# Deploy with constructor arguments
forge create src/Token.sol:Token \
  --rpc-url $SEPOLIA_RPC_URL \
  --private-key $PRIVATE_KEY \
  --constructor-args "MyToken" "MTK"

# Deploy with script
forge script script/Deploy.s.sol:DeployScript \
  --rpc-url $SEPOLIA_RPC_URL \
  --private-key $PRIVATE_KEY \
  --broadcast \
  --verify
```

**Deployment script:**

```solidity
// script/Deploy.s.sol
pragma solidity ^0.8.20;

import "forge-std/Script.sol";
import "../src/MyContract.sol";

contract DeployScript is Script {
    function run() external {
        uint256 deployerPrivateKey = vm.envUint("PRIVATE_KEY");

        vm.startBroadcast(deployerPrivateKey);

        MyContract myContract = new MyContract();
        console.log("Deployed at:", address(myContract));

        vm.stopBroadcast();
    }
}
```

---

### 26. Hardhat Deployment

```ts
// scripts/deploy.ts
import { ethers } from "hardhat";

async function main() {
  const [deployer] = await ethers.getSigners();
  console.log("Deploying with:", deployer.address);

  const MyContract = await ethers.getContractFactory("MyContract");
  const myContract = await MyContract.deploy();
  await myContract.waitForDeployment();

  console.log("Deployed at:", await myContract.getAddress());
}

main().catch((error) => {
  console.error(error);
  process.exitCode = 1;
});
```

```shell
# Deploy to testnet
npx hardhat run scripts/deploy.ts --network sepolia

# Verify on Etherscan
npx hardhat verify --network sepolia <CONTRACT_ADDRESS> <CONSTRUCTOR_ARGS>
```

---

## Gas Optimization Tips

### 27. Common Optimizations

**1. Use `calldata` instead of `memory` for function parameters (external functions):**

```solidity
// Expensive
function process(uint256[] memory data) external {
    // Copies array to memory
}

// Cheaper
function process(uint256[] calldata data) external {
    // Reads directly from calldata
}
```

**2. Cache storage variables in memory:**

```solidity
// Expensive: Multiple SLOADs
function bad() public view returns (uint256) {
    return myStorageVar + myStorageVar + myStorageVar;
}

// Cheaper: One SLOAD
function good() public view returns (uint256) {
    uint256 cached = myStorageVar;
    return cached + cached + cached;
}
```

**3. Use `++i` instead of `i++` in loops:**

```solidity
// More expensive
for (uint256 i = 0; i < array.length; i++) { }

// Cheaper
for (uint256 i = 0; i < array.length; ++i) { }
```

**4. Use custom errors instead of require strings:**

```solidity
// Expensive
require(balance >= amount, "Insufficient balance");

// Cheaper
error InsufficientBalance();
if (balance < amount) revert InsufficientBalance();
```

**5. Pack storage variables:**

```solidity
// Expensive: 3 storage slots
contract Bad {
    uint128 a;  // Slot 0
    uint256 b;  // Slot 1
    uint128 c;  // Slot 2
}

// Cheaper: 2 storage slots
contract Good {
    uint128 a;  // Slot 0 (first 16 bytes)
    uint128 c;  // Slot 0 (last 16 bytes)
    uint256 b;  // Slot 1
}
```

---

## Quick Reference

### Security Checklist

Before deploying to mainnet:

- [ ] All external calls use checks-effects-interactions pattern
- [ ] ReentrancyGuard on functions with external calls
- [ ] Access control on privileged functions (onlyOwner, roles)
- [ ] Input validation (non-zero addresses, amount limits)
- [ ] Integer overflow protection (use Solidity 0.8+)
- [ ] No `tx.origin` for authentication
- [ ] Proper handling of external call failures
- [ ] Events for all state changes
- [ ] Slippage protection on DEX swaps
- [ ] Timelock on critical admin functions
- [ ] Pausable pattern for emergency stops
- [ ] Comprehensive test coverage (>90%)
- [ ] Fuzz testing and invariant testing
- [ ] External audit from reputable firm
- [ ] Bug bounty program
- [ ] Testnet deployment and monitoring

---

## Resources

- **Foundry:** [Foundry Book](https://book.getfoundry.sh/)
- **Hardhat:** [Hardhat Documentation](https://hardhat.org/docs)
- **Solidity:** [Solidity Documentation](https://docs.soliditylang.org/)
- **Security:** [Consensys Smart Contract Best Practices](https://consensys.github.io/smart-contract-best-practices/)
- **Vulnerabilities:** [SWC Registry](https://swcregistry.io/), [Rekt News](https://rekt.news/)
- **OpenZeppelin:** [OpenZeppelin Contracts](https://docs.openzeppelin.com/contracts)
- **Testing:** [Foundry Testing Tutorial](https://book.getfoundry.sh/forge/tests)
- **Gas Optimization:** [EVM Codes](https://www.evm.codes/), [Solidity Gas Optimization Tips](https://github.com/iskdrews/awesome-solidity-gas-optimization)
- **Auditing:** [Trail of Bits](https://www.trailofbits.com/), [OpenZeppelin Audits](https://www.openzeppelin.com/security-audits), [Consensys Diligence](https://consensys.net/diligence/)

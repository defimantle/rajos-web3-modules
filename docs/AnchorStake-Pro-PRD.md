# Product Requirements Document (PRD): AnchorStake Pro (Modular Yield & Asset Lockup Engine)

## 1. Objective & Scope
**Objective:** Develop a highly flexible, secure, and customizable on-chain asset staking protocol on Solana. This protocol enables third-party projects to easily deploy decentralized lockup pools for their ecosystem tokens or NFTs, rewarding long-term community retention with automated yield emissions.
**Target Audience:** Newly launched token ecosystems, NFT communities, and DAOs looking for a plug-and-play, audited staking contract without writing custom rust logic.

## 2. Product Architecture & Technology Stack
Due to the immutability of Web3 infrastructure, the stack focuses heavily on security, standardized frameworks, and reliable UI adapters.

### 2.1 Smart Contract Layer
- **Framework:** Anchor Framework (Rust) for secure, standardized Solana smart contract development.
- **Token Standard:** Solana Token Program & Token 2022 Extensions (supporting interest-bearing or transfer-fee tokens).
- **Testing:** Bankrun / Mocha + Chai for local validator simulation and edge-case testing.

### 2.2 Frontend & Integration
- **Framework:** Next.js 14+ (App Router) combined with Tailwind CSS for the staking dashboard.
- **Web3 Integration:** `@solana/web3.js` and `@coral-xyz/anchor` for contract interactions.
- **Wallet Connection:** Solana Wallet Adapter (supporting Phantom, Solflare, etc.).
- **Data Fetching:** React Query for reliable RPC polling and state management of on-chain data.

## 3. Core Functional Requirements
### 3.1 Multi-Tier Pool Factory
- **Initialization:** Allow project owners to dynamically instantiate staking pools.
- **Configuration:** Owners can specify lockup duration multipliers (e.g., 1x for 30 days, 1.5x for 90 days), penalty fees for early withdrawal, and total emission schedules.
- **Funding:** Secure mechanism to deposit reward tokens into a vault PDA (Program Derived Address).

### 3.2 Dynamic Reward Calculations
- **Time-Based Accrual:** Secure on-chain calculation of accumulated rewards relying on Solana's `Clock` sysvar (tracking block time intervals or slots).
- **Claiming:** Users can claim accumulated rewards without unstaking the principal, or configure the contract to auto-compound if applicable.

### 3.3 Emergency Unstaking Penalties
- **Early Withdrawal:** Users can bypass lockup periods in emergencies via an `early_unstake` function.
- **Penalty Enforcement:** A heavy percentage fee (configured at pool initialization) is slashed from the principal amount and returned to the pool ecosystem or a designated treasury vault.

## 4. Security & Safety Requirements (Critical)
- **PDA Constraints:** User stake accounts and vault reserves must strictly utilize deterministic PDAs tied to the user's wallet public key and the pool ID to eliminate account hijacking.
- **Drift Variance:** Clock-based rewards must account for Solana slot drift variance, relying on `Clock.unix_timestamp` rather than raw slots for consistent time measurement.
- **Overflow Protection:** Strict checked math (using `checked_add`, `checked_mul`, etc.) across all reward formulas to prevent integer overflow exploits.
- **Vault Integrity:** Hard assertions ensuring reward pool reserves cannot be drawn past their initialization bounds under any circumstance.

## 5. Account Structures (High Level)
- **Pool State Account:** Stores Authority Pubkey, Reward Mint, Vault PDA, Lockup Durations, Multipliers, Penalty Percentage, and Total Staked.
- **User Stake Account:** PDA derived from `[user_pubkey, pool_pubkey]`. Stores Staked Amount, Lockup End Time, Accumulated Rewards, and Last Update Timestamp.
- **Vault Account:** Token Account owned by the Pool PDA containing the locked principal and reward reserves.

## 6. Acceptance Criteria (Vetting Benchmarks)
- [ ] Clock-based calculation logic passes rigorous edge-case tests simulating extreme network time drift.
- [ ] Unstake instructions revert explicitly if the lockup duration hasn't passed and the normal `unstake` function is called.
- [ ] The `early_unstake` function properly executes the mathematical penalty and routes the fee to the correct treasury.
- [ ] The Next.js frontend seamlessly handles wallet connection, accurately displays projected yield, and signs complex transactions smoothly.

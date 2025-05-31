# Blueshift Anchor Vault

A simple Solana program for managing SOL deposits and withdrawals using program-derived addresses (PDAs).

## Overview

This vault program allows users to deposit SOL into a secure program-controlled account and withdraw it later. Each user gets their own unique vault derived from their wallet address, ensuring isolation and security.

## Features

- **Deposit SOL**: Transfer SOL from your wallet to a program-derived vault
- **Withdraw SOL**: Retrieve all SOL from your vault back to your wallet
- **PDA Security**: Vaults are controlled by the program using cryptographic derivation
- **User Isolation**: Each wallet gets its own unique vault address

## Program Structure

### Instructions

- `deposit(amount: u64)` - Deposits the specified amount of SOL into the user's vault
- `withdraw()` - Withdraws all SOL from the user's vault back to the user

### Account Structure

```rust
#[derive(Accounts)]
pub struct VaultAction<'info> {
    #[account(mut)]
    pub signer: Signer<'info>,
    
    #[account(
        mut,
        seeds = [b"vault", signer.key().as_ref()],
        bump,
    )]
    pub vault: SystemAccount<'info>,
    
    pub system_program: Program<'info, System>,
}
```

## Build & Deploy

### Prerequisites

- [Rust](https://rustup.rs/)
- [Solana CLI](https://docs.solana.com/cli/install-solana-cli-tools)
- [Anchor Framework](https://www.anchor-lang.com/docs/installation)

### Building

```bash
# Build the program
cargo build-sbf --manifest-path programs/blueshift_anchor_vault/Cargo.toml

# Or use Anchor (if you have compatible versions)
anchor build --skip-lint
```

### Testing

```bash
anchor test
```

### Deployment

```bash
# Deploy to devnet
anchor deploy --provider.cluster devnet

# Deploy to mainnet-beta
anchor deploy --provider.cluster mainnet-beta
```

## Usage Example

```typescript
import * as anchor from "@coral-xyz/anchor";

const program = anchor.workspace.BlueshiftAnchorVault;
const provider = anchor.AnchorProvider.env();

// Deposit 1 SOL
await program.methods
  .deposit(new anchor.BN(1000000000)) // 1 SOL in lamports
  .accounts({
    signer: provider.wallet.publicKey,
    systemProgram: anchor.web3.SystemProgram.programId,
  })
  .rpc();

// Withdraw all SOL
await program.methods
  .withdraw()
  .accounts({
    signer: provider.wallet.publicKey,
    systemProgram: anchor.web3.SystemProgram.programId,
  })
  .rpc();
```

## Security Considerations

- Vaults are derived using `[b"vault", user_pubkey]` seeds
- Only the vault owner can withdraw funds
- Deposit requires minimum rent exemption amount
- Program uses Anchor's built-in security checks

## Program ID

**Devnet/Localnet**: `22222222222222222222222222222222222222222222`

## Error Codes

- `VaultAlreadyExists` - Attempted to deposit into an already funded vault
- `InvalidAmount` - Deposit amount too small (must cover rent exemption)

## License

This project is licensed under the terms specified in the LICENSE file. 
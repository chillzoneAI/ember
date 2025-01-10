# EMBER

A decentralized digital time capsule protocol built on Solana, enabling secure, time-locked data storage and inheritance transfer through smart contracts.

## Protocol Overview

EMBER implements a secure data storage and time-based transfer system using Solana's programmable runtime environment. The protocol leverages threshold encryption and temporal checkpoints to manage access control and automatic transfer of digital assets.

### Core Components

- **Program ID**: `Check Twitter for updates.`
- **Runtime**: Solana (1.17.0)
- **Language**: Rust
- **Encryption**: AES-256-GCM with threshold key sharing

## Architecture

### Smart Contract Structure

```rust
pub struct Capsule {
    pub authority: Pubkey,
    pub recipients: Vec<Recipient>,
    pub metadata: CapsuleMetadata,
    pub time_lock: i64,
    pub encryption_key: [u8; 32],
    pub data_hash: [u8; 32],
}

pub struct Recipient {
    pub address: Pubkey,
    pub share_threshold: u8,
    pub access_time: i64,
}

pub struct CapsuleMetadata {
    pub created_at: i64,
    pub size: u64,
    pub content_type: String,
    pub checksum: [u8; 32],
}
```

### Data Flow

1. **Initialization**
   - Client generates AES-256 key
   - Data is encrypted client-side
   - Encrypted data is stored on Arweave
   - References stored in Solana state

2. **Time-Lock Implementation**
   - Uses Solana's native clock for timestamps
   - Implements Shamir's Secret Sharing for key distribution
   - Threshold-based access control

3. **Asset Transfer**
   - Automatic checkpoint verification
   - Multi-signature recipient validation
   - Atomic transfer execution

## Setup & Development

### Prerequisites

```bash
# Install Rust
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# Install Solana CLI
sh -c "$(curl -sSfL https://release.solana.com/v1.17.0/install)"

# Install Project Dependencies
cargo install --git https://github.com/emberai-sol/ember-cli
```

### Local Development

```bash
# Clone repository
git clone https://github.com/emberai-sol/ember
cd ember

# Install dependencies
cargo build

# Run local tests
cargo test

# Deploy to local validator
solana-test-validator
cargo run --bin deploy
```

### Configuration

Create a `.env` file:

```env
SOLANA_CLUSTER=devnet
ARWEAVE_WALLET_PATH=./arweave-wallet.json
ENCRYPTION_STRENGTH=256
MAX_CAPSULE_SIZE=5242880  # 5MB
```

## Smart Contract Interaction

### Creating a Time Capsule

```typescript
import { Connection, PublicKey, TransactionInstruction } from '@solana/web3.js';
import { emberai-sol } from '@ember/sdk';

const createCapsule = async (
  connection: Connection,
  authority: PublicKey,
  recipients: PublicKey[],
  timelock: number,
  data: Buffer
) => {
  const ember = new emberai-sol(connection);
  
  // Generate encryption key
  const encryptionKey = await ember.generateKey();
  
  // Encrypt data
  const encryptedData = await ember.encrypt(data, encryptionKey);
  
  // Store data on Arweave
  const dataHash = await ember.storeData(encryptedData);
  
  // Create capsule instruction
  const ix = await ember.createCapsuleInstruction({
    authority,
    recipients,
    timelock,
    dataHash,
  });
  
  return ix;
};
```

### Retrieving Data

```typescript
const retrieveCapsule = async (
  connection: Connection,
  capsuleAddress: PublicKey,
  recipient: PublicKey
) => {
  const ember = new emberai-sol(connection);
  
  // Verify time conditions
  await ember.verifyAccess(capsuleAddress, recipient);
  
  // Retrieve and decrypt data
  const capsule = await ember.getCapsule(capsuleAddress);
  const encryptedData = await ember.retrieveData(capsule.dataHash);
  const data = await ember.decrypt(encryptedData, capsule.encryptionKey);
  
  return data;
};
```

## Security Considerations

### Encryption Process
- Client-side AES-256-GCM encryption
- Key splitting using Shamir's Secret Sharing
- Zero-knowledge proofs for recipient verification

### Access Control
- Time-based access validation
- Multi-signature requirement option
- Threshold encryption for shared access

### Data Storage
- Content-addressed storage on Arweave
- Encrypted metadata on Solana
- Checksum verification

## Program Instructions

```rust
pub enum EmberInstruction {
    /// Initialize a new time capsule
    CreateCapsule {
        recipients: Vec<Pubkey>,
        timelock: i64,
        data_hash: [u8; 32],
    },
    
    /// Add recipients to existing capsule
    AddRecipients {
        new_recipients: Vec<Pubkey>,
    },
    
    /// Update time lock settings
    UpdateTimelock {
        new_timelock: i64,
    },
    
    /// Retrieve capsule data (requires recipient verification)
    RetrieveCapsule,
}
```

## Error Handling

```rust
pub enum EmberError {
    /// Invalid authority signature
    InvalidAuthority,
    
    /// Time lock conditions not met
    TimelockActive,
    
    /// Recipient not authorized
    UnauthorizedRecipient,
    
    /// Data verification failed
    InvalidChecksum,
    
    /// Encryption error
    EncryptionError,
}
```

## Testing

Run the test suite:

```bash
# Unit tests
cargo test

# Integration tests
cargo test --features integration

# Specific test suite
cargo test capsule_creation
```

## License

MIT License - see LICENSE file for details

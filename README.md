🚀 Solana Hello World 

📘 Description

This repository contains our implementation of a basic "Hello, World!" smart contract on the "Solana blockchain"

⚙️ Prerequisites

Before getting started, we made sure to install the following tools:

- Rust — for writing the Solana program.
- Solana CLI — to interact with the Solana blockchain.


🛠️ Project Initialization

We began by creating the project using the following commands:

bash
cargo init hello_world --lib
cd hello_world
cargo add solana-program@1.18.26


- `cargo init hello_world --lib` — initialized a new Rust library project.
- `cd hello_world` — navigated into the newly created folder.
- `cargo add solana-program@1.18.26` — added Solana's core SDK to the project.

We also edited the `Cargo.toml` file to include:

```toml
[lib]
crate-type = ["cdylib", "lib"]

[dependencies]
solana-program = "1.18.26"
```


📁 Project Structure

Our folder structure looked like this:

```
.
├── src/lib.rs
├── examples/client.rs
├── test-ledger/
├── Cargo.toml
└── README.md
```

---

🧠 Program Logic (`src/lib.rs`)

Here is the code we wrote for the main program:

```rust
use solana_program::{
    account_info::AccountInfo, entrypoint, entrypoint::ProgramResult, msg, pubkey::Pubkey,
};

entrypoint!(process_instruction);

pub fn process_instruction(
    _program_id: &Pubkey,
    _accounts: &[AccountInfo],
    _instruction_data: &[u8],
) -> ProgramResult {
    msg!("Hello, world!");
    Ok(())
}
```

- `entrypoint!` defines the entry for the program.
- `msg!` logs output to the program log.

---

🔨 Building the Program

To compile the program, we used:

```bash
cargo build-sbf
```

This generated the `.so` and keypair files in `target/deploy/`. We got our program ID with:

```bash
solana address -k ./target/deploy/hello_world-keypair.json
```

---

🔧 Local Validator Setup

We configured the Solana CLI for local development and started a test validator:

```bash
solana config set -ul
solana-test-validator
```

---

🚀 Deploying the Program

While the validator was running, we deployed the program using:

```bash
solana program deploy ./target/deploy/hello_world.so
```

This returned the program ID and transaction signature.

---

✅ Unit Testing with solana-program-test

We added testing dependencies:

```bash
cargo add solana-program-test@1.18.26 --dev
cargo add solana-sdk@1.18.26 --dev
cargo add tokio --dev
```

Then, we added the following unit test in `lib.rs`:

```rust
#[cfg(test)]
mod test {
    use solana_program_test::*;
    use solana_sdk::{instruction::Instruction, pubkey::Pubkey, signature::Signer, transaction::Transaction};

    #[tokio::test]
    async fn test_hello_world() {
        let program_id = Pubkey::new_unique();
        let mut program_test = ProgramTest::default();
        program_test.add_program("hello_world", program_id, None);
        let (mut banks_client, payer, recent_blockhash) = program_test.start().await;

        let instruction = Instruction { program_id, accounts: vec![], data: vec![] };
        let mut transaction = Transaction::new_with_payer(&[instruction], Some(&payer.pubkey()));
        transaction.sign(&[&payer], recent_blockhash);

        let result = banks_client.process_transaction(transaction).await;
        assert!(result.is_ok());
    }
}
```

We ran the test using:

```bash
cargo test-sbf
```

---

💻 Writing a Rust Client (`examples/client.rs`)

We created a simple Rust client that airdrops SOL and sends a transaction to the program.

First, we registered the example in `Cargo.toml`:

```toml
[[example]]
name = "client"
path = "examples/client.rs"
```

Then added the dependency:

```bash
cargo add solana-client@1.18.26 --dev
```

Here’s the code we wrote:

```rust
use solana_client::rpc_client::RpcClient;
use solana_sdk::{
    commitment_config::CommitmentConfig,
    instruction::Instruction,
    pubkey::Pubkey,
    signature::{Keypair, Signer},
    transaction::Transaction,
};
use std::str::FromStr;

#[tokio::main]
async fn main() {
    let program_id = Pubkey::from_str("YOUR_PROGRAM_ID").unwrap();
    let client = RpcClient::new_with_commitment("http://127.0.0.1:8899".into(), CommitmentConfig::confirmed());

    let payer = Keypair::new();
    let airdrop_amount = 1_000_000_000;
    let signature = client.request_airdrop(&payer.pubkey(), airdrop_amount).unwrap();

    loop {
        if client.confirm_transaction(&signature).unwrap() {
            break;
        }
    }

    let instruction = Instruction::new_with_borsh(program_id, &(), vec![]);
    let mut transaction = Transaction::new_with_payer(&[instruction], Some(&payer.pubkey()));
    transaction.sign(&[&payer], client.get_latest_blockhash().unwrap());

    match client.send_and_confirm_transaction(&transaction) {
        Ok(sig) => println!("Transaction Signature: {}", sig),
        Err(err) => eprintln!("Error: {}", err),
    }
}
```

We ran the client with:

```bash
cargo run --example client
```

🔁 Updating the Program

To change the message, we updated the `msg!` macro in `src/lib.rs` to:

```rust
msg!("Hello, Solana!");
```

Then rebuilt and redeployed:

```bash
cargo build-sbf
solana program deploy ./target/deploy/hello_world.so
```
🧹 Closing the Program

To reclaim SOL from the deployed program, we closed it using:

```bash
solana program close <PROGRAM_ID> --bypass-warning
```

If we needed to redeploy, we generated a new program ID with:

```bash
solana-keygen new -o ./target/deploy/hello_world-keypair.json --force
```



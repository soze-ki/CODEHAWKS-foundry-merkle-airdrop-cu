# CODEHAWKS-foundry-merkle-airdrop-cu

Cyfrin Updraft – Security & Auditing Course  
Author: **Jason King**  


This lesson shows you how to eliminate plain-text private keys from your workflow by using Foundry’s built-in ERC-2335–encrypted keystores. Follow the step-by-step guide below to import, encrypt, and deploy with cast—no `.env` file required.

# https://www.youtube.com/watch?v=VQe7cIpaE54

This YouTube video from Cyfrin Audits demonstrates a crucial security improvement for Foundry developers: **how to encrypt private keys instead of storing them in plain text .env files**. The tutorial shows developers how to use Foundry's built-in `cast` tool to securely handle private keys through password encryption.[^1_2]

## The Security Problem

The video addresses a fundamental security issue where developers traditionally store private keys in `.env` files as plain text. This practice creates multiple security risks:[^1_2]

- **Accidental exposure**: Private keys might be accidentally pushed to GitHub repositories
- **Terminal exposure**: Keys could be displayed in terminal history or output
- **Plain text vulnerability**: Unencrypted keys are easily compromised if accessed[^1_2]


## The Encrypted Solution

The tutorial demonstrates using **ERC-2335** encryption standard to secure private keys in JSON format. Here's the step-by-step process:[^1_2]

### 1. Import and Encrypt Private Key

```bash
cast wallet import default_key --interactive
```

This command creates an interactive shell where you paste your private key (which appears hidden) and set a password for encryption.[^1_2]

### 2. Deploy Using Encrypted Key

Instead of the insecure method:

```bash
forge script --private-key YOUR_PLAIN_TEXT_KEY
```

Use the secure encrypted approach:

```bash
forge script script/DeployFundMe.s.sol:DeployFundMe \
  --rpc-url http://localhost:8545 \
  --account default_key \
  --sender YOUR_ADDRESS \
  --broadcast
```

The system will prompt for your password during execution, keeping the private key encrypted.[^1_2]

## Advanced Security Features

The video covers additional security measures:

- **Password files**: Using `--password-file` to automate authentication while maintaining security
- **History cleanup**: Commands like `history -c` to remove private keys from shell history
- **Keystore location**: Encrypted keys are stored in `~/.foundry/keystores/` as encrypted JSON files[^1_2]


## Key Security Principles

The instructor emphasizes that developers should develop an instinctive aversion to seeing private keys in plain text. **Any time a private key appears unencrypted, it should trigger immediate concern and action to secure it**.[^1_2]

This method represents a significant improvement in blockchain development security practices, eliminating the need for `.env` files containing sensitive cryptographic material while maintaining developer workflow efficiency.



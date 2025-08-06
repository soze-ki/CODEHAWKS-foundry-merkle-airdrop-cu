# 🔐 Secure Private Key Management with Foundry

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






# Merkle Airdrop Extravaganza 

This is a section of the [Cyfrin Updraft Advanced Foundry Course](https://updraft.cyfrin.io/). In this repo, we will learn about signatures, merkle drops, and more. 

- [Merkle Airdrop Extravaganza](#merkle-airdrop-extravaganza)
- [Getting Started](#getting-started)
  - [Requirements](#requirements)
  - [Quickstart](#quickstart)
- [Usage](#usage)
  - [Pre-deploy: Generate merkle proofs](#pre-deploy-generate-merkle-proofs)
- [Deploy](#deploy)
  - [Deploy to Anvil](#deploy-to-anvil)
  - [Deploy to a zkSync local node](#deploy-to-a-zksync-local-node)
    - [zkSync prerequisites](#zksync-prerequisites)
    - [Setup local zkSync node](#setup-local-zksync-node)
    - [Deploy to a local zkSync network](#deploy-to-a-local-zksync-network)
    - [Deploy to zkSync Sepolia](#deploy-to-zksync-sepolia)
  - [Interacting - zkSync local network](#interacting---zksync-local-network)
    - [Setup local zksync node, deploy contracts, and run airdrop claim](#setup-local-zksync-node-deploy-contracts-and-run-airdrop-claim)
  - [Interacting - Local anvil network](#interacting---local-anvil-network)
    - [Setup anvil and deploy contracts](#setup-anvil-and-deploy-contracts)
    - [Sign your airdrop claim](#sign-your-airdrop-claim)
    - [Claim your airdrop](#claim-your-airdrop)
    - [Check claim amount](#check-claim-amount)
  - [Testing](#testing)
    - [Test Coverage](#test-coverage)
  - [Estimate gas](#estimate-gas)
- [Formatting](#formatting)
- [Thank you!](#thank-you)

# Getting Started

## Requirements

- [git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
  - You'll know you did it right if you can run `git --version` and you see a response like `git version x.x.x`
- [foundry](https://getfoundry.sh/)
  - You'll know you did it right if you can run `forge --version` and you see a response like `forge 0.2.0 (816e00b 2023-03-16T00:05:26.396218Z)`

To get started, we are assuming you're working with vanilla `foundry` and not `foundry-zksync` to start. 


## Quickstart

```bash
git clone https://github.com/Cyfrin/foundry-merkle-airdrop-cu
cd foundry-merkle-airdrop-cu
make # or forge install && forge build if you don't have make 
```

# Usage

## Pre-deploy: Generate merkle proofs

We are going to generate merkle proofs for an array of addresses to airdrop funds to. If you'd like to work with the default addresses and proofs already created in this repo, skip to [deploy](#deploy)

If you'd like to work with a different array of addresses (the `whitelist` list in `GenerateInput.s.sol`), you will need to follow the following:

First, the array of addresses to airdrop to needs to be updated in `GenerateInput.s.sol. To generate the input file and then the merkle root and proofs, run the following:

Using make:

```bash
make merkle
```

Or using the commands directly:

```bash
forge script script/GenerateInput.s.sol:GenerateInput && forge script script/MakeMerkle.s.sol:MakeMerkle
```

Then, retrieve the `root` (there may be more than 1, but they will all be the same) from `script/target/output.json` and paste it in the `Makefile` as `ROOT` (for zkSync deployments) and update `s_merkleRoot` in `DeployMerkleAirdrop.s.sol` for Ethereum/Anvil deployments.

# Deploy 

## Deploy to Anvil

```bash
# Optional, ensure you're on vanilla foundry
foundryup
# Run a local anvil node
make anvil
# Then, in a second terminal
make deploy
```

## Deploy to a zkSync local node

### zkSync prerequisites

- [foundry-zksync](https://github.com/matter-labs/foundry-zksync)
  - You'll know you did it right if you can run `forge --version` and you see a response like `forge 0.0.2 (816e00b 2023-03-16T00:05:26.396218Z)`. 
- [npx & npm](https://docs.npmjs.com/cli/v10/commands/npm-install)
  - You'll know you did it right if you can run `npm --version` and you see a response like `7.24.0` and `npx --version` and you see a response like `8.1.0`.
- [docker](https://docs.docker.com/engine/install/)
  - You'll know you did it right if you can run `docker --version` and you see a response like `Docker version 20.10.7, build f0df350`.
  - Then, you'll want the daemon running, you'll know it's running if you can run `docker --info` and in the output you'll see something like the following to know it's running:
```bash
Client:
 Context:    default
 Debug Mode: false
```

### Setup local zkSync node 

Run the following:

```bash
npx zksync-cli dev config
```

And select: `In memory node` and do not select any additional modules.

Then run:
```bash
npx zksync-cli dev start
```

And you'll get an output like:

```
In memory node started v0.1.0-alpha.22:
 - zkSync Node (L2):
  - Chain ID: 260
  - RPC URL: http://127.0.0.1:8011
  - Rich accounts: https://era.zksync.io/docs/tools/testing/era-test-node.html#use-pre-configured-rich-wallets
```

This will save your zkSync configuration so you won't have to run `npx zksync-cli dev config` again. 

In the future, you can just run:
```bash
make zk-anvil
``` 

To close the zkSync node (in the future, leave it running for now), run:
```bash
docker ps
```

To get the container ID from the result. If there is no result, then docker might not be running, and you're all set. Otherwise run:
```bash
docker kill ${CONTAINER_ID}
```

### Deploy to a local zkSync network 
```bash
# Optional, ensure you're on foundry-zksync
foundryup-zksync
# Setup a docker container for zkSync (if you haven't already)
# make zk-anvil
# deploy
make deploy-zk
```

### Deploy to zkSync Sepolia

Be sure you have the following:
- `ZKSYNC_SEPOLIA_RPC_URL` set in your `.env` file
- An account named `default` set up your `cast`
  - [See here to set one up](https://www.youtube.com/watch?v=VQe7cIpaE54)

```bash
# Optional, ensure you're on foundry-zksync
foundryup-zksync
# Deploy
make deploy-zk-sepolia
# You'll be prompted to enter your password
```

## Interacting - zkSync local network

The following steps allow the second default anvil address (0x70997970C51812dc3A010C7d01b50e0d17dc79C8) to call claim and pay for the gas on behalf of the first default anvil address (0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266) which will recieve the airdrop. 

### Setup local zksync node, deploy contracts, and run airdrop claim

> See [Deploy to a zkSync local node prerequisites](#zksync-prerequisites) for prerequisites.

On `foundry-zksync`, setup a local node and deploy the zkSync contracts. You can do both steps with this two commands:

```bash
foundryup-zksync
chmod +x interactZk.sh && ./interactZk.sh
```

You'll see the output of:
1. Deploying zkSync smart contracts
2. Signing your airdrop claim
3. Claiming the airdrop

All from the `./interactZk.sh` script.

## Interacting - Local anvil network

### Setup anvil and deploy contracts

Swap back to vanilla foundry and run an anvil node:

```bash
foundryup
make anvil
make deploy
# Copy the BagelToken address & Airdrop contract address
```
Copy the Bagel Token and Aidrop contract addresses and paste them into the `AIRDROP_ADDRESS` and `TOKEN_ADDRESS` variables in the `MakeFile`

The following steps allow the second default anvil address (`0x70997970C51812dc3A010C7d01b50e0d17dc79C8`) to call claim and pay for the gas on behalf of the first default anvil address (`0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266`) which will recieve the airdrop. 

### Sign your airdrop claim  

```bash
# in another terminal
make sign
```

Retrieve the signature bytes outputted to the terminal and add them to `Interact.s.sol` *making sure to remove the `0x` prefix*. 

Additionally, if you have modified the claiming addresses in the merkle tree, you will need to update the proofs in this file too (which you can get from `output.json`)


### Claim your airdrop

Then run the following command:

```bash
make claim
```

### Check claim amount

Then, check the claiming address balance has increased by running

```bash
make balance
```

NOTE: `0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266` is the default anvil address which has recieved the airdropped tokens.


## Testing

```bash
foundryup
forge test
```

for for zkSync

```bash
# This will run `foundryup-zksync && forge test --zksync && foundryup`
make zktest
```

### Test Coverage

```bash
forge coverage
```

## Estimate gas

You can estimate how much gas things cost by running:

```
forge snapshot
```

And you'll see an output file called `.gas-snapshot`


# Formatting


To run code formatting:
```
forge fmt
```

# Thank you!

If you appreciated this, please follow [Cyfrin Updraft!](https://x.com/CyfrinUpdraft)

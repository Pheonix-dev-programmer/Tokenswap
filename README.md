
# Script Overview

This script automates the process of exchanging USDC for LINK via Uniswap, and then depositing the LINK tokens into the Aave Lending Pool. It handles token approval, token swapping, and yield farming across multiple DeFi protocols.

## Key Functions and Logic

### 1. `approveToken(tokenAddress, tokenABI, amount, wallet)`

- **Purpose**: Grants the Swap Router contract permission to spend a specified amount of USDC tokens.
- **Interaction**: Invokes the `approve` method on the ERC20 token contract to authorize spending.

### 2. `getPoolInfo(factoryContract, tokenIn, tokenOut)`

- **Purpose**: Retrieves the Uniswap pool address for swapping USDC to LINK and obtains pool details.
- **Interaction**: Queries the factory contract to find the pool address and fetches details such as tokens and fee structure.

### 3. `prepareSwapParams(poolContract, signer, amountIn)`

- **Purpose**: Constructs the parameters required for the token swap transaction.
- **Interaction**: Prepares parameters for the `exactInputSingle` method of the Swap Router contract.

### 4. `executeSwap(swapRouter, params, signer)`

- **Purpose**: Executes the swap transaction from USDC to LINK.
- **Interaction**: Uses the `exactInputSingle` method on the Swap Router contract with the prepared parameters.

### 5. `supplyToAave(tokenAddress, amount, signer)`

- **Purpose**: Deposits LINK tokens into the Aave Lending Pool to earn yield.
- **Interaction**: 
  - **Step 1**: Approves Aave to manage LINK tokens.
  - **Step 2**: Uses the `deposit` method of the Aave Lending Pool contract to deposit the tokens.

## Interaction with DeFi Protocols

- **ERC20 Token Approval**: Interacts with the token contract (e.g., USDC) to permit the Swap Router to handle tokens.
- **Uniswap Pool**: Uses the factory contract to identify the pool address and details necessary for swapping tokens.
- **Swap Router**: Conducts the token swap from USDC to LINK through Uniswap’s liquidity pool.
- **Aave Lending Pool**: Approves and deposits LINK tokens into Aave to earn interest.

## Diagram Illustration

### DeFi Automation Script Flow Diagram

```
+---------------------+
| 1. Approve USDC    |
+---------------------+
          |
          v
+---------------------+
| 2. Get Uniswap Pool |
|    Info             |
+---------------------+
          |
          v
+---------------------+
| 3. Prepare Swap     |
|    Params           |
+---------------------+
          |
          v
+---------------------+
| 4. Execute Swap     |
+---------------------+
          |
          v
+---------------------+
| 5. Approve LINK     |
|    for Aave         |
+---------------------+
          |
          v
+---------------------+
| 6. Deposit LINK     |
|    to Aave          |
+---------------------+
```

## Code Explanation

### 1. Import Dependencies and Setup

```javascript
require('dotenv').config();
const { ethers } = require('ethers');
const USDC_ABI = require('./abis/usdc.json');
const LINK_ABI = require('./abis/link.json');
const POOL_FACTORY_ABI = require('./abis/poolFactory.json');
const SWAP_ROUTER_ABI = require('./abis/swapRouter.json');
const AAVE_ABI = require('./abis/aave.json');
```

**Function:**
- **dotenv:** Loads environment variables from a `.env` file, allowing secure management of sensitive data like API keys and private keys.
- **ethers:** A library for interacting with the Ethereum blockchain, including smart contracts and transactions.
- **ABI files:** JSON files describing the interface of Ethereum smart contracts, necessary for interaction.

### 2. Setup Provider and Signer

```javascript
const provider = new ethers.providers.JsonRpcProvider(process.env.RPC_URL);
const wallet = new ethers.Wallet(process.env.PRIVATE_KEY, provider);
```

**Function:**
- **provider:** Connects to the Ethereum network using a JSON-RPC URL.
- **wallet:** Creates a wallet instance with a private key, used for signing transactions and interacting with the blockchain.

### 3. Define Contract Addresses

```javascript
const POOL_FACTORY_CONTRACT_ADDRESS = '0x...'; // Uniswap Pool Factory address
const SWAP_ROUTER_CONTRACT_ADDRESS = '0x...'; // Uniswap Swap Router address
const AAVE_LENDING_POOL_ADDRESS = '0x...';   // Aave Lending Pool address
```

**Function:**
- Defines the Ethereum addresses for the Uniswap Pool Factory, Swap Router, and Aave Lending Pool contracts, which are essential for interaction.

### 4. Define Key Functions

#### `approveToken(tokenAddress, tokenABI, amount, wallet)`

```javascript
async function approveToken(tokenAddress, tokenABI, amount, wallet) {
    const tokenContract = new ethers.Contract(tokenAddress, tokenABI, wallet);
    const tx = await tokenContract.approve(SWAP_ROUTER_CONTRACT_ADDRESS, amount);
    await tx.wait();
}
```

**Function:**
- **Create Contract Instance:** Initializes an ERC20 token contract using its address and ABI.
- **Approval:** Sends a transaction to approve the Swap Router contract to spend a specified amount of the token (USDC).
- **Confirmation:** Waits for the approval transaction to be mined and confirmed.

#### `getPoolInfo(factoryContract, tokenIn, tokenOut)`

```javascript
async function getPoolInfo(factoryContract, tokenIn, tokenOut) {
    const poolAddress = await factoryContract.getPool(tokenIn, tokenOut, 3000);
    const poolContract = new ethers.Contract(poolAddress, POOL_ABI, wallet);
    const poolDetails = await poolContract.getPoolDetails();
    return poolDetails;
}
```

**Function:**
- **Get Pool Address:** Queries the Uniswap Pool Factory contract to get the pool address for the token pair (USDC and LINK).
- **Pool Details:** Fetches details such as token addresses and fee from the pool contract.

#### `prepareSwapParams(poolContract, signer, amountIn)`

```javascript
function prepareSwapParams(poolContract, signer, amountIn) {
    return {
        tokenIn: '0x...', // USDC address
        tokenOut: '0x...', // LINK address
        fee: 3000,
        recipient: signer.address,
        deadline: Math.floor(Date.now() / 1000) + 60 * 20, // 20 minutes from now
        amountIn: ethers.utils.parseUnits(amountIn.toString(), 6),
        amountOutMinimum: 0,
    };
}
```

**Function:**
- **Construct Parameters:** Prepares the parameters for the `exactInputSingle` method of the Uniswap Swap Router contract, including token addresses, fee, recipient, and transaction details.

#### `executeSwap(swapRouter, params, signer)`

```javascript
async function executeSwap(swapRouter, params, signer) {
    const swapRouterContract = new ethers.Contract(swapRouter, SWAP_ROUTER_ABI, signer);
    const tx = await swapRouterContract.exactInputSingle(params);
    await tx.wait();
}
```

**Function:**
- **Create Contract Instance:** Initializes the Swap Router contract.
- **Execute Swap:** Calls the `exactInputSingle` method to perform the swap from USDC to LINK.
- **Confirmation:** Waits for the swap transaction to be mined and confirmed.

#### `supplyToAave(tokenAddress, amount, signer)`

```javascript
async function supplyToAave(tokenAddress, amount, signer) {
    const aaveContract = new ethers.Contract(AAVE_LENDING_POOL_ADDRESS, AAVE_ABI, signer);
    await approveToken(tokenAddress, LINK_ABI, amount, signer); // Approve Aave to spend LINK
    const tx = await aaveContract.deposit(tokenAddress, amount, signer.address, 0);
    await tx.wait();
}
```

**Function:**
- **Approval:** Calls `approveToken` to authorize Aave to spend LINK tokens.
- **Deposit:** Sends a transaction to deposit LINK tokens into the Aave Lending Pool to earn yield.
- **Confirmation:** Waits for the deposit transaction to be mined and confirmed.

### 5. Workflow Summary

1. **Approval:** Grants the Swap Router permission to handle USDC tokens.
2. **Get Pool Info:** Retrieves the Uniswap pool address and details necessary for the swap.
3. **Prepare Swap Params:** Constructs the parameters for the swap transaction.
4. **Execute Swap:** Performs the token swap from USDC to LINK via Uniswap.
5. **Supply to Aave:** Approves and deposits LINK tokens into the Aave Lending Pool.
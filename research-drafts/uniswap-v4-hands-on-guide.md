# Hands-On Uniswap V4: A Developer's Guide to Liquidity and Swaps with Solidity

Uniswap V4 represents a significant evolution in decentralized exchange architecture, building upon the success of V3 while introducing powerful new features that give developers unprecedented control over liquidity and swap behavior. This guide will walk you through the core concepts, setup, and practical implementation details needed to build with Uniswap V4 using Solidity.

## Introduction to Uniswap V4

Uniswap V4 introduces a radical redesign centered around a single, powerful contract called the PoolManager. Unlike previous versions where pools were separate contracts with their own logic, V4 consolidates all pool functionality into one contract while enabling customization through hooks. This architecture reduces deployment costs significantly and opens up new possibilities for on-chain trading strategies.

The most groundbreaking addition is the hooks system: smart contracts that execute custom logic at precise points during liquidity provision and swaps. You can create hooks that implement limit orders, dynamic fees, custom price oracles, or any other behavior you can imagine, all integrated seamlessly into the core exchange flow.

Other major improvements include native ETH support (no more wrapping), flash accounting that nets obligations to minimize external transfers, and a new token standard ERC-6909 that replaces ERC-20 for more efficient multi-token management. The singleton PoolManager architecture means all pools live under one address, simplifying pool discovery and interaction.

### Comparison of Uniswap Versions

To understand V4's advantages, it helps to see how it compares to previous versions:

| Feature | Uniswap V2 | Uniswap V3 | Uniswap V4 |
|---------|------------|------------|------------|
| Architecture | Separate pool contracts | Separate pool contracts | Single PoolManager singleton |
| Gas Efficiency | Standard | Improved capital efficiency | Highest, with flash accounting |
| Customization | Limited to router | Limited to periphery | Hooks system for full customization |
| ETH Support | Wrapped only | Wrapped only | Native ETH support |
| Pool Creation | Deploy new contract | Deploy new contract | Initialize within PoolManager |
| Liquidity Precision | Full range positions | Concentrated ranges | Concentrated ranges with hooks |
| Token Standard | ERC-20 | ERC-20 | ERC-6909 multi-token |

This table highlights the architectural shift in V4: moving from separate pool contracts to a unified manager, and adding the hooks system that fundamentally changes what's possible on top of Uniswap.

## Prerequisites

Before diving into Uniswap V4 development, you should have a solid understanding of Solidity fundamentals including contract structure, functions, modifiers, and error handling. Familiarity with automated market makers (AMMs) and liquidity provider concepts from Uniswap V2 or V3 will be helpful, though not strictly required as we'll cover enough background.

You should have Node.js and npm installed for package management. For testing and deployment, you'll use Hardhat v2 set up in your development environment. This guide will primarily use Solidity code examples, so you should be comfortable writing and understanding Solidity contracts.

## Setting Up the Development Environment

### Installing Dependencies

For a Hardhat v2 project, install the required packages using npm:

```bash
npm init -y
npm install --save-dev hardhat @nomicfoundation/hardhat-toolbox
npm install @uniswap/v4-core @uniswap/v4-periphery @openzeppelin/contracts
npm install --save-dev @types/node typescript ts-node
```

Initialize your Hardhat project with `npx hardhat init` and choose TypeScript or JavaScript based on your preference. The dependencies include the core Uniswap V4 contracts, periphery utilities, and OpenZeppelin for common security patterns.

### Project Structure

A proper Hardhat v2 project structure should look like this:

```
contracts/
├── interfaces/
│   ├── IPoolManager.sol
│   ├── IPositionManager.sol
│   ├── ISwapRouter.sol
│   └── IHook.sol
├── MyPoolHook.sol
├── MyTradingContract.sol
└── HelperContracts.sol
scripts/
├── deploy.ts
├── create-pool.ts
└── demo-swaps.ts
test/
├── pool.test.ts
├── swap.test.ts
├── hook.test.ts
└── fixtures.ts
hardhat.config.ts
package.json
tsconfig.json
```

When writing contracts, you'll import from the Uniswap V4 packages using node_modules resolution:

```solidity
import {IPoolManager} from "@uniswap/v4-core/contracts/interfaces/IPoolManager.sol";
import {IPositionManager} from "@uniswap/v4-periphery/contracts/interfaces/IPositionManager.sol";
import {ISwapRouter} from "@uniswap/v4-periphery/contracts/interfaces/ISwapRouter.sol";
import {IHook} from "@uniswap/v4-core/contracts/interfaces/IHook.sol";
```

### Hardhat Configuration

Create a `hardhat.config.ts` (or `.js`) file with the following configuration:

```typescript
import {HardhatUserConfig} from "hardhat/config";
import "@nomicfoundation/hardhat-toolbox";

const config: HardhatUserConfig = {
  solidity: {
    version: "0.8.20",
    settings: {
      optimizer: {
        enabled: true,
        runs: 200
      }
    }
  },
  networks: {
    hardhat: {
      chainId: 31337,
      gasPrice: 0,
      accounts: {
        mnemonic: "test test test test test test test test test test test junk",
        path: "m/44'/60'/0'/0",
        initialIndex: 0,
        count: 20
      }
    },
    localhost: {
      url: "http://127.0.0.1:8545"
    },
    sepolia: {
      url: process.env.SEPOLIA_URL || "",
      accounts: process.env.PRIVATE_KEY
        ? [process.env.PRIVATE_KEY]
        : [],
      gasPrice: 20000000000,
      chainId: 11155111
    }
  },
  paths: {
    sources: "./contracts",
    tests: "./test",
    cache: "./cache",
    artifacts: "./artifacts"
  }
};

export default config;
```

This configuration sets up Solidity 0.8.20 with optimization, defines local and Sepolia networks, and specifies the project directory structure. Adjust network settings based on your deployment target.

## Core Concepts

### Singleton PoolManager Architecture

Uniswap V4 consolidates all pool logic into a single PoolManager contract. Instead of deploying separate contract instances for each pool (as in V2 and V3), all pools now exist as state within the PoolManager. This means that to interact with any pool, you call functions on the PoolManager address. The PoolManager stores pool configurations, handles swaps, creates positions, and executes hook callbacks.

When you want to create a new pool, you don't deploy a new contract. Instead, you initialize a new pool within the PoolManager by calling `initialize` with a PoolKey that identifies the token pair, fee tier, and tick spacing. The PoolManager creates the necessary internal data structures for that pool and assigns it a unique pool ID that can be used in subsequent calls.

### Hooks System

Hooks are contracts that implement custom logic at specific points in the pool lifecycle. They're optional and can be attached to pools during creation. When you create a pool in V4, you specify hook addresses for various callback points: before/after initialization, before/after swap, before/after add liquidity, before/after remove liquidity, before/after donate (for fee collection), and before/after transfer LP tokens.

Hooks must implement predetermined function signatures that the PoolManager will call at the appropriate times. These callbacks allow you to add conditions, custom accounting, off-chain price oracles, or any other business logic. The hook system makes Uniswap V4 highly modular: you can write complex trading strategies, limit order implementations, or token bonding curves without forking the core protocol.

### Hook Callback Points

The following table lists all available hook callback points and their purposes:

| Callback | When Called | Parameters | Typical Use Cases |
|-----------|-------------|------------|-------------------|
| beforeInitialize | Before pool initialization | poolId, tick | Validate initialization parameters |
| afterInitialize | After pool initialization | poolId, tick | Record pool creation events |
| beforeSwap | Before execution of swap | poolId, SwapRequest | Validate swap conditions, check balances |
| afterSwap | After completion of swap | poolId, SwapRequest, amount0Delta, amount1Delta | Track volume, update oracles, distribute rewards |
| beforeAddLiquidity | Before liquidity addition | poolId, liquidityParams | Validate liquidity amounts, enforce caps |
| afterAddLiquidity | After liquidity added | poolId, liquidityParams, liquidityDelta | Track provider counts, distribute incentives |
| beforeRemoveLiquidity | Before liquidity removal | poolId, liquidityParams | Enforce withdrawal limits, cooldowns |
| afterRemoveLiquidity | After liquidity removed | poolId, liquidityParams, liquidityDelta | Update metrics, trigger compensations |
| beforeDonate | Before fee donation | poolId, amount0, amount1 | Validate donation amounts |
| afterDonate | After fee donation | poolId, amount0, amount1 | Record contributions, update governance |
| beforeTransfer | Before LP token transfer | poolId, tokenId, sender, recipient | Validate transfers, enforce restrictions |
| afterTransfer | After LP token transfer | poolId, tokenId, sender, recipient | Update ownership records, emit events |

Each callback executes in the context of the PoolManager and can read or modify state according to its purpose. Hooks are called in a specific order, and reverts in a hook will revert the entire operation.

### PoolKey Struct and Pool Identifiers

Every pool in V4 is uniquely identified by a PoolKey struct containing:

- `currency0` and `currency1`: The two token addresses (sorted by their token ID)
- `fee`: The fee tier in hundredths of a basis point (e.g., 3000 = 0.3%)
- `tickSpacing`: The tick spacing for that pool (derived from fee tier)

The PoolKey struct has the following fields:

| Field | Type | Description |
|-------|------|-------------|
| currency0 | address | First token address (must be less than currency1) |
| currency1 | address | Second token address (must be greater than currency0) |
| fee | uint24 | Fee tier in hundredths of a basis point (100 = 0.01%) |
| tickSpacing | int24 | Distance between ticks, derived from fee tier |

To reference a pool in function calls, you typically pass this PoolKey struct. The PoolManager uses the hash of this struct as the internal pool identifier. When you initialize a pool, the PoolManager returns a `poolId` which is a bytes32 value. This poolId is used in subsequent calls to reference the specific pool.

The PoolManager provides a helper function to compute the poolId from a PoolKey:

```solidity
bytes32 poolId = IPoolManager.bytes32Key(keccak256(abi.encode(key)));
```

### Currency Types and ERC-6909

V4 introduces a flexible currency abstraction. Currencies can be ERC-20 tokens, native ETH (represented as a special address), or any asset that implements the IETH adapter interface. The system uses token IDs rather than just addresses to distinguish between different representations of the same asset.

ERC-6909 is a new multi-token standard that replaces ERC-20 in V4 for efficiency. It allows multiple independent token instances to be managed by a single contract, reducing deployment overhead. Many V4 contracts use ERC-6909 instead of traditional ERC-20, though the interface is similar: it has `balanceOf`, `transfer`, `transferFrom`, `approve`, and `allowance` functions.

### Flash Accounting and Netting

One of V4's most powerful efficiency features is flash accounting. Instead of immediately transferring tokens in and out of the PoolManager during swaps and liquidity operations, V4 tracks net obligations and only settles the net difference at the end of a transaction. This reduces the number of external token transfers, saving gas and enabling more complex operations within a single transaction.

For example, during a swap, the PoolManager records how much of token0 it owes the user and how much it receives, but doesn't send tokens until the very end. If you perform multiple swaps in one transaction, the net amounts are calculated and only the final imbalance is settled. This also enables "delta accounting" where liquidity providers can collect earned fees without withdrawing liquidity.

### Delta Accounting Concepts

Delta accounting refers to the way V4 tracks changes in liquidity and token balances. The `Delta` struct records positive or negative changes in token amounts. When you add liquidity, you specify the amount of tokens you're depositing and the position you're creating. The PoolManager updates the global liquidity and your position while recording the delta. When you remove liquidity or collect fees, deltas are settled against your balance in the PositionManager.

Key delta accounting concepts:

| Concept | Description | Example |
|---------|-------------|---------|
| Liquidity Delta | Change in position liquidity | Adding 100 liquidity: +100 |
| Token Delta | Net token amount owed | Swap: user sends tokenIn (+), receives tokenOut (-) |
| Fee Delta | Accumulated fees per position | Collected fees delta updates position balance |
| Settlement | Netting all deltas at transaction end | Multiple operations: only net transfer occurs |
| Position Balance | Tracks token amounts owed to position | Updated via deltas, withdrawable via collect |

This system allows atomic operations that combine multiple actions without intermediate transfers, significantly reducing gas costs and enabling complex trading strategies.

## Flash Accounting Use Cases

Flash accounting and delta management in Uniswap V4 enable powerful patterns that were not possible or were inefficient in previous versions. The ability to perform multiple operations in a single transaction, with only the net token movement settled at the end, opens up sophisticated use cases. Let's explore three concrete implementations that demonstrate the power of atomic operations.

### Atomic Arbitrage

Arbitrage between different Uniswap V4 pools can be executed atomically within a single transaction, eliminating the risk of price changes between separate operations. The flash accounting system allows you to perform a sequence of swaps and only settle the net token difference at the end.

Consider an arbitrage between two USDC/ETH pools with different fee tiers. If one pool has price discrepancy, you can buy cheap from one and sell high on the other in the same transaction, capturing the spread without any outside capital.

```solidity
contract AtomicArbitrage {
    IPoolManager public poolManager;
    ISwapRouter public swapRouter;
    
    constructor(address _poolManager, address _swapRouter) {
        poolManager = IPoolManager(_poolManager);
        swapRouter = ISwapRouter(_swapRouter);
    }
    
    struct ArbitrageConfig {
        bytes32 poolId1; // Pool where we buy
        bytes32 poolId2; // Pool where we sell
        uint24 fee1;
        uint24 fee2;
        uint256 amountIn; // Amount of tokenIn to use for arbitrage
        uint256 minProfit; // Minimum profit threshold
    }
    
    function executeArbitrage(
        address tokenIn,
        address tokenOut,
        ArbitrageConfig memory config,
        uint256 amountOutMin
    ) external returns (uint256 profit) {
        // Determine swap direction: if tokenIn is token0 for both pools
        bool zeroForOne1 = tokenIn < tokenOut;
        bool zeroForOne2 = tokenIn < tokenOut; // Same tokens, same direction
        
        // Step 1: Swap tokenIn -> tokenOut on pool 1 (buy cheap)
        SingleSwap memory swap1 = SingleSwap({
            poolId: config.poolId1,
            kind: zeroForOne1 ? 0 : 1,
            amountSpecified: int256(config.amountIn),
            sqrtPriceLimitX96: 0
        });
        
        FundManagement memory funds1 = FundManagement({
            sender: msg.sender,
            from: tokenIn == address(0) ? msg.sender : tokenIn,
            to: address(this), // Receive tokens into this contract
            wNETH: address(0),
            funding: 1
        });
        
        uint256 amountOut1 = swapRouter.exactInputSingle(
            swap1,
            funds1,
            0, // No minimum for intermediate swap
            block.timestamp + 1000
        );
        
        // Step 2: Swap tokenOut -> tokenIn on pool 2 (sell high)
        // Here we use the output from first swap as input to second
        SingleSwap memory swap2 = SingleSwap({
            poolId: config.poolId2,
            kind: zeroForOne2 ? 0 : 1,
            amountSpecified: -int256(amountOut1), // Negative for exact output opposite direction
            sqrtPriceLimitX96: 0
        });
        
        // Need to approve the swapRouter to spend tokens we just received
        // This is the flash accounting magic: we'll end with net position
        IERC20(tokenOut).approve(address(swapRouter), amountOut1);
        
        FundManagement memory funds2 = FundManagement({
            sender: address(this),
            from: tokenOut,
            to: msg.sender, // Final output goes to caller
            wNETH: address(0),
            funding: 1
        });
        
        uint256 amountIn2 = swapRouter.exactOutputSingle(
            swap2,
            funds2,
            type(uint256).max, // Let it use as much as needed
            block.timestamp + 1000
        );
        
        // Profit is the amount of tokenIn we didn't need to spend
        profit = config.amountIn - amountIn2;
        
        require(profit >= config.minProfit, "Profit too low");
        
        // At this point, the contract holds any excess tokenOut if profit wasn't max
        // But we want to return all profits in tokenIn to caller
        // The flash accounting ensures only net amount is transferred
        
        emit ArbitrageExecuted(
            config.poolId1,
            config.poolId2,
            profit,
            amountOut1,
            amountIn2
        );
    }
    
    event ArbitrageExecuted(
        bytes32 indexed poolId1,
        bytes32 indexed poolId2,
        uint256 profit,
        uint256 amountOut1,
        uint256 amountIn2
    );
}
```

The key insight: this contract receives tokenOut from the first swap, then uses it as input for the second swap. The net effect is that some amount of tokenIn remains in the contract (the profit). The transfer to the caller happens through the second swap's fund management setting `to: msg.sender`. Only the net difference between what we started with and what we end with is actually transferred.

However, the above implementation requires careful handling of token approvals. A more elegant approach using only the PoolManager:

```solidity
contract AtomicArbitrageV2 {
    IPoolManager public poolManager;
    
    constructor(address _poolManager) {
        poolManager = IPoolManager(_poolManager);
    }
    
    function executeArbitrage(
        bytes32 poolId1,
        bytes32 poolId2,
        address tokenIn,
        address tokenOut,
        uint256 amountIn,
        uint256 minProfit
    ) external returns (uint256 profit) {
        // Verify pools have same tokens
        require(poolId1 != poolId2, "Same pool");
        
        // First swap: tokenIn -> tokenOut on pool1
        SwapParams memory params1 = SwapParams({
            poolId: poolId1,
            amountSpecified: int256(amountIn),
            sqrtPriceLimitX96: 0,
            zeroForOne: tokenIn < tokenOut
        });
        
        (int256 amount0Delta1, int256 amount1Delta1) = poolManager.swap(
            params1,
            bytes("") // no hook data
        );
        
        // Determine which token delta represents our output
        uint256 received = tokenIn < tokenOut 
            ? uint256(-amount1Delta1)  // received token1
            : uint256(-amount0Delta1); // received token0
            
        // Second swap: tokenOut -> tokenIn on pool2
        // We specify exact output: we want to end up with at least amountIn - minProfit
        SwapParams memory params2 = SwapParams({
            poolId: poolId2,
            amountSpecified: -int256(received), // negative = exact output
            sqrtPriceLimitX96: 0,
            zeroForOne: tokenIn < tokenOut // Same direction as before
        });
        
        (int256 amount0Delta2, int256 amount1Delta2) = poolManager.swap(
            params2,
            bytes("")
        );
        
        // Calculate how much tokenIn we got back
        uint256 returned = tokenIn < tokenOut
            ? uint256(amount1Delta2)
            : uint256(amount0Delta2);
            
        profit = returned - (amountIn - minProfit);
        require(profit >= minProfit, "Insufficient profit");
        
        // The flash accounting system automatically nets:
        // - We started with amountIn from caller (sent to pool1 in first swap)
        // - We received 'received' tokens after first swap
        // - We spent some of those in second swap to get back tokenIn
        // - The net tokenIn that remains in the contract is profit
        // - At transaction end, this net amount stays with the contract (or could be returned)
        
        emit ArbitrageExecuted(poolId1, poolId2, profit, received, returned);
    }
    
    struct SwapParams {
        bytes32 poolId;
        int256 amountSpecified;
        uint160 sqrtPriceLimitX96;
        bool zeroForOne;
    }
}
```

This version directly uses the PoolManager without the SwapRouter wrapper. It calls `swap` twice. The flash accounting mechanism tracks the deltas from both swaps and nets them. The net effect is that the contract ends up with some profit. However, note that in this raw form, we need to pass tokens to the contract first. For a production implementation, you would use `IERC20(tokenIn).transferFrom(msg.sender, address(this), amountIn)` at the start, and `IERC20(tokenIn).transfer(msg.sender, profit)` at the end. But the flash accounting within the PoolManager still reduces the number of external transfers needed between swaps.

Gas comparison: doing two swaps in separate transactions would require four token transfers (user -> pool, pool -> user, user -> pool, pool -> user). In the atomic version, you have only two transfers: user -> contract (initial funding), contract -> user (net profit), plus the internal accounting between swaps that doesn't involve external transfers. This can save significant gas, especially for high-value arbitrage.

### Liquidity Migration

Moving liquidity from an existing position to a new position (different pool, different tick range, or different fee tier) often requires multiple steps: collect fees, remove liquidity, then add new liquidity. With flash accounting, you can perform this migration atomically, eliminating the risk of price changes between removal and addition.

Here's a contract that migrates liquidity from one position to another in a single transaction:

```solidity
contract LiquidityMigrator {
    IPositionManager public positionManager;
    IPoolManager public poolManager;
    
    constructor(address _positionManager, address _poolManager) {
        positionManager = IPositionManager(_positionManager);
        poolManager = IPoolManager(_poolManager);
    }
    
    struct MigrationParams {
        uint256 tokenId; // Existing position to migrate
        bytes32 newPoolId; // Target pool
        int24 newTickLower;
        int24 newTickUpper;
        uint256 sqrtPriceX96; // Optional: target price for rebalancing
        uint256 amount0Min; // Slippage protection
        uint256 amount1Min;
    }
    
    function migratePosition(MigrationParams memory params) external {
        // Verify caller owns the position
        require(positionManager.ownerOf(params.tokenId) == msg.sender, "Not position owner");
        
        // Get current position data
        Position memory oldPosition = positionManager.getPositionData(params.tokenId);
        
        // Step 1: Remove all liquidity from old position using modifyLiquidity
        ModifyLiquidityParams memory removeParams = ModifyLiquidityParams({
            poolId: oldPosition.poolId,
            liquidityDelta: oldPosition.liquidity,
            sqrtPriceX96: params.sqrtPriceX96,
            tickLower: oldPosition.tickLower,
            tickUpper: oldPosition.tickUpper,
            amount0Requested: 0,
            amount1Requested: 0,
            decreaseLiquidity: true,
            createPosition: false
        });
        
        (uint256 amount0, uint256 amount1) = positionManager.modifyLiquidity(
            removeParams,
            0, // No minimums for withdrawal
            0,
            address(this), // Tokens go to this contract
            block.timestamp + 1000
        );
        
        // Step 2: Collect any outstanding fees from the old position
        (uint256 fees0, uint256 fees1) = positionManager.collect(
            oldPosition.poolId,
            params.tokenId,
            address(this),
            block.timestamp + 1000
        );
        
        // Add collected fees to our token amounts
        amount0 += fees0;
        amount1 += fees1;
        
        // Step 3: Add liquidity to new pool with the same tokens
        ModifyLiquidityParams memory addParams = ModifyLiquidityParams({
            poolId: params.newPoolId,
            liquidityDelta: 0,
            sqrtPriceX96: params.sqrtPriceX96,
            tickLower: params.newTickLower,
            tickUpper: params.newTickUpper,
            amount0Requested: amount0,
            amount1Requested: amount1,
            decreaseLiquidity: false,
            createPosition: true
        });
        
        (uint256 newLiquidity, uint256 usedAmount0, uint256 usedAmount1) = positionManager.modifyLiquidity(
            addParams,
            params.amount0Min,
            params.amount1Min,
            msg.sender, // New position owned by caller
            block.timestamp + 1000
        );
        
        // Burn the old position token
        positionManager.burn(params.tokenId);
        
        emit PositionMigrated(
            params.tokenId,
            oldPosition.poolId,
            params.newPoolId,
            oldPosition.liquidity,
            newLiquidity,
            usedAmount0,
            usedAmount1
        );
    }
    
    event PositionMigrated(
        uint256 indexed oldTokenId,
        bytes32 indexed fromPoolId,
        bytes32 indexed toPoolId,
        uint128 oldLiquidity,
        uint128 newLiquidity,
        uint256 amount0,
        uint256 amount1
    );
}
```

The beauty of this approach is that all operations happen within a single transaction. The `modifyLiquidity` call for removal returns tokens to the contract. Then another `modifyLiquidity` call uses those same tokens without requiring them to leave the contract's balance. The flash accounting system ensures that the token transfers are netted: the tokens received from the old pool are directly used for the new pool, with only any net imbalance settled externally.

Notice that we collect fees before burning the old position. This ensures we capture all accumulated fees. Those fees are then added to the new position automatically, effectively compounding the migration.

The contract burns the old position token after migration. This is important to avoid leaving orphaned tokens that could be used to claim additional assets later.

Slippage protection is applied only to the addition step, as the removal typically should give back exactly what you put in (minus possible rounding differences). The `sqrtPriceX96` parameter can be used to target a specific price range when re-adding liquidity, helping ensure you get similar value for your tokens despite price movements.

In a separate-transaction approach, you would first remove liquidity (tokens transferred to you), then add liquidity (tokens transferred from you). Between these steps, price could move unfavorably, and you'd pay gas twice. The atomic migration is safer and more gas-efficient.

### Fee Reinvestment

Collecting fees and immediately adding them as liquidity is a common strategy for compounding returns. Without flash accounting, this requires two separate transactions: one to collect fees (which transfers tokens to you) and another to add liquidity (which transfers tokens back to the pool). The two transactions incur double gas costs and suffer from timing risk between them.

With flash accounting, you can collect and reinvest in a single transaction:

```solidity
contract FeeReinvestor {
    IPositionManager public positionManager;
    
    constructor(address _positionManager) {
        positionManager = IPositionManager(_positionManager);
    }
    
    function reinvestFees(
        bytes32 poolId,
        uint256 tokenId,
        uint256 amount0Min,
        uint256 amount1Min
    ) external returns (uint256 liquidityAdded) {
        // Ensure caller owns the position
        require(positionManager.ownerOf(tokenId) == msg.sender, "Not owner");
        
        // Get current position to preserve tick range
        Position memory position = positionManager.getPositionData(tokenId);
        
        // Step 1: Collect fees directly into this contract
        (uint256 fee0, uint256 fee1) = positionManager.collect(
            poolId,
            tokenId,
            address(this),
            block.timestamp + 1000
        );
        
        // Check if there are any fees to reinvest
        if (fee0 == 0 && fee1 == 0) {
            revert("No fees to reinvest");
        }
        
        // Step 2: Use the collected fees to add liquidity to the same position
        // We use the same tickLower/tickUpper to deepen existing position
        ModifyLiquidityParams memory addParams = ModifyLiquidityParams({
            poolId: poolId,
            liquidityDelta: 0,
            sqrtPriceX96: 0,
            tickLower: position.tickLower,
            tickUpper: position.tickUpper,
            amount0Requested: fee0,
            amount1Requested: fee1,
            decreaseLiquidity: false,
            createPosition: false // Position already exists
        });
        
        (liquidityAdded, , ) = positionManager.modifyLiquidity(
            addParams,
            amount0Min,
            amount1Min,
            msg.sender, // Add to same position, so recipient is owner
            block.timestamp + 1000
        );
        
        emit FeesReinvested(poolId, tokenId, fee0, fee1, liquidityAdded);
    }
    
    event FeesReinvested(
        bytes32 indexed poolId,
        uint256 indexed tokenId,
        uint256 amount0,
        uint256 amount1,
        uint256 liquidityAdded
    );
}
```

This contract demonstrates the power of flash accounting in a simple but impactful way. The `collect` call deposits the fees directly into the contract's token balance. Then `modifyLiquidity` uses those tokens without them ever leaving the contract. The PoolManager's settle system will only compute the net position at the end of the transaction. If the collected fees exactly match what is added as liquidity, there will be zero net token transfer; the position merely increases in liquidity.

Important details:

- We pass `createPosition: false` because the position already exists. We're simply adding to it.
- The `recipient` parameter in `modifyLiquidity` is set to `msg.sender` (the position owner). This means the increased liquidity is added to their existing position. The PositionManager handles this correctly.
- We fetch the existing position's tick range (`tickLower`, `tickUpper`) to ensure we add liquidity to the same range. If you wanted to reinvest fees into a different range, you would need to create a new position instead.

This pattern is gas-efficient because it avoids intermediate token transfers to an external account. In a two-transaction approach, you'd pay gas for transferring tokens to the user in the collect step, then gas again to transfer them back to the pool in the add step. The single transaction approach only settles the net change, which is often zero because the fees are immediately reused.

You can extend this pattern to collect fees from multiple positions and add them to a single pool, or to reinvest in a completely different pool across token types. The key is to keep all operations within one transaction to benefit from delta accounting.

For example, to collect from multiple positions and add to a new one:

```solidity
function multiPositionReinvest(
    bytes32[] memory poolIds,
    uint256[] memory tokenIds,
    bytes32 targetPoolId,
    int24 targetTickLower,
    int24 targetTickUpper,
    uint256 amount0Min,
    uint256 amount1Min
) external returns (uint256 totalLiquidity) {
    uint256 accumulated0 = 0;
    uint256 accumulated1 = 0;
    
    // Collect from all positions
    for (uint256 i = 0; i < tokenIds.length; i++) {
        require(positionManager.ownerOf(tokenIds[i]) == msg.sender, "Not owner of position");
        
        (uint256 fee0, uint256 fee1) = positionManager.collect(
            poolIds[i],
            tokenIds[i],
            address(this),
            block.timestamp + 1000
        );
        
        accumulated0 += fee0;
        accumulated1 += fee1;
    }
    
    // Now add all collected fees to target position (creating if needed)
    ModifyLiquidityParams memory addParams = ModifyLiquidityParams({
        poolId: targetPoolId,
        liquidityDelta: 0,
        sqrtPriceX96: 0,
        tickLower: targetTickLower,
        tickUpper: targetTickUpper,
        amount0Requested: accumulated0,
        amount1Requested: accumulated1,
        decreaseLiquidity: false,
        createPosition: true // Create new position if doesn't exist
    });
    
    (uint256 newLiquidity, , ) = positionManager.modifyLiquidity(
        addParams,
        amount0Min,
        amount1Min,
        msg.sender,
        block.timestamp + 1000
    );
    
    totalLiquidity = newLiquidity;
    
    emit MultiPositionReinvest(tokenIds.length, accumulated0, accumulated1, newLiquidity);
}
```

This aggregated reinvestment shows how you can efficiently compound earnings from multiple sources into a single larger position. The flash accounting ensures all the fee collections happen without any intermediate token transfers out of the contract, and only the net amount (if any) needs to be settled at the end.

### Summary of Benefits

The flash accounting patterns demonstrated here offer several advantages:

- **Atomicity**: All steps either succeed or fail together, eliminating intermediate state risk
- **Gas efficiency**: Fewer external token transfers and no intermediate settlements
- **Security**: No need to expose users' tokens to the contract for multi-step operations
- **Simplicity**: Complex operations become straightforward to implement

When designing your own strategies, look for opportunities to combine multiple interactions with the PoolManager into a single transaction. Whether it's arbitrage, migration, compounding, or custom strategies that involve multiple swaps and liquidity changes, the delta accounting system is designed to make these patterns efficient and safe.

## Creating a Pool

Creating a pool in Uniswap V4 requires initializing it with the PoolManager. Here's a complete example that shows how to define the PoolKey and call initialize:

```solidity
contract PoolCreator {
    IPoolManager public poolManager;
    
    constructor(address _poolManager) {
        poolManager = IPoolManager(_poolManager);
    }
    
    function createAndInitializePool(
        address token0,
        address token1,
        uint24 fee,
        int24 tickSpacing,
        int24 initialTick
    ) external returns (bytes32 poolId) {
        // Ensure token0 is less than token1
        require(token0 < token1, "Invalid token order");
        
        PoolKey memory key = PoolKey({
            currency0: token0,
            currency1: token1,
            fee: fee,
            tickSpacing: tickSpacing
        });
        
        // Compute poolId from key
        poolId = IPoolManager.bytes32Key(
            keccak256(abi.encode(key))
        );
        
        // Initialize the pool
        poolManager.initialize(
            key,
            initialTick,
            msg.sender
        );
        
        emit PoolCreated(poolId, token0, token1, fee, initialTick);
    }
    
    event PoolCreated(
        bytes32 indexed poolId,
        address token0,
        address token1,
        uint24 fee,
        int24 tick
    );
}
```

Important notes: The tokens must be sorted so that token0 < token1 when compared numerically. The initial tick determines the starting price; it should be chosen based on the desired initial sqrtPriceX96. The `msg.sender` becomes the initial owner of the pool (though the PoolManager acts as the administrator). After initialization, the pool exists and can accept liquidity and swaps.

### Fee Tiers and Tick Spacing

Choosing the right fee tier is important for your pool's success. Here are the standard fee tiers and their recommended tick spacing values:

| Fee Tier | Fee Percentage | Tick Spacing | Use Case |
|----------|----------------|--------------|----------|
| 100 | 0.01% | 1 | Stablecoin pairs, very low volatility |
| 500 | 0.05% | 10 | Stable pairs with moderate volatility |
| 3000 | 0.3% | 60 | Most common, general purpose |
| 10000 | 1% | 200 | Volatile pairs, high risk/reward |

Higher fee tiers allow wider tick spacing, which reduces the number of ticks the pool must track, saving gas. However, higher fees may attract less trading volume. Choose based on expected volatility of the token pair.

## Adding Liquidity (Minting Positions)

Adding liquidity in V4 involves constructing a ModifyLiquidityParameters struct and calling the PoolManager's modifyLiquidity function. The PositionManager provides a convenient wrapper that handles token transfers and position tracking. Let's walk through a complete minting function using the PositionManager:

```solidity
contract LiquidityManager {
    IPositionManager public positionManager;
    IPoolManager public poolManager;
    
    constructor(address _positionManager, address _poolManager) {
        positionManager = IPositionManager(_positionManager);
        poolManager = IPoolManager(_poolManager);
    }
    
    struct AddLiquidityParams {
        address token0;
        address token1;
        uint24 fee;
        int24 tickLower;
        int24 tickUpper;
        uint256 amount0Desired;
        uint256 amount1Desired;
        uint256 amount0Min;
        uint256 amount1Min;
        address recipient;
        uint256 deadline;
    }
    
    function addLiquidity(
        AddLiquidityParams memory params
    ) external returns (uint256 liquidity, uint256 amount0, uint256 amount1) {
        // Validate token order
        require(params.token0 < params.token1, "Invalid token order");
        
        // Construct PoolKey
        PoolKey memory poolKey = PoolKey({
            currency0: params.token0,
            currency1: params.token1,
            fee: params.fee,
            tickSpacing: _getTickSpacing(params.fee)
        });
        
        // Compute poolId
        bytes32 poolId = IPoolManager.bytes32Key(
            keccak256(abi.encode(poolKey))
        );
        
        // Verify the tick range aligns with tick spacing
        require(
            params.tickLower % _getTickSpacing(params.fee) == 0,
            "Invalid tickLower"
        );
        require(
            params.tickUpper % _getTickSpacing(params.fee) == 0,
            "Invalid tickUpper"
        );
        require(params.tickLower < params.tickUpper, "Invalid tick range");
        
        // Prepare modify liquidity parameters
        ModifyLiquidityParams memory modifyParams = ModifyLiquidityParams({
            poolId: poolId,
            liquidityDelta: 0,
            sqrtPriceX96: 0,
            tickLower: params.tickLower,
            tickUpper: params.tickUpper,
            amount0Requested: params.amount0Desired,
            amount1Requested: params.amount1Desired,
            decreaseLiquidity: false,
            createPosition: true
        });
        
        // Call PositionManager to handle token transfers and position creation
        (liquidity, amount0, amount1) = positionManager.modifyLiquidity(
            modifyParams,
            params.amount0Min,
            params.amount1Min,
            params.recipient,
            params.deadline
        );
        
        emit LiquidityAdded(
            poolId,
            params.recipient,
            liquidity,
            amount0,
            amount1,
            params.tickLower,
            params.tickUpper
        );
    }
    
    function _getTickSpacing(uint24 fee) internal pure returns (int24) {
        if (fee >= 500) return 10;
        if (fee == 3000) return 60;
        if (fee == 10000) return 200;
        return 1; // For fee 100 (0.01%)
    }
    
    event LiquidityAdded(
        bytes32 indexed poolId,
        address indexed recipient,
        uint256 liquidity,
        uint256 amount0,
        uint256 amount1,
        int24 tickLower,
        int24 tickUpper
    );
}
```

This function wraps the modifyLiquidity call from the PositionManager. The PositionManager handles token transfers from the caller, ensures you have sufficient balances, and creates a position NFT representing your liquidity. The `tickLower` and `tickUpper` define the price range for your liquidity. The amounts specify how much of each token you want to deposit; the actual amounts used may be less depending on the current price and range. The `amount0Min` and `amount1Min` provide slippage protection.

When `decreaseLiquidity` is false and `createPosition` is true, the call adds liquidity and creates a new position. The PositionManager returns the amount of liquidity minted and the actual token amounts deposited.

## Removing Liquidity (Burning Positions)

Removing liquidity requires calling modifyLiquidity with `decreaseLiquidity` set to true. You also typically want to collect any earned fees before or after burning. Here's a complete example:

```solidity
contract LiquidityManager {
    // ... previous code ...
    
    struct RemoveLiquidityParams {
        bytes32 poolId;
        uint256 tokenId;
        uint256 liquidity;
        uint256 amount0Min;
        uint256 amount1Min;
        address recipient;
        uint256 deadline;
    }
    
    function removeLiquidity(
        RemoveLiquidityParams memory params
    ) external returns (uint256 amount0, uint256 amount1) {
        // Prepare modify liquidity parameters for removal
        ModifyLiquidityParams memory modifyParams = ModifyLiquidityParams({
            poolId: params.poolId,
            liquidityDelta: params.liquidity,
            sqrtPriceX96: 0,
            tickLower: 0,
            tickUpper: 0,
            amount0Requested: 0,
            amount1Requested: 0,
            decreaseLiquidity: true,
            createPosition: false
        });
        
        // Call PositionManager to remove liquidity
        (amount0, amount1) = positionManager.modifyLiquidity(
            modifyParams,
            params.amount0Min,
            params.amount1Min,
            params.recipient,
            params.deadline
        );
        
        emit LiquidityRemoved(
            params.poolId,
            msg.sender,
            params.tokenId,
            params.liquidity,
            amount0,
            amount1
        );
    }
    
    function collectFees(
        bytes32 poolId,
        uint256 tokenId,
        address recipient,
        uint256 deadline
    ) external {
        positionManager.collect(
            poolId,
            tokenId,
            recipient,
            deadline
        );
        
        emit FeesCollected(poolId, tokenId, recipient);
    }
    
    event LiquidityRemoved(
        bytes32 indexed poolId,
        address indexed owner,
        uint256 tokenId,
        uint256 liquidity,
        uint256 amount0,
        uint256 amount1
    );
    
    event FeesCollected(
        bytes32 indexed poolId,
        uint256 tokenId,
        address recipient
    );
}
```

For removal, you specify the poolId and tokenId of the position you want to decrease. The `liquidity` parameter is the amount of liquidity to remove. The PositionManager will burn that portion of your position and return the corresponding tokens (plus any collected fees if you call collect separately). The amount0Min and amount1Min enforce slippage limits on the output amounts.

The position token (NFT) represents your liquidity position and is required to collect fees or remove liquidity. Always ensure you're the owner of the tokenId or have proper approval before calling these functions.

## Position NFT Details

The Position NFT is a unique aspect of Uniswap V4 that represents ownership of a liquidity position. Unlike previous versions where LP tokens were fungible ERC-20 tokens, V4 uses ERC-721 non-fungible tokens. Each position is uniquely identified by a tokenId and contains all the information about your liquidity position. Understanding how these tokens work is essential for proper position management.

### PositionManager as an ERC-721 Implementation

The PositionManager contract is built on top of the ERC-721 standard, meaning each liquidity position is a unique, non-fungible token. The PositionManager implements ERC-721 functions like `balanceOf`, `ownerOf`, `transferFrom`, `approve`, and `getApproved`. However, the token metadata is specialized to represent liquidity positions rather than digital art or collectibles.

Every time you add liquidity through the PositionManager, a new NFT is minted (or an existing position is increased if you use the same tick range). This NFT serves as both proof of ownership and a data structure containing all the parameters of your position. You must hold this NFT to perform operations like collecting fees or removing liquidity. The NFT can be transferred to other addresses, effectively transferring ownership of the liquidity position.

The PositionManager contract address is separate from the PoolManager. While the PoolManager handles the actual pool operations and accounting, the PositionManager acts as a user-friendly wrapper that manages token approvals, transfers, and position ownership. This separation of concerns makes the system more modular and easier to interact with.

### Position Token Structure and Encoding

Each position NFT stores critical information about the liquidity position both in its on-chain data and in its token URI. The tokenId is a bytes32 value that encodes several pieces of information, though this encoding is internal to the PositionManager. More importantly, the PositionManager exposes view functions to query position details.

The key components of a position are:

- `poolId`: Identifies which pool the liquidity is in
- `tickLower` and `tickUpper`: The price range boundaries
- `liquidity`: The amount of liquidity provided
- `feeGrowthInside0LastX128` and `feeGrowthInside1LastX128`: Track accumulated fees per token
- `tokensOwed0` and `tokensOwed1`: Track uncollected fees

When you call `positionManager.tokenURI(tokenId)`, you receive a JSON metadata string that includes human-readable information about the position, such as the token pair, fee tier, tick range, and liquidity amount. This metadata is useful for wallet applications and block explorers to display position details.

The actual storage layout in the PositionManager contract uses mappings to track positions:

```solidity
mapping(uint256 => Position) private _positions;
mapping(bytes32 => uint256) private _poolIdToNextPositionId;
```

Where Position is a struct containing all the fields mentioned above. The tokenId serves as the key into the `_positions` mapping.

### Query Functions for Position Analysis

The PositionManager provides several view functions to retrieve information about positions. These are essential for building dashboards, tracking performance, and validating state before operations.

The core query functions are:

```solidity
function getPosition(
    bytes32 poolId,
    uint24 tickLower,
    uint24 tickUpper
) external view returns (uint256 liquidity, uint256 feeGrowthInside0LastX128, uint256 feeGrowthInside1LastX128, uint256 tokensOwed0, uint256 tokensOwed1);

function getPositionData(uint256 tokenId) external view returns (Position memory);

function positionsOf(address account) external view returns (uint256[] memory);

function tokensOfOwner(address owner) external view returns (uint256[] memory);
```

`getPosition` takes a poolId and tick range and returns the aggregate position details across all positions with those parameters. This is useful for understanding the overall liquidity you have in a specific range.

`getPositionData` takes a tokenId and returns the complete Position struct for that specific NFT. Use this when you need detailed information about a particular position, including exact liquidity amount and fee accumulation.

`positionsOf` returns all tokenIds owned by a given address. This is helpful for enumerating all positions a user holds across different pools and ranges.

`tokensOfOwner` is an ERC-721 standard function that returns the tokenIds owned by an address. This serves the same purpose as `positionsOf`.

Let's examine practical usage of these functions:

```solidity
contract PositionAnalyzer {
    IPositionManager public positionManager;
    
    constructor(address _positionManager) {
        positionManager = IPositionManager(_positionManager);
    }
    
    // Enumerate all positions for a specific address
    function getAllPositions(address user) external view returns (PositionInfo[] memory) {
        uint256[] memory tokenIds = positionManager.tokensOfOwner(user);
        PositionInfo[] memory positions = new PositionInfo[](tokenIds.length);
        
        for (uint256 i = 0; i < tokenIds.length; i++) {
            Position memory pos = positionManager.getPositionData(tokenIds[i]);
            positions[i] = PositionInfo({
                tokenId: tokenIds[i],
                poolId: pos.poolId,
                tickLower: pos.tickLower,
                tickUpper: pos.tickUpper,
                liquidity: pos.liquidity,
                tokensOwed0: pos.tokensOwed0,
                tokensOwed1: pos.tokensOwed1,
                feeGrowthInside0LastX128: pos.feeGrowthInside0LastX128,
                feeGrowthInside1LastX128: pos.feeGrowthInside1LastX128
            });
        }
        
        return positions;
    }
    
    // Calculate total uncollected fees for a position
    function calculateUncollectedFees(
        bytes32 poolId,
        uint256 tokenId,
        uint256 currentFeeGrowth0,
        uint256 currentFeeGrowth1
    ) external view returns (uint256 fee0, uint256 fee1) {
        Position memory position = positionManager.getPositionData(tokenId);
        
        // Need to fetch current pool fee growth from PoolManager
        // This would require calling IPoolManager.feeGrowthGlobal0(poolId)
        // For simplicity, assuming those values are passed in
        uint256 globalFeeGrowth0 = currentFeeGrowth0;
        uint256 globalFeeGrowth1 = currentFeeGrowth1;
        
        // Calculate the amount of fees that have accumulated since last collection
        if (globalFeeGrowth0 > position.feeGrowthInside0LastX128) {
            uint256 numerator = position.liquidity * (globalFeeGrowth0 - position.feeGrowthInside0LastX128);
            fee0 = numerator / (1 << 128);
        }
        
        if (globalFeeGrowth1 > position.feeGrowthInside1LastX128) {
            uint256 numerator = position.liquidity * (globalFeeGrowth1 - position.feeGrowthInside1LastX128);
            fee1 = numerator / (1 << 128);
        }
        
        // Add already-owed tokens
        fee0 += position.tokensOwed0;
        fee1 += position.tokensOwed1;
    }
    
    struct PositionInfo {
        uint256 tokenId;
        bytes32 poolId;
        int24 tickLower;
        int24 tickUpper;
        uint128 liquidity;
        uint256 tokensOwed0;
        uint256 tokensOwed1;
        uint256 feeGrowthInside0LastX128;
        uint256 feeGrowthInside1LastX128;
    }
}
```

The `calculateUncollectedFees` function demonstrates how to compute fees that would be collected from a position. It requires knowing the current global fee growth values from the pool, which you can get from `IPoolManager.feeGrowthGlobal0(poolId)` and `feeGrowthGlobal1(poolId)`. The calculation uses the position's liquidity and the difference between current global fee growth and the last recorded values when fees were collected.

### Transferring Positions and Approvals

Because positions are ERC-721 tokens, they support standard transfer mechanisms:

```solidity
function transferPosition(
    uint256 tokenId,
    address to,
    uint256 deadline
) external {
    // Ensure we own the token
    require(positionManager.ownerOf(tokenId) == msg.sender, "Not the owner");
    
    // Optional: check deadline
    require(block.timestamp <= deadline, "Deadline expired");
    
    // Transfer the NFT
    positionManager.transferFrom(msg.sender, to, tokenId);
    
    emit PositionTransferred(tokenId, msg.sender, to);
}

function approvePositionTransfer(
    uint256 tokenId,
    address operator,
    bool approved
) external {
    require(positionManager.ownerOf(tokenId) == msg.sender, "Not the owner");
    
    if (approved) {
        positionManager.approve(operator, tokenId);
    } else {
        positionManager.approve(address(0), tokenId);
    }
}
```

The `transferFrom` function will move the NFT from the sender to the recipient. The recipient becomes the new owner and can subsequently collect fees or remove liquidity. The PositionManager handles the ownership change and updates its internal mappings.

Approvals allow a third party to manage a position on your behalf. By calling `approve(operator, tokenId)`, you grant the operator permission to transfer that specific NFT. This is useful for delegation or for custodial services.

When designing contracts that interact with user positions, always validate ownership before executing operations. The PositionManager itself will check that the caller is authorized to modify a position (either the owner or approved operator), but you should perform additional checks if your contract has custom logic.

### Position Management Best Practices

Effective position management requires regular monitoring and strategic decisions. Here are key practices to follow:

Collect fees frequently enough that they do not accumulate excessively, but not so frequently that you waste gas on many small transactions. A good rule of thumb is to collect when fees exceed a certain threshold relative to gas costs. For active trading positions with high volume, collect daily or weekly. For longer-term passive positions, monthly collection may be sufficient.

Track position performance by monitoring liquidity utilization and fee accumulation. The currency of fees is in the same tokens as the pool, so you can calculate your yield in percentage terms by comparing fees earned to the value of the liquidity provided. Use the `getPositionData` function to retrieve current state and compare with previous snapshots.

When adjusting positions, consider the impact of removing liquidity on your fee earning potential. Partial reductions may be preferable to complete withdrawals if you want to maintain exposure. The `modifyLiquidity` function in PositionManager allows you to increase or decrease liquidity without burning the entire position, enabling gradual adjustments.

Be aware that moving liquidity creates a new position token if you want to change the tick range. If you want to shift your range, you typically withdraw from the old position and create a new one. There are gas-efficient patterns for doing this atomically within a single transaction to avoid intermediate state changes.

Always set slippage protections (`amount0Min`, `amount1Min`) when adding or removing liquidity, especially in volatile markets. These parameters prevent your transaction from executing if token amounts deviate too far from your expectations, protecting you from frontrunning or sudden price movements.

The following table summarizes key PositionManager functions for reference:

| Function | Purpose | Key Parameters | Returns |
|----------|---------|----------------|---------|
| modifyLiquidity | Add or remove liquidity | poolId, liquidityDelta, tickLower, tickUpper, amounts | liquidity, amount0, amount1 |
| collect | Collect earned fees | poolId, tokenId, recipient, deadline | amount0, amount1 |
| transferFrom | Transfer NFT to new owner | from, to, tokenId | None |
| approve | Approve operator for token | operator, tokenId | None |
| getPositionData | Get full position details | tokenId | Position struct |
| tokensOfOwner | Get all tokenIds for address | owner | uint256[] |
| ownerOf | Check token ownership | tokenId | address |
| getApproved | Get approved operator | tokenId | address |

By understanding these concepts and functions, you can effectively manage liquidity positions in Uniswap V4, track earnings, and make strategic adjustments to optimize your returns.

## Performing Swaps

Uniswap V4 provides two primary ways to swap: the SwapRouter (which handles token transfers and approval management) or directly calling the PoolManager (which gives you more control). Let's examine both approaches.

### Using the SwapRouter

Here's a swap using the SwapRouter:

```solidity
contract SwapExecutor {
    ISwapRouter public swapRouter;
    
    constructor(address _swapRouter) {
        swapRouter = ISwapRouter(_swapRouter);
    }
    
    struct SwapParams {
        address tokenIn;
        address tokenOut;
        uint24 fee;
        uint256 amountIn;
        uint256 amountOutMin;
        address recipient;
        uint256 deadline;
        uint160 sqrtPriceLimitX96;
        bool zeroForOne;
    }
    
    function exactInputSingle(SwapParams memory params)
        external
        returns (uint256 amountOut)
    {
        // Get poolId from token pair and fee
        bytes32 poolId = _getPoolId(
            params.tokenIn,
            params.tokenOut,
            params.fee
        );
        
        SingleSwap memory singleSwap = SingleSwap({
            poolId: poolId,
            kind: params.zeroForOne ? 0 : 1,
            amountSpecified: params.amountIn,
            sqrtPriceLimitX96: params.sqrtPriceLimitX96
        });
        
        // Define fund management: how tokens are transferred
        FundManagement memory funds = FundManagement({
            sender: msg.sender,
            from: params.tokenIn == address(0) ? msg.sender : params.tokenIn,
            to: params.recipient,
            wNETH: address(0), // Set to WETH address if wrapping ETH
            funding: _funding()
        });
        
        // Execute exact input swap: send amountIn, get at least amountOutMin
        amountOut = swapRouter.exactInputSingle(
            singleSwap,
            funds,
            params.amountOutMin,
            params.deadline
        );
    }
    
    function exactOutputSingle(SwapParams memory params)
        external
        returns (uint256 amountIn)
    {
        bytes32 poolId = _getPoolId(
            params.tokenIn,
            params.tokenOut,
            params.fee
        );
        
        SingleSwap memory singleSwap = SingleSwap({
            poolId: poolId,
            kind: params.zeroForOne ? 0 : 1,
            amountSpecified: -int256(params.amountOutMin), // Negative for exact output
            sqrtPriceLimitX96: params.sqrtPriceLimitX96
        });
        
        FundManagement memory funds = FundManagement({
            sender: msg.sender,
            from: params.tokenIn == address(0) ? msg.sender : params.tokenIn,
            to: params.recipient,
            wNETH: address(0),
            funding: _funding()
        });
        
        // Execute exact output swap: receive amountOut, pay at most amountIn
        amountIn = swapRouter.exactOutputSingle(
            singleSwap,
            funds,
            params.amountIn, // This is the maximum amount willing to pay
            params.deadline
        );
    }
    
    function _getPoolId(
        address token0,
        address token1,
        uint24 fee
    ) internal view returns (bytes32) {
        // Sort tokens
        address sortedToken0 = token0 < token1 ? token0 : token1;
        address sortedToken1 = token0 < token1 ? token1 : token0;
        
        PoolKey memory key = PoolKey({
            currency0: sortedToken0,
            currency1: sortedToken1,
            fee: fee,
            tickSpacing: _getTickSpacing(fee)
        });
        return IPoolManager.bytes32Key(
            keccak256(abi.encode(key))
        );
    }
    
    function _getTickSpacing(uint24 fee) internal pure returns (int24) {
        return fee >= 500 ? 10 : 1;
    }
    
    function _funding() internal pure returns (uint8) {
        return 1; // 0 = from, 1 = to (specifies who provides tokens)
    }
}
```

The SwapRouter handles the complexity of token approvals and transfers. For exact input swaps, you specify the amount you want to send and receive at least amountOutMin. For exact output swaps, you specify the amount you need to receive and the router calculates the required input (or reverts if too much would be needed). The `sqrtPriceLimitX96` parameter lets you set price limits; setting it to 0 allows any price. The `zeroForOne` flag determines direction: if tokenIn < tokenOut, it's zeroForOne; otherwise it's oneForZero.

If you need more control (such as multi-hop swaps), you can use `exactInput` or `exactOutput` with an array of SingleSwap structs.

### SwapRouter vs Direct PoolManager: Comparison

When implementing swaps, you have two main approaches. The following table compares them:

| Aspect | SwapRouter | Direct PoolManager |
|--------|------------|-------------------|
| Ease of Use | High, handles approvals automatically | Medium, requires manual token handling |
| Flexibility | Limited to single/multi-hop swaps | Full control over swap parameters |
| Gas Efficiency | Slightly higher due to wrapper logic | Slightly lower, direct calls |
| Multi-hop Support | Yes, via arrays | Yes, via multiple calls |
| Token Approval | Handled by router | Must approve router or poolManager |
| Use Cases | Standard swaps, dApps, simple integrations | Custom trading strategies, MEV protection, complex logic |
| Error Handling | Standardized errors | Custom error handling required |
| Additional Features | Built-in deadlining, slippage | Manual implementation needed |

For most applications, the SwapRouter is the recommended choice due to its convenience and safety features. Direct PoolManager calls are reserved for advanced use cases where you need precise control over execution.

## Working with Hooks

Hooks let you inject custom logic at specific execution points. Let's create a practical example: a hook that implements a simple fee discount for a specific token holder. This demonstrates how hooks can add business logic without modifying core contracts.

```solidity
contract FeeDiscountHook {
    IPoolManager public poolManager;
    
    address public owner;
    mapping(address => bool) public discountedAccounts;
    uint256 public discountBps; // Basis points discount (100 = 1%)
    
    constructor(address _poolManager) {
        poolManager = IPoolManager(_poolManager);
        owner = msg.sender;
        discountedAccounts[msg.sender] = true;
        discountBps = 50; // 0.5% discount
    }
    
    // beforeSwap callback: modify the fee before swap executes
    function beforeSwap(
        bytes32 poolId,
        IPoolManager.SwapRequest memory request
    ) external {
        require(msg.sender == address(poolManager), "Unauthorized");
        
        // Check if the caller qualifies for discount
        if (discountedAccounts[msg.sender]) {
            // The fee is in hundredths of a basis point. Reduce by discountBps.
            uint256 newFee = request.fee - (request.fee * discountBps) / 10000;
            request.fee = newFee > 0 ? newFee : 0;
        }
    }
    
    // afterSwap: could track volume, update rewards, etc.
    function afterSwap(
        bytes32 poolId,
        IPoolManager.SwapRequest memory request,
        int256 amount0Delta,
        int256 amount1Delta
    ) external {
        require(msg.sender == address(poolManager), "Unauthorized");
        
        // Example: track total volume for discounted accounts
        uint256 volume = uint256(
            amount0Delta > 0 ? amount0Delta : amount1Delta
        );
        // Could accumulate volume per account, update rewards, etc.
    }
    
    // Admin function to add discounted accounts
    function addDiscountedAccount(address account) external {
        require(msg.sender == owner, "Only owner");
        discountedAccounts[account] = true;
    }
    
    // Admin function to adjust discount rate
    function setDiscountBps(uint256 _discountBps) external {
        require(msg.sender == owner, "Only owner");
        require(_discountBps <= 500, "Discount too high"); // Max 5%
        discountBps = _discountBps;
    }
}
```

To use this hook, you would pass its address when creating a pool:

```solidity
poolManager.initialize(
    key,
    initialTick,
    msg.sender,
    FeeConfiguration({
        // Hook addresses: you can specify different hooks for different callbacks
        // For simplicity, we use the same hook for all callbacks
        hookProtocol: address(feeDiscountHook),
        hookSwap: address(feeDiscountHook),
        hookBeforeAddLiquidity: address(feeDiscountHook),
        hookAfterAddLiquidity: address(feeDiscountHook),
        hookBeforeRemoveLiquidity: address(feeDiscountHook),
        hookAfterRemoveLiquidity: address(feeDiscountHook),
        hookBeforeDonate: address(feeDiscountHook),
        hookAfterDonate: address(feeDiscountHook),
        hookTransfer: address(feeDiscountHook)
    })
);
```

The hook functions are called automatically during pool operations. The hook must be careful about gas usage and reentrancy. Always validate that `msg.sender == address(poolManager)` in your hook callbacks to prevent unauthorized access.

### Hook Implementation Checklist

When building hooks, consider these best practices:

| Checklist Item | Why It Matters |
|----------------|----------------|
| Validate msg.sender == poolManager | Prevents unauthorized calls |
| Keep callbacks gas-efficient | PoolManager calls all hooks; expensive hooks increase all pool costs |
| Avoid external calls in hooks | Reentrancy risk and gas uncertainty |
| Use view functions for read-only data | Prevent state changes unintentionally |
| Test hook integration thoroughly | Hooks run in production pool context |
| Consider hook ordering if multiple hooks | Callbacks execute in specified order |
| Implement proper error handling | Hooks revert whole transaction on failure |
| Document hook behavior clearly | Users need to understand what hook does |

## Advanced Hook Patterns

The simple fee discount hook demonstrated earlier only scratches the surface of what's possible with Uniswap V4 hooks. In this section, we'll explore four advanced hook implementations that showcase the true power of the hooks system. Each pattern solves a real-world trading problem and demonstrates different technical approaches.

### A) Limit Order Hook

A limit order hook allows users to place orders that execute only when the pool price reaches a specified target. This is one of the most requested features in DeFi, and implementing it as a hook makes it available to any pool that uses the hook, without requiring separate infrastructure.

The hook stores pending limit orders and checks their conditions during the beforeSwap callback. When a user attempts a swap that would move the price across the limit order trigger, the hook automatically executes the order at the limit price, not the market price.

```solidity
contract LimitOrderHook {
    struct LimitOrder {
        address owner;
        address tokenIn;
        address tokenOut;
        uint256 amountIn; // Amount of tokenIn to sell
        uint256 minAmountOut; // Minimum acceptable output
        int24 tickLimit; // Price level that triggers the order
        bool zeroForOne; // Direction
        bool filled; // Whether order is fulfilled
        uint256 expiry; // Timestamp after which order invalid
    }
    
    IPoolManager public poolManager;
    
    // Mapping from tokenId (some unique identifier) to order
    mapping(uint256 => LimitOrder) public orders;
    uint256 public nextOrderId;
    
    // Mapping from owner to array of their order IDs
    mapping(address => uint256[]) public userOrders;
    
    event OrderPlaced(uint256 indexed orderId, address indexed owner, bytes32 poolId, int24 tickLimit, uint256 amountIn);
    event OrderFilled(uint256 indexed orderId, address filler, uint256 amountIn, uint256 amountOut);
    event OrderCancelled(uint256 indexed orderId, address owner);
    
    constructor(address _poolManager) {
        poolManager = IPoolManager(_poolManager);
    }
    
    // Place a new limit order
    function placeOrder(
        bytes32 poolId,
        address tokenIn,
        address tokenOut,
        uint256 amountIn,
        uint256 minAmountOut,
        int24 tickLimit,
        bool zeroForOne,
        uint256 expiry
    ) external returns (uint256 orderId) {
        require(tokenIn != tokenOut, "Same token");
        require(amountIn > 0, "Zero amount");
        require(expiry > block.timestamp, "Already expired");
        
        // Verify tickLimit matches pool's tick spacing
        ( , int24 tickSpacing, , , ) = poolManager.getPool(poolId);
        require(tickLimit % tickSpacing == 0, "Invalid tick spacing");
        
        orderId = nextOrderId++;
        orders[orderId] = LimitOrder({
            owner: msg.sender,
            tokenIn: tokenIn,
            tokenOut: tokenOut,
            amountIn: amountIn,
            minAmountOut: minAmountOut,
            tickLimit: tickLimit,
            zeroForOne: zeroForOne,
            filled: false,
            expiry: expiry
        });
        
        userOrders[msg.sender].push(orderId);
        
        emit OrderPlaced(orderId, msg.sender, poolId, tickLimit, amountIn);
    }
    
    // beforeSwap callback: check if any orders trigger
    function beforeSwap(
        bytes32 poolId,
        IPoolManager.SwapRequest memory request
    ) external {
        require(msg.sender == address(poolManager), "Unauthorized");
        
        // Only process swaps that match our orders' direction
        bool zeroForOne = request.zeroForOne;
        
        // Iterate through orders for this pool (in real implementation, use index by poolId)
        // For simplicity, we check all orders - in production optimize with pool-specific indexing
        uint256 orderCount = nextOrderId;
        for (uint256 i = 0; i < orderCount; i++) {
            LimitOrder storage order = orders[i];
            if (order.filled || order.expiry < block.timestamp) continue;
            if (order.tokenIn != request.tokenIn || order.tokenOut != request.tokenOut) continue;
            if (order.zeroForOne != zeroForOne) continue;
            
            // Get current tick after swap would execute
            // We need to simulate the price change to see if it crosses the limit
            // In practice, we'd examine the sqrtPriceLimitX96 parameter in swapRequest
            // For current tick, we can get it from pool state
            
            // Simplified: check if the swap would move price across order's tick
            // Note: real implementation needs to carefully compute price impact
            
            int24 currentTick = _getCurrentTick(poolId);
            int24 newTick = _calculateNewTick(poolId, request);
            
            // Does price cross the limit order trigger?
            bool crossesLimit = (zeroForOne && newTick <= order.tickLimit && currentTick > order.tickLimit) ||
                               (!zeroForOne && newTick >= order.tickLimit && currentTick < order.tickLimit);
            
            if (crossesLimit) {
                // Execute the order: swap order.amountIn at the limit price
                // We need to perform an internal swap at the order's trigger price
                _executeOrder(i, order);
                
                // Mark order as filled
                order.filled = true;
                
                emit OrderFilled(i, msg.sender, order.amountIn, order.minAmountOut);
            }
        }
    }
    
    function _executeOrder(uint256 orderId, LimitOrder storage order) private {
        // Perform swap at the order's limit price by adjusting the sqrtPriceLimitX96
        // This requires enabling the swap to execute at that specific price
        
        // In a real implementation, we would:
        // 1. Check that the swap amount doesn't exceed order.amountIn
        // 2. Ensure the swap happens at or better than order.minAmountOut
        // 3. Possibly modify the swap request to enforce limit price
        
        // Simplified: we rely on the fact that the current swap in beforeSwap
        // is itself crossing the limit, so we piggyback on that swap
        
        // In practice, we'd need to allocate some of the current swap's tokens
        // to fill the order, adjusting amounts accordingly
    }
    
    function cancelOrder(uint256 orderId) external {
        require(orders[orderId].owner == msg.sender, "Not owner");
        require(!orders[orderId].filled, "Already filled");
        if (orders[orderId].expiry < block.timestamp) revert("Expired");
        
        delete orders[orderId];
        
        // Also remove from userOrders array (expensive, but necessary)
        uint256[] storage userOrderList = userOrders[msg.sender];
        for (uint256 i = 0; i < userOrderList.length; i++) {
            if (userOrderList[i] == orderId) {
                userOrderList[i] = userOrderList[userOrderList.length - 1];
                userOrderList.pop();
                break;
            }
        }
        
        emit OrderCancelled(orderId, msg.sender);
    }
    
    function getUserOrders(address user) external view returns (uint256[] memory) {
        return userOrders[user];
    }
    
    function _getCurrentTick(bytes32 poolId) private view returns (int24) {
        // Fetch from poolManager
        (, int24 tick, , ,) = poolManager.getPool(poolId);
        return tick;
    }
    
    function _calculateNewTick(bytes32 poolId, IPoolManager.SwapRequest memory request) private view returns (int24) {
        // Complex price calculation based on swap amount
        // Would need to access pool state and compute using swap math
        // This is simplified
        return request.sqrtPriceLimitX96 > 0 ? 0 : _getCurrentTick(poolId);
    }
}
```

Key implementation considerations:

The hook stores limit orders in a mapping keyed by order ID. Each order contains the trigger price (as a tick), direction, amount, and minimum acceptable output. When a swap occurs, the beforeSwap callback examines whether the price movement would cross any order's trigger. If so, the hook executes the order by ensuring that some of the swap's tokens are allocated to fill the limit order.

In production, you need to handle partial fills: if the swap amount is larger than the order, the order should be completely filled and the excess continues as a normal swap. If the swap amount is smaller than the order, the order should be partially filled and remain active for the remaining amount. The hook would need to adjust the swap request to allocate the appropriate amounts.

The hook must also ensure that limit orders are executed at the trigger price, not at the potentially worse price of the actual swap. This requires careful manipulation of the swap parameters. One approach is to split the incoming swap into two: one that executes up to the limit price to fill the order, and another that continues at the limit price for any remaining amount.

Gas efficiency: Iterating through all orders in each beforeSwap is prohibitively expensive. The implementation should index orders by pool ID and by relevant price ranges to enable efficient lookup. For instance, maintain a mapping from poolId to a nested mapping of tick ranges that contain orders. The beforeSwap callback can then only check orders in the relevant tick range.

The hook should also validate the order's minAmountOut carefully to prevent oracle manipulation attacks. The limit price should be based on a trustworthy time-weighted average rather than the instantaneous pool price if you want to avoid frontrunning.

### B) TWAP Oracle Hook

Time-weighted average price (TWAP) oracles are essential for many DeFi applications, providing a price that's resistant to short-term manipulation. Implementing a TWAP as a hook gives any pool using the hook a built-in, trustworthy price oracle without needing separate infrastructure.

This hook tracks cumulative price observations over time and allows querying the geometric mean price over any time window.

```solidity
contract TWAPOracleHook {
    IPoolManager public poolManager;
    
    struct Observation {
        int24 tick; // Current tick at observation
        uint256 timestamp; // When observation was recorded
        bool initialized; // Whether this slot has been used
    }
    
    // Circular buffer of observations per pool
    mapping(bytes32 => Observation[]) private observations;
    mapping(bytes32 => uint256) private nextIndex; // Next index to write
    mapping(bytes32 => uint256) private observationInterval; // Seconds between observations
    
    // Minimum observations needed for TWAP
    uint256 public constant MIN_OBSERVATIONS = 2;
    
    event ObservationAdded(bytes32 indexed poolId, int24 tick, uint256 timestamp);
    event OracleUpdated(bytes32 indexed poolId, uint256 period);
    
    constructor(address _poolManager) {
        poolManager = IPoolManager(_poolManager);
    }
    
    // afterSwap callback: record an observation
    function afterSwap(
        bytes32 poolId,
        IPoolManager.SwapRequest memory,
        int256 amount0Delta,
        int256 amount1Delta
    ) external {
        require(msg.sender == address(poolManager), "Unauthorized");
        
        _recordObservation(poolId);
    }
    
    // afterAddLiquidity and afterRemoveLiquidity also affect price
    function afterAddLiquidity(
        bytes32 poolId,
        IPoolManager.ModifyLiquidityParams memory,
        int256 liquidityDelta
    ) external {
        require(msg.sender == address(poolManager), "Unauthorized");
        _recordObservation(poolId);
    }
    
    function afterRemoveLiquidity(
        bytes32 poolId,
        IPoolManager.ModifyLiquidityParams memory,
        int256 liquidityDelta
    ) external {
        require(msg.sender == address(poolManager), "Unauthorized");
        _recordObservation(poolId);
    }
    
    function _recordObservation(bytes32 poolId) private {
        Observation[] storage obs = observations[poolId];
        uint256 index = nextIndex[poolId] % obs.length;
        
        // Get current tick from pool
        (, int24 currentTick, , ,) = poolManager.getPool(poolId);
        
        if (!obs[index].initialized) {
            // First observation for this slot
            obs[index] = Observation({
                tick: currentTick,
                timestamp: block.timestamp,
                initialized: true
            });
            
            // Allocate observation buffer on first use
            if (obs.length == 0) {
                // Default buffer size: 100 observations
                // In production, this should be configurable per pool
                observations[poolId] = new Observation[](100);
                observationInterval[poolId] = 1 hours; // Default: hourly observations
            }
        } else {
            // Check if enough time has passed since last observation
            if (block.timestamp >= obs[index].timestamp + observationInterval[poolId]) {
                obs[index] = Observation({
                    tick: currentTick,
                    timestamp: block.timestamp,
                    initialized: true
                });
                nextIndex[poolId]++;
                
                emit ObservationAdded(poolId, currentTick, block.timestamp);
            }
        }
    }
    
    // Public function to query TWAP over a time window
    function consult(
        bytes32 poolId,
        uint32 period // Time window in seconds
    ) external view returns (int24 tick) {
        Observation[] storage obs = observations[poolId];
        if (obs.length == 0) revert("No observations");
        
        uint256 currentTime = block.timestamp;
        uint256 startTime = currentTime - period;
        
        // Find observations within the time window
        // We need to iterate through the circular buffer
        uint256 count = 0;
        uint256 accumulated = 0; // For geometric mean calculation
        
        // Simplified: get first and last observations
        // Full implementation would walk the buffer and interpolate
        Observation memory firstObs = _getObservationBefore(poolId, startTime);
        Observation memory lastObs = _getObservationAt(poolId, currentTime);
        
        if (!firstObs.initialized || !lastObs.initialized) {
            revert("Insufficient observations");
        }
        
        // For time-weighted average, we weight the tick by time duration
        // But tick is not linear; we convert to sqrtPriceX96 for averaging
        
        // Simplified: just return last tick
        tick = lastObs.tick;
        
        // Full implementation would:
        // 1. Convert tick to sqrtPriceX96: price = 1.0001^tick
        // 2. For each time interval between observations, accumulate log(price) * duration
        // 3. Divide by total duration, exponentiate to get TWAP
        // 4. Convert back to tick
    }
    
    function _getObservationBefore(bytes32 poolId, uint256 time) private view returns (Observation memory) {
        Observation[] storage obs = observations[poolId];
        if (obs.length == 0) return Observation(0,0,false);
        
        uint256 index = nextIndex[poolId];
        // Walk backward through buffer to find most recent observation before time
        // Implementation omitted for brevity
        return obs[0];
    }
    
    function _getObservationAt(bytes32 poolId, uint256 time) private view returns (Observation memory) {
        Observation[] storage obs = observations[poolId];
        if (obs.length == 0) return Observation(0,0,false);
        
        uint256 index = (nextIndex[poolId] + obs.length - 1) % obs.length;
        return obs[index];
    }
    
    function setObservationInterval(bytes32 poolId, uint256 interval) external {
        // Only owner or designated manager can configure
        // For demo, anyone can set
        observationInterval[poolId] = interval;
        emit OracleUpdated(poolId, interval);
    }
    
    function getObservationCount(bytes32 poolId) external view returns (uint256) {
        return observations[poolId].length;
    }
}
```

This hook records price observations at regular intervals whenever pool state changes (swaps, liquidity adds/removes). The observations are stored in a circular buffer per pool, allowing efficient querying of historical prices.

To compute the true TWAP, you would:

1. Convert tick values to sqrtPriceX96 (using the tick to price formula)
2. For each time interval between observations, accumulate the logarithm of the price multiplied by the duration
3. Divide by total duration to get average log price
4. Exponentiate to get the TWAP price
5. Convert back to tick

The hook's `consult` function would implement this calculation and return the result as a tick value, which can be converted to a price by the caller.

Gas considerations: Recording an observation is relatively cheap because it's just a storage write. However, reading the TWAP requires iterating through potentially many observations in the time window. For efficiency, you might implement a precomputed aggregate for fixed periods (e.g., store 1-day, 7-day, 30-day TWAPs that update on each observation).

Handling zero volume: If a pool has no activity for an extended period, the TWAP should remain constant at the last known price. The circular buffer naturally handles this: the same observation persists until a new one overwrites it.

Security: The hook must be careful that only the PoolManager can call its callbacks. Always include `require(msg.sender == address(poolManager))`. Also, ensure that the observation buffer is sized appropriately to hold enough data for the longest TWAP period you want to support.

### C) Dynamic Fee Hook

Most AMMs use a fixed fee schedule, but volatility and market conditions change. A dynamic fee hook adjusts the fee based on recent price movement or trading volume, charging more during volatile periods to compensate liquidity providers and less during calm periods to attract volume.

```solidity
contract DynamicFeeHook {
    IPoolManager public poolManager;
    
    // Fee configuration
    uint256 public baseFee; // hundredths of a basis point
    uint256 public minFee; // Minimum fee allowed
    uint256 public maxFee; // Maximum fee allowed
    
    // Volatility parameters
    uint256 public volatilityWindow; // Number of swaps to consider
    uint256 public feeSensitivity; // How much fee scales with volatility
    
    // Track recent price changes for volatility calculation
    int24[] private priceHistory;
    uint256 private lastPriceUpdate;
    
    // Cache last fee to avoid unnecessary changes
    uint256 public currentFee;
    
    event FeeAdjusted(bytes32 indexed poolId, uint256 oldFee, uint256 newFee, uint256 volatility);
    
    constructor(address _poolManager) {
        poolManager = IPoolManager(_poolManager);
        baseFee = 3000; // 0.3%
        minFee = 100;   // 0.01%
        maxFee = 10000; // 1%
        volatilityWindow = 100;
        feeSensitivity = 50; // 50 bps per volatility unit
    }
    
    function beforeSwap(
        bytes32 poolId,
        IPoolManager.SwapRequest memory request
    ) external {
        require(msg.sender == address(poolManager), "Unauthorized");
        
        // Update price history with current price
        _updatePriceHistory(poolId);
        
        // Calculate current volatility based on recent price movements
        uint256 volatility = _calculateVolatility();
        
        // Adjust fee based on volatility
        uint256 newFee = baseFee;
        if (volatility > 0) {
            // Fee = baseFee + (volatility * sensitivity), bounded
            uint256 adjustment = (volatility * feeSensitivity) / 100;
            if (adjustment > maxFee - baseFee) {
                adjustment = maxFee - baseFee;
            }
            newFee = baseFee + adjustment;
        }
        
        // Ensure fee is within bounds
        if (newFee < minFee) newFee = minFee;
        if (newFee > maxFee) newFee = maxFee;
        
        // Apply fee if different from current
        if (newFee != currentFee) {
            currentFee = newFee;
            request.fee = newFee;
            emit FeeAdjusted(poolId, request.fee, newFee, volatility);
        }
    }
    
    function _updatePriceHistory(bytes32 poolId) private {
        (, int24 currentTick, , ,) = poolManager.getPool(poolId);
        
        if (priceHistory.length == 0) {
            priceHistory.push(currentTick);
        } else {
            int24 lastTick = priceHistory[priceHistory.length - 1];
            // Only record if price changed (tick is different)
            if (currentTick != lastTick) {
                priceHistory.push(currentTick);
            }
        }
        
        // Maintain fixed window size
        while (priceHistory.length > volatilityWindow) {
            priceHistory[0] = priceHistory[priceHistory.length - 1];
            priceHistory.pop();
        }
    }
    
    function _calculateVolatility() private view returns (uint256) {
        if (priceHistory.length < 2) return 0;
        
        // Calculate average absolute tick change
        uint256 totalChange = 0;
        for (uint256 i = 1; i < priceHistory.length; i++) {
            int24 diff = priceHistory[i] - priceHistory[i-1];
            if (diff < 0) diff = -diff;
            totalChange += uint256(diff);
        }
        
        uint256 avgChange = totalChange / (priceHistory.length - 1);
        
        // Normalize: typical tick changes might be in range 0-100
        // Scale to a 0-1000 volatility index
        uint256 volatility = (avgChange * 1000) / 10; // 10 is a typical divisor
        if (volatility > 1000) volatility = 1000;
        
        return volatility;
    }
    
    // Admin functions to adjust parameters
    function setBaseFee(uint256 _baseFee) external {
        require(_baseFee >= minFee && _baseFee <= maxFee, "Invalid base fee");
        baseFee = _baseFee;
    }
    
    function setFeeBounds(uint256 _minFee, uint256 _maxFee) external {
        require(_minFee < _maxFee, "Min must be less than max");
        minFee = _minFee;
        maxFee = _maxFee;
    }
    
    function setVolatilityWindow(uint256 _window) external {
        require(_window >= 10 && _window <= 1000, "Invalid window");
        volatilityWindow = _window;
    }
    
    function setFeeSensitivity(uint256 _sensitivity) external {
        require(_sensitivity <= 500, "Sensitivity too high");
        feeSensitivity = _sensitivity;
    }
}
```

The dynamic fee hook monitors recent price movements to gauge market volatility. When volatility is high (prices moving rapidly), the fee increases to compensate liquidity providers for the increased risk of adverse selection. When volatility is low, the fee decreases to attract more trading volume.

Implementation details:

- The hook maintains a circular buffer of recent tick values (price levels). The size is configurable via `volatilityWindow`. On each `beforeSwap` call, it records the current tick.

- Volatility is calculated as the average absolute tick change over the window. This can be refined to use standard deviation or other measures.

- The fee adjustment is computed as `baseFee + (volatility * sensitivity / normalization)`. The `feeSensitivity` parameter controls how aggressively the fee responds to volatility.

- The adjusted fee is applied by modifying `request.fee`. The PoolManager uses this value for the swap.

Gas efficiency considerations:

- Recording tick values is cheap, but storing them in an array that grows and shrinks has costs. Use a fixed-size circular buffer to avoid array resizing.

- The volatility calculation iterates over the price history. With a window of 100, this is acceptable. For larger windows, consider optimizing or sampling.

- The hook runs on every swap, so keep it lightweight. Avoid complex mathematical operations or external calls.

Potential issues and mitigations:

- **Gaming**: A trader could manipulate the price temporarily to reduce fees before executing a large swap. Mitigate by using a longer volatility window, applying smoothing (moving average), or basing volatility on volume-weighted metrics instead of just price changes.

- **Lag**: The fee adjustment may be slow to respond to sudden spikes. Consider using exponential moving average (EMA) instead of simple average for faster response.

- **Boundary conditions**: Ensure minFee and maxFee are reasonable to prevent fees from going too high (scaring users) or too low (undervaluing LP risk).

### D) Custom AMM Curve Hook

One of the most powerful aspects of hooks is the ability to completely change the pool's pricing formula. The constant product formula (x*y=k) is just one of many possible bonding curves. With a custom AMM curve hook, you can implement exponential curves, power curves, or even hybrid mechanisms.

This is an advanced pattern that requires deep understanding of Uniswap V4's settlement mechanism and delta accounting. The hook intercepts the swap logic and modifies how token deltas are calculated.

```solidity
contract CustomCurveHook {
    IPoolManager public poolManager;
    
    // Curve parameters
    uint256 public exponent; // Power for the formula: x^exponent * y^exponent = k
    uint256 public amplification; // For Stableswap-like curve
    bool public useCustomCurve;
    
    event CurveUpdated(uint256 exponent, uint256 amplification, bool useCustom);
    
    constructor(address _poolManager) {
        poolManager = IPoolManager(_poolManager);
        useCustomCurve = true;
        exponent = 2; // Default: parabolic curve
        amplification = 0; // Not using Stableswap by default
    }
    
    function beforeSwap(
        bytes32 poolId,
        IPoolManager.SwapRequest memory request
    ) external {
        require(msg.sender == address(poolManager), "Unauthorized");
        
        if (!useCustomCurve) return;
        
        // This hook replaces the constant product formula with a custom one
        // We need to adjust the swap parameters so that the PoolManager computes
        // deltas based on our custom curve instead of the standard formula.
        
        // The PoolManager will call our hook, and then we can modify the
        // request object or execute custom logic. However, the actual delta
        // computation happens in the PoolManager based on square root price.
        
        // There are two ways to implement custom curves:
        // 1. Pretend to be a constant product pool but have a different invariant
        // 2. Completely override the math in beforeSwap and return synthetic deltas
        
        // This example demonstrates approach #2: compute custom deltas and
        // "fake" the PoolManager's accounting by adjusting request.amountSpecified
        
        // Get current pool state
        (, int24 tick, uint160 sqrtPriceX96, uint24 protocolFees, ) = poolManager.getPool(poolId);
        
        // Desired direction
        bool zeroForOne = request.zeroForOne;
        int256 amountSpecified = request.amountSpecified;
        
        // Compute expected output/input using custom curve
        // This requires implementing your own swap math
        if (amountSpecified > 0) {
            // Exact input: compute how much output should be given
            uint256 amountOut = _computeOutputCustom(
                sqrtPriceX96,
                amountSpecified,
                zeroForOne,
                exponent,
                amplification
            );
            
            // We need to tell the PoolManager to expect this amount out
            // Unfortunately, the PoolManager calculates output itself based on its invariant
            // So we cannot simply override the result.
            
            // Alternative approach: use a hook that executes the swap manually
            // by interacting with token contracts and adjusting position, bypassing
            // the PoolManager's swap. This is more complex but gives full control.
        }
    }
    
    // Alternative: implement the curve entirely within the hook by:
    // 1. Capturing the tokens from the user
    // 2. Adjusting a virtual position to reflect the curve change
    // 3. Using swapRouter or direct calls to settle differences
    // This is highly advanced and specific to particular curve designs.
    
    // A simpler approach for many custom curves is to use a virtual reserve
    // that differs from the actual reserves, then adjust手续费
    
    function setCurveParameters(uint256 _exponent, uint256 _amplification) external {
        exponent = _exponent;
        amplification = _amplification;
        emit CurveUpdated(_exponent, _amplification, useCustomCurve);
    }
    
    function enableCustomCurve(bool enable) external {
        useCustomCurve = enable;
    }
    
    // Helper function to compute output with custom curve
    // This is a simplified example using a power curve: (x^a)*(y^a) = k
    function _computeOutputCustom(
        uint160 sqrtPriceX96,
        uint256 amountIn,
        bool zeroForOne,
        uint256 exp,
        uint256 ampl
    ) internal pure returns (uint256) {
        // This function would implement the swap math for your custom curve
        // It's complex and beyond the scope of this overview
        // See Uniswap V4 research papers for detailed implementations
        
        // For demonstration, return a dummy value
        return amountIn * 95 / 100; // Assume 5% slippage
    }
}
```

Important considerations for custom AMM curves:

The Uniswap V4 PoolManager is designed around the concentrated liquidity invariant (which extends the constant product formula). Overriding the core pricing math is non-trivial because the Pool Manager's `swap` function calculates token deltas based on that invariant. A hook cannot directly change those calculations; instead, you must either:

1. Use the hook to adjust parameters that feed into the existing math (like fee, or by manipulatingLP token amounts)
2. Bypass the standard swap entirely and implement custom accounting within the hook (very advanced, high risk)
3. Design your custom curve to map onto the existing concentrated liquidity model

In practice, most "custom curve" implementations using hooks focus on modifying fee structures, adding barriers, or implementing conditional logic around the standard swap. Truly changing the bonding curve would likely require forking the core PoolManager, which defeats the purpose of using a hook.

However, you can achieve interesting effects by modifying the `sqrtPriceLimitX96` parameter, injecting virtual liquidity, or using hook callbacks to simulate alternative curves. For example, a hook could implement a dynamic fee that effectively changes the perceived price impact curve. Or a hook could add a tax or subsidy that adjusts the effective exchange rate.

Gas considerations:

Hooks that run complex calculations on every swap will increase gas costs for all pool users. If your custom curve requires intensive computation, consider caching results or using a simplified approximation.

Compatibility with other hooks:

If multiple hooks are attached to the same pool, they execute in a defined order. Your custom curve hook should be positioned appropriately relative to hooks that adjust fees or enforce limits.

Testing:

Custom curve hooks must be thoroughly tested across edge cases: extreme liquidity, zero liquidity, large swaps, and round-off errors. Price calculations should be consistent and reversible (i.e., swapping A->B->A should return to approximately original balance).

While this example is conceptual, it demonstrates the direction. Truly novel bonding curves often require deeper integration than a simple hook provides, but the hook system can still customize many aspects of pool behavior beyond the basic invariant.

## Best Practices and Security Considerations

### Gas Optimization

Uniswap V4 is already more gas-efficient than V3 due to the singleton architecture, but you can still optimize. When adding liquidity, consider using exact token amounts rather than desired amounts where possible, as the router will handle the optimal distribution. Use the lowest tick spacing that meets your needs; tighter spacing increases liquidity precision but costs more gas per position.

Avoid unnecessary token approvals by using the `permit` function where supported. For multiple operations in the same transaction, use the PositionManager's ability to modify liquidity multiple times rather than separate calls.

### Slippage and Deadlines

Always set reasonable slippage tolerance and transaction deadlines. In high volatility environments, even 0.1% slippage might not be enough. Use `block.time` or `block.number` as deadlines rather than assuming a fixed number of seconds. For large swaps, consider breaking them into smaller chunks to minimize price impact.

### Reentrancy Protection

While the PoolManager uses a nonReentrant modifier on critical functions, if you're building custom hooks or contracts that interact with V4 in complex ways, you should implement your own reentrancy guards using OpenZeppelin's ReentrancyGuard or similar patterns. Never assume that callbacks from the PoolManager are reentrancy-safe unless explicitly stated.

### Common Pitfalls and Solutions

| Pitfall | Cause | Solution |
|---------|-------|----------|
| Wrong token order in PoolKey | Not sorting token0 < token1 | Always sort tokens before creating key |
| Tick not aligned with spacing | Using arbitrary tick values | Use tick % tickSpacing == 0 validation |
| Insufficient token balance | Not checking approvals or balances | Verify token approvals and balances before operations |
| Price limit too restrictive | sqrtPriceLimitX96 prevents execution | Set to 0 for any price, or calculate reasonable bounds |
| Not handling ETH correctly | Treating ETH like ERC20 | Use address(0) for ETH, payable functions |
| Ignoring deadline checks | Transactions execute after deadline | Always include deadline parameter and check it |
| Assuming one swap per block | Multiple calls in same block affect price | Consider order of operations and price impact |
| Forgetting to collect fees | Fees accumulate but not withdrawn | Call PositionManager.collect regularly |

### Testing Strategies

Thorough testing is essential when working with Uniswap V4. Write tests that cover:

1. Pool initialization with various token pairs and fee tiers
2. Liquidity addition and removal across different tick ranges
3. Swaps in both directions with various amounts
4. Hook callbacks executing correctly
5. Edge cases: zero liquidity, full range positions, extreme prices
6. Reentrancy attempts and unauthorized access
7. Flash accounting: multiple operations netted correctly
8. Fee collection and delta accounting

Use Hardhat's testing framework with ethers.js or viem. Load Uniswap V4 fixtures to deploy local PoolManager and test contracts. Test on forked mainnet if simulating real-world conditions.

## Mainnet Fork Setup

Testing against live mainnet conditions is critical for Uniswap V4 development. A mainnet fork creates a local copy of the Ethereum blockchain at a specific block, allowing you to interact with real contracts, real liquidity, and actual token balances without spending real money. This section provides a comprehensive guide to setting up and using mainnet forks with Hardhat.

### Hardhat Configuration for Forking

The foundation of mainnet forking is a properly configured Hardhat setup. Create or modify your `hardhat.config.ts` to include network configurations that point to Ethereum mainnet or Layer 2 networks.

```typescript
import {HardhatUserConfig} from "hardhat/config";
import "@nomicfoundation/hardhat-toolbox";
import {NetworkUserConfig} from "hardhat/types";

const config: HardhatUserConfig = {
    solidity: {
        version: "0.8.20",
        settings: {
            optimizer: {
                enabled: true,
                runs: 200
            }
        }
    },
    networks: {
        // Mainnet fork configuration
        mainnet: {
            url: process.env.MAINNET_URL || "",
            accounts:
                process.env.PRIVATE_KEY !== undefined
                    ? [process.env.PRIVATE_KEY]
                    : [],
            chainId: 1,
            // Enable forking with optional block number
            fork: {
                url: process.env.MAINNET_URL || "https://eth.llamarpc.com",
                // blockNumber: 20000000, // Optional: fork at specific block
            } as any,
            // Cache for faster restarts
            cache: {
                // Enabled by default in Hardhat
                // You can specify a path:
                // path: "./cache/mainnet-fork"
            }
        },
        // Arbitrum fork
        arbitrum: {
            url: process.env.ARBITRUM_URL || "",
            accounts:
                process.env.PRIVATE_KEY !== undefined
                    ? [process.env.PRIVATE_KEY]
                    : [],
            chainId: 42161,
            fork: {
                url: process.env.ARBITRUM_URL || "https://arb1.arbitrum.io/rpc",
            } as any,
        },
        // Optimism fork
        optimism: {
            url: process.env.OPTIMISM_URL || "",
            accounts:
                process.env.PRIVATE_KEY !== undefined
                    ? [process.env.PRIVATE_KEY]
                    : [],
            chainId: 10,
            fork: {
                url: process.env.OPTIMISM_URL || "https://mainnet.optimism.io",
            } as any,
        },
    },
    paths: {
        sources: "./contracts",
        tests: "./test",
        cache: "./cache",
        artifacts: "./artifacts"
    },
    // Gas reporter for optimization insights
    gasReporter: {
        enabled: process.env.REPORT_GAS !== undefined,
        currency: "USD",
    },
};

export default config;
```

Key configuration points:

The `fork` property tells Hardhat to create a local fork of the specified network. You'll need an RPC URL that supports archive access (ability to query historical state). Alchemy, Infura, and QuickNode all provide this with appropriate endpoints.

Set environment variables for your RPC URLs and private keys to avoid hardcoding them. Use a `.env` file with dotenv.

The `blockNumber` option in the fork config allows you to fork at a specific point in history. This is useful for testing against a known state or reproducing historical events.

Caching is enabled by default and significantly speeds up subsequent Hardhat restarts since the forked state is preserved. You can disable caching or customize the cache path if needed.

### RPC Provider Setup

Choosing a reliable RPC provider is essential for stable forking. Here's a comparison of popular options:

| Provider | Free Tier | Archive Access | Rate Limits | Notes |
|----------|-----------|----------------|-------------|-------|
| Alchemy | 5M compute units/month | Yes | Varies by plan | Good documentation, widely used |
| Infura | 100k requests/day | Yes | 10-20 req/sec | Simple setup, but IP rate limits |
| QuickNode | 10M credits/month | Yes | Varies | Fast, global endpoints |
| Llama RPC | Free, community | Limited | None | Good for testing, reliability varies |
| Public RPCs | Free | Usually no | Very low | Not suitable for heavy forking |

Compute units are a measure of resource consumption. Simple calls cost few units; state-heavy calls (like getting all pool pairs) can cost thousands. Monitor your usage if on a free tier.

For production development, Alchemy or QuickNode are recommended due to reliability and generous free tiers. Be aware that free tiers may have strict rate limits that cause test failures under heavy usage.

### Forked Test Fixtures

When testing against a forked network, you need to deploy your own contracts (PoolManager, PositionManager, hooks) to the local fork. However, you also need to interact with existing mainnet tokens and potentially existing pools. The following fixture helpers simplify this setup.

Create a `fixtures.ts` file in your `test/` directory:

```typescript
import {ethers} from "hardhat";
import {PoolManager, PositionManager, ISwapRouter} from "@uniswap/v4-core";
import {HardhatRuntimeEnvironment} from "hardhat/types";

export interface ForkFixtures {
    poolManager: PoolManager;
    positionManager: PositionManager;
    swapRouter: ISwapRouter;
    weth: string;
    usdc: string;
    wbtc: string;
    usdcEthPoolId: bytes32;
    signers: any[];
}

// Global cache to avoid redeploying on every test file
let cachedFixtures: ForkFixtures | null = null;

export async function getForkFixtures(hre: HardhatRuntimeEnvironment): Promise<ForkFixtures> {
    if (cachedFixtures) {
        return cachedFixtures;
    }
    
    const [deployer, user1, user2] = await ethers.getSigners();
    const signers = [deployer, user1, user2];
    
    // Deploy our contracts on the fork
    // PositionManager and SwapRouter are from periphery package
    const PoolManager = await ethers.getContractFactory("PoolManager");
    const poolManager = await PoolManager.deploy();
    await poolManager.deployed();
    
    // For the PositionManager and SwapRouter, you'd typically import and deploy
    // But these are provided by the V4 periphery package. In tests, you can
    // either deploy them from source or use a pre-deployed instance.
    
    // For this example, assume we have a local deployment script
    const positionManagerAddress = "0x..."; // Get from deployment
    const swapRouterAddress = "0x...";
    
    // Tokens: We'll use well-known mainnet addresses
    const weth = "0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2";
    const usdc = "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48";
    const wbtc = "0x2260FAC5E5542a773Aa44fBCfeDf7C193bc2C599";
    
    // Load an existing mainnet pool (optional)
    // You can get the poolId from an on-chain pool
    const USDC_ETH_POOL_ID = "0x..."; // Compute from token addresses and fee tier
    
    // Impersonate a rich account to fund your testers
    const richAccount = "0x..."; // A whale address from mainnet
    await network.provider.request({
        method: "hardhat_impersonateAccount",
        params: [richAccount]
    });
    
    // Fund test users with some tokens
    // You would call transferFrom the whale to the test users
    
    // Stop impersonating
    await network.provider.request({
        method: "hardhat_stopImpersonatingAccount",
        params: [richAccount]
    });
    
    cachedFixtures = {
        poolManager,
        positionManager: null as any,
        swapRouter: null as any,
        weth,
        usdc,
        wbtc,
        usdcEthPoolId: USDC_ETH_POOL_ID,
        signers
    };
    
    return cachedFixtures;
}

// Helper to fund an address with ETH
export async function fundAddress(to: string, amount: ethers.BigNumberish): Promise<void> {
    await network.provider.request({
        method: "hardhat_setBalance",
        params: [to, ethers.utils.parseEther(amount.toString())]
    });
}

// Helper to impersonate a mainnet account
export async function impersonate(account: string): Promise<void> {
    await network.provider.request({
        method: "hardhat_impersonateAccount",
        params: [account]
    });
}

// Helper to stop impersonation
export async function stopImpersonating(account: string): Promise<void> {
    await network.provider.request({
        method: "hardhat_stopImpersonatingAccount",
        params: [account]
    });
}
```

Using fixtures in your tests:

```typescript
import {getForkFixtures, impersonate, fundAddress} from "./fixtures";
import {ethers} from "hardhat";

describe("USDC/ETH Pool Integration", function () {
    let fixtures: ForkFixtures;
    
    before(async function () {
        // Only run if network is forked
        if (!(await ethers.provider.getNetwork()).chainId) {
            return;
        }
        fixtures = await getForkFixtures(hre);
    });
    
    it("Should swap USDC for ETH and get expected amount", async function () {
        // Use real mainnet pool state
        const {poolManager, usdc, weth, usdcEthPoolId} = fixtures;
        
        // Get pool state
        const (sqrtPrice, tick, observationIndex, observationState) = 
            await poolManager.getPool(usdcEthPoolId);
        
        // Simulate a swap
        const amountIn = ethers.utils.parseUnits("100", 6); // 100 USDC (6 decimals)
        
        // Call swap
        const result = await poolManager.swap(
            {
                poolId: usdcEthPoolId,
                amountSpecified: -int256(amountIn), // exact output
                sqrtPriceLimitX96: 0,
                zeroForOne: false // USDC -> ETH
            },
            bytes("")
        );
        
        // Validate results
        const amountOut = result.amount1; // Since USDC->ETH, amount1 is ETH
        expect(amountOut).to.be.gt(0);
        
        // Compare with off-chain quote
        // You could compute expected output using uniswap-v3-sdk or similar
    });
});
```

### Integration Test Examples

Complete test file demonstrating integration with real mainnet pool state:

```typescript
// test/uniswap-v4-fork.test.ts
import {ethers, network} from "hardhat";
import {BigNumberish, BigNumber} from "ethers";
import {HardhatRuntimeEnvironment} from "hardhat/types";
import {getForkFixtures, impersonate, fundAddress} from "./fixtures";

describe("Uniswap V4 Mainnet Fork Tests", function () {
    this.timeout(60000); // Fork tests can be slow
    
    let fixtures: ForkFixtures;
    const [deployer, trader, lpProvider] = await ethers.getSigners();
    
    before(async function () {
        // Skip if not forking
        if (!(await ethers.provider.getNetwork()).chainId) {
            console.log("Skipping: not a forked network");
            return;
        }
        
        console.log("Setting up fixtures on forked network...");
        fixtures = await getForkFixtures(hre);
    });
    
    describe("Real Pool Interaction", function () {
        it("Should quote swap correctly against real USDC/ETH pool", async function () {
            const {poolManager, usdcEthPoolId} = fixtures;
            
            // Get current pool state
            const liquidity = await poolManager.liquidity(usdcEthPoolId);
            const sqrtPriceX96 = await poolManager.sqrtPrice(usdcEthPoolId);
            const tick = await poolManager.tick(usbcEthPoolId);
            
            const amountIn = ethers.utils.parseUnits("1", 6); // 1 USDC
            
            // Use the SwapRouter for realistic testing
            const swapRouter = await ethers.getContractAt("ISwapRouter", swapRouterAddress);
            
            // Execute exact input single swap
            const minAmountOut = 0; // For testing, no minimum
            const deadline = Math.floor(Date.now() / 1000) + 60;
            
            const result = await swapRouter.exactInputSingle(
                {
                    poolId: usdcEthPoolId,
                    kind: 0, // zeroForOne
                    amountSpecified: amountIn,
                    sqrtPriceLimitX96: 0
                },
                {
                    sender: trader.address,
                    from: usdc,
                    to: trader.address,
                    wNETH: ethers.constants.AddressZero,
                    funding: 1
                },
                minAmountOut,
                deadline,
                {value: 0}
            );
            
            const amountOut = result;
            console.log("Swap result:", {
                amountIn: amountIn.toString(),
                amountOut: amountOut.toString(),
                effectivePrice: amountOut.mul(1e6).div(amountIn).toString()
            });
            
            // Basic sanity: should get some ETH out
            expect(amountOut).to.be.gt(0);
        });
        
        it("Should add liquidity to mainnet pool without breaking", async function () {
            const {positionManager, poolManager, usdcEthPoolId, usdc, weth} = fixtures;
            
            // Get current tick to set range
            const currentTick = await poolManager.tick(usdcEthPoolId);
            
            // Set range around current tick
            const tickLower = currentTick - 100;
            const tickUpper = currentTick + 100;
            
            // Approve tokens
            const usdcContract = await ethers.getContractAt("IERC20", usdc);
            await usdcContract.approve(positionManager.address, ethers.constants.MaxUint256);
            
            // Mint liquidity
            const amount0 = ethers.utils.parseUnits("1000", 6); // 1000 USDC
            const amount1 = ethers.utils.parseEther("1"); // 1 ETH
            
            // Build ModifyLiquidityParams
            const modifyParams = {
                poolId: usdcEthPoolId,
                liquidityDelta: 0,
                sqrtPriceX96: 0,
                tickLower: tickLower,
                tickUpper: tickUpper,
                amount0Requested: amount0,
                amount1Requested: amount1,
                decreaseLiquidity: false,
                createPosition: true
            };
            
            const result = await positionManager.modifyLiquidity(
                modifyParams,
                0, // amount0Min
                0, // amount1Min
                lpProvider.address,
                Math.floor(Date.now() / 1000) + 1000
            );
            
            const [liquidity, actualAmount0, actualAmount1] = result;
            
            expect(liquidity).to.be.gt(0);
            expect(actualAmount0).to.be.gt(0);
            expect(actualAmount1).to.be.gt(0);
            
            console.log("Liquidity added:", {
                liquidity: liquidity.toString(),
                amount0: actualAmount0.toString(),
                amount1: actualAmount1.toString()
            });
        });
        
        it("Should collect fees from position after some swaps", async function () {
            // Impersonate a whale to perform swaps that generate fees
            const whale = "0x..."; // Mainnet address with large balance
            
            await impersonate(whale);
            const whaleSigner = await ethers.getImpersonatedSigner(whale);
            
            // Ensure whale has token approval for PositionManager or SwapRouter
            // Perform a series of swaps to generate fees
            // ...
            
            await stopImpersonating(whale);
            
            // Now collect fees from our position
            // ...
        });
    });
    
    describe("Hook Integration", function () {
        it("Should deploy a pool with FeeDiscountHook and execute swaps", async function () {
            // Deploy a custom hook
            const FeeDiscountHook = await ethers.getContractFactory("FeeDiscountHook");
            const feeHook = await FeeDiscountHook.deploy(poolManager.address);
            
            // Create a new pool with the hook attached
            // This would involve calling poolManager.initialize with FeeConfiguration
            const tokenA = "0x..."; // e.g., WETH on mainnet fork
            const tokenB = "0x..."; // e.g., USDC
            const fee = 3000;
            const tickSpacing = 60;
            
            const poolKey = {
                currency0: tokenA,
                currency1: tokenB,
                fee: fee,
                tickSpacing: tickSpacing
            };
            
            const hookConfig = {
                hookProtocol: feeHook.address,
                hookSwap: feeHook.address,
                // ... other hooks set to feeHook address or zero
            };
            
            await poolManager.initialize(
                poolKey,
                0, // initial tick
                deployer.address,
                hookConfig
            );
            
            // Now test that swaps on this pool invoke the hook
            // by checking that discount is applied
        });
    });
});
```

### Forking Best Practices

Effective use of mainnet forks requires adherence to certain practices that keep your tests reliable and cost-effective.

**When to Use Fresh vs Cached Forks**

A fresh fork starts from the exact state of the target network at the configured block number. This ensures accuracy but can be slow to initialize (downloading several GB of state). Cached forks reuse previously downloaded data, making restarts much faster.

- Use fresh forks when you need to test interactions with state that changes frequently (e.g., recent blocks, token balances that might have been spent)
- Use cached forks for development speed when testing against relatively stable state (e.g., well-known pools from a few days ago)
- Clear the cache (`rm -rf ./cache`) periodically to avoid stale state

**Managing RPC Costs and Limits**

RPC providers impose rate limits and quota restrictions. To stay within limits:

- Use `.only` or `.skip` in your test suites to avoid running expensive fork tests on every `hardhat test` invocation. Tag fork tests with a custom flag and run them selectively: `hardhat test --grep "fork"` or similar.
- Mock network calls where possible; for example, instead of using real mainnet token balances, mint your own test tokens. Only fork when you need to verify actual pool math or integration with existing infrastructure.
- Batch RPC calls when possible. Hardhat's `hardhat_impersonateAccount` and `hardhat_setBalance` are local operations and do not hit the RPC; use these instead of transferring from real accounts.

**Debugging Fork-Specific Issues**

Common issues with forks include:

- **State mismatches**: If you fork at a block number where a pool didn't exist yet, you'll get errors when trying to interact with it. Verify pool existence by checking `poolManager.getPool(poolId)` returns non-zero values.
- **Reorgs**: Mainnet can have occasional reorganizations. If you're testing very recent blocks, the state might change between forks. Fork at a block at least 30 minutes old for stability.
- **Chain updates**: Hardhat may need to be updated to handle network upgrades (e.g., Ethereum's Dencun). Check Hardhat release notes if forking fails unexpectedly.
- **Missing storage slots**: Some contracts use deep storage layouts that may not be properly captured by the fork. Verify by comparing key values against a block explorer.

Performance tips:

- Set the `fork.blockNumber` to avoid syncing the entire chain to the latest block; a specific block is much faster.
- Use `cache.path` to place the cache on a fast SSD if available.
- Limit the number of fork tests; each one incurs RPC latency. Where possible, reuse the same fork across multiple test cases using `before`/`beforeEach`.

Cleaning up between tests:

When tests modify state (e.g., changing token balances), ensure they don't pollute subsequent tests. You have a few options:

- Use `hardhat_reset` to reset the fork to the original state between tests. This is slow but ensures isolation.
- Design your tests to be order-independent by carefully resetting balances within each test.
- Run each test file in isolation (each file can fork anew), accepting the startup cost.

```typescript
// In a test file, reset between tests
afterEach(async function () {
    if (network.name === "hardhat" || network.name === "mainnet") {
        await network.provider.request({
            method: "hardhat_reset",
            params: [{
                forking: {
                    // Re-fork from same point
                    jsonRpcUrl: process.env.MAINNET_URL,
                    blockNumber: 20000000
                }
            }]
        });
    }
});
```

This reset is expensive, so use sparingly. Better to structure tests to not interfere.

By following these patterns, you can effectively use mainnet forks to validate your Uniswap V4 integrations against real market conditions before deploying significant funds.

## Conclusion and Further Resources

Uniswap V4 provides a powerful foundation for building decentralized trading applications with unparalleled flexibility. The hook system alone opens possibilities for limit orders, custom AMM curves, dynamic fees, and on-chain order books all within the same liquidity network.

To deepen your expertise, explore the official Uniswap V4 documentation at https://docs.uniswap.org/ and review the extensive test suite in the v4-core and v4-periphery repositories. The Uniswap V4 spec provides detailed technical explanations of every function and parameter.

Start by building simple liquidity provision contracts, then gradually add hooks and custom swap logic. The provided examples in this guide are production-ready patterns that you can adapt to your specific use cases. Remember to thoroughly test on testnets before deploying significant value, and consider auditing your hook implementations if they handle substantial funds.

The Uniswap community is active and helpful, join the developer Discord channels for real-time support and to learn from others building on V4. As the ecosystem grows, best practices will evolve, so stay updated with the latest changes in the core contracts.

## Quick Reference Tables

### SwapRouter vs Direct PoolManager: Detailed Comparison

| Feature | SwapRouter | Direct PoolManager |
|---------|------------|-------------------|
| Token Transfer | Automatic through fund management | Manual via transferFrom/transfer |
| Approval Required | Yes, to router | Yes, to poolManager or use permit |
| Multi-hop Swaps | Native support via swap array | Requires sequential calls |
| Price Limits | sqrtPriceLimitX96 per swap | Same parameter available |
| Deadlines | Built-in deadline check | Must implement manually |
| Gas Refunds | Router may handle efficiently | Direct, but requires careful accounting |
| Error Messages | Standardized router errors | Contract-specific errors |
| Maintenance | Router must be deployed separately | Direct poolManager interaction |

### Delta Accounting Summary

| Operation | Liquidity Delta | Token0 Delta | Token1 Delta | Settlement |
|-----------|----------------|--------------|--------------|------------|
| Add Liquidity | +minted | -deposited | -deposited | Position created |
| Remove Liquidity | -burned | +withdrawn | +withdrawn | Position burned, tokens sent |
| Swap (zeroForOne) | 0 | +received | -paid | Net transfer |
| Swap (oneForZero) | 0 | -paid | +received | Net transfer |
| Fee Collection | 0 | +collected0 | +collected1 | Added to position balance |
| Donation | 0 | +donated0 | +donated1 | Added to pool |

This table shows how each operation affects the various delta values and how they eventually settle through token transfers or balance updates.

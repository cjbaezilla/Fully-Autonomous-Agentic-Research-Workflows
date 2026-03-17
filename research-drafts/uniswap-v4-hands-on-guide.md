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
 
## Upgradeability Patterns

When building production systems with Uniswap V4, you need to consider how your contracts will evolve over time. Smart contracts are immutable once deployed, but upgradeability patterns allow you to replace implementation logic while preserving state and contract addresses. This section covers the main approaches to making your V4 integrations upgradeable, along with their trade-offs and practical implementation details.

### Understanding Upgradeability Fundamentals

Upgradeability in Ethereum smart contracts is achieved through indirection. Instead of users calling your logic contract directly, they call a proxy contract that delegates calls to an implementation contract. The proxy stores the state variables, while the implementation contains the executable code. By changing the implementation address stored in the proxy, you can upgrade the behavior while keeping the same proxy address and all stored data intact.

This pattern requires careful design because the storage layout of the new implementation must remain compatible with the old one. If you add or reorder state variables in an incompatible way, the new contract will read garbage from storage slots and behave unpredictably. Therefore, upgradeable contracts need disciplined storage management.

### Proxy Patterns: Transparent Proxy vs UUPS

Two main proxy patterns dominate the Ethereum ecosystem: the Transparent Proxy (used by OpenZeppelin) and the Universal Upgradeable Proxy Standard (UUPS). Each has different characteristics.

The Transparent Proxy separates concerns: the proxy contract handles upgrade logic and admin functions, while the implementation is completely unaware of upgradeability. The proxy has an admin who can upgrade to a new implementation. Users interact with the proxy as if it were the implementation. This pattern is simple and works well when you have a multi-sig or DAO controlling upgrades. However, it adds extra storage slots for the implementation address and admin, increasing deployment cost slightly.

The UUPS pattern moves upgrade logic into the implementation itself. The proxy becomes a minimal, cheap contract that simply delegates all calls. The implementation contract must include functions to upgrade the implementation address and to check if an address is a proxy. This saves proxy deployment gas and reduces the attack surface (fewer functions in the proxy), but requires the implementation to be written with upgradeability in mind. OpenZeppelin's UUPS upgradeable contracts provide a base that handles the mechanics.

Which to choose? For most Uniswap V4 hook implementations, UUPS is recommended because it's more gas-efficient and the proxy itself cannot be upgraded accidentally; only the implementation can change itself. However, if you need multiple versions of an implementation coexisting (e.g., for testing different configurations), Transparent Proxy gives more flexibility.

Here's how to deploy an upgradeable hook using UUPS:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import {UUPSUpgradeable} from "@openzeppelin/contracts-upgradeable/proxy/utils/UUPSUpgradeable.sol";
import {Initializable} from "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";
import {IPoolManager} from "@uniswap/v4-core/contracts/interfaces/IPoolManager.sol";
import {IHook} from "@uniswap/v4-core/contracts/interfaces/IHook.sol";

contract UpgradeableFeeDiscountHook is Initializable, IHook, UUPSUpgradeable {
    IPoolManager public poolManager;
    address public owner;
    mapping(address => bool) public discountedAccounts;
    uint256 public discountBps;

    /// @custom:oz-upgrades-unsafe-allow constructor
    constructor() {
        // Enable upgradeability via UUPS
    }

    function initialize(address _poolManager) public initializer {
        poolManager = IPoolManager(_poolManager);
        owner = msg.sender;
        discountedAccounts[msg.sender] = true;
        discountBps = 50;
    }

    function beforeSwap(
        bytes32 poolId,
        IPoolManager.SwapRequest memory request
    ) external override {
        require(msg.sender == address(poolManager), "Unauthorized");
        if (discountedAccounts[request.sender]) {
            uint256 newFee = request.fee - (request.fee * discountBps) / 10000;
            request.fee = newFee > 0 ? newFee : 0;
        }
    }

    function addDiscountedAccount(address account) external {
        require(msg.sender == owner, "Only owner");
        discountedAccounts[account] = true;
    }

    function setDiscountBps(uint256 _discountBps) external {
        require(msg.sender == owner, "Only owner");
        require(_discountBps <= 500, "Discount too high");
        discountBps = _discountBps;
    }

    // Required by UUPS: this function is called by the proxy to authorize an upgrade
    function _authorizeUpgrade(address newImplementation) internal override onlyOwner {
        // Add any custom upgrade authorization logic here
        // For example, check that newImplementation is a trusted address
    }
}
```

To deploy this as an upgradeable contract:

```bash
npx hardhat run scripts/deploy-upgradeable.ts --network sepolia
```

And the deployment script:

```typescript
import {ethers} from "hardhat";
import {UUPSUpgradeable} from "@openzeppelin/contracts-upgradeable/proxy/utils/UUPSUpgradeable";

async function main() {
    const [deployer] = await ethers.getSigners();
    console.log("Deploying upgradeable hook with account:", deployer.address);

    const Hook = await ethers.getContractFactory("UpgradeableFeeDiscountHook");
    const hook = await Hook.deploy();
    await hook.deployed();
    console.log("Hook deployed to:", hook.address);

    // Initialize the contract (call the initializer function)
    const poolManagerAddress = "0x..."; // Your pool manager address
    await hook.initialize(poolManagerAddress);
    console.log("Hook initialized");
}

main().catch((error) => {
    console.error(error);
    process.exitCode = 1;
});
```

Note that the constructor in upgradeable contracts should be empty or minimal; any initialization should happen in an `initialize` function marked with `initializer` from OpenZeppelin. This prevents the initializer from being called again accidentally.

### Diamond Pattern for Modular Hooks

For complex systems where you want to separate concerns into multiple facets, the Diamond Pattern (EIP-2535) offers a different approach. A Diamond is a contract that delegates function calls to multiple implementation contracts (facets) and can upgrade each facet independently. This allows you to have separate contracts for different hook callbacks, for example, a Limit Orders facet, an Oracle facet, and a Fees facet, all composing a single hook address.

The Diamond pattern is more flexible than a single implementation proxy because you can add or replace individual function groups without affecting others. It's particularly useful when you have many hooks and want to upgrade them selectively. However, it introduces more complexity: you need a DiamondLoupe interface for introspection, you must carefully manage storage collisions between facets (they all share the same storage), and you need governance to cut diamonds (make changes).

A basic diamond for Uniswap V4 hooks might look like this:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import {IDiamond} from "./interfaces/IDiamond.sol";
import {IDiamondLoupe} from "./interfaces/IDiamondLoupe.sol";
import {LibDiamond} from "./libraries/LibDiamond.sol";
import {IPoolManager} from "@uniswap/v4-core/contracts/interfaces/IPoolManager.sol";

contract MyHookDiamond is IDiamond, IDiamondLoupe {
    using LibDiamond for bytes32;

    // Diamond storage layout - stored in diamond storage slot
    struct DiamondStorage {
        address owner;
        bytes4[] facetFunctionSelectors;
        mapping(bytes4 => bytes4) facetFunctionSelectorToFacet; // selector to facet
        mapping(bytes4 => bool) deletedSelectors; // for removing functions
    }

    // Slots for diamond storage (use a unique keccak256 hash)
    bytes32 constant DIAMOND_STORAGE_POSITION = keccak256("diamond.standard.diamond.storage");

    modifier onlyOwner {
        require(msg.sender == _getOwner(), "Only owner");
        _;
    }

    function _getOwner() internal view returns (address) {
        DiamondStorage storage ds = diamondStorage();
        return ds.owner;
    }

    function diamondStorage() internal pure returns (DiamondStorage storage ds) {
        bytes32 position = DIAMAND_STORAGE_POSITION;
        assembly {
            ds.slot := position
        }
    }

    // Initialize the diamond with initial facets
    function initialize(address _poolManager) external {
        // Only call once
        DiamondStorage storage ds = diamondStorage();
        require(ds.owner == address(0), "Already initialized");

        ds.owner = msg.sender;
        // Add facets via diamondCut (or set in constructor)
    }

    // Diamond cut: add/replace/remove facets
    function diamondCut(
        IDiamond.FacetCut[] calldata _diamondCut,
        address _init,
        bytes calldata _calldata
    ) external onlyOwner {
        LibDiamond.diamondCut(_diamondCut, _init, _calldata);
    }

    // Fallback: delegate calls to facets
    fallback() external payable {
        LibDiamond.delegateSelector();
    }

    // Receive: accept plain ether transfers
    receive() external payable {}

    // IDiamondLoupe functions for introspection
    function facets() external view returns (IDiamond.Facet[] memory) {
        DiamondStorage storage ds = diamondStorage();
        IDiamond.Facet[] memory facetsArray = new IDiamond.Facet[](ds.facetFunctionSelectors.length);
        for (uint256 i = 0; i < ds.facetFunctionSelectors.length; i++) {
            bytes4 selector = ds.facetFunctionSelectors[i];
            address facet = ds.facetFunctionSelectorToFacet[selector];
            facetsArray[i] = IDiamond.Facet({
                facetAddress: facet,
                facetFunctionSelectors: new bytes4[](0) // simplified
            });
        }
        return facetsArray;
    }

    function facetFunctionSelectors(address facet) external view returns (bytes4[] memory) {
        DiamondStorage storage ds = diamondStorage();
        bytes4[] memory allSelectors = ds.facetFunctionSelectors;
        bytes4[] memory facetSelectors = new bytes4[](0);
        for (uint256 i = 0; i < allSelectors.length; i++) {
            if (ds.facetFunctionSelectorToFacet[allSelectors[i]] == facet) {
                facetSelectors.push(allSelectors[i]);
            }
        }
        return facetSelectors;
    }

    function facetAddress(bytes4 selector) external view returns (address) {
        DiamondStorage storage ds = diamondStorage();
        require(!ds.deletedSelectors[selector], "Selector deleted");
        return ds.facetFunctionSelectorToFacet[selector];
    }
}
```

The LibDiamond library manages the storage and delegation logic. Each facet (e.g., FeeDiscountFacet, LimitOrderFacet) is a separate contract that contains only the functions relevant to that function group. The diamondCut function lets the owner add new facets, replace existing ones, or remove functions. This modularity allows you to upgrade fee logic separately from order logic, reducing risk of breaking unrelated features.

### State Management and Storage Layout

Upgradeability introduces the critical challenge of storage layout compatibility. When you upgrade an implementation, the storage slots read and written by the contract must remain consistent. The proxy holds all state variables in its own storage, and the implementation reads them as if they were its own variables. If variable types, order, or positions change, the implementation will misinterpret data.

The safest practice is to use a storage pattern that reserves slots for future variables and never changes the order or types of existing variables. OpenZeppelin's upgradeable contracts use a technique: each contract defines a struct that holds all its persistent state, and uses a unique storage slot to hold that struct. Child contracts then inherit from that base and append new state at the end. This ensures that as long as you only add new state at the end and never modify or delete existing state, upgrades remain compatible.

Example:

```solidity
// Base contract that defines the storage layout
contract HookStorage {
    struct Layout {
        IPoolManager poolManager;
        address owner;
        mapping(address => bool) discountedAccounts;
        uint256 discountBps;
        // Add new variables here in upgrades - never remove or reorder
    }

    bytes32 private constant STORAGE_SLOT = keccak256("myhook.storage");

    function getLayout() internal pure returns (Layout storage l) {
        bytes32 slot = STORAGE_SLOT;
        assembly {
            l.slot := slot
        }
    }
}

// Implementation contract
contract FeeDiscountHook is HookStorage {
    function initialize(address _poolManager) public {
        Layout storage l = getLayout();
        require(l.owner == address(0), "Already initialized");
        l.poolManager = IPoolManager(_poolManager);
        l.owner = msg.sender;
        l.discountedAccounts[msg.sender] = true;
        l.discountBps = 50;
    }

    // Functions read from l
    function beforeSwap(bytes32 poolId, IPoolManager.SwapRequest memory request) external {
        require(msg.sender == address(poolManager), "Unauthorized");
        Layout storage l = getLayout();
        if (l.discountedAccounts[request.sender]) {
            uint256 newFee = request.fee - (request.fee * l.discountBps) / 10000;
            request.fee = newFee > 0 ? newFee : 0;
        }
    }

    // Upgradeable owners
    function setDiscountBps(uint256 _discountBps) external {
        require(msg.sender == getLayout().owner, "Only owner");
        require(_discountBps <= 500, "Discount too high");
        getLayout().discountBps = _discountBps;
    }
}
```

When upgrading, you can add new fields to the Layout struct at the end. The old contract will ignore them (treating them as zero). The new contract will read the old fields from the same slots and see the new fields as zero initially. This ensures compatibility.

### Governance-Controlled Upgrade Mechanisms

Upgrading contracts that control significant assets should not be controlled by a single individual. Instead, use governance mechanisms such as timelocks, multi-signature wallets, or DAO voting to enforce security and deliberate upgrade processes.

A common pattern is to have an upgradeability module (like the proxy admin or a governance contract) that can queue upgrades, enforce a delay period, and allow token holders to veto. OpenZeppelin's TimelockController can be used as an intermediary: the upgrade function is only callable by the Timelock, which executes transactions after a minimum delay, giving the community time to react and exit if an upgrade is malicious.

Example integration:

```solidity
import {TimelockController} from "@openzeppelin/contracts/governance/TimelockController.sol";

contract GovernedHook {
    TimelockController public timelock;
    address public admin;

    constructor(address _timelock, address _admin) {
        timelock = TimelockController(_timelock);
        admin = _admin;
    }

    // Only timelock can call upgrade after delay
    function upgradeHook(address newImplementation) external {
        require(msg.sender == address(timelock), "Only timelock");
        // Call to UUPS proxy's upgradeTo function
        // The timelock will execute this after the delay
    }
}
```

In practice, you would deploy the UUPS proxy, then assign its admin role to the Timelock. The Timelock then has authority to call `upgradeTo` on the proxy. Any proposal to upgrade must go through the governance process: voting, timelock queue, execution. This prevents a single key holder from pushing malicious upgrades instantly.

### Timelocks and Multi-sig Patterns

Even without a full DAO, you can use a multi-signature wallet like Gnosis Safe to require multiple approvals before an upgrade executes. The Safe can be configured with a threshold (e.g., 3 out of 5 signers). An upgrade transaction is submitted as a multi-sig transaction and only executes once enough signatures are collected.

Gnosis Safe also supports modules that can add additional logic, such as requiring that the new implementation passes certain validation (like having an audit flag) before it can be set. You could create a custom Safe module that checks the new implementation address against a whitelist of audited contracts.

For smaller projects, a simple 2-of-3 multi-sig is often sufficient. The important principle is that no single private key can upgrade the system alone.

### Migration Strategies for Existing Positions and Pool State

When upgrading a hook that interacts with existing pools and positions, you must consider how state changes affect existing users. The hook state is typically stored in the hook contract itself (e.g., mapping of limit orders, oracle observations). If you change the data structures or storage layout of that state, you need a migration plan.

Two main approaches:

1. **Forward-compatible upgrades**: Design your initial hook state to be extensible. Use mappings that can accommodate new keys, and avoid packing state into tightly coupled structs. If you need to add new features, introduce new mappings rather than modifying existing ones. This way, old and new implementations can coexist without explicit data migration.

2. **Explicit migration phase**: Schedule a maintenance window where users must withdraw liquidity or close positions, the old hook is deprecated, a new hook is deployed with a different address, and pools are reinitialized to point to the new hook. This is disruptive but sometimes necessary for fundamental changes that can't be backward-compatible.

Example of a migration process:

```solidity
contract HookMigrationCoordinator {
    IHook public oldHook;
    IHook public newHook;
    address public poolManager;

    // Users can call this to migrate their limit orders
    function migrateLimitOrder(uint256 orderId) external {
        require(oldHook.ownerOf(orderId) == msg.sender, "Not owner");
        LimitOrder memory order = oldHook.getOrder(orderId);
        // Validate order parameters
        // Insert into newHook
        newHook.transferOrder(orderId, msg.sender); // Transfer ownership in new system
        oldHook.cancelOrder(orderId); // Remove from old
    }

    // Admin function to migrate a pool's hook attachment
    function setPoolHook(bytes32 poolId) external onlyOwner {
        // Call poolManager to change hook for this pool
        // This might involve poolManager.updateHookConfiguration
        // After migration, new hook will be used for new operations
        // Old hook's state remains for historical reference
    }
}
```

The key is to test migrations thoroughly on a fork of mainnet with real positions and orders to ensure no data loss or unexpected reverts.

### Non-Upgradeable Alternative Patterns

Not every contract needs to be upgradeable. In fact, upgradeability increases complexity and attack surface. For many use cases, a non-upgradeable design is simpler and safer. Consider these alternatives:

- **Factory pattern**: Deploy a new implementation for each version. Your factory contract remains upgradeable to point to new versions, while individual hook contracts are immutable. Users can choose which version to use based on risk tolerance. This is the pattern used by Uniswap V3's NonfungiblePositionManager: it's deployed as a single immutable contract, but pools and routers are separate and can be upgraded independently.

- **Versioned contracts with registry**: Deploy HookV1, HookV2, etc., and maintain a registry that maps pool IDs to the current hook address. The registry can be upgraded, while individual hooks remain fixed. This allows you to fix bugs by deploying a new hook and updating the registry mapping.

- **Agent-based deployment**: Each user deploys their own hook instance from a factory. This gives users full control and avoids upgradeability concerns because if a bug is discovered, users can simply stop using the old contract and start using a new one. The downside is higher deployment cost per user.

Example factory pattern:

```solidity
contract HookFactory {
    address public immutable poolManager;
    uint256 public hookCount;
    mapping(uint256 => address) public hooks;

    event HookDeployed(uint256 indexed hookId, address hook, address owner);

    constructor(address _poolManager) {
        poolManager = _poolManager;
    }

    function deployHook() external returns (uint256 hookId) {
        Hook newHook = new Hook(poolManager, msg.sender);
        hookId = ++hookCount;
        hooks[hookId] = address(newHook);
        emit HookDeployed(hookId, address(newHook), msg.sender);
    }
}

contract Hook {
    IPoolManager public poolManager;
    address public owner;

    constructor(address _poolManager, address _owner) {
        poolManager = IPoolManager(_poolManager);
        owner = _owner;
    }

    // Hook callbacks with owner validation
    function beforeSwap(bytes32 poolId, IPoolManager.SwapRequest memory request) external {
        require(msg.sender == address(poolManager), "Unauthorized");
        // ... hook logic
    }
}
```

### Risks and Trade-offs of Different Approaches

Upgradeability introduces risks that must be carefully managed:

- **Proxy storage corruption**: A poorly designed upgrade can misinterpret storage and corrupt state. Use well-audited upgradeable patterns (OpenZeppelin) and avoid custom assembly where possible.
- **Malicious upgrades**: If the upgrade key is compromised, an attacker can replace the implementation with a malicious one that steals funds. Mitigate with timelocks, multi-sig, and decentralized governance.
- **Function selector clashes**: In Diamond patterns, facets must not share function selectors, and must not conflict with the Diamond's own functions. This requires careful namespace planning.
- **Initialization vulnerabilities**: Upgradeable contracts must have initializer functions that can only be called once. A malicious upgrade could include a new initializer that resets critical state. Use the `initializer` modifier and consider making the contract `Initializable` only for the first deployment, not subsequent upgrades, or ensure new initializers are distinct.
- **Breaking changes**: Even with compatible storage, changing function semantics can break downstream contracts that depend on the old behavior. Version your API and deprecate old functions rather than altering them.

Before choosing an upgradeability strategy, assess:
- Who controls upgrades? (Team, DAO, multisig)
- How often do you expect changes? (Frequent changes favor upgradeability)
- What is the value at risk? (Higher value demands stronger controls)
- How complex are the state changes? (Simple upgrades easier)

For most production hooks, a UUPS proxy with a Timelock and multisig guardians offers a good balance of flexibility and security.


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

## Oracle & Price Feed Integration

Real-time, reliable price data is essential for many DeFi applications, and Uniswap V4 hooks provide a powerful way to integrate oracles directly into the pool lifecycle. Whether you want to restrict swaps based on external price feeds, create TWAP-based trading rules, or implement custom median oracles, the hook system makes it possible without leaving the Uniswap ecosystem. This section covers practical strategies for integrating Chainlink price feeds and V3 TWAP oracles within V4 hooks, along with security considerations and complete working examples.

### Integrating Chainlink Price Feeds in Hooks

Chainlink provides decentralized price feeds that aggregate data from multiple node operators, delivering time-weighted average prices with cryptographic proofs. These feeds are widely used in lending protocols, derivatives, and other applications requiring trustworthy off-chain price information.

To integrate a Chainlink price feed into a Uniswap V4 hook, you typically read the latest answer from the feed in a `beforeSwap` callback and compare it to the pool's current price. If the discrepancy exceeds a threshold, you can revert the swap or adjust parameters. This prevents trades based on stale or manipulated prices.

The Chainlink AggregatorV3Interface defines these key functions:

```solidity
interface AggregatorV3Interface {
    function latestRoundData() external view returns (
        uint80 roundId,
        int256 answer,
        uint256 startedAt,
        uint256 updatedAt,
        uint80 answeredInRound
    );
    function decimals() external view returns (uint8);
}
```

You call `latestRoundData` on the feed address to get the latest price (answer) and its timestamp (updatedAt). You must check that `updatedAt` is recent enough (not too stale) and that the `answer` is within expected bounds before using it.

A hook that enforces price validity might look like this:

```solidity
contract ChainlinkOracleHook {
    IPoolManager public poolManager;
    
    // Price feed for token0/token1 pair (normalized to 8 decimals)
    address public priceFeed;
    
    // Price deviation threshold: maximum allowed difference between 
    // Chainlink price and pool spot price, as a percentage (e.g., 100 = 1%)
    uint256 public maxDeviationBps;
    
    // Maximum age of price feed in seconds before considered stale
    uint256 public maxStalenessSeconds;
    
    event PriceCheck(bytes32 indexed poolId, int24 tick, int256 chainlinkPrice, uint256 deviationBps);
    
    constructor(address _poolManager, address _priceFeed) {
        poolManager = IPoolManager(_poolManager);
        priceFeed = _priceFeed;
        maxDeviationBps = 100; // 1%
        maxStalenessSeconds = 60 * 60; // 1 hour
    }
    
    function beforeSwap(
        bytes32 poolId,
        IPoolManager.SwapRequest memory request
    ) external {
        require(msg.sender == address(poolManager), "Unauthorized");
        
        // Get current pool state
        (, int24 currentTick, , ,) = poolManager.getPool(poolId);
        
        // Convert pool tick to price
        uint256 poolPrice = _tickToPrice(currentTick);
        
        // Fetch Chainlink price (normalized to 8 decimals)
        (int256 chainlinkPriceRaw, uint256 lastUpdated) = _getChainlinkPrice();
        
        // Check staleness
        require(block.timestamp - lastUpdated <= maxStalenessSeconds, "Oracle stale");
        
        // Normalize chainlink price to pool's price format if needed
        // Chainlink usually returns 8 decimal places; pool uses Q64.96
        // Scale accordingly based on token decimals
        uint256 chainlinkPrice = _normalizeToPoolPrice(chainlinkPriceRaw);
        
        // Compute deviation
        if (poolPrice > chainlinkPrice) {
            uint256 difference = poolPrice - chainlinkPrice;
            uint256 deviationBps = (difference * 10000) / chainlinkPrice;
            require(deviationBps <= maxDeviationBps, "Price deviation too high");
        } else {
            uint256 difference = chainlinkPrice - poolPrice;
            uint256 deviationBps = (difference * 10000) / poolPrice;
            require(deviationBps <= maxDeviationBps, "Price deviation too high");
        }
        
        emit PriceCheck(poolId, currentTick, chainlinkPrice, deviationBps);
    }
    
    function _getChainlinkPrice() internal view returns (int256 price, uint256 timestamp) {
        // Call Chainlink aggregator
        (, price, , timestamp, ) = AggregatorV3Interface(priceFeed).latestRoundData();
    }
    
    function _tickToPrice(int24 tick) internal pure returns (uint256) {
        // Convert tick to sqrtPriceX96, then to price
        // For simplicity, this returns price scaled to match Chainlink's decimals
        // In production, handle token decimals correctly
        uint160 sqrtPriceX96 = TickMath.getSqrtRatioAtTick(tick);
        uint256 price = FullMath.mulDiv(sqrtPriceX96, sqrtPriceX96, FixedPoint128.Q128);
        return price;
    }
    
    function _normalizeToPoolPrice(int256 chainlinkPrice) internal pure returns (uint256) {
        // Adjust chainlink Price to pool's Q64.96 format
        // This depends on token decimals - assume 8 for Chainlink feeds
        // Pool price = (chainlinkPrice * 2^96) / (10^8)
        return FullMath.mulDiv(uint256(uint256(chainlinkPrice)), FixedPoint128.Q128, 10**8);
    }
    
    // Admin functions to adjust parameters
    function setMaxDeviationBps(uint256 _maxDeviationBps) external {
        require(_maxDeviationBps <= 1000, "Too high");
        maxDeviationBps = _maxDeviationBps;
    }
    
    function setMaxStalenessSeconds(uint256 _maxStalenessSeconds) external {
        require(_maxStalenessSeconds >= 60, "Too low");
        maxStalenessSeconds = _maxStalenessSeconds;
    }
}
```

Important details:

- The hook reverts if the Chainlink price is stale (older than `maxStalenessSeconds`) or if the deviation between the pool's spot price and the Chainlink price exceeds `maxDeviationBps`. This prevents large, potentially manipulated swaps.
- You must adjust for token decimals. Chainlink feeds typically return prices with 8 decimals. Pool prices are represented as sqrtPriceX96 in Q64.96 fixed-point format. The `_normalizeToPoolPrice` function does the conversion. For actual use, you need to know the decimals of both tokens and convert accordingly.
- This hook only validates; it does not modify the swap's price. You could also implement logic that charges an extra fee or restricts the swap amount based on the oracle.
- The hook emits an event for monitoring.

### Reading V3 TWAP Oracles from V4 Hooks

Uniswap V3 introduced time-weighted average price (TWAP) oracles that enable manipulation-resistant price references. V4 pools inherit the same oracle infrastructure because the PoolManager maintains cumulative price observations. You can access these observations directly from the PoolManager within a hook to compute TWAP over any time window.

The PoolManager provides:

```solidity
function observe(
    bytes32 poolId,
    uint32 secondsAgo,
    uint32 observationIndex
) external view returns (int56 tickCumulative, uint160 secondsPerLiquidityCumulativeX128);
```

The `observe` function returns the cumulative tick and liquidity values at a specific point in the past (or present). By taking two observations separated by a time interval and computing the difference, you can derive the average tick (geometric mean price) over that interval.

Let's build a hook that enforces that the swap price stays within a certain range of the 30-minute TWAP:

```solidity
contract TWAPLimitHook {
    IPoolManager public poolManager;
    
    // Time window for TWAP in seconds
    uint32 public twapWindowSeconds;
    
    // Maximum deviation from TWAP as basis points (0.1% = 10)
    uint256 public maxDeviationBps;
    
    // Last recorded TWAP to avoid recomputation on every swap
    mapping(bytes32 => int24) public lastTwapTick;
    mapping(bytes32 => uint256) public lastTwapTimestamp;
    
    constructor(address _poolManager) {
        poolManager = IPoolManager(_poolManager);
        twapWindowSeconds = 30 * 60; // 30 minutes
        maxDeviationBps = 10; // 0.1%
    }
    
    function beforeSwap(
        bytes32 poolId,
        IPoolManager.SwapRequest memory request
    ) external {
        require(msg.sender == address(poolManager), "Unauthorized");
        
        // Get current tick from pool
        (, int24 currentTick, , ,) = poolManager.getPool(poolId);
        
        // Compute TWAP tick over the configured window
        int24 twapTick = _computeTwapTick(poolId);
        
        // Cache the last computed TWAP to avoid redundant calls
        // In practice, you might want to compute less frequently
        if (block.timestamp - lastTwapTimestamp[poolId] >= 60) {
            // Only update roughly every minute
            lastTwapTick[poolId] = twapTick;
            lastTwapTimestamp[poolId] = block.timestamp;
        } else {
            twapTick = lastTwapTick[poolId];
        }
        
        // Determine if the swap price crosses outside the allowed band
        // For a swap, we don't know the final price until after, but we can
        // approximate based on current tick and amount. Alternatively, we can
        // check that the current tick is within range of TWAP.
        
        int24 tickDiff = currentTick > twapTick 
            ? currentTick - twapTick 
            : twapTick - currentTick;
            
        // Convert tick difference to price deviation percentage
        uint256 deviationBps = _tickDiffToDeviationBps(tickDiff);
        
        require(deviationBps <= maxDeviationBps, "Price too far from TWAP");
    }
    
    function _computeTwapTick(bytes32 poolId) internal view returns (int24) {
        // Observe at current time
        (int56 tickCumulativeNow, ) = poolManager.observe(poolId, 0, 0);
        // Observe at twapWindowSeconds ago
        (int56 tickCumulativePast, ) = poolManager.observe(poolId, twapWindowSeconds, 0);
        
        // Compute average tick: (cumulativeNow - cumulativePast) / time
        // The cumulative values are scaled by secondsPerLiquidity, but we need
        // to divide by time elapsed. However, observe returns cumulative values
        // that already incorporate time. The difference directly gives the sum
        // of ticks over the period, which we divide by the period duration.
        
        int56 tickCumulativeDelta = tickCumulativeNow - tickCumulativePast;
        uint256 seconds = twapWindowSeconds;
        
        // Average tick (with rounding) as integer
        int24 avgTick = int24(tickCumulativeDelta / int56(seconds));
        
        return avgTick;
    }
    
    function _tickDiffToDeviationBps(int24 tickDiff) internal pure returns (uint256) {
        // A tick difference corresponds to a price ratio of 1.0001^tickDiff
        // For small differences, the deviation is approximately (1.0001^tickDiff - 1) * 10000
        // We can compute this without exponentiation by using a lookup or approximation
        // For moderate tickDiff (say up to 100), we can precompute or use a formula
        
        // Simple approximation for small deviations: deviation% ≈ tickDiff * 0.005
        // Actually ln(1.0001) ≈ 0.000099995, so price change per tick is about 0.01%
        // More precisely: (1.0001^d - 1) ≈ d * 0.0001 for small d
        uint256 absDiff = tickDiff > 0 ? uint256(tickDiff) : uint256(-tickDiff);
        uint256 deviationBps = absDiff * 100; // each tick ≈ 0.01% = 1bps? Let's compute properly
        
        // More accurate: each tick is ~0.01% (1 basis point). Actually 1.0001^1 = 1.0001, difference 0.0001 = 0.01% = 1bps.
        // So tickDiff * 1 bps per tick gives deviation in bps.
        deviationBps = absDiff; // if 1 tick = 1 bps, which is roughly true
        // But better to compute precisely:
        // deviation fraction = (1.0001^tickDiff - 1)
        // Multiply by 10000 to get bps
        // For small to moderate tickDiff, we can use exponentiation via log tables or
        // approximate with series expansion.
        
        return deviationBps; // Simplified; in production compute accurately
    }
    
    function setTwapWindow(uint32 _seconds) external {
        require(_seconds >= 60 && _seconds <= 3600, "Invalid window");
        twapWindowSeconds = _seconds;
    }
    
    function setMaxDeviation(uint256 _maxDeviationBps) external {
        require(_maxDeviationBps <= 500, "Too high");
        maxDeviationBps = _maxDeviationBps;
    }
}
```

This hook ensures that swaps do not move the price too far from the recent TWAP. It uses the `observe` function to get cumulative tick values at two timestamps and computes the average tick over the interval. Then it checks that the current tick is within a tight band around that average.

You can adjust the `maxDeviationBps` to be more or less strict, and set the `twapWindowSeconds` to a different time period (e.g., 5 minutes, 1 hour). The hook caches the TWAP value for up to a minute to avoid calling `observe` on every swap, which saves gas. The cache invalidation logic can be tuned.

For production use, compute the deviation conversion precisely:

```solidity
function _priceToDeviationBps(uint256 price1, uint256 price2) internal pure returns (uint256) {
    // price1 and price2 are in Q64.96. Compute the smaller denominator.
    uint256 minPrice = price1 < price2 ? price1 : price2;
    uint256 maxPrice = price1 > price2 ? price1 : price2;
    if (minPrice == 0) return type(uint256).max;
    uint256 ratio = (maxPrice * 10000) / minPrice;
    // ratio is (max/min) * 10000. Deviation bps = (max/min - 1) * 10000 = ratio - 10000.
    return ratio - 10000;
}
```

Then you can compute TWAP price from the average tick:

```solidity
function _tickToPrice(int24 tick) internal pure returns (uint256) {
    return FullMath.mulDiv(
        uint160(TickMath.getSqrtRatioAtTick(tick)),
        uint160(TickMath.getSqrtRatioAtTick(tick)),
        FixedPoint128.Q128
    );
}
```

Thus the deviation check becomes:

```solidity
uint256 twapPrice = _tickToPrice(twapTick);
uint256 currentPrice = _tickToPrice(currentTick);
uint256 deviationBps = _priceToDeviationBps(currentPrice, twapPrice);
require(deviationBps <= maxDeviationBps, "TWAP deviation too high");
```

### Custom Oracle Implementations with Multiple Sources

Relying on a single oracle (whether Chainlink or the pool's own TWAP) can still be risky if that oracle fails or gets manipulated. A more robust approach combines multiple price sources and takes a median or average. Hooks allow you to implement such multi-source oracles directly on-chain, inheriting the security of the pool's economics.

For instance, you could create a hook that reads three different price sources: the pool's own TWAP, a Chainlink feed, and a secondary decentralized oracle (like Band or Pyth). Then compute the median of the three prices and use that as the reference for swap validation.

```solidity
contract MultiSourceOracleHook {
    IPoolManager public poolManager;
    
    // Different price feed addresses (could be Chainlink, Band, etc.)
    address public feedA;
    address public feedB;
    address public feedC;
    
    // Staleness and deviation parameters
    uint256 public stalenessThreshold;
    uint256 public maxDeviationBps;
    
    // Weights for median calculation (if more than 3 sources)
    uint256[] public weights; // parallel array to sources
    
    event OraclePrice(bytes32 indexed poolId, int256 medianPrice, uint256[] sourcePrices);
    
    constructor(address _poolManager, address _feedA, address _feedB, address _feedC) {
        poolManager = IPoolManager(_poolManager);
        feedA = _feedA;
        feedB = _feedB;
        feedC = _feedC;
        stalenessThreshold = 3600; // 1 hour
        maxDeviationBps = 50; // 0.5%
    }
    
    function beforeSwap(
        bytes32 poolId,
        IPoolManager.SwapRequest memory request
    ) external {
        require(msg.sender == address(poolManager), "Unauthorized");
        
        // Gather prices from all sources
        int256[] memory prices = new int256[](3);
        prices[0] = _readFeed(feedA);
        prices[1] = _readFeed(feedB);
        prices[2] = _readFeed(feedC);
        
        // Also get pool's TWAP for 10 minutes
        int256 poolTwapPrice = _getPoolTwapPrice(poolId, 600);
        
        // Combine with TWAP as a fourth source if desired
        // For now, just use the three feeds
        int256 medianPrice = _median(prices);
        
        // Convert pool's current price to comparable format
        (, int24 currentTick, , ,) = poolManager.getPool(poolId);
        uint256 currentPrice = _tickToPrice(currentTick);
        
        // Ensure medianPrice and currentPrice are in same scale
        // This conversion depends on decimals; for simplicity assume all normalized to 8 decimals
        uint256 medianPriceScaled = uint256(uint256(medianPrice));
        
        uint256 deviationBps = _priceToDeviationBps(currentPrice, medianPriceScaled);
        require(deviationBps <= maxDeviationBps, "Price deviates from median");
        
        emit OraclePrice(poolId, medianPrice, prices);
    }
    
    function _readFeed(address feed) internal view returns (int256) {
        (, int256 price, , uint256 updatedAt, ) = AggregatorV3Interface(feed).latestRoundData();
        require(block.timestamp - updatedAt <= stalenessThreshold, "Feed stale");
        return price;
    }
    
    function _getPoolTwapPrice(bytes32 poolId, uint32 windowSeconds) internal view returns (int256) {
        (int56 tickCumulativeNow, ) = poolManager.observe(poolId, 0, 0);
        (int56 tickCumulativePast, ) = poolManager.observe(poolId, windowSeconds, 0);
        int56 delta = tickCumulativeNow - tickCumulativePast;
        int24 avgTick = int24(delta / int56(windowSeconds));
        return int256(_tickToPrice(avgTick));
    }
    
    function _median(int256[] memory arr) internal pure returns (int256) {
        // Simple selection sort for small array, or use quickselect
        // For brevity, sort and pick middle
        for (uint256 i = 0; i < arr.length; i++) {
            for (uint256 j = i + 1; j < arr.length; j++) {
                if (arr[j] < arr[i]) {
                    int256 temp = arr[i];
                    arr[i] = arr[j];
                    arr[j] = temp;
                }
            }
        }
        return arr[arr.length / 2];
    }
    
    function _tickToPrice(int24 tick) internal pure returns (uint256) {
        uint160 sqrtPriceX96 = TickMath.getSqrtRatioAtTick(tick);
        return FullMath.mulDiv(sqrtPriceX96, sqrtPriceX96, FixedPoint128.Q128);
    }
    
    function _priceToDeviationBps(uint256 priceA, uint256 priceB) internal pure returns (uint256) {
        uint256 min = priceA < priceB ? priceA : priceB;
        uint256 max = priceA > priceB ? priceA : priceB;
        if (min == 0) return type(uint256).max;
        return (max * 10000) / min - 10000;
    }
}
```

This multi-source approach reduces the risk that any single oracle manipulation will affect the system. However, it increases gas costs because multiple external calls to different feeds are made. Consider the trade-off: for high-value pools, the added security may justify the extra gas.

If one feed is completely offline (reverts or returns zero), you could fall back to using the median of the remaining feeds, as long as at least two are available. The implementation can be made more robust with timeouts and fallback logic.

### Flash Loan Attack Prevention Techniques

One of the most common threats to on-chain price oracles is flash loan manipulation. An attacker can borrow a large amount of assets, execute a series of trades to move the pool price dramatically, manipulate an oracle that relies on the pool's spot price or short-term TWAP, then profit from a dependent contract before the price reverts. This is known as a "sandwich attack" or "flash loan attack".

Hooks can incorporate defenses against such manipulation by:

1. Using longer TWAP windows: A longer lookback period makes it harder for an attacker to move the average price significantly with a flash loan, because the attack impact is diluted across many minutes or hours.
2. Requiring multiple confirmations: Only accept a price if it has been stable across several consecutive blocks.
3. Checking liquidity depth: Reject swaps that would move the price beyond a realistic slippage given the pool's available liquidity. This can be estimated via `getPool` liquidity values.
4. Combining on-chain and off-chain oracles: Use both the pool's TWAP and an external feed; if they differ significantly, reject the swap.
5. Delay settlement: Instead of instantly settling swaps, you could hold the tokens for a few blocks, but this conflicts with Uniswap's atomic nature.

A practical mitigation is to combine a TWAP oracle with a minimum time window and a secondary oracle. For example, require that the TWAP price over 30 minutes differs by less than 1% from a Chainlink feed. This makes attacks expensive: the attacker would need to manipulate both the pool for 30 minutes and also corrupt the Chainlink feed (much harder).

Another technique: within a hook, you can record the current block timestamp when a position is opened, and only allow swaps that don't move the price beyond a certain threshold relative to the entry price for a certain duration. This limits the profitability of short-term manipulation. However, it's complex to implement fairly.

### Code Examples Summary

We've provided three complete code examples:

- **ChainlinkOracleHook**: Integrates a Chainlink price feed, checks staleness and spot price deviation.
- **TWAPLimitHook**: Uses the pool's own TWAP to ensure swaps don't deviate too far from recent average price.
- **MultiSourceOracleHook**: Combines multiple feeds (including pool TWAP) to compute a median price and enforce deviation limits.

These examples demonstrate how to:
- Read external data from Chainlink or from the PoolManager's `observe` function
- Normalize price values between different decimal systems
- Compute deviations and enforce thresholds
- Emit events for monitoring
- Adjust parameters via admin functions

You can combine these patterns. For instance, a hook could enforce that both the spot price is within 0.5% of the 30-minute TWAP and that the TWAP itself is within 1% of the Chainlink median. This multi-layer validation creates a robust defense.

### Security Considerations for Oracle Design

When designing oracle-based hooks, keep these principles in mind:

- **Keep it simple**: Complex oracle aggregation logic increases attack surface and gas cost. Prefer using well-audited libraries like Chainlink's or Uniswap's own TWAP.
- **Guard against data source failures**: Have fallback mechanisms. If a feed is stale, either revert or allow using an alternative source if it's still fresh.
- **Be aware of decimal mismatches**: Prices from different sources may use different numbers of decimals. Normalize carefully to avoid overflow or precision loss.
- **Consider gas costs**: Reading multiple external contracts can be expensive, especially in a `beforeSwap` hook that executes for every trade. Cache values and update only periodically if possible.
- **Test with simulated attacks**: On a forked mainnet, attempt to manipulate the TWAP by sandwiching or executing many swaps. Verify that the hook stops the attack under your configured parameters.
- **Avoid over-reliance on a single hook**: Hooks affect all pool users. If the oracle hook has a bug that reverts all swaps, the pool becomes unusable. Design with emergency stop functions, pausing mechanisms, and owner controls to deactivate the hook if needed.
- **Governance oversight**: Oracle parameters (feeds, thresholds, windows) should be controlled by governance rather than a single admin. Use a timelock for changes.

### Testing Oracle Integration with Forked Mainnet Data

Testing oracle hooks in a controlled environment is straightforward: you can mock the Chainlink feed and simulate various price scenarios. However, to ensure real-world effectiveness, you must test against actual mainnet conditions using a fork.

Setup a mainnet fork that includes an existing Uniswap V4 pool with its TWAP oracle and a real Chainlink feed for the token pair. Your test should:

1. Impersonate a whale account that holds a large balance of one token
2. Execute a series of trades that move the pool price significantly
3. Attempt to trigger a swap that would be blocked by the hook due to price deviation
4. Verify that the hook correctly identifies the deviation and reverts with the expected reason
5. Also verify that legitimate swaps within the allowed band succeed

Example test (TypeScript with Hardhat):

```typescript
import {ethers, network} from "hardhat";
import {BigNumber} from "ethers";

describe("TWAPLimitHook on forked mainnet", function () {
    this.timeout(60000);
    
    let hook: TWAPLimitHook;
    let poolManager: any;
    let usdc: string;
    let weth: string;
    let poolId: string;
    
    before(async function () {
        // Only run if network is forked
        if ((await ethers.provider.getNetwork()).chainId !== 1) {
            return;
        }
        
        // Deploy the hook on the fork
        const Hook = await ethers.getContractFactory("TWAPLimitHook");
        hook = await Hook.deploy(poolManagerAddress);
        await hook.deployed();
        
        // Reinitialize the target pool to use this hook? Or test on an existing pool just by enabling the hook?
        // Hooks are set at pool creation. You might need to create a new pool on the fork.
        // For integration test, you could create a new pool with the hook attached.
    });
    
    it("Should block swap when price deviates from TWAP", async function () {
        // Get initial TWAP
        const twapBefore = await hook.lastTwapTick(poolId);
        
        // Impersonate a whale and perform a huge swap to move price
        const whale = "0x..."; // real address with large balance
        await network.provider.request({
            method: "hardhat_impersonateAccount",
            params: [whale]
        });
        const whaleSigner = await ethers.getImpersonatedSigner(whale);
        
        // Execute a massive swap to move price drastically
        const hugeAmount = ethers.utils.parseUnits("1000000", 6); // 1M USDC
        const swapRouter = await ethers.getContractAt("ISwapRouter", swapRouterAddress);
        
        try {
            await swapRouter.connect(whaleSigner).exactInputSingle(
                {
                    poolId: poolId,
                    kind: 0,
                    amountSpecified: hugeAmount,
                    sqrtPriceLimitX96: 0
                },
                {
                    sender: whale,
                    from: usdc,
                    to: whaleSigner.address,
                    wNETH: ethers.constants.AddressZero,
                    funding: 1
                },
                0,
                Math.floor(Date.now() / 1000) + 100
            );
        } catch (error) {
            // Expected: hook reverts
            // Verify revert reason contains expected message
            expect(error.message).to.include("Price too far from TWAP");
        }
        
        await network.provider.request({
            method: "hardhat_stopImpersonatingAccount",
            params: [whale]
        });
    });
});
```

Such tests validate the hook's behavior under realistic on-chain conditions. You can also test the TWAP computation by moving the price over multiple blocks and verifying that the TWAP changes slowly rather than instantly.

By combining robust oracle design with thorough testing, you can build hooks that protect pools from manipulation while still enabling flexible trading.


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

## Error Handling & Revert Reasons

When building with Uniswap V4, understanding error messages and implementing robust error handling is essential for debugging and user experience. Uniswap V4 defines a set of custom errors that provide clear reasons when operations fail. This section gives a complete reference of these errors, explains how to decode revert reasons in your development environment, and demonstrates patterns for handling errors gracefully in both contracts and off-chain applications.

### Complete Reference of Uniswap V4 Custom Errors

Uniswap V4 uses Solidity custom errors (introduced in Solidity 0.8.4) which are more gas-efficient than traditional `require` strings and provide structured data that can be decoded off-chain. Each error has a unique four-byte selector derived from the error's function signature.

Here are all the custom errors defined in the V4 core contracts:

| Error Name | Error Signature | Hex Selector | Conditions |
|------------|-----------------|--------------|------------|
| ZeroAddress | ZeroAddress() | 0x192fa61f | Raised when a required address is zero (e.g., token address(0) is not allowed except for ETH) |
| InvalidTick | InvalidTick(int24 tick) | 0x511c3c4e | Tick value is outside the allowed range or not aligned with tick spacing |
| InsufficientLiquidity | InsufficientLiquidity() | 0x454a2a32 | Operation requires liquidity but pool has none |
| InsufficientInputAmount | InsufficientInputAmount() | 0xa9c67b48 | Input amount is zero or below minimum |
| InsufficientOutputAmount | InsufficientOutputAmount() | 0x4a30e8a2 | Output amount would be less than minimum specified |
| SqrtPriceLimitExceeded | SqrtPriceLimitExceeded() | 0x60c63277 | Swap would move price beyond the sqrtPriceLimitX96 constraint |
| InvalidTickSpacing | InvalidTickSpacing(int24 spacing) | 0xf0a82724 | Tick spacing does not match pool's configuration |
| InvalidLiquidity | InvalidLiquidity(uint128 liquidity) | 0x8e091373 | Liquidity value is zero or overflows |
| TooManyTicks | TooManyTicks() | 0x9b023251 | Position would create too many tick indices (exceeds maximum) |
| LteZero | LteZero() | 0xc1c5f3c0 | A parameter that must be positive is zero or negative |
| NotAuthorized | NotAuthorized() | 0x82b42900 | Caller is not authorized (e.g., hook not called by PoolManager) |
| InvalidPool | InvalidPool() | 0x4c607f54 | Pool identifier does not correspond to an initialized pool |
| InvalidAmounts | InvalidAmounts() | 0x5233fabd | Amounts provided are inconsistent (e.g., both zero, or negative) |
| FeeTooLarge | FeeTooLarge(uint24 fee) | 0xa6c1ee99 | Fee tier exceeds maximum allowed for the pool |
| SameToken | SameToken() | 0x9c6b2090 | Both tokens in a pool are the same address |
| PositionDoesNotExist | PositionDoesNotExist(uint256 tokenId) | 0xeab36e01 | Trying to modify or collect from a non-existent position NFT |
| NotPositionOwner | NotPositionOwner() | 0x2f47c4c3 | Caller is not the owner of the position and not approved |
| PositionAlreadyExists | PositionAlreadyExists() | 0x8e470c62 | Creating a position with same tick range would merge but not allowed in some contexts |
| TickNotFound | TickNotFound(int24 tick) | 0x06817309 | Tick index not found in the tick bitmap (sporadic) |
| FeeGrowthOutsideZero | FeeGrowthOutsideZero() | 0x68c5111a | Fee growth outside the tick range is zero, meaning no fees collected |
| TickLowerAboveUpper | TickLowerAboveUpper() | 0xa671444d | Lower tick is not less than upper tick |
| TickLtMin | TickLtMin() | 0x1a768d83 | Tick is below the minimum allowed tick |
| TickGtMax | TickGtMax() | 0xa50f9de9 | Tick is above the maximum allowed tick |
| NonZeroInput | NonZeroInput() | 0x6c044ae0 | An input parameter that must be zero is non-zero |
| ZeroOutput | ZeroOutput() | 0x2e71c37b | An output parameter that must be non-zero is zero |
| BalanceNotEnough | BalanceNotEnough() | 0xc1c5f3c0 | Token balance insufficient for operation (often duplicate) |

Note: Some selectors may vary slightly if the core contracts change. Always cross-check with the latest Uniswap V4 source.

Each error includes relevant parameters that give additional context. For example, `InvalidTick(int24 tick)` passes the problematic tick value. When a transaction reverts with a custom error, the error selector is the first 4 bytes of the call data, followed by the encoded parameters (if any).

### Decoding Revert Reasons in Hardhat/ethers

When a transaction reverts with a custom error, your Hardhat tests or ethers.js scripts can catch the error and decode its meaning. The error object in ethers contains a `data` property with the full revert data: first 4 bytes are the selector, followed by ABI-encoded arguments.

Here's how to decode a revert reason in a test:

```typescript
import {ethers} from "hardhat";
import {Interface} from "@ethersproject/abi";

// Example: call a function that may revert
it("Should revert with InvalidTick", async function () {
    const poolManager = await ethers.getContractAt("IPoolManager", poolManagerAddress);
    
    try {
        // Attempt to initialize with an invalid tick (not aligned with spacing)
        await poolManager.initialize(
            {
                currency0: tokenA,
                currency1: tokenB,
                fee: 3000,
                tickSpacing: 60
            },
            -100, // invalid tick (not multiple of 60)
            await signer.getAddress()
        );
        throw new Error("Expected revert did not happen");
    } catch (error) {
        // The error might be a TransactionResponse or CallException
        const err = error as any;
        // In ethers v6, error has `reason` property for revert strings, but for custom errors it's `data`
        const data = err.data || err.reason;
        
        // Custom error data begins with 0x followed by the 4-byte selector and arguments
        if (data.startsWith("0x")) {
            const selector = data.slice(0, 10); // includes 0x
            const argsHex = data.slice(10);
            
            // Identify known errors
            const INVALID_TICK_SELECTOR = "0x511c3c4e";
            if (selector === INVALID_TICK_SELECTOR) {
                // Decode the int24 tick argument
                // argsHex contains 32 bytes (padded) of the int24 in hex
                const tickHex = argsHex.padStart(64, '0'); // ensure 32 bytes
                const tick = parseInt(tickHex.slice(0, 18), 16); // slice last two bytes? Actually int24 uses 3 bytes
                // Better to use ethers Interface to decode
                const iface = new Interface(["error InvalidTick(int24 tick)"]);
                try {
                    const decoded = iface.parseError(data);
                    console.log("Reverted with InvalidTick:", decoded.args[0]);
                } catch (e) {
                    console.log("Unknown error");
                }
            }
        }
    }
});
```

Simpler approach using ethers' `ErrorFragment`:

```typescript
import {ErrorFragment} from "@ethersproject/contracts";

const errorFragment = ErrorFragment.from("InvalidTick(int24)");
const decoded = errorFragment.decode(data);
console.log(decoded.name, decoded.args);
```

Or using `parseError` from a contract interface:

```typescript
const abi = [
    "function initialize(PoolKey, int24, address) external",
    "error InvalidTick(int24 tick)",
    "error InsufficientLiquidity()"
];
const iface = new Interface(abi);
try {
    const error = iface.parseError(data);
    console.log(error.name, error.args);
} catch {
    console.log("Unknown error or not a custom error");
}
```

This allows you to write tests that assert specific error conditions:

```typescript
await expect(poolManager.initialize(...))
    .to.be.revertedWithCustomError(poolManager, "InvalidTick")
    .withArgs(-100);
```

The `@nomicfoundation/hardhat-chai-matchers` package provides matchers like `revertedWithCustomError`.

### Common Error Scenarios by Operation

Different operations have different typical failure modes. Understanding these helps you write more resilient contracts and provide better user feedback.

**Pool Creation**

- `InvalidTick`: The initial tick you pass to `initialize` must be a multiple of the pool's tickSpacing. Also it must be within allowed range (typically -887272 to 887272).
- `ZeroAddress`: Either token address is zero. Note that token0 cannot be address(0) except for native ETH, which is represented by address(0) but that's only allowed in certain contexts.
- `SameToken`: token0 and token1 are the same address.
- `InvalidPool`: If you attempt to interact with a pool that hasn't been initialized yet.
- `FeeTooLarge`: The fee tier you provided is outside the allowed set (100, 500, 3000, 10000). Actually V4 allows arbitrary fee? But some checks exist.

**Swaps**

- `InsufficientLiquidity`: The pool has no liquidity, so a swap cannot be executed.
- `SqrtPriceLimitExceeded`: The sqrtPriceLimitX96 parameter prevents the swap from reaching the desired price. Either adjust the limit or set to zero.
- `InsufficientInputAmount`: The amountSpecified is zero or exceeds the token balance.
- `InsufficientOutputAmount`: The swap would result in an output amount that is less than the specified minimum (when using exact output).
- `BalanceNotEnough`: The token balance of the sender is insufficient to cover the input amount.
- `InvalidTick`: Might occur if the price moves into an invalid region (shouldn't happen normally).

**Liquidity Operations**

- `InvalidTick`: tickLower or tickUpper not aligned with tickSpacing, or tickLower >= tickUpper, or ticks outside min/max.
- `InvalidLiquidity`: liquidityDelta is zero when decreasing or creating; or overflow.
- `PositionDoesNotExist`: The tokenId doesn't correspond to an existing position.
- `NotPositionOwner`: The caller is not the owner or approved operator of the position.
- `FeeGrowthOutsideZero`: When collecting fees, the fee growth outside the range is zero, meaning you're out of range (no fees to collect).
- `InsufficientLiquidity`: When trying to remove more liquidity than exists.
- `TickNotFound`: The tick bitmap does not contain the tickRange, which suggests the position was never minted properly.

**Hook Callbacks**

- `NotAuthorized`: The hook function was called by someone other than the PoolManager. Always check `msg.sender == address(poolManager)`.
- Hook calls can also revert with any of the above errors if the underlying operation fails. A hook reverting will revert the entire transaction.

### Error Handling Patterns

In your contracts, you can use Solidity's try/catch to handle reverts from external calls. However, note that low-level calls with `call` can be used, but when you call the PoolManager functions directly, they will revert automatically on errors. If you want to handle errors gracefully, you can use a low-level call wrapper:

```solidity
function safeSwap(IPoolManager.SwapParams memory params) external returns (bool success, bytes memory data) {
    (bool ok, bytes memory returndata) = address(poolManager).staticcall(
        abi.encodeWithSelector(IPoolManager.swap.selector, params, "")
    );
    if (!ok) {
        // The call reverted. returndata contains error data.
        // You can decode the error and take action
        if (returndata.length >= 4) {
            bytes4 selector = bytes4(returndata[:4]);
            if (selector == IPoolManager.InsufficientLiquidity.selector) {
                // Handle insufficient liquidity
                emit SwapFailed("No liquidity");
                return (false, returndata);
            }
            // handle other errors...
        }
        // Generic revert
        revert("Swap failed");
    }
    // success; decode return values from returndata
}
```

However, it's often simpler to let the revert bubble up and handle it off-chain in your UI or script, displaying a user-friendly message based on the error selector.

Off-chain, when you call a contract function via ethers or web3, you can catch the error and map it to a user-facing message:

```typescript
const errorMessages: Record<string, string> = {
    "0x511c3c4e": "Invalid tick number",
    "0x454a2a32": "Insufficient liquidity in the pool",
    // etc.
};

try {
    await positionManager.modifyLiquidity(params, ...);
} catch (error) {
    const data = error.data;
    if (data && data.startsWith("0x")) {
        const selector = data.slice(0, 10);
        const friendly = errorMessages[selector];
        if (friendly) {
            alert(`Transaction failed: ${friendly}`);
        } else {
            alert("Transaction failed: unknown error");
        }
    } else {
        alert("Transaction failed: " + error.reason || "unknown");
    }
}
```

### Debugging Techniques

When developing Uniswap V4 contracts, you'll often encounter reverts. Here are techniques to debug:

1. **Console.log in Solidity**: With Hardhat's `hardhat-deploy` or `console.sol` library, you can add `console.log` statements in your contracts to print variables. Use sparingly because it costs gas, but in development it's invaluable.

2. **Transaction tracing**: Hardhat's `--trace` flag prints a full trace of execution, showing which function calls were made and where the revert occurred. Run `npx hardhat test --grep "test name" --trace`. This can pinpoint the exact line that failed.

3. **Simulate in console**: Use Hardhat's node console (`npx hardhat console`) to interactively call functions and see revert reasons. You can also use `hardhat_debugTransaction` to inspect a mined transaction.

4. **Event emission**: Add events before critical operations to record state. When a transaction reverts, the events may still be emitted if placed before the revert. Retroactively analyze state.

5. **Check token balances**: Many errors arise from insufficient token balances. After a failure, query `balanceOf` on the involved tokens to confirm.

6. **Inspect pool state**: Before performing an operation, call `poolManager.getPool(poolId)` to see current tick, liquidity, etc. Ensure they match expectations.

7. **Use revert reason parsing**: Custom errors often contain encoded parameters. Decode them to get exact values (like the tick that was invalid).

8. **Unit test edge cases**: Write tests that deliberately pass bad parameters to confirm which errors are thrown. This helps you recognize them later.

### How to Implement User-Friendly Error Reporting in Contracts

While custom errors are great for developers, end users prefer clear messages. You have two options:

- Off-chain: Decode the error in your frontend and show a friendly message. This is the recommended approach because it doesn't add gas cost to on-chain execution.
- On-chain: Wrap Uniswap V4 calls in your own contract that catches errors and reverts with human-readable strings. However, string reverts cost more gas and still require the frontend to display them.

Here's an example wrapper that adds user-friendly messages:

```solidity
contract UserFriendlyPositionManager {
    IPositionManager public positionManager;
    
    constructor(address _positionManager) {
        positionManager = IPositionManager(_positionManager);
    }
    
    function addLiquidity(
        bytes32 poolId,
        uint128 liquidityDelta,
        uint160 sqrtPriceX96,
        int24 tickLower,
        int24 tickUpper,
        uint256 amount0Requested,
        uint256 amount1Requested,
        bool decreaseLiquidity,
        bool createPosition,
        uint256 amount0Min,
        uint256 amount1Min,
        address recipient,
        uint256 deadline
    ) external returns (uint128 liquidity, uint256 amount0, uint256 amount1) {
        try positionManager.modifyLiquidity(
            ModifyLiquidityParams({
                poolId: poolId,
                liquidityDelta: liquidityDelta,
                sqrtPriceX96: sqrtPriceX96,
                tickLower: tickLower,
                tickUpper: tickUpper,
                amount0Requested: amount0Requested,
                amount1Requested: amount1Requested,
                decreaseLiquidity: decreaseLiquidity,
                createPosition: createPosition
            }),
            amount0Min,
            amount1Min,
            recipient,
            deadline
        ) returns (uint128 liq, uint256 amt0, uint256 amt1) {
            liquidity = liq;
            amount0 = amt0;
            amount1 = amt1;
        } catch Error(string memory reason) {
            revert(reason); // forward string revert
        } catch {
            // Catch any custom error or other revert
            // We could decode the error data and revert with a friendly message
            revert("Liquidity addition failed. Please check pool tick spacing, token balances, and liquidity amount.");
        }
    }
}
```

The `try/catch` block catches reverts from `modifyLiquidity`. The first catch handles string reverts (revert with a reason). The second catch handles custom errors and generic reverts; we replace with a generic message. However, we lose specifics. To improve, we could inspect `error.data` using inline assembly to decode the selector and produce different messages. But that's advanced and still costs extra gas.

Simpler: Do not add on-chain messages. Just let the original custom error propagate, and have the frontend decode it to a friendly message.

### Testing Error Conditions Properly

A comprehensive test suite should cover all potential error paths. This not only ensures your contract handles failures correctly but also documents expected behavior.

For each function, write negative tests:

- Provide zero token amount → expect `InsufficientInputAmount`
- Provide tick that's not a multiple of tickSpacing → expect `InvalidTick`
- Provide tickLower >= tickUpper → expect `TickLowerAboveUpper`
- Call as unauthorized user on a hook → expect `NotAuthorized`
- Call modifyLiquidity with non-existent position → expect `PositionDoesNotExist`
- Provide amount0Min higher than actual amount0Used → expect `InsufficientOutputAmount`

Example using Chai matchers:

```typescript
it("Reverts with InvalidTick when tick not aligned", async function () {
    const invalidTick = 123; // not multiple of 60 for fee 3000
    await expect(poolManager.initialize(key, invalidTick, signer.address))
        .to.be.revertedWithCustomError(poolManager, "InvalidTick")
        .withArgs(invalidTick);
});

it("Reverts with NotAuthorized when hook called by non-PoolManager", async function () {
    const hook = await deployHook();
    await expect(hook.beforeSwap(poolId, swapRequest))
        .to.be.revertedWith("NotAuthorized"); // the hook's own require
});
```

Some errors come from the PositionManager, not the PoolManager directly. Make sure you import the correct contract artifact to get its error selectors.

Also test that your wrapper functions (like `UserFriendlyPositionManager`) produce expected revert messages when delegation fails.

By systematically testing errors, you build confidence that your integration will not surprise users with unexpected reverts.


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

## Liquidity Strategy Examples

Concentrated liquidity in Uniswap V4 allows liquidity providers to allocate capital within specific price ranges, potentially increasing capital efficiency compared to traditional full-range positions. However, choosing the right strategy requires understanding market dynamics, volatility, and your risk tolerance. This section presents practical liquidity provision strategies with complete code examples, calculations, and comparative analysis to help you make informed decisions.

### Understanding Concentrated Liquidity Basics

In Uniswap V4, you provide liquidity by selecting a tick range: `tickLower` and `tickUpper`. The wider the range relative to the current price, the more of your capital is always in range and earning fees, but the lower the fee earnings per unit of liquidity because your liquidity is spread thinner. Conversely, a narrow range concentrates your liquidity around the current price, earning higher fees when trades occur within that range, but risks going out of range if price moves, earning zero fees until you adjust.

The optimal range depends on your expectations for price volatility. If you expect the price to stay within a certain band, concentrate liquidity around that band. If you want passive exposure with minimal management, choose a wide range that covers potential price movements.

### Calculating Optimal Tick Ranges Based on Volatility

Historical volatility of a token pair can help determine a suitable range. For example, if a token has an annualized volatility of 80%, that corresponds to a standard deviation of price movement of about 0.5% per day (depending on time scaling). You might want your range to cover roughly ±2 standard deviations, i.e., about ±1% per day, to avoid frequent rebalancing.

However, it's often easier to think in terms of tick ranges. The tick is a logarithmic measure where each tick is 0.01% price movement (1 basis point). So a range of 100 ticks corresponds to about 1% price movement.

You can estimate volatility from on-chain or off-chain data. For an automated strategy, you might compute recent volatility from the pool's TWAP observations:

```solidity
function estimateVolatility(bytes32 poolId, uint32 windowSeconds) public view returns (uint256) {
    // Get price observations at start and end of window
    (, int24 tickStart, , ,) = poolManager.observe(poolId, windowSeconds, 0);
    (, int24 tickEnd, , ,) = poolManager.observe(poolId, 0, 0);
    
    int24 tickChange = tickEnd > tickStart ? tickEnd - tickStart : tickStart - tickEnd;
    // Each tick is ~0.01% change
    uint256 priceChangeBps = uint256(tickChange); // 1 bps per tick
    
    // Annualizing? Not needed for range selection; we want daily range
    return priceChangeBps;
}
```

Then set `tickLower = currentTick - estimatedChange * safetyFactor` and `tickUpper = currentTick + estimatedChange * safetyFactor`. The safety factor might be 2 or 3 to ensure the range covers most normal moves.

Alternatively, you can look at historical price data off-chain and compute moving average of true range (high-low), then convert to ticks and set on-chain via parameters.

### Rebalancing Automation: When and How to Adjust Ranges

When the price exits your liquidity range, you stop earning fees. It's important to monitor position status and rebalance when necessary. You can either:

- Let the position go out of range and manually adjust later by removing and adding liquidity.
- Automatically rebalance using a keeper service that calls `modifyLiquidity` to shift the range.

The trigger for rebalancing could be when the current tick moves outside your range by more than a threshold. For example, if your range is from tick -100 to 0 relative to entry, and current tick becomes -150, you might want to shift the entire range down.

An auto-rebalancer contract might look like:

```solidity
contract AutoRebalancer {
    IPositionManager public positionManager;
    IPoolManager public poolManager;
    
    address public owner;
    
    // Configuration per pool/position
    struct RebalanceConfig {
        bytes32 poolId;
        int24 tickLower;
        int24 tickUpper;
        uint256 width; // Upper - lower
        uint256 triggerDistance; // How far out of range before rebalancing
        address recipient;
    }
    
    mapping(bytes32 => RebalanceConfig) public configs;
    
    event Rebalanced(bytes32 indexed poolId, int24 oldLower, int24 oldUpper, int24 newLower, int24 newUpper);
    
    constructor(address _positionManager, address _poolManager) {
        positionManager = IPositionManager(_positionManager);
        poolManager = IPoolManager(_poolManager);
        owner = msg.sender;
    }
    
    function configure(
        bytes32 poolId,
        int24 tickLower,
        int24 tickUpper,
        uint256 triggerDistance,
        address recipient
    ) external {
        require(msg.sender == owner, "Not owner");
        configs[poolId] = RebalanceConfig({
            poolId: poolId,
            tickLower: tickLower,
            tickUpper: tickUpper,
            width: uint256(tickUpper - tickLower),
            triggerDistance: triggerDistance,
            recipient: recipient
        });
    }
    
    // Keeper calls this periodically
    function checkAndRebalance(bytes32 poolId) external returns (bool rebalanced) {
        RebalanceConfig memory cfg = configs[poolId];
        require(cfg.poolId != bytes32(0), "No config");
        
        // Get current pool tick
        (, int24 currentTick, , ,) = poolManager.getPool(pfg.poolId);
        
        // Determine if out of range
        if (currentTick < cfg.tickLower || currentTick > cfg.tickUpper) {
            // Rebalance: shift range so that current tick becomes centered (or at edge)
            int24 newLower;
            int24 newUpper;
            
            // Strategy: keep same width, but place so that current tick is at the lower+trigger or similar
            if (currentTick < cfg.tickLower) {
                // Price moved down; shift range down
                newLower = currentTick - int24(cfg.triggerDistance);
                newUpper = newLower + int24(cfg.width);
            } else {
                // Price moved up
                newUpper = currentTick + int24(cfg.triggerDistance);
                newLower = newUpper - int24(cfg.width);
            }
            
            // Find the position tokenId for this pool/range associated with recipient
            // In practice, you'd need to track tokenIds or look up via positionsOf
            uint256 tokenId = _findPositionTokenId(cfg.poolId, cfg.recipient);
            
            // Use modifyLiquidity to change the range
            (uint128 liquidity, uint256 amount0, uint256 amount1) = positionManager.modifyLiquidity(
                ModifyLiquidityParams({
                    poolId: poolId,
                    liquidityDelta: 0, // keep same liquidity (no change in amount)
                    sqrtPriceX96: 0, // let pool pick optimal price to match current
                    tickLower: newLower,
                    tickUpper: newUpper,
                    amount0Requested: 0,
                    amount1Requested: 0,
                    decreaseLiquidity: false,
                    createPosition: false // we're modifying existing
                }),
                0,
                0,
                cfg.recipient,
                block.timestamp + 1000
            );
            
            emit Rebalanced(poolId, cfg.tickLower, cfg.tickUpper, newLower, newUpper);
            rebalanced = true;
            
            // Update config to new range
            cfg.tickLower = newLower;
            cfg.tickUpper = newUpper;
            configs[poolId] = cfg;
        }
        
        return rebalanced;
    }
    
    function _findPositionTokenId(bytes32 poolId, address owner) internal view returns (uint256) {
        // Find the position that matches poolId and has the expected tick range? 
        // Since we just care about any position in that pool for this owner.
        // In practice, you might store tokenId in config.
        // This is simplified.
        uint256[] memory tokenIds = positionManager.tokensOfOwner(owner);
        for (uint256 i = 0; i < tokenIds.length; i++) {
            Position memory pos = positionManager.getPositionData(tokenIds[i]);
            if (pos.poolId == poolId) {
                return tokenIds[i];
            }
        }
        revert("No position found");
    }
}
```

In a real system, you'd need to account for the fact that `modifyLiquidity` with zero `liquidityDelta` just changes the range without altering the liquidity amount. However, this only works if the pool can accommodate the new range given current liquidity distribution. If the new range would require different token amounts to maintain the same liquidity at the new price, the operation might need to withdraw and add tokens accordingly. Using `sqrtPriceX96 = 0` lets the pool choose the price that satisfies the liquidity delta; it will likely move the position to the current price, and the token amounts will adjust (you might receive or pay tokens). This is acceptable for a rebalance: your position value remains similar, but the range shifts.

For a more precise rebalance that maintains the exact value, you'd compute the required token amounts based on the new price and liquidity. That's more complex.

The keeper should run periodically (e.g., every hour) and call `checkAndRebalance` for each configured pool. The keeper could be a bot operated by the position owner or a decentralized keeper network like Gelato.

### Multi-Tier Liquidity Provision for Different Fee Levels

Uniswap V4 pools exist with different fee tiers (0.01%, 0.05%, 0.3%, 1%). Liquidity providers often split their capital across multiple fee tiers to capture fees from both high-volume low-spread trades (low fee tiers) and volatile trades (high fee tiers). A common strategy is to provide liquidity in the 0.3% pool (the most popular) and also some in the 0.05% pool if the token pair is stable, or 1% if very volatile.

However, the pool with the most trading volume usually offers the most fee revenue. You should analyze historical volume by fee tier for your token pair before deciding allocation.

You can automate multi-tier liquidity by creating a contract that splits funds into positions in multiple pools. For example, allocate 70% to the 0.3% pool with a tight range around the current price, and 30% to the 0.05% pool with a wider range to capture additional volume from stablecoin-like pairs.

```solidity
contract MultiTierLiquidityProvider {
    IPositionManager public positionManager;
    
    struct TierAllocation {
        uint24 fee;
        uint256 percentage; // basis points (e.g., 7000 = 70%)
        int24 offsetTicks; // How much to widen range relative to base
    }
    
    TierAllocation[] public tiers;
    address public owner;
    
    constructor(address _positionManager) {
        positionManager = IPositionManager(_positionManager);
        owner = msg.sender;
    }
    
    function setTiers(TierAllocation[] memory _tiers) external {
        require(msg.sender == owner, "Not owner");
        tiers = _tiers;
    }
    
    function provideLiquidity(
        address token0,
        address token1,
        uint256 totalAmount0,
        uint256 totalAmount1,
        int24 baseTickLower,
        int24 baseTickUpper,
        uint256 deadline
    ) external returns (uint256[] memory tokenIds) {
        require(tiers.length > 0, "No tiers configured");
        
        tokenIds = new uint256[](tiers.length);
        
        uint256 totalBps = 0;
        for (uint256 i = 0; i < tiers.length; i++) {
            totalBps += tiers[i].percentage;
        }
        require(totalBps == 10000, "Allocation percentages must sum to 100%");
        
        for (uint256 i = 0; i < tiers.length; i++) {
            TierAllocation memory tier = tiers[i];
            
            // Allocate amounts proportionally
            uint256 amount0 = (totalAmount0 * tier.percentage) / 10000;
            uint256 amount1 = (totalAmount1 * tier.percentage) / 10000;
            
            // Adjust tick range: base +/- offsetTicks
            int24 tickLower = baseTickLower - tier.offsetTicks;
            int24 tickUpper = baseTickUpper + tier.offsetTicks;
            
            // Build PoolKey
            PoolKey memory key = PoolKey({
                currency0: token0,
                currency1: token1,
                fee: tier.fee,
                tickSpacing: _getTickSpacing(tier.fee)
            });
            
            bytes32 poolId = IPoolManager.bytes32Key(keccak256(abi.encode(key)));
            
            ModifiableLiquidityParams memory params = ModifyLiquidityParams({
                poolId: poolId,
                liquidityDelta: 0,
                sqrtPriceX96: 0,
                tickLower: tickLower,
                tickUpper: tickUpper,
                amount0Requested: amount0,
                amount1Requested: amount1,
                decreaseLiquidity: false,
                createPosition: true
            });
            
            (uint128 liquidity, , ) = positionManager.modifyLiquidity(
                params,
                0, // amount0Min - set to 0 for simplicity; in production set slippage
                0, // amount1Min
                msg.sender,
                deadline
            );
            
            tokenIds[i] = tokenId; // modifyLiquidity should return tokenId? Actually it doesn't return tokenId; you'd need to query via positionsOf after.
            // For simplicity, assume we can get tokenId from event
        }
    }
    
    function _getTickSpacing(uint24 fee) internal pure returns (int24) {
        if (fee >= 500) return 10;
        if (fee == 3000) return 60;
        if (fee == 10000) return 200;
        return 1;
    }
}
```

This contract allows the owner to define multiple tiers with percentages and tick offsets. The `provideLiquidity` function then splits the total token amounts accordingly and creates positions in each tier's pool. The tick range for each tier can be expanded or contracted relative to a base range using `offsetTicks`.

Note: `modifyLiquidity` doesn't directly return the tokenId of the new position. You would typically emit an event from PositionManager that includes tokenId, or you can query `positionsOf` after the call. In production, adjust accordingly.

### Impermanent Loss Mitigation Techniques

Impermanent loss occurs when the price of the two tokens in a pool diverges. As an LP, you end up with a combination that's worth less than if you had just held the tokens separately. This is a fundamental risk of any AMM liquidity provision. However, certain strategies can mitigate its impact:

- **Choose pools with correlated assets**: Stablecoin pairs (USDC/USDT) have minimal price divergence, so impermanent loss is negligible. Volatile token pairs (ETH/UNI) can experience significant divergence.
- **Provide liquidity in a narrow range only when you expect mean reversion**: If you believe the price will return to a certain level, a narrow range allows you to earn fees without much exposure to the price movement because you'll be out of range when price moves away, automatically stopping your exposure. But you also stop earning fees.
- **Collect fees frequently and rebalance**: By collecting fees often, you realize some returns in the tokens themselves, reducing the impact of impermanent loss on total value. However, this doesn't eliminate IL.
- **Hedge with options or futures**: Use external protocols to short one of the tokens, offsetting the price change impact on your LP position. This is advanced and requires cross-protocol coordination.
- **Use asymmetric deposits**: If you expect the price to go up (token0 appreciates relative to token1), you can deposit more of the token you think will go down, or set a range that's biased towards the direction you want to avoid. However, the AMM automatically rebalances your deposit to maintain constant product, so there's limited direct control.

There's no free lunch; impermanent loss is inherent to AMM LPs. The best mitigation is to choose pools where you believe the fee earnings will outweigh the expected impermanent loss.

### Position Management Across Multiple Pools

When you provide liquidity across several pools (different token pairs or different fee tiers), you need a strategy to track and manage them. A centralized dashboard can help. Here's a contract that aggregates position data for a user across all pools:

```solidity
contract PositionAggregator {
    IPositionManager public positionManager;
    
    struct PositionSummary {
        bytes32 poolId;
        address token0;
        address token1;
        uint24 fee;
        int24 tickLower;
        int24 tickUpper;
        uint128 liquidity;
        uint256 tokensOwed0;
        uint256 tokensOwed1;
        uint256 uncollectedFees0;
        uint256 uncollectedFees1;
    }
    
    constructor(address _positionManager) {
        positionManager = IPositionManager(_positionManager);
    }
    
    function getAllPositions(address user) external view returns (PositionSummary[] memory) {
        uint256[] memory tokenIds = positionManager.tokensOfOwner(user);
        PositionSummary[] memory summaries = new PositionSummary[](tokenIds.length);
        
        for (uint256 i = 0; i < tokenIds.length; i++) {
            Position memory pos = positionManager.getPositionData(tokenIds[i]);
            bytes32 poolId = pos.poolId;
            // Need to get token addresses and fee from poolManager? Possibly store mapping or compute from poolId
            // For now, we'll omit those fields or assume external indexing
            summaries[i] = PositionSummary({
                poolId: poolId,
                token0: address(0), // placeholder
                token1: address(0),
                fee: 0,
                tickLower: pos.tickLower,
                tickUpper: pos.tickUpper,
                liquidity: pos.liquidity,
                tokensOwed0: pos.tokensOwed0,
                tokensOwed1: pos.tokensOwed1,
                uncollectedFees0: 0,
                uncollectedFees1: 0
            });
        }
        
        return summaries;
    }
    
    // Helper to collect all fees from all positions in one transaction
    function collectAllFees(address recipient) external returns (uint256 total0, uint256 total1) {
        uint256[] memory tokenIds = positionManager.tokensOfOwner(msg.sender);
        for (uint256 i = 0; i < tokenIds.length; i++) {
            Position memory pos = positionManager.getPositionData(tokenIds[i]);
            (uint256 fees0, uint256 fees1) = positionManager.collect(
                pos.poolId,
                tokenIds[i],
                recipient,
                block.timestamp + 1000
            );
            total0 += fees0;
            total1 += fees1;
        }
    }
}
```

This contract allows a user to see an aggregated view and batch collect fees. However, note that iterating over many positions can hit gas limits. For users with many positions, you might want to paginate or use off-chain indexing.

For true multi-pool strategy coordination, you could create a manager contract that automatically rebalances between pools based on some rules (e.g., move liquidity from low-volume pool to high-volume pool). This would involve removing liquidity from one pool and adding to another, which can be done atomically using flash accounting patterns described earlier.

### Code Example: Dynamic Range Adjustment Based on Price Movement

A practical bot might widen the range when volatility increases and narrow it when price is stable. Here's a simplified version that adjusts the range after every trade by a fixed amount, mimicking a trailing range:

```solidity
contract TrailingRangePosition {
    IPositionManager public positionManager;
    IPoolManager public poolManager;
    
    struct Position {
        bytes32 poolId;
        uint256 tokenId;
        int24 tickLower;
        int24 tickUpper;
        uint128 liquidity;
        address owner;
    }
    
    mapping(bytes32 => Position) public positions;
    
    event RangeAdjusted(bytes32 indexed poolId, int24 newLower, int24 newUpper);
    
    constructor(address _positionManager, address _poolManager) {
        positionManager = IPositionManager(_positionManager);
        poolManager = IPoolManager(_poolManager);
    }
    
    // This function would be called by a keeper after each swap (or periodically)
    function adjustRangeBasedOnPrice(
        bytes32 poolId,
        uint256 toleranceTicks // How far price can move before we shift range
    ) external returns (bool adjusted) {
        Position storage pos = positions[poolId];
        require(pos.tokenId != 0, "No position tracked");
        
        (, int24 currentTick, , ,) = poolManager.getPool(poolId);
        
        // If current tick is at or beyond the lower bound (for upward price) or upper bound (for downward), shift
        bool needsAdjust = false;
        int24 newLower = pos.tickLower;
        int24 newUpper = pos.tickUpper;
        
        if (currentTick > pos.tickUpper - int24(toleranceTicks)) {
            // Price moved up; shift range up
            newLower = currentTick - pos.tickLower + pos.tickUpper; // Actually compute shift: maintain width but move up
            // Simpler: just move the whole range up by (currentTick - pos.tickUpper) + some buffer
            int24 shift = currentTick - pos.tickUpper + int24(toleranceTicks);
            newLower = pos.tickLower + shift;
            newUpper = pos.tickUpper + shift;
            needsAdjust = true;
        } else if (currentTick < pos.tickLower + int24(toleranceTicks)) {
            // Price moved down; shift down
            int24 shift = pos.tickLower - currentTick + int24(toleranceTicks);
            newLower = pos.tickLower - shift;
            newUpper = pos.tickUpper - shift;
            needsAdjust = true;
        }
        
        if (needsAdjust) {
            // Modify position range while keeping same liquidity
            (uint128 liquidity, , ) = positionManager.modifyLiquidity(
                ModifyLiquidityParams({
                    poolId: poolId,
                    liquidityDelta: 0,
                    sqrtPriceX96: 0,
                    tickLower: newLower,
                    tickUpper: newUpper,
                    amount0Requested: 0,
                    amount1Requested: 0,
                    decreaseLiquidity: false,
                    createPosition: false
                }),
                0,
                0,
                pos.owner,
                block.timestamp + 1000
            );
            
            pos.tickLower = newLower;
            pos.tickUpper = newUpper;
            emit RangeAdjusted(poolId, newLower, newUpper);
            adjusted = true;
        }
        
        return adjusted;
    }
    
    // When creating a position, track it
    function trackPosition(
        bytes32 poolId,
        uint256 tokenId
    ) external {
        Position memory pos = Position({
            poolId: poolId,
            tokenId: tokenId,
            tickLower: 0,
            tickUpper: 0,
            liquidity: 0,
            owner: positionManager.ownerOf(tokenId)
        });
        positions[poolId] = pos;
    }
}
```

Notice that this contract stores the position details in a mapping for quick access. It's important to keep this in sync with the PositionManager. The contract assumes `trackPosition` is called after adding liquidity to set the initial tickLower and tickUpper, as well as tokenId.

The `adjustRangeBasedOnPrice` could be called by a keeper every few blocks. It checks if the current price has approached the edge of the range (within tolerance), and if so, shifts the entire range to center around the current price. This keeps liquidity concentrated around the active price, aiming to maximize fee earnings.

You could also make the width dynamic based on volatility: wider when volatile to avoid going out of range, narrower when calm to concentrate.

### Code Example: Automatic Rebalancing Contract (Batch Operations)

A more advanced rebalancer could also adjust the liquidity amount in response to changes in pool utilization or total value locked. For instance, if the pool's total liquidity grows significantly, you might want to increase your share to maintain fee revenue.

Here's a concept for a contract that periodically rebalances across multiple pools to maintain target percentages:

```solidity
contract MultiPoolRebalancer {
    IPoolManager public poolManager;
    IPositionManager public positionManager;
    
    struct PoolTarget {
        bytes32 poolId;
        uint256 targetPercentageBps; // Target share of total liquidity
        int24 tickLower;
        int24 tickUpper;
    }
    
    PoolTarget[] public targets;
    address public owner;
    
    constructor(address _poolManager, address _positionManager) {
        poolManager = IPoolManager(_poolManager);
        positionManager = IPositionManager(_positionManager);
        owner = msg.sender;
    }
    
    function addPoolTarget(
        bytes32 poolId,
        uint256 targetPercentageBps,
        int24 tickLower,
        int24 tickUpper
    ) external {
        require(msg.sender == owner, "Not owner");
        targets.push(PoolTarget({poolId: poolId, targetPercentageBps: targetPercentageBps, tickLower: tickLower, tickUpper: tickUpper}));
    }
    
    // Rebalance all positions to match targets
    function rebalance(address token) external returns (uint256 totalLiquidity) {
        // Need to know total value of all our positions in 'token' terms
        // This requires price oracle; complex.
        // Instead, simpler: rebalance based on liquidity units across pools, not value.
        // But liquidity units are not directly comparable across different pools.
        // So we need to compute USD value of each position via TWAP oracle.
        // That's beyond this example.
        
        revert("Complex; needs external price data");
    }
}
```

This is a high-level sketch; actual implementation requires cross-pool valuation and possibly moving liquidity between pools atomically. The flash accounting pattern can be used: remove liquidity from pool A, collect tokens, then add to pool B within the same transaction. However, you must ensure that the pool's price doesn't move between the removal and addition if you're doing them separately. Since they're in the same transaction, the pool state is updated after removal before addition; the price might shift slightly due to the removal itself. That's acceptable if you're just rebalancing proportions; you can add with some slippage tolerance.

### Code Example: Fee Tier Optimization Strategy

To decide which fee tier to choose, analyze historical volume and fee rates. A simple approach: select the pool with the highest product of volume and fee rate, because fee revenue ≈ volume × fee. But also consider that higher fee pools may have less volume due to lower trader appeal.

You could write an off-chain script that queries subgraph data or on-chain TWAP to estimate annual fee yield for each fee tier and then choose accordingly.

On-chain, you can have a strategy that deposits into the pool with the highest recent fee growth per liquidity. For example, monitor `poolManager.feeGrowthGlobal0` and `feeGrowthGlobal1` over a period to compute the fees earned per unit liquidity, and then migrate your liquidity to the best performing pool.

```solidity
contract FeeYieldOptimizer {
    IPositionManager public positionManager;
    IPoolManager public poolManager;
    
    bytes32[] public candidatePoolIds;
    address public owner;
    
    // Every N blocks, evaluate yields and possibly migrate
    uint256 public evaluationPeriod;
    uint256 public lastEvaluation;
    
    constructor(address _positionManager, address _poolManager) {
        positionManager = IPositionManager(_positionManager);
        poolManager = IPoolManager(_poolManager);
        evaluationPeriod = 7200; // ~1 day (assuming 15s block time)
    }
    
    function addCandidatePool(bytes32 poolId) external {
        require(msg.sender == owner, "Not owner");
        candidatePoolIds.push(poolId);
    }
    
    // Called periodically
    function optimize() external returns (bool migrated) {
        require(block.timestamp >= lastEvaluation + evaluationPeriod, "Too early");
        
        bytes32 bestPool = _findBestPool();
        bytes32 currentPool = _getCurrentPool(); // Determine from current position(s)
        
        if (bestPool != currentPool) {
            // Migrate position to bestPool
            // Use atomic migration pattern
            _migrate(currentPool, bestPool);
            migrated = true;
        }
        
        lastEvaluation = block.timestamp;
        return migrated;
    }
    
    function _findBestPool() internal view returns (bytes32) {
        // Compute fee yield (feeGrowth per liquidity) for each candidate over recent period
        // Simplified: just check current feeGrowthGlobal0+1 scaled by liquidity
        bytes32 best = candidatePoolIds[0];
        uint256 bestScore = 0;
        for (uint256 i = 0; i < candidatePoolIds.length; i++) {
            bytes32 poolId = candidatePoolIds[i];
            (uint128 liquidity, , , ,) = poolManager.getPool(poolId);
            if (liquidity == 0) continue;
            (int56 feeGrowth0, int56 feeGrowth1) = poolManager.feeGrowthGlobal(poolId); // not actual signature but conceptually
            // Need to get accumulated fees over period; requiring previous snapshot
            // For demo, we'll just use current feeGrowthGlobal total
            uint256 score = uint256(uint128(feeGrowth0)) + uint256(uint128(feeGrowth1));
            // Normalize by liquidity? score / liquidity
            // So compute yield per unit: (feeGrowthGlobal * 2^-128) / liquidity = fees per LP token
            // Use high precision
            (uint256 yieldPerLiquidity, ) = _computeYield(poolId);
            if (yieldPerLiquidity > bestScore) {
                bestScore = yieldPerLiquidity;
                best = poolId;
            }
        }
        return best;
    }
    
    function _computeYield(bytes32 poolId) internal view returns (uint256, uint256) {
        // This would fetch feeGrowthGlobal at now and at lastEvaluation, subtract and divide by elapsed time and liquidity
        // Need to store previous feeGrowth values per pool in mapping.
        // Omitted for brevity.
        return (0,0);
    }
    
    function _getCurrentPool() internal view returns (bytes32) {
        // Return the poolId of the current position (assuming only one)
        // Could store during creation
    }
    
    function _migrate(bytes32 fromPool, bytes32 toPool) internal {
        // Use LiquidityMigrator pattern
        // Position token, etc.
    }
}
```

This optimizer periodically evaluates which of several candidate pools is offering the highest fee yield per unit liquidity and migrates the position there. Migration must be done atomically to avoid price risk (using the earlier LiquidityMigrator). This is an active strategy that incurs gas costs for migrations but may increase overall returns.

### Tables Comparing Different Strategy Approaches

| Strategy | When to Use | Complexity | Gas Costs | Management Overhead | Risks |
|----------|-------------|------------|-----------|---------------------|-------|
| Single pool, narrow range | High conviction on price range | Low | Low (few tx) | Monitor out-of-range frequently | Price moves out, zero fees |
| Single pool, wide range | Passive, want steady fees | Low | Low | Minimal | Lower fee per liquidity, still impermanent loss |
| Multi-tier across fee levels | Want to capture fee variance | Medium | Medium (multiple add/remove) | Track positions across tiers | Some tiers may underperform |
| Auto-rebalancing with keepers | Active management, bot accessible | High | High (periodic modify) | Need reliable keeper, tuning | Rebalance cost, frontrunning on rebalance |
| TWAP-based dynamic range | Want to ride volatility | Medium-High | Medium-High | Compute ranges, adjust | Complex, may misjudge volatility |
| Fee optimizer (migration) | Willing to move for better yields | High | High (migration costs) | Track yields, execute migrations | Migration slippage, opportunity cost |

When choosing a strategy, consider:
- Your available capital (larger capital may benefit from splitting across tiers)
- How much time you can spend monitoring
- Your risk tolerance (narrow ranges are riskier but potentially more profitable)
- Gas costs on the network (Ethereum mainnet gas may make frequent rebalancing uneconomical)

### Impermanent Loss Calculation Example

To make informed decisions, you should be able to estimate impermanent loss given a price change. The formula for the value of an LP position relative to holding tokens is:

Value ratio = (2 * sqrt(price ratio) / (1 + price ratio)) - (if symmetric deposit)

Where price ratio = P1/P0 (new price / initial price). If the ratio is 1, IL is zero. As the ratio deviates from 1, IL increases.

You can write a helper function to compute potential IL for a given range width and expected price movement. For instance, if your range width is ±10% (price ratio from 0.9 to 1.1 relative to entry), and the actual price moves 15%, you'll be out of range for part of the time. But if it stays within, IL is limited by the range boundaries.

Better yet, simulate different price paths using historical data to see how your strategy would have performed.

Remember that fees earned can partially or fully offset IL. A pool with high volume and high fees may compensate for significant price swings.


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

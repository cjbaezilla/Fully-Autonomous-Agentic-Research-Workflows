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

## Conclusion and Further Resources

Uniswap V4 provides a powerful foundation for building decentralized trading applications with unparalleled flexibility. The hook system alone opens possibilities for limit orders, custom AMM curves, dynamic fees, and on-chain order books—all within the same liquidity network.

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

# Hands-On Uniswap V4: A Developer's Guide to Liquidity and Swaps with Solidity

Uniswap V4 represents a significant evolution in decentralized exchange architecture, building upon the success of V3 while introducing powerful new features that give developers unprecedented control over liquidity and swap behavior. This guide will walk you through the core concepts, setup, and practical implementation details needed to build with Uniswap V4 using Solidity.

## Introduction to Uniswap V4

Uniswap V4 introduces a radical redesign centered around a single, powerful contract called the PoolManager. Unlike previous versions where pools were separate contracts with their own logic, V4 consolidates all pool functionality into one contract while enabling customization through hooks. This architecture reduces deployment costs significantly and opens up new possibilities for on-chain trading strategies.

The most groundbreaking addition is the hooks system: smart contracts that execute custom logic at precise points during liquidity provision and swaps. You can create hooks that implement limit orders, dynamic fees, custom price oracles, or any other behavior you can imagine, all integrated seamlessly into the core exchange flow.

Other major improvements include native ETH support (no more wrapping), flash accounting that nets obligations to minimize external transfers, and a new token standard ERC-6909 that replaces ERC-20 for more efficient multi-token management. The singleton PoolManager architecture means all pools live under one address, simplifying pool discovery and interaction.

## Prerequisites

Before diving into Uniswap V4 development, you should have a solid understanding of Solidity fundamentals including contract structure, functions, modifiers, and error handling. Familiarity with automated market makers (AMMs) and liquidity provider concepts from Uniswap V2 or V3 will be helpful, though not strictly required as we'll cover enough background.

You should have Node.js and npm installed for package management. For testing and deployment, you'll need either Foundry or Hardhat set up in your development environment. This guide will primarily use Solidity code examples, so you should be comfortable writing and understanding Solidity contracts.

## Core Concepts

### Singleton PoolManager Architecture

Uniswap V4 consolidates all pool logic into a single PoolManager contract. Instead of deploying separate contract instances for each pool (as in V2 and V3), all pools now exist as state within the PoolManager. This means that to interact with any pool, you call functions on the PoolManager address. The PoolManager stores pool configurations, handles swaps, creates positions, and executes hook callbacks.

When you want to create a new pool, you don't deploy a new contract. Instead, you initialize a new pool within the PoolManager by calling `initialize` with a PoolKey that identifies the token pair, fee tier, and tick spacing. The PoolManager creates the necessary internal data structures for that pool and assigns it a unique pool ID that can be used in subsequent calls.

### Hooks System

Hooks are contracts that implement custom logic at specific points in the pool lifecycle. They're optional and can be attached to pools during creation. When you create a pool in V4, you specify hook addresses for various callback points: before/after initialization, before/after swap, before/after add liquidity, before/after remove liquidity, before/after donate (for fee collection), and before/after transfer LP tokens.

Hooks must implement predetermined function signatures that the PoolManager will call at the appropriate times. These callbacks allow you to add conditions, custom accounting, off-chain price oracles, or any other business logic. The hook system makes Uniswap V4 highly modular: you can write complex trading strategies, limit order implementations, or token bonding curves without forking the core protocol.

### PoolKey Struct and Pool Identifiers

Every pool in V4 is uniquely identified by a PoolKey struct containing:
- `currency0` and `currency1`: The two token addresses (sorted by their token ID)
- `fee`: The fee tier in hundredths of a basis point (e.g., 3000 = 0.3%)
- `tickSpacing`: The tick spacing for that pool (derived from fee tier)

To reference a pool in function calls, you typically pass this PoolKey struct. The PoolManager uses the hash of this struct as the internal pool identifier. When you initialize a pool, the PoolManager returns a `poolId` which is a bytes32 value. This poolId is used in subsequent calls to reference the specific pool.

### Currency Types and ERC-6909

V4 introduces a flexible currency abstraction. Currencies can be ERC-20 tokens, native ETH (represented as a special address), or any asset that implements the IETH adapter interface. The system uses token IDs rather than just addresses to distinguish between different representations of the same asset.

ERC-6909 is a new multi-token standard that replaces ERC-20 in V4 for efficiency. It allows multiple independent token instances to be managed by a single contract, reducing deployment overhead. Many V4 contracts use ERC-6909 instead of traditional ERC-20, though the interface is similar.

### Flash Accounting and Netting

One of V4's most powerful efficiency features is flash accounting. Instead of immediately transferring tokens in and out of the PoolManager during swaps and liquidity operations, V4 tracks net obligations and only settles the net difference at the end of a transaction. This reduces the number of external token transfers, saving gas and enabling more complex operations within a single transaction.

For example, during a swap, the PoolManager records how much of token0 it owes the user and how much it receives, but doesn't send tokens until the very end. If you perform multiple swaps in one transaction, the net amounts are calculated and only the final imbalance is settled. This also enables "delta accounting" where liquidity providers can collect earned fees without withdrawing liquidity.

### Delta Accounting

Delta accounting refers to the way V4 tracks changes in liquidity and token balances. The `Delta` struct records positive or negative changes in token amounts. When you add liquidity, you specify the amount of tokens you're depositing and the position you're creating. The PoolManager updates the global liquidity and your position while recording the delta. When you remove liquidity or collect fees, deltas are settled against your balance in the PositionManager.

## Setting Up the Development Environment

### Installing Dependencies

For a Foundry project, add Uniswap V4 dependencies to your `foundry.toml`:

```toml
[dependencies]
uniswap-v4-core = { git = "https://github.com/uniswap/v4-core", rev = "main" }
uniswap-v4-periphery = { git = "https://github.com/uniswap/v4-periphery", rev = "main" }
solidity = "0.8.20"
```

For Hardhat, install the packages:

```bash
npm install @uniswap/v4-core @uniswap/v4-periphery
npm install --save-dev @openzeppelin/contracts
```

### Project Structure

A typical Uniswap V4 project structure might look like:

```
contracts/
├── interfaces/
│   ├── IUniswapV4PoolManager.sol
│   ├── IUniswapV4PositionManager.sol
│   └── ISwapRouter.sol
├── MyPoolHook.sol
├── MyTradingContract.sol
└── HelperContracts.sol
test/
  ├── pool.test.sol
  ├── swap.test.sol
  └── hook.test.sol
```

When writing contracts, you'll import from the Uniswap V4 packages:

```solidity
import {IPoolManager} from "@uniswap/v4-core/contracts/interfaces/IPoolManager.sol";
import {IPositionManager} from "@uniswap/v4-periphery/contracts/interfaces/IPositionManager.sol";
import {ISwapRouter} from "@uniswap/v4-periphery/contracts/interfaces/ISwapRouter.sol";
```

## Creating a Pool

Creating a pool in Uniswap V4 requires initializing it with the PoolManager. Here's a complete example that shows how to define the PoolKey and call initialize:

```solidity
contract PoolCreator {
    IPoolManager public poolManager;
    
    struct PoolKey {
        address currency0;
        address currency1;
        uint24 fee;
        int24 tickSpacing;
    }
    
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
        PoolKey memory key = PoolKey({
            currency0: token0,
            currency1: token1,
            fee: fee,
            tickSpacing: tickSpacing
        });
        
        poolId = IPoolManager.bytes32Key(
            keccak256(abi.encode(key))
        );
        
        poolManager.initialize(
            key,
            initialTick,
            msg.sender
        );
    }
}
```

Important notes: The tokens must be sorted so that token0 < token1 when compared numerically. The initial tick determines the starting price; it should be chosen based on the desired initial sqrtPriceX96. The `msg.sender` becomes the initial owner of the pool (though the PoolManager acts as the administrator). After initialization, the pool exists and can accept liquidity and swaps.

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
        PoolKey memory poolKey = PoolKey({
            currency0: params.token0,
            currency1: params.token1,
            fee: params.fee,
            tickSpacing: _getTickSpacing(params.fee)
        });
        
        bytes32 poolId = IPoolManager.bytes32Key(
            keccak256(abi.encode(poolKey))
        );
        
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
        
        (liquidity, amount0, amount1) = positionManager.modifyLiquidity(
            modifyParams,
            params.amount0Min,
            params.amount1Min,
            params.recipient,
            params.deadline
        );
    }
    
    function _getTickSpacing(uint24 fee) internal pure returns (int24) {
        return fee >= 500 ? 10 : 1;
    }
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
        modifyParams = ModifyLiquidityParams({
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
        
        (amount0, amount1) = positionManager.modifyLiquidity(
            modifyParams,
            params.amount0Min,
            params.amount1Min,
            params.recipient,
            params.deadline
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
    }
}
```

For removal, you specify the poolId and tokenId of the position you want to decrease. The `liquidity` parameter is the amount of liquidity to remove. The PositionManager will burn that portion of your position and return the corresponding tokens (plus any collected fees if you call collect separately). The amount0Min and amount1Min enforce slippage limits on the output amounts.

## Performing Swaps

Uniswap V4 provides two primary ways to swap: the SwapRouter (which handles token transfers and approval management) or directly calling the PoolManager (which gives you more control). Here's a swap using the SwapRouter:

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
        SingleSwap memory singleSwap = SingleSwap({
            poolId: _getPoolId(
                params.tokenIn,
                params.tokenOut,
                params.fee
            ),
            kind: params.zeroForOne ? 0 : 1,
            amountSpecified: params.amountIn,
            sqrtPriceLimitX96: params.sqrtPriceLimitX96
        });
        
        FundManagement memory funds = FundManagement({
            sender: msg.sender,
            from: params.tokenIn == address(0) ? msg.sender : params.tokenIn,
            to: params.recipient,
            wNETH: address(0), // For ETH wrapping, set to WETH9 address
            funding: _funding()
        });
        
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
        SingleSwap memory singleSwap = SingleSwap({
            poolId: _getPoolId(
                params.tokenIn,
                params.tokenOut,
                params.fee
            ),
            kind: params.zeroForOne ? 0 : 1,
            amountSpecified: -int256(params.amountOutMin),
            sqrtPriceLimitX96: params.sqrtPriceLimitX96
        });
        
        FundManagement memory funds = FundManagement({
            sender: msg.sender,
            from: params.tokenIn == address(0) ? msg.sender : params.tokenIn,
            to: params.recipient,
            wNETH: address(0),
            funding: _funding()
        });
        
        amountIn = swapRouter.exactOutputSingle(
            singleSwap,
            funds,
            params.amountIn,
            params.deadline
        );
    }
    
    function _getPoolId(
        address token0,
        address token1,
        uint24 fee
    ) internal view returns (bytes32) {
        PoolKey memory key = PoolKey({
            currency0: token0 < token1 ? token0 : token1,
            currency1: token0 < token1 ? token1 : token0,
            fee: fee,
            tickSpacing: _getTickSpacing(fee)
        });
        return IPoolManager.bytes32Key(
            keccak256(abi.encode(key))
        );
    }
    
    function _funding() internal pure returns (uint8) {
        return 1; // 0 = from, 1 = to (handle token transfers)
    }
}
```

The SwapRouter handles the complexity of token approvals and transfers. For exact input swaps, you specify the amount you want to send and receive at least amountOutMin. For exact output swaps, you specify the amount you need to receive and the router calculates the required input (or reverts if too much would be needed). The `sqrtPriceLimitX96` parameter lets you set price limits; setting it to 0 allows any price. The `zeroForOne` flag determines direction: if tokenIn < tokenOut, it's zeroForOne; otherwise it's oneForZero.

If you need more control (such as multi-hop swaps), you can use `exactInput` or `exactOutput` with an array of SingleSwap structs.

## Working with Hooks

Hooks let you inject custom logic at specific execution points. Let's create a simple example hook that tracks swap volume in a separate mapping:

```solidity
contract VolumeHook {
    IPoolManager public poolManager;
    
    mapping(bytes32 => uint256) public swapVolume;
    mapping(bytes32 => uint256) public lastUpdate;
    
    address public owner;
    
    constructor(address _poolManager) {
        poolManager = IPoolManager(_poolManager);
        owner = msg.sender;
    }
    
    function beforeSwap(
        bytes32 poolId,
        IPoolManager.SwapRequest memory request
    ) external {
        require(msg.sender == address(poolManager), "Unauthorized");
        
        lastUpdate[poolId] = block.timestamp;
    }
    
    function afterSwap(
        bytes32 poolId,
        IPoolManager.SwapRequest memory request,
        int256 amount0Delta,
        int256 amount1Delta
    ) external {
        require(msg.sender == address(poolManager), "Unauthorized");
        
        uint256 absoluteAmount = uint256(
            amount0Delta > amount1Delta ? amount0Delta : amount1Delta
        );
        swapVolume[poolId] += absoluteAmount;
    }
    
    function getVolume(bytes32 poolId) external view returns (uint256) {
        return swapVolume[poolId];
    }
}
```

To use this hook, you would pass its address when creating a pool. The hook functions are called automatically during swap operations. The hook must be careful about gas usage and reentrancy. Check the IHook interface in the Uniswap V4 contracts for the complete set of callbacks you can implement.

## Best Practices and Security Considerations

### Gas Optimization

Uniswap V4 is already more gas-efficient than V3 due to the singleton architecture, but you can still optimize. When adding liquidity, consider using exact token amounts rather than desired amounts where possible, as the router will handle the optimal distribution. Use the lowest tick spacing that meets your needs; tighter spacing increases liquidity precision but costs more gas per position.

Avoid unnecessary token approvals by using the `permit` function where supported. For multiple operations in the same transaction, use the PositionManager's ability to modify liquidity multiple times rather than separate calls.

### Slippage and Deadlines

Always set reasonable slippage tolerance and transaction deadlines. In high volatility environments, even 0.1% slippage might not be enough. Use `block.time` or `block.number` as deadlines rather than assuming a fixed number of seconds. For large swaps, consider breaking them into smaller chunks to minimize price impact.

### Reentrancy Protection

While the PoolManager uses a nonReentrant modifier on critical functions, if you're building custom hooks or contracts that interact with V4 in complex ways, you should implement your own reentrancy guards using OpenZeppelin's ReentrancyGuard or similar patterns. Never assume that callbacks from the PoolManager are reentrancy-safe unless explicitly stated.

### Common Pitfalls

One common mistake is passing the wrong token order to PoolKey. Always ensure token0 < token1 when constructing the key. Another is misunderstanding tick spacing: if you try to create a position at a tick that doesn't align with the pool's tick spacing, it will revert. Use _getTickSpacing(fee) to ensure alignment.

Be careful with isolated fee tiers: you cannot mix liquidity across different fee tiers for the same token pair. Each fee tier creates a separate pool with independent liquidity. Choose your fee tier based on expected volatility: lower fees for stable pairs, higher fees for volatile pairs.

When working with native ETH, remember that the PoolManager uses address(0) to represent ETH. You'll need to handle ETH transfers differently from ERC-20 tokens, using payable functions and address(0) checks.

## Conclusion and Further Resources

Uniswap V4 provides a powerful foundation for building decentralized trading applications with unparalleled flexibility. The hook system alone opens possibilities for limit orders, custom AMM curves, dynamic fees, and on-chain order books—all within the same liquidity network.

To deepen your expertise, explore the official Uniswap V4 documentation at https://docs.uniswap.org/ and review the extensive test suite in the v4-core and v4-periphery repositories. The Uniswap V4 spec provides detailed technical explanations of every function and parameter.

Start by building simple liquidity provision contracts, then gradually add hooks and custom swap logic. The provided examples in this guide are production-ready patterns that you can adapt to your specific use cases. Remember to thoroughly test on testnets before deploying significant value, and consider auditing your hook implementations if they handle substantial funds.

The Uniswap community is active and helpful; join the developer Discord channels for real-time support and to learn from others building on V4. As the ecosystem grows, best practices will evolve, so stay updated with the latest changes in the core contracts.
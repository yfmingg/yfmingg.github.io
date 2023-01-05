# Uniswap概览

官网：<https://uniswap.org/>

Uniswap是用来交易ERC20代币的。Uniswap有3个主要功能:

- 在不同的代币之间进行兑换
- 添加代币对流动性，获得LP ERC-20流动性代币
- 销毁 LP ERC-20流动性代币，取回配对的ERC-20代币

## Uniswap简单实现

```solidity
//SPDX-License-Identifier: Unlicense
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/utils/math/Math.sol";
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";

contract LooneySwapPool is ERC20 {
  address public token0;
  address public token1;

  // Reserve of token 0
  uint public reserve0;

  // Reserve of token 1
  uint public reserve1;

  uint public constant INITIAL_SUPPLY = 10**5;

  constructor(address _token0, address _token1) ERC20("LooneyLiquidityProvider", "LP") {
    token0 = _token0;
    token1 = _token1;
  }

  /**
   * Adds liquidity to the pool.
   * 1. Transfer tokens to pool
   * 2. Emit LP tokens
   * 3. Update reserves
   */
  function add(uint amount0, uint amount1) public {
    assert(IERC20(token0).transferFrom(msg.sender, address(this), amount0));
    assert(IERC20(token1).transferFrom(msg.sender, address(this), amount1));

    uint reserve0After = reserve0 + amount0;
    uint reserve1After = reserve1 + amount1;

    if (reserve0 == 0 && reserve1 == 0) {
      _mint(msg.sender, INITIAL_SUPPLY);
    } else {
      uint currentSupply = totalSupply();
      uint newSupplyGivenReserve0Ratio = reserve0After * currentSupply / reserve0; // x <==  (reserve0After / reserve0) = x / currentSupply
      uint newSupplyGivenReserve1Ratio = reserve1After * currentSupply / reserve1;
      uint newSupply = Math.min(newSupplyGivenReserve0Ratio, newSupplyGivenReserve1Ratio);
      _mint(msg.sender, newSupply - currentSupply);
    }

    reserve0 = reserve0After;
    reserve1 = reserve1After;
  }

  /**
   * Removes liquidity from the pool.
   * 1. Transfer LP tokens to pool
   * 2. Burn the LP tokens
   * 3. Update reserves
   */
  function remove(uint liquidity) public {
    assert(transfer(address(this), liquidity));

    uint currentSupply = totalSupply();
    uint amount0 = liquidity * reserve0 / currentSupply; // （lp / currentSupply) * reserve0
    uint amount1 = liquidity * reserve1 / currentSupply;

    _burn(address(this), liquidity);

    assert(IERC20(token0).transfer(msg.sender, amount0));
    assert(IERC20(token1).transfer(msg.sender, amount1));
    reserve0 = reserve0 - amount0;
    reserve1 = reserve1 - amount1;
  }

  /**
   * Uses x * y = k formula to calculate output amount.
   * 1. Calculate new reserve on both sides
   * 2. Derive output amount
   * 
   * fromToken 用来交换的token，即要花掉的token
   * amountIn  被换token的数量，即想得到的pair中另一个token的数量
   */
  function getAmountOut (uint amountIn, address fromToken) public view returns (uint amountOut, uint _reserve0, uint _reserve1) {
    uint newReserve0;
    uint newReserve1;
    uint k = reserve0 * reserve1; 

    // x (reserve0) * y (reserve1) = k (constant)
    // (reserve0 + amountIn) * (reserve1 - amountOut) = k
    // (reserve1 - amountOut) = k / (reserve0 + amount)
    // newReserve1 = k / (newReserve0)
    // amountOut = newReserve1 - reserve1

    if (fromToken == token0) { 
      newReserve0 = amountIn + reserve0;
      newReserve1 = k / newReserve0;
      amountOut = reserve1 - newReserve1;
    } else {
      newReserve1 = amountIn + reserve1;
      newReserve0 = k / newReserve1;
      amountOut = reserve0 - newReserve0;
    }

    _reserve0 = newReserve0;
    _reserve1 = newReserve1;
  }

  /**
   * Swap to a minimum of `minAmountOut`
   * 1. Calculate new reserve on both sides
   * 2. Derive output amount
   * 3. Check output against minimum requested
   * 4. Update reserves
   */
  function swap(uint amountIn, uint minAmountOut, address fromToken, address toToken, address to) public {
    require(amountIn > 0 && minAmountOut > 0, 'Amount invalid');
    require(fromToken == token0 || fromToken == token1, 'From token invalid');
    require(toToken == token0 || toToken == token1, 'To token invalid');
    require(fromToken != toToken, 'From and to tokens should not match');

    (uint amountOut, uint newReserve0, uint newReserve1) = getAmountOut(amountIn, fromToken);

    require(amountOut >= minAmountOut, 'Slipped... on a banana');

    assert(IERC20(fromToken).transferFrom(msg.sender, address(this), amountIn));
    assert(IERC20(toToken).transfer(to, amountOut));

    reserve0 = newReserve0;
    reserve1 = newReserve1;
  }
}
```

### 对接 Uniswap V2 兑换代币

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8;

import "./interfaces/IERC20.sol";
import "./interfaces/Uniswap.sol";

contract testSwap {
    uint256 deadline;
    //address of the uniswap v2 router
    //https://docs.uniswap.org/contracts/v2/reference/smart-contracts/router-02
    address private constant UNISWAP_V2_ROUTER =
        0x7a250d5630B4cF539739dF2C5dAcb4c659F2488D;


    // swap function
    /**
    _tokenIn： 是我们要兑换的代币的地址。
    _tokenOut：是我们想从这次交易中获得的代币的地址。
    _amountIn： 是我们要交易的代币的数量。
    _to：交易兑换出的代币发送到这个地址。
    _deadline：是交易应该被执行的时间期限。如果超过了最后期限，交易就会失败。
     */
    function swap(
        address _tokenIn,
        address _tokenOut,
        uint256 _amountIn,
        address _to,
        uint256 _deadline
      
    ) external {
        // transfer the amount in tokens from msg.sender to this contract
        IERC20(_tokenIn).transferFrom(msg.sender, address(this), _amountIn);

        //by calling IERC20 approve you allow the uniswap contract to spend the tokens in this contract
        IERC20(_tokenIn).approve(UNISWAP_V2_ROUTER, _amountIn);

        address[] memory path;
        path = new address[](2);
        path[0] = _tokenIn; 
        path[1] = _tokenOut; 

        uint256[] memory amountsExpected = IUniswapV2Router(UNISWAP_V2_ROUTER).getAmountsOut(
            _amountIn,
            path
        );
     
        IUniswapV2Router(UNISWAP_V2_ROUTER).swapExactTokensForTokens(
            amountsExpected[0],
            (amountsExpected[1]*990)/1000, // accpeting a slippage of 1%
            path,
            _to,
            _deadline
        );
    }
}
```

### Uniswap上执行闪电兑换(Flash Swaps)

与传统贷款不同，在闪电贷中，在一个交易里完成资金借入和归还。这一点是必须的。

在Defi里，交易者（通常通过机器人）不断寻找套利机会，通过在为同一资产提供不同价格的平台之间进行交易来获得利益。这就是闪电贷出现的地方（通常是）。

在闪电贷的帮助下，交易者可以借到一大笔钱来执行套利交易。闪电贷和闪电兑换其实是一回事。

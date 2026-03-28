# -Simple-AMM-Liquidity-Pool-Constant-Product-
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract SimpleAMM is Ownable {
    IERC20 public tokenA;
    IERC20 public tokenB;
    uint256 public reserveA;
    uint256 public reserveB;

    event LiquidityAdded(uint256 amountA, uint256 amountB);
    event Swap(address indexed user, address tokenIn, uint256 amountIn, uint256 amountOut);

    constructor(address _tokenA, address _tokenB, address initialOwner) Ownable(initialOwner) {
        tokenA = IERC20(_tokenA);
        tokenB = IERC20(_tokenB);
    }

    function addLiquidity(uint256 amountA, uint256 amountB) external onlyOwner {
        tokenA.transferFrom(msg.sender, address(this), amountA);
        tokenB.transferFrom(msg.sender, address(this), amountB);
        reserveA += amountA;
        reserveB += amountB;
        emit LiquidityAdded(amountA, amountB);
    }

    function swap(address tokenIn, uint256 amountIn) external returns (uint256 amountOut) {
        require(tokenIn == address(tokenA) || tokenIn == address(tokenB), "Invalid token");

        if (tokenIn == address(tokenA)) {
            amountOut = (amountIn * reserveB) / (reserveA + amountIn);
            tokenA.transferFrom(msg.sender, address(this), amountIn);
            tokenB.transfer(msg.sender, amountOut);
            reserveA += amountIn;
            reserveB -= amountOut;
        } else {
            amountOut = (amountIn * reserveA) / (reserveB + amountIn);
            tokenB.transferFrom(msg.sender, address(this), amountIn);
            tokenA.transfer(msg.sender, amountOut);
            reserveB += amountIn;
            reserveA -= amountOut;
        }
        emit Swap(msg.sender, tokenIn, amountIn, amountOut);
    }
}

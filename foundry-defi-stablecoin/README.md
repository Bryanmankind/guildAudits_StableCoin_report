## Findings 

## [M - 1] - **Precission Loss in _calculateHealthFact Method :**

## Severity.
Impact : Medium,  sequence can lead to precision loss due to integer division truncation.
Likelihood: Medium, loss allways happen when calculating the health factor of users. 

## Description
The _calculateHealthFactor function in the given code calculates the health factor by adjusting the collateral value for the liquidation threshold and then dividing by the total DSC minted. The current implementation performs multiplication by 1e18 in the return statement, after the division operation. This sequence can lead to precision loss due to integer division truncation.

https://github.com/Cyfrin/2023-07-foundry-defi-stablecoin/blob/8db4820ee67210c6c41b7dbe6372c05a652ee73c/src/DSCEngine.sol#L324


```solidity
function _calculateHealthFactor(uint256 totalDscMinted, uint256 collateralValueInUsd)
    internal
    pure
    returns (uint256)
{
    if (totalDscMinted == 0) return type(uint256).max;
    uint256 collateralAdjustedForThreshold = (collateralValueInUsd * LIQUIDATION_THRESHOLD) / LIQUIDATION_PRECISION;
    return (collateralAdjustedForThreshold * 1e18) / totalDscMinted;
}
```

**Proof of Concept (PoC)**
Consider the following example inputs:

totalDscMinted = 500
collateralValueInUsd = 1000
LIQUIDATION_THRESHOLD = 150 (assuming 150%)
LIQUIDATION_PRECISION = 100 (assuming to keep percentages in integer form)
Current implementation:

```solidity
uint256 collateralAdjustedForThreshold = (1000 * 150) / 100; // results in 1500
return (1500 * 1e18) / 500; // results in 3e18 (3 * 10^18)
```

Correct implementation should ensure precision by multiplying before dividing:

```solidity
uint256 collateralAdjustedForThreshold = (collateralValueInUsd * LIQUIDATION_THRESHOLD * 1e18) / LIQUIDATION_PRECISION;
return collateralAdjustedForThreshold / totalDscMinted;
```

Modified implementation:
```solidity
Copy code
uint256 collateralAdjustedForThreshold = (1000 * 150 * 1e18) / 100; // results in 1.5e21
return 1.5e21 / 500; // results in 3e18 (3 * 10^18)
```

## Recommendation 
To prevent precision loss, the multiplication by 1e18 should be performed before any division operation. Here's the corrected code:

```solidity
function _calculateHealthFactor(uint256 totalDscMinted, uint256 collateralValueInUsd)
    internal
    pure
    returns (uint256)
{
    if (totalDscMinted == 0) return type(uint256).max;
    uint256 collateralAdjustedForThreshold = (collateralValueInUsd * LIQUIDATION_THRESHOLD * 1e18) / LIQUIDATION_PRECISION;
    return collateralAdjustedForThreshold / totalDscMinted;
}
```

## [M - 2] -**Stale period of 3 hours is too large for Ethereum *
## Severity.
Impact: Medium, the protocol can consume stale price data.
Likelihood: whenever price data is needed. 

## Description
uint256 private constant TIMEOUT = 3 hours; // 3 * 60 * 60 = 10800 seconds
https://github.com/Cyfrin/2023-07-foundry-defi-stablecoin/blob/8db4820ee67210c6c41b7dbe6372c05a652ee73c/src/libraries/OracleLib.sol#L30
A timeout value of 3 hours is too long, especially in volatile markets where price updates need to be more frequent to maintain accurate pricing and ensure the stability of the protocol. If the data becomes stale within this period, the protocol could operate on outdated prices, leading to potential miscalculations and economic losses.

Given that on Ethereum, the oracle updates the price data approximately every hour [https://data.chain.link/feeds/ethereum/mainnet/eth-usd], the current timeout period of 3 hours may be excessively long. This can lead to scenarios where the protocol operates on outdated prices, increasing the risk of inaccuracies and potential financial discrepancies.

## Tools Used 
Manual Review

## Recommendation 
Adjust the timeout value to be closer to the oracle's update frequency. Since the Chainlink oracle updates approximately every hour, a timeout value of 1 hour (3600 seconds) would be more appropriate.


- **Method mintDsc does not emite tokens :** The function call does not emit the tokens minted to the user, this is important to show the amount of tokens minted.  



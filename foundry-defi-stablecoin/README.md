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

## [C - 0] - **Title :**
[H - 0] - Title
[M - 0] - Title
[L - 0] - Title

## Severity.
Impact 
Likelihood 

## Description
	poc

## Recommendation 


- **Method mintDsc does not emite tokens :** The function call does not emit the tokens minted to the user, this is important to show the amount of tokens minted.  

- **Deposited Funds May Become Stuck if mintDsc Reverts :** If the mintDsc function reverts due to the user's health factor, the funds already deposited in the contract will remain stuck. This issue arises because there is no method in place to refund the user in such cases.

```solidity
function mintDsc(uint256 amountDscToMint) public moreThanZero(amountDscToMint) nonReentrant {
        s_DSCMinted[msg.sender] += amountDscToMint;
        // if they minted too much ($150 DSC, $100 ETH)
        _revertIfHealthFactorIsBroken(msg.sender);
        bool minted = i_dsc.mint(msg.sender, amountDscToMint);
        if (!minted) {
            revert DSCEngine__MintFailed();
        }
    }

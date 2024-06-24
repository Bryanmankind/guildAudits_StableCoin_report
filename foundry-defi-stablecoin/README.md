## Findings 

- **Method mintDsc does not emite tokens :** The function call does not emit the tokens minted to the user, this is important to show the amount of tokens minted.  

- **Deposited Funds May Become Stuck if mintDsc Reverts:** If the mintDsc function reverts due to the user's health factor, the funds already deposited in the contract will remain stuck. This issue arises because there is no method in place to refund the user in such cases.

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

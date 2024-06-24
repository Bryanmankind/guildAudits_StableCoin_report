## Findings 

- **Method mintDsc does not emite tokens :** The function call does not emit the tokens minted to the user, this is important to show the amount of tokens minted.  

- **Deposited funds can get stocked in the contract if the mintDsc reverts :** If the health factor of a user reverts the alredy deposited funds in the contract will be stacked since thier is no methode to refund the user.

function mintDsc(uint256 amountDscToMint) public moreThanZero(amountDscToMint) nonReentrant {
        s_DSCMinted[msg.sender] += amountDscToMint;
        // if they minted too much ($150 DSC, $100 ETH)
        _revertIfHealthFactorIsBroken(msg.sender);
        bool minted = i_dsc.mint(msg.sender, amountDscToMint);
        if (!minted) {
            revert DSCEngine__MintFailed();
        }
    }

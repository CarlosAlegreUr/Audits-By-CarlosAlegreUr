## Summary 📌

This report is about how malicious actors could hijack the system's solvency.

## Vulnerability Details 🔍

Any address with adequate funds can artificially manipulate the perceived solvency state of the `DSCEngine.sol` contract.

The issue stems from these two functions in `DSCEngine.sol`:

```
function depositCollateral(address tokenCollateralAddress, uint256 amountCollateral) public;

function redeemCollateral(address tokenCollateralAddress, uint256 amountCollateral) external;
```

By using the `depositCollateral()` function, addresses can deposit collateral which isn't actively backing any DSC debt. Moreover, the address that deposited this "collateral" can retrieve it anytime using `redeemCollateral()`, without any restrictions or penalties.

Although `_revertIfHealthFactorIsBroken()` is invoked inside `redeemCollateral()`, for users with 0 DSC minted, this function won't revert. This is due to the health factor having it's maximum value if a user has no debt, a.k.a., 0 DSC minted.

<details>
<summary>🗄️ Step-by-step Breakdown of the Exploitation Scenario</summary>

1. **System Initiation**: Users begin interacting with the `DecentralizedStableCoin.sol` and `DSCEngine.sol` contracts.

2. **Spotting the Loophole**: A group of "mafiosos" identify a vulnerability. They deposit substantial collateral into the system, without incurring any actual debt.

3. **False Security**: The value of collateral falls, edging the system towards insolvency. However, the mafiosos' deposited funds paint a misleading picture of solvency. Other users, seeing the fund balance, remain unaware of the imminent risk.

4. **The Blackmail Phase**: Leveraging their unique position, the mafiosos approach users who invested big amounts of money in the stablecoin. They present an ultimatum: pay a "protection fee" or witness the system's solvency crumble. Moreover, any attempt to alert others will be met with an immediate system shutdown by the mafiosos.

This scenario exemplifies the potential dangers of such vulnerabilities. Not only can malicious entities manipulate the stablecoin's fate, but they can also exploit their position for extortion.

</details>

## Impact 📈

This vulnerability can result in scenarios where numerous user positions appear solvent due to the collateral that's held and "owned" by the engine. In reality, they are controlled by the original depositors, primarily due to the unrestricted nature of the `redeemCollateral()` function and the capability to deposit collateral without minting any DSC.

The system may seem robust when tested with `invariant_protocolMustHaveMoreValueThanTotalSupply()`. Yet, if the collateral's value drops, affecting many users' solvency, this can change abruptly. Addresses with "fake collateral" (collateral not backing any debt) can dictate the protocol's solvency.

This vulnerability enables single or multiple addresses to dominate the system's solvency, going against the goal of a decentralized and trustworthy engine. It also contradicts the principle that only natural fluctuations in the collateral's value should influence the system's solvency.

## Tools Used 🛠️

- Manual audit.

## Recommendations 🎯

1. **Restrictive Collateral Entry**: Make sure every collateral introduced is directly linked to some debt. This would prevent "fake collateral" entries that could be freely withdrawn.

   <details> <summary> Implementation Pseudocode Example 💻 </summary>

   Implement extra checkings in the `depositCollateral()` function to make sure everyone has some degree of debt in the system:

   ```
   function depositCollateral(address tokenCollateralAddress, uint256 amountCollateral)
       public
       moreThanZero(amountCollateral)
       isAllowedToken(tokenCollateralAddress)
       nonReentrant
   {
       if(s_DSCMinted[msg.sender] != 0) { // Changed here 🟢
           // original code...
       }
       else { // All this is else block is new code too 🟢
           uint256 amountDscToMint = /* Calculate it */
           // Not refering the contracts current function as it
           // uses msg.sender.
           depositCollateralAndMintDsc(
               tokenCollateralAddress,
               amountCollateral,
               amountDscToMint
           );
       }
   }
   ```

   > 🚧 **Notice** ⚠️: This change has not been tested, it's been added to help guiding the devs.

</details>

2. **No minted DSC should be considered unhealthy**: Make the situations where `totalDscMinted` == 0 an unhealthy ones.

   <details> <summary> Implementation Pseudocode Example 💻 </summary>

   Return an unhealthy health factor value when `totalDscMinted` == 0 like:

   ```
    function _calculateHealthFactor(uint256 totalDscMinted, uint256 collateralValueInUsd)
    internal
    pure
    returns (uint256)
    {
        if (totalDscMinted == 0) return 0; // Change here 🟢
        // original code here...
    }
   ```

   > 🚧 **Notice** ⚠️: This change has not been tested, it's been added to help guiding the devs.

</details>

> 🚧 **Note** ⚠️: Both solutions might be implemented independently or in combination to avoid this vulnerability, based on the system's desired complexity and goals. Regardless of the devs decision, after these changes,another audit is necessary.

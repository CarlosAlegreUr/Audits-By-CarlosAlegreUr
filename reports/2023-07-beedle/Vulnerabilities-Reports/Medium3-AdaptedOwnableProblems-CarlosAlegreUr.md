## **Summary 📌**

A medium risk vulnerability was identified in relation to the contracts that `Lender.sol` inherits.

## **Vulnerability Details 🔍**

The `Lender.sol` codebase utilizes a custom version of OpenZeppelin's `Ownable.sol` contract. A notable discrepancy is:

- **Missing Zero Address Check**: The `transferOwnership()` function omits the crucial check for the zero address `address(0)`.

Furthermore note the **IERC20 Interface Redundancy**. The `IERC20.sol` located at `./src/interfaces` mimics OpenZeppelin's version. However, it lacks comments, leading to redundancy and a degradation in code quality.

## **Impact 📈**

1️⃣ **Loss of Funds** 🔻

- Transferring ownership, either accidentally or maliciously, to the zero address could permanently lock up gathered fees.

2️⃣ **Uncallable functions** 🔻

- Ownership by `address(0)` makes onlyOwner() functions inoperable. Like `setLenderFee()`, `setBorrowerFee()`, `setFeeReceiver()`.

3️⃣ **Protocol Disruption** 🔻

- This ownership change can disrupt the protocol's future governance. This could severely halt the progress and development of the protocol.

4️⃣ **Potential New Vulnerabilities** 🔻

- The modifications to the `Ownable` contract expose the codebase to unforeseen vulnerabilities. The OpenZeppelin team's meticulous work, which may sometimes seem superfluous, often addresses deeply hidden vulnerabilities. often addresses really hidden vulnerabilities. Modifying or omitting parts of their code could reintroduce these vulnerabilities.

## **Tools Used 🛠️**

- Manual audit.

## **Recommendations 🎯**

Use the official, up-to-date versions of `Ownable.sol` and `IERC20.sol` from [OpenZeppelin 🌐](https://docs.openzeppelin.com/contracts/4.x/). Their extensive testing ensures reliability and security.

# QA Report for veRWA 🕵️ 
###  by [CarlosAlegreUr](https://github.com/CarlosAlegreUr)

---

## Summary 📌

Overall this report aims to make the code more readable, maintainable and easier to understand for new developers to the codebase. Furthermore it detects and provides a way to fix some useless storage consumption that could be avoided.
--- 

## Index 🗂️

- [QA]()
    + [Comment Issues 📝](#1️⃣-comment-issues-📝)
    + [Modularity and DRY Principles 🧩](#2️⃣-modularity-and-dry-principles-🧩)
    + [Readability and Consistency 📖](#3️⃣-readability-and-consistency-📖)
    + [Abstract Recommendation and Concern 📜](#4️⃣-abstract-recommendation-and-concern-📜)
- [Low Risk 🔅](#low-risk-🔅)

---

## 1️⃣ **`Comment Issues`** 📝

### **General** 🌐

The application of these changes will enable new developers to **understand the code easily** and eliminate contradictions present.

This section includes:

+ **Unclear** comment.
+ **Better arrangement** of comments.
+ **Inconsistencies** within DOCS.

> 📘 **Note:** ℹ️ It's in the bot reports already, but **please comment more your code before audits**; it will significantly save time for auditors.

#### 1️⃣.1️⃣ **Unclear Comment** 🤔

Make sure the comments are written correctly and don't lead to misinterpretation.

For example, in line 155 of **`VotingEscrow.sol`**, the following comment is confusing: "Than zeros?", "by in the FUTURE?" It may need clarification or correction to properly convey the intended meaning.


```
// From 🔴
// newLocked.end can ONLY by in the FUTURE unless everything expired: than zeros
// To 🟢
// newLocked.end can ONLY be in the FUTURE unless everything has expired; then it becomes zero.
```

#### 1️⃣.2️⃣ **Better Arrangement of Comments** 📐

Apply this to the whole codebase if necessary. Move the comments on top of the line they refer to, not at its right, for enhanced readability.

Example from a code snippet at `LendingLedger.sol`:

```
mapping(address => mapping(address => uint256)) public lendingMarketBalancesEpoch; //🔴2️⃣ Instead of starting here
// 🟢1️⃣ NOTICE COMMENT COULD GO HERE REFERING TO THE LINE BELOW ⏬
mapping(address => mapping(address => uint256)) public lendingMarketBalancesEpoch;
```

#### 1️⃣.3️⃣ **Inconsistencies within DOCS** 📜

- **Contradictory docs** at **`VotingEscrow.sol`**. Some functions have this comment:
```
 // See IVotingEscrow for documentation
```
The documentation in [IVotingEscrow](https://github.com/code-423n4/2022-08-fiatdao/blob/main/contracts/interfaces/IVotingEscrow.sol) doesn't match with the implementation in the `VotingEscrow.sol` contract, e.g.:

- **`createLock()`**: missing the `__unlockTime_` parameter.
- **`increaseAmount()`**: IVotingEscrow docs say it doesn't update the end lock time; the project's code does.
- **`delegate()`**: IVotingEscrow's comment says: "Can only undelegate to longer lock duration". In your project this doesn't apply, lock times are always 5 years.

It's recommended to **adapt the comments** and place them in the project repo, not external links if possible, facilitating the understanding process for anyone new to the codebase.

---

## 2️⃣ **`Modularity and DRY Principles`** 🧩

### General 🌐

Some functions and variables are used in different contracts at the same time, consider implementing another contract that implements them.

> 🚧 **Note** ⚠️: This might increase gas consumption. It has not been tested. However, sometimes a slight increase in gas consumption is worth the increase adherence to modularity or DRY principles.

Features that contracts share and might be interesting to isolate in some kind of **`Utilities.sol`**:

- **`onlyGovernance()`** modifier.
- **WEEK** constant and **10**\*\*18 multiplier.
- **`_floorToWeek()`** function in **`VotingEscrow.sol`**. Its functionality is used in the other contracts as:

```
uint256 currEpoch = (block.timestamp / WEEK) * WEEK;
```

Which is essentially the same computation **`_floorToWeek()`** does:
```
function _floorToWeek(uint256 _t) internal pure returns (uint256) {
    🟢 return (_t / WEEK) * WEEK;
}

// 🟢 So you could do:
uint256 currEpoch = _floorToWeek(block.timestamp);
```

### GaugeController.sol 🎛️

Apply DRY at **`gauge_relative_weight_write()`** by leveraging the **`checkpoint_gauge()`** function:

```
// Before 🟡:
    function gauge_relative_weight_write(address _gauge, uint256 _time) external returns (uint256) {
        _get_weight(_gauge);
        _get_sum();
        return _gauge_relative_weight(_gauge, _time);
    }

// After 🟢:
   function gauge_relative_weight_write(address _gauge, uint256 _time) external returns (uint256) {
       checkpoint_gauge(_gauge);
       return _gauge_relative_weight(_gauge, _time);
   }
```

> 📘 **Note** ℹ️: Tests still pass with this change.


---

## 3️⃣ **`Readability and Consistency`** 📖

### **General** 🌐

These recommendations align with industry best practices for code readability and consistency. Implementing these suggestions will significantly improve the codebase, making it more accessible and maintainable for current and future developers.

### **Specific Changes and Suggestions** 📝

#### **Gauges and Lending Markets Clarification**:

- Since the only gauges used are Lending Markets, include a comments in **`LendingLedger.sol`** or in **`GaugeController.sol`** indicating this to prevent confusion.

### **Variables' Name Changes and Enhancements** 🛠️

> 📘 **Note** ℹ️: While these changes may make the code more verbose, they will provide greater clarity for new developers, enhancing readability and maintainability.

- **Clarify Mathematical Terms**: Insert comments to better illustrate the correlation between slope points, biases, and their functional representations. While links to CRV finance docs are useful, direct comments in the code will facilitate understanding. For example:

```
🔴 Before
    struct Point {
        int128 bias;
        int128 slope;
        uint256 ts; //🔴 Abbreviated variables are less intuitive
        uint256 blk;
    }

🟢 After
    struct Point {
        int128 weightBias;
        int128 votingPowerSlope;
        uint256 timestamp;
        uint256 blockNumber;
    }
```

- **Rename Confusing Variables**: Adjust confusing names in the **`LockAction`** enum to better align with their functionality.
    - `INCREASE_AMOUNT_AND_DELEGATION` to `INCREASE_NOT_DELEGATED_AMOUNT`

- **Enhance Readability within GaugeController.sol**:
    - In **`_get_weight()`** and **`_get_sum()`**, rename `d_bias` to `decrease_bias`.
    - Replace state variable name `points_sum` with `summation_of_gauges_weights`.
    - Consider renaming other function varibales and state variables along the codebase too.

### **Commenting and Spacing Improvements** 🖋️

- **Enhance Commenting**: Introduce more comments, especially in **`LendingLedger.sol`**, to clarify the code's purpose.

- **Improve Spacing**: Add spacing within functions for segmentation and understanding.

> 🚧 **Note** ⚠️: Consider adding private helper functions to improve modularity, especially where additional spacing is utilized within the code.

**Example from `LendingLedger.sol` in the `claim()` function**:
(spacings added are marked with 🟢)

```
function claim(
        address _market,
        uint256 _claimFromTimestamp,
        uint256 _claimUpToTimestamp
    ) external is_valid_epoch(_claimFromTimestamp) is_valid_epoch(_claimUpToTimestamp) {
        address lender = msg.sender;
        uint256 userLastClaimed = userClaimedEpoch[_market][lender];
        require(userLastClaimed > 0, "No deposits for this user");
🟢
        // Updating
        _checkpoint_lender(_market, lender, _claimUpToTimestamp);
        _checkpoint_market(_market, _claimUpToTimestamp);
🟢
        // Getting time ranges from 
        uint256 currEpoch = (block.timestamp / WEEK) * WEEK;
        uint256 claimStart = Math.max(userLastClaimed, _claimFromTimestamp);
        uint256 claimEnd = Math.min(currEpoch - WEEK, _claimUpToTimestamp);
🟢
        uint256 cantoToSend;
        if (claimEnd >= claimStart) {
            // This ensures that we only set userClaimedEpoch when a claim actually happened
            for (uint256 i = claimStart; i <= claimEnd; i += WEEK) {
                uint256 userBalance = lendingMarketBalances[_market][lender][i];
                uint256 marketBalance = lendingMarketTotalBalance[_market][i];
                RewardInformation memory ri = rewardInformation[i];
                // Can only claim for epochs where rewards are set, even if it is set to 0
                require(ri.set, "Reward not set yet"); 
                uint256 marketWeight = gaugeController.gauge_relative_weight_write(_market, i); // Normalized to 1e18
                cantoToSend += (marketWeight * userBalance * ri.amount) / (1e18 * marketBalance); // (marketWeight / 1e18) * (userBalance / marketBalance) * ri.amount;
            }
            userClaimedEpoch[_market][lender] = claimEnd + WEEK;
        }
🟢        
        // Sending canto if so needed
        if (cantoToSend > 0) {
            (bool success, ) = msg.sender.call{value: cantoToSend}("");
            require(success, "Failed to send CANTO");
        }
    }
```

---

## 4️⃣ **`Abstract Recommendation and Concern`** 📜

### **Recommendation** 📣

#### **Better Way of Analyzing LendingMarkets Used by `LendingLedger.sol`** 

Developers are aware of the importance of thoroughly reviewing a whitelisted lending market. Currently, the analysis of lending markets is manual and the time it consumes is proportional to the lending markets' codebase complexity.

Forcing lending markets to adhere to a general protocol or pattern could streamline this process, regardless of market complexity. For example, mandating lending markets that wish to be whitelisted to implement state variables (either in an existing contract on their system or a specific one crafted by the veRAW dev team) that track or verify users' funds, deposits, or withdrawals.

The existence of balances and their traceability via variable states, events, or a mix of both could facilitate the validation process for new lending markets.

### **Concern** 🚨

As I am not knowledgeable in the CANTO network and didn't have time for a thorough analysis, I have a concern that might be inconsequential but is worth mentioning.

If active users of the veRAW system are not influential enough to have veto capability in CANTO governance, this could potentially influence the protocol in a malicious way, rendering it inoperable.

Due to my lack of time to research, I'm not sure about the veracity of these claims. Therefore, I include them in this QA report. If true, I believe they belong to the Medium Risk classification.

Malicious influential governors could dilute liquidity rewards distribution by whitelisting fake lending markets or directly changing the weights of existing ones after rewards are set, effectively misappropriating funds that should have belonged to others. That's why I would rate this concern as medium. They could extract the funds from **`LendingLedger`** using the `claim()` function and fake `sync_ledger()` function calls.

---

# **Low Risk** 🔅

### In **`LendingLedger.sol`**

There is a risk of unnecessary storage usage that can be mitigated.

This situation arises if a whitelisted market later becomes "blacklisted," meaning it's no longer whitelisted.

For users that participated in blacklisted markets, they can still claim the rewards, leading to `checkpoint()` calls that use storage spaces for markets that are no longer relevant. Although the weight of blacklisted markets is set to 0, meaning no rewards are claimed, storage usage is still accessed and filled through different checkpoint functions.

**Recommended Action** 🛠️

Add a revert condition for blacklisted markets claims using the following already existing code snippet:

```
require(lendingMarketWhitelist[lendingMarket], "Market not whitelisted");
```

This action will prevent unnecessary storage slots from being occupied.

Since this `require` statement is already used in the `syncLedger()` function, consider making it a modifier to leverage its usage and further enhance the code's efficiency and readability.

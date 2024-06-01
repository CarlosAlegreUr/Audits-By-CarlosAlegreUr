
### by [CarlosAlegreUr](https://github.com/CarlosAlegreUr)

## Summary

Chainlink price feeds are `RESTRICTED` and the checks made on `aggregator.latestRoundData();` from `DataFeed.sol` at `_getDataInBase18()` function are insufficient and inorrect.

## Vulnerability Detail

There is a key check that is missing for Chainlink data feeds, and the one implemented is wrong.

The eventual impact of all of them is the usage of incorrect pricing data which can lead to the `_validateAmountUsdIn()` letting users deposit when they should not or prohibiting users from depositing when they should, this would depend on the stale data provided being higher or lower than the actual price.

- 1️⃣ There are no checks for `hearbeat` on the used [EUR/USD data feed](https://docs.chain.link/data-feeds/price-feeds/addresses?network=ethereum&page=1&search=EUR%2FUSD). This means that during Forex market valid days, the price feed could be working on stale data and the protocol would be using it. _Example: If one Tuesday price feed becomes stale for more than 1h in Arbitrum, that price would be used._

- 2️⃣ Check for Market Hours for Forex is incorrect. The current check is:
 
```solidity
    /**
     * @dev healty difference between `block.timestamp` and `updatedAt` timestamps
     */
    uint256 private constant _HEALTHY_DIFF = 3 days;

    // more contract code...

    require(
            // solhint-disable-next-line not-rely-on-time
            block.timestamp - updatedAt <= _HEALTHY_DIFF,
            "DF: feed is unhealthy"
        );
```

A staleness of > than 3 days does not mean weekend nor that it is _`18:00 ET Sunday to 17:00 ET Friday`_ as the Forex Market Hours are for **EUR/USD** price feed. It could be that price has been stale for 3 days, starting on **Monday**, and now it is **Wednesday**. Now in this case the code would execute but the `heartbeat` on any of the chains that the procotol works on would not be checked and it is crucial so as to not to use stale data. (**EUR/USD** on Ethereum Hearbeat is 1 day, Arbitrum is 1h) 

Due to a 3 days interval not properly meaning weekend, this check is incorrect and can clearly allow for stale data to be used. As as none of the heartbeats is > 3 days, thus the stale time can pass yet the unhealthy check will still deem it valid.

> 📘 **Note** ℹ️ It is said that sequencer being down it is a `TRUSTED` risk thus invalid issue for this audit. I just want to recommend its check, it is vital to use valid data to create trust. And your system is heavely centralized thus maximum trust must be a given.

## Impact

`_validateAmountUsdIn()` from `DepositVualt.sol` can revert valid deposits or allow invalid ones due to using stale data from Chainlink. Thus breaking one of the protocol restrictions of creating restrains of minimal deposit amounts for some addresses.

## Code Snippet

`_validateAmountUsdIn()` calls `minAmountToDepositInUsd()` on its `require` statement [here](https://github.com/sherlock-audit/2024-05-midas/blob/main/midas-contracts/contracts/DepositVault.sol#L161).

Then `minAmountToDepositInUsd()` calls `_getDataInBase18()` from `DataFeed.sol` [here](https://github.com/sherlock-audit/2024-05-midas/blob/main/midas-contracts/contracts/DepositVault.sol#L139).

If the data is stale, wheter because of being a higher value or a smaller value, the minimum deposit limits will be incorrect. This can make valid depositors revert or invalid depositors pass.

The `_getDataInBase18()` function from `DataFeed.sol`, with its checks and lack of them, can be seen [here](https://github.com/sherlock-audit/2024-05-midas/blob/main/midas-contracts/contracts/feeds/DataFeed.sol#L64).

## Tool used

Manual Review

## Recommendation

Check the Chainlink data feed correctly and implement the proper checks in the `_getDataInBase18()` function. Add all the following recommendations:

- 1️⃣ Add the `hearbeat` check as the Chainlink docs say [here](https://docs.chain.link/data-feeds#check-the-timestamp-of-the-latest-answer).

- 2️⃣ Fix the Market Hours problem by checking that the current time is actually a valid time for the Forex market. This can be done with on-chain logic or, due to the centralized nature of the protocol, signaling it to the contracts on chain with a flag that a trusted party can set every weekend.

The checks must confirm that when using Forex data the timestamp is, as per [Chainlink docs](https://docs.chain.link/data-feeds/selecting-data-feeds#market-hours):

```text
18:00 ET Sunday to 17:00 ET Friday.
The feeds also follow the global Forex market Christmas and New Year's Day holiday schedule. Many non-G12 currencies primarily trade during local market hours. It is recommended to use those feeds only during local trading hours.
```

Here is a `chat-GPT-4o` generated example on how to implement the check on-chain:

> 🚧 **Note** ⚠️ Gpt code has not been audited, it is meant to serve just as a guide.

<details> <summary> See code 👁️ </summary>

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract ForexTimeCheck {

    // Converts a timestamp to the corresponding day of the week (0 = Sunday, 1 = Monday, ..., 6 = Saturday)
    function getDayOfWeek(uint256 timestamp) public pure returns (uint8) {
        return uint8((timestamp / 86400 + 4) % 7); // 86400 seconds in a day, 4 because 1970-01-01 was a Thursday
    }

    // Converts a timestamp to the year, month, and day
    function getYearMonthDay(uint256 timestamp) public pure returns (uint16 year, uint8 month, uint8 day) {
        int __days = int(timestamp / 86400);

        int L = __days + 68569 + 2440588;
        int N = (4 * L) / 146097;
        L = L - (146097 * N + 3) / 4;
        int _year = (4000 * (L + 1)) / 1461001;
        L = L - (1461 * _year) / 4 + 31;
        int _month = (80 * L) / 2447;
        int _day = L - (2447 * _month) / 80;
        L = _month / 11;
        _month = _month + 2 - 12 * L;
        _year = 100 * (N - 49) + _year + L;

        year = uint16(_year);
        month = uint8(_month);
        day = uint8(_day);
    }

    // Check if the timestamp is within the restricted time range
    function isWithinRestrictedTime(uint256 timestamp) public pure returns (bool) {
        uint8 dayOfWeek = getDayOfWeek(timestamp);
        uint256 timeOfDay = timestamp % 86400; // number of seconds since midnight UTC

        // Convert times to seconds since midnight
        uint256 sundayStart = 23 * 3600; // 23:00 UTC on Sunday
        uint256 fridayEnd = 22 * 3600; // 22:00 UTC on Friday

        // Check if it's Sunday and after 23:00 UTC
        if (dayOfWeek == 0 && timeOfDay >= sundayStart) {
            return true;
        }

        // Check if it's Friday and before 22:00 UTC
        if (dayOfWeek == 5 && timeOfDay < fridayEnd) {
            return true;
        }

        // Check if it's Monday to Thursday (1 to 4)
        if (dayOfWeek >= 1 && dayOfWeek <= 4) {
            return true;
        }

        return false;
    }

    // Check if the date is a holiday (Christmas or New Year's Day)
    function isHoliday(uint256 timestamp) public pure returns (bool) {
        (uint16 year, uint8 month, uint8 day) = getYearMonthDay(timestamp);
        // Check for Christmas (December 25) and New Year's Day (January 1)
        if ((month == 12 && day == 25) || (month == 1 && day == 1)) {
            return true;
        }
        return false;
    }

    // Main function to check if the timestamp falls within restricted time or holiday
    function isRestricted(uint256 timestamp) public pure returns (bool) {
        if (isHoliday(timestamp)) {
            return true;
        }
        return isWithinRestrictedTime(timestamp);
    }
}
```

</details>

- 3️⃣ Please check for sequencer up-time in **Arbitrum** as [Chainlink says](https://docs.chain.link/data-feeds/l2-sequencer-feeds).

All logic being implemented would look something like this:

```solidity
// THIS IS JUST A TEMPLATE CODE TO SHOW THE LOGIC, IT IS NOT AUDITED 🔴
function _getDataInBase18()
        private
        view
        returns (uint80 roundId, uint256 answer)
    {
        // more code...
        (uint80 _roundId, int256 _answer, , uint256 updatedAt, ) = aggregator
            .latestRoundData();

        // isRestricted() is from GPTs code mentioned above
        // Though here you could use just a boolean flag if you use the centralized solution
        if(isRestricted(updatedAt)){
            revert "Invalid data: forex is closed";
        }

        if( block.timestamp - updatedAt <= HEARTBEAT){
            revert "Invalid data: heartbeat stale";
        }
        // Sequencer check should also be included...
        // more code...
    }
```

> 🚧 **Note** ⚠️ In the README of the repo for this Sherlock contest, the [**IB01/USD**](https://docs.chain.link/data-feeds/price-feeds/addresses?network=ethereum&page=1&search=IB01) feed is mentioned. Yet is not present nor used in the code. Depite that, if it were to be used it would face the same problems of _heartbeat_ and _Market Hours_, as it is treated as **UK_ETF** in Chainlink ([see market hours here](https://docs.chain.link/data-feeds/selecting-data-feeds#market-hours)). And for **UK_ETF** 3 days since last update does not propely reflect the market hours either and can lead to stale data. 

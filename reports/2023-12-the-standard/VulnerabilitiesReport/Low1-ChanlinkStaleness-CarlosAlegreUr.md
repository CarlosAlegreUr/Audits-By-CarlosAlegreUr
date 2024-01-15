### by [CarlosAlegreUr](https://github.com/CarlosAlegreUr)

### **Lack of check for Chainlink price feed's staleness** 🔢

## **Summary 📌**

When checking Chainlink Data Feeds its a good practice to check how long its been
since the last update. If its been too long, the data provided might be stale and should
not be used.

Although a fail in the Chainlink network is rare, it can happen and it is
something your code should be ready for.

---

## **Vulnerability Details 🔍 && Impact 📈**

The code uses the `EUR/USD` data feed when calling `distributeAssets()` in the `LiquidationPool`. Each feed on its own and in different chains can be updated at different rates (Chainlink calls them `Heartbeat`). For `EUR/USD ArbitrumOne`, the data feed
is given new data every 3600s => 1h. As you can see [here](https://docs.chain.link/data-feeds/price-feeds/addresses?network=arbitrum&page=1&search=EUR) in the Chainlink docs.

If the price feed has not been updated in the last hour an incorrect price can be used whether
for good or bad as this can harm users given outdated more expensive prices or benefit them giving outdated cheaper prices.

---

## **Tools Used 🛠️**

- Manual audit.

---

## **Recommendations 🎯**

To avoid this scenario you can use the `uint256 updatedAt` return data from `AggregatorV3.latestRoundData()`. You should check `block.timestamp - updatedAt >= heartbeatOfDataFeed` to see if its been too long since the last price update before proceeding to use the data. And then, according to that, handle the situation as you see fit, maybe with a revert.

---
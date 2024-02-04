# Gas Price

This document describes the rules for the gas price calculation for the projects using Coreum.

### Fee model <a href="#fee-model" id="fee-model"></a>

The Coreum chain uses the [Coreum Fee Model](https://docs.coreum.dev/modules/feemodel.html) for the `min gas price` calculation. If the sent transaction's gas price is lower than the `min gas price` the transaction will be rejected. The minimum gas price is a dynamic value and can be changed after each block.

### How to calculate the min gas price for the next block <a href="#how-to-calculate-the-min-gas-price-for-the-next-block" id="how-to-calculate-the-min-gas-price-for-the-next-block"></a>

There are three ways how it can be done:

#### Use recommended gas price endpoint <a href="#use-recommended-gas-price-endpoint" id="use-recommended-gas-price-endpoint"></a>

[RecommendedGasPrice](https://github.com/CoreumFoundation/coreum/blob/master/proto/coreum/feemodel/v1/query.proto) endpoint returns a quite accurate gas price prediction for future blocks. It will return low, med and high values to determine the possible range of gas price changes. You can use med value with a decent certainty that your transaction will go through, but to be almost 100% sure, you can use the high value.

#### Use optimized gas price calculation <a href="#use-optimized-gas-price-calculation" id="use-optimized-gas-price-calculation"></a>

For the optimized gas price calculation you can use the [MinGasPrice Query](https://github.com/CoreumFoundation/coreum/blob/master/proto/coreum/feemodel/v1/query.proto) to get the current min gas price. It is recalculated after each block, so it is recommended to use a multiplier.

| Waiting time | Multiplier          |
| ------------ | ------------------- |
| 1s           | 1.1                 |
| 30s          | 1.3                 |
| 60s          | 1.5                 |
| > 60s        | `initial gas price` |

The table above shows the correlation of the `waiting time` - the time from the `MinGasPrice Query` until the transaction submission, and the recommended `multiplier` to use for the gas price in the result.

![](https://docs.coreum.dev/assets/min\_gas\_price\_chart.f93d7520.png)

In the picture, you can see how it's changing over time, and explains the values in the tables.

For example, you can use 1.1 as a multiplier if you send the tx right after querying MinGasPrice.

#### Use non-optimized `initial gas price` <a href="#use-non-optimized-initial-gas-price" id="use-non-optimized-initial-gas-price"></a>

The `initial gas price` is the gas price used in the [Coreum Fee Model](https://docs.coreum.dev/modules/feemodel.html) as initial. That value is a parameter, which is set initially in the genesis and can be updated by the governance. This value can be used as a gas price, for cases when the tools you use don't support additional queries. To get that value automatically you can use: [Params Query](https://github.com/CoreumFoundation/coreum/blob/master/proto/coreum/feemodel/v1/query.proto) and `ModelParams.initial_gas_price` in the result. Or you can get it manually on the [explorer params page (Fee Model section)](https://explorer.coreum.com/coreum/params). Using that value the transactions will pass with most of the cases, except the cases with high chain load, in that case the gas price might go above the `initial gas price`.

Also be aware that if you hardcode gas price instead of querying and the value is increased by governance voting, txs might start failing, so it is recommended to query it

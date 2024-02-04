# Fee Model

### Abstract <a href="#abstract" id="abstract"></a>

This document specifies the `feemodel` module. The module is responsible for calculating minimum gas price required by the chain based on the [parameters](https://docs.coreum.dev/modules/feemodel.html#parameters) of fee model.

Two charts are presented below, showing how the implemented fee model behaves. Keep in mind that data presented on those charts were generated using `MaxGasPrice` set to `0.15` for better readability, while in reality we use `62.5`.

TERMS:

* _long average block gas_ is the EMA (exponential moving average) of gas consumed by previous blocks using `LongEmaBlockLength` parameter for computing the EMA.
* _short average block gas_ is the EMA (exponential moving average) of gas consumed by previous blocks using `ShortEmaBlockLength` parameter for computing the EMA.
* `MaxGasPrice = InitialGasPrice * MaxGasPriceMultiplier`
* `EscalationStartBlockGas = MaxBlockGas * EscalationStartFraction`

The chart below presents the dependency between _short average block gas_ and minimum gas price required by the network on next block.

![Fee model curve](https://docs.coreum.dev/assets/curve.54b8c951.png)

There are four regions on the fee model curve:

* green - between 0 and _long average block gas_ where gas price goes down exponentially from `InitialGasPrice` to gas price with maximum discount (`InitialGasPrice * (1 - MaxDiscount)`),
* red - between _long average block gas_ and `EscalationStartBlockGas` where we offer gas price with maximum discount all the time,
* yellow - between `EscalationStartBlockGas` and `MaxBlockGas` where price goes up rapidly (being an output of a power function) from gas price with maximum discount to `MaxGasPrice`,
* blue - above `MaxBlockGas` (if it happens for any reason) where price is equal to `MaxGasPrice`.

The input (x value) for that function is calculated by taking _short block gas average_. Price (y value) being an output of the fee model is used as the minimum gas price for the next block.

The second chart presents the model behavior over time, presenting how changes in gas consumed by blocks affect the minimum gas price required by the network. The chart was calculated using fixed \*long average block gas\` equal to 5 millions.

![Fee model time series](https://docs.coreum.dev/assets/time\_series.61a3dfc3.png)

* x axis represents block number
* left y axis is related to `ShortEMA` (red line) and `LongEMA` (orange line),
* right axis presents the minimum gas price computed after a particular block,
* blue line (almost completely covered by the red one) represents the raw gas consumed by blocks,
* red line is the _short average block gas_,
* orange line is the _long average block gas_,
* green line is the minimum gas price required by the network.

A few things to note:

* whenever `ShortEMA` goes above `MaxBlockGas` price is set to `MaxGasPrice`,
* whenever `ShortEMA` goes above `EscalationStartBlockGas` price starts growing rapidly up to `MaxGasPrice`,
* whenever `ShortEMA` is 0, price is set to `InitialGasPrice`,
* whenever `ShortEMA` is equal to or greater than `LongEMA` maximum discount (`MaxDiscount`) is applied on top of `InitialGasPrice`,
* when `ShortEMA` goes from 0 to `LongEMA` price drops until price with maximum discount is reached.

### &#x20;<a href="#state" id="state"></a>

### State <a href="#state" id="state"></a>

The `x/feemodel` module at the end of each block computes the minimum gas price required by the chain for the next block.

State managed by feemodel module:

* MinGasPrice: `0x01 | -> string(minGasPrice)`
* ShortEMAGas: `0x02 | -> int64(shortEMAGas)`
* LongEMAGasKey: `0x03 | -> int64(longEMAGas)`

#### MinGasPrice <a href="#mingasprice" id="mingasprice"></a>

Minimum gas price required by chain

**ShortEMAGas**

Short moving average of gas consumed by previous blocks

**LongEMAGasKey**

Long moving average of gas consumed by previous blocks

### &#x20;<a href="#keeper" id="keeper"></a>

### Keeper <a href="#keeper" id="keeper"></a>

The feemodel module provides a keeper providing these methods:

```go
type Keeper interface {
// TrackedGas returns gas limits declared by transactions executed so far in current block
TrackedGas(ctx sdk.Context) int64

// TrackGas increments gas tracked for current block
TrackGas(ctx sdk.Context, gas int64)

// SetParams sets the parameters of the model
SetParams(ctx sdk.Context, params types.Params)

// GetParams gets the parameters of the model
GetParams(ctx sdk.Context) types.Params

// GetShortEMAGas retrieves average gas used by previous blocks, used as a representation of smoothed gas used by latest block
GetShortEMAGas(ctx sdk.Context) int64

// SetShortEMAGas sets average gas used by previous blocks, used as a representation of smoothed gas used by latest block
SetShortEMAGas(ctx sdk.Context, emaGas int64)

// GetLongEMAGas retrieves long average gas used by previous blocks, used for determining average block load where maximum discount is applied
GetLongEMAGas(ctx sdk.Context) int64

// SetLongEMAGas sets long average gas used by previous blocks, used for determining average block load where maximum discount is applied
SetLongEMAGas(ctx sdk.Context, emaGas int64)

// GetMinGasPrice returns current minimum gas price required by the network
GetMinGasPrice(ctx sdk.Context) sdk.DecCoin

// SetMinGasPrice sets minimum gas price required by the network on current block
SetMinGasPrice(ctx sdk.Context, minGasPrice sdk.DecCoin)
}
```

From all of these methods only `GetMinGasPrice` should be used by other modules. All the other ones serve internal needs of feemodel module.

### &#x20;<a href="#parameters" id="parameters"></a>

### Parameters <a href="#parameters" id="parameters"></a>

The feemodel module contains the following parameters:

| Key                     | Type         | Example  |
| ----------------------- | ------------ | -------- |
| InitialGasPrice         | string (dec) | "0.0625" |
| MaxGasPriceMultiplier   | string (dec) | "1000"   |
| MaxDiscount             | string (dec) | "0.5"    |
| EscalationStartFraction | string (dec) | "0.8"    |
| MaxBlockGas             | int64        | 50000000 |
| ShortEmaBlockLength     | uint32       | 50       |
| LongEmaBlockLength      | uint32       | 1000     |

#### &#x20;<a href="#initialgasprice" id="initialgasprice"></a>

#### InitialGasPrice <a href="#initialgasprice" id="initialgasprice"></a>

`InitialGasPrice` is the minimum gas price required when _block gas short average_ is 0. It happens when there are no transactions being broadcasted. This value is also used to initialize gas price on a brand-new chain.

#### MaxGasPriceMultiplier <a href="#maxgaspricemultiplier" id="maxgaspricemultiplier"></a>

`MaxGasPriceMultiplier` is used to multiply `InitialGasPrice` to get the minimum gas price required when _block gas short average_ is greater than or equal to `MaxBlockGas`.This value is used to limit gas price escalation to avoid having possible infinity gas price value otherwise.

#### MaxDiscount <a href="#maxdiscount" id="maxdiscount"></a>

`MaxDiscount` is the maximum discount we offer on top of `InitialGasPrice` if _short average block gas_ is between _long average block gas_ and `EscalationStartBlockGas` (`EscalationStartBlockGas = MaxBlockGas * EscalationStartFraction`).

#### EscalationStartFraction <a href="#escalationstartfraction" id="escalationstartfraction"></a>

`EscalationStartFraction` is used to multiply `MaxBlockGas` to get the block gas usage where gas price escalation starts if _short average block gas_ is higher than this value.

#### MaxBlockGas <a href="#maxblockgas" id="maxblockgas"></a>

`MaxBlockGas` sets the maximum capacity of a block. This is enforced on tendermint level in genesis configuration. Once short average block gas goes above this value, gas price is a flat line equal to `MaxGasPrice` (`MaxGasPrice = InitialGasPrice * MaxGasPriceMultiplier`).

#### &#x20;<a href="#shortemablocklength" id="shortemablocklength"></a>

#### ShortEmaBlockLength <a href="#shortemablocklength" id="shortemablocklength"></a>

`ShortEmaBlockLength` defines inertia for short average long gas in EMA model. The equation is:

`NewAverage = ((ShortAverageBlockLength - 1)*PreviousAverage + GasUsedByCurrentBlock) / ShortAverageBlockLength`

The value might be interpreted as the number of blocks which are taken to calculate the average. It would be exactly like that in SMA model, in EMA this is an approximation.

#### LongEmaBlockLength <a href="#longemablocklength" id="longemablocklength"></a>

`LongEmaBlockLength` defines inertia for long average block gas in EMA model. The equation is:

`NewAverage = ((LongAverageBlockLength - 1)*PreviousAverage + GasUsedByCurrentBlock) / LongAverageBlockLength`

The value might be interpreted as the number of blocks which are taken to calculate the average. It would be exactly like that in SMA model, in EMA this is an approximation.

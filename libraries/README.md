# Library
&nbsp;

## Uptrend Downtrend Loopback Candle Identification Lib

File Script: [Uptrend Downtrend Loopback Candle Identification Lib](uptrend_downtrend_loopback_candle_identification_lib.pine)

Link Script: [Uptrend Downtrend Loopback Candle Identification Lib](https://www.tradingview.com/v/7XVVW2CK/)

&nbsp;

> ### uptrendLoopbackCandleIdentification:
- The function starts by declaring a variable *isUptrend* and initializes it as *false*. This variable will be used to store the result of the loop analysis.
- A *for loop* is used to iterate through the specified *lookbackPeriod* with the loop index variable *i*.
- Inside the loop, the code checks if the *low* of the current bar (low) is greater than or equal to the *low* of the bar *i* bars ago (low).
- If the condition is met, the variable *isUptrend* is set to *true*.
- If the *current low* is *lower* than the *previous lows*, the *isUptrend* boolean variable is set to *false* and the loop is broken using the *break* statement.
- After the loop has finished, the function returns the final value of *isUptrend*.

&nbsp;

> ### downtrendLoopbackCandleIdentification:
- The function starts by declaring a variable *isDowntrend* and initializes it as *false*. This variable will be used to store the result of the loop analysis.
- A *for loop* is used to iterate through the specified *lookbackPeriod* with the loop index variable *i*.
- Inside the loop, the code checks if the *low* of the current bar (low) is less than or equal to the *low* of the bar *i* bars ago (low).
- If the condition is met, the variable *isDowntrend* is set to *true*.
- If the *current low* is *higher* than the *previous lows*, the *isDowntrend* boolean variable is set to *false* and the loop is broken using the *break* statement.
- After the loop has finished, the function returns the final value of *isDowntrend*.

&nbsp;

_Each function will return true if the respective condition is met for any bar within the lookback period. The loops will not terminate early if the conditions are met._
___
___
&nbsp;

## EMA Lib

The **Exponential Moving Average** (EMA) is a *trend following*, when the price of a asset is above the EMA, it could indicate an *upward* trend, while prices below of the asset it could indicate a *downward* trend.

File Script: [Exponential Moving Average Lib](ema_lib.pine)

- **emaFunction(emaLengthPeriod)** it defines the function *emaFunction* which takes an argument *emaLengthPeriod* is the length of the **EMA**, the *number of periods* to consider in the EMA calculation.
- **source** defines the source of the data for the EMA calculation as the *closing price* of each *period*. Setting the **source** as the **close** variable from pinescript.
- **alpha** (*alpha = 2 / (emaLengthPeriod + 1)*) calculates the *smoothing factor* for the EMA.
- **sma = 0.0** and **ema = 0.0** it initializes the *SMA* and *EMA* to zero.
- **for i = 0 to emaLengthPeriod - 1** it *loops* from *0 to emaLengthPeriod - 1* which is used to calculate the *SMA* over the *EMA* length *period*.
- **sma := sma + source[i] / emaLengthPeriod** it is inside the loop, the *SMA* is calculated by *summing* up the *closing* prices of the *previous* bars (*source[i]*) and *dividing* by the *EMA* length *period*, which calculates the *SMA* over the *EMA* length *period*.
- **ema := na(ema[1]) ? sma : ((alpha * source) + ((1 - alpha) * nz(ema[1])))** when *SMA* is calculated, the *EMA* is then computed. If the *previous EMA* value (*ema[1]*) is not available (*na*), the *EMA* is set to the *SMA*. If the *previous EMA* value is available, the *EMA* is calculated as the *weighted sum* of the *current close* price and the *previous EMA* value. Using **na()** function which _returns true if something is na, false otherwise._

&nbsp;

_Observation: This code is a basic implementation of an EMA, but it's worth noting that built-in function for calculating EMA is available in Pine Script, which is optimized and easier to use_.

_For EMA, can be used ta.ema(source, length)_.
___
___
&nbsp;

## RSI Lib

The **Relative Strength Index** (RSI) is a *momentum oscillator* that measures the speed and change of price movements. Can be used to identify **overbought** and/or **oversold** conditions in a market.

File Script: [Relative Strength Index Lib](rsi_lib.pine)

- **rsiFunction(rsiLengthPeriod)** it defines the function *rsiFunction* which takes an argument *rsiLengthPeriod* is the length of the **RSI**, the *number of periods* to consider in the RSI calculation.
- **lengthPeriod = rsiLengthPeriod** it represents the *length period* used in the *RSI* calculation.
- **source = close** assigns the *closing price* to *source*.
- **differenceSource = ta.change(source)** calculates the *difference* between the *current* source value and the *previous* one, then is stored in the variable **differenceSource**.
- **gain = na(differenceSource) ? na : math.max(differenceSource, 0)** calculates the *gain* for each bar, if the **differenceSource** value is *not available* (**na**), then *gain* is also set to *na*. Otherwise, *gain* is calculated as the *maximum* between **differenceSource** and *0*, it ensures that only *positive* values are considered for the *gain*.
- **loss = na(differenceSource) ? na : -math.min(differenceSource, 0)** it calculates the *loss* for each bar, if the **differenceSource** value is *not available* (*na*), then *loss* is also set to *na*. Otherwise, *loss* is calculated as the *negative* of the *minimum* between **differenceSource** and *0*, it ensures that only *negative* values (or 0) are considered for the *loss*.
- **averageGain = ta.rma(gain, lengthPeriod)** it calculates the *relative moving average* (**RMA**) of the *gain* values over the specified **lengthPeriod**.
- **averageLoss = ta.rma(loss, lengthPeriod)** it calculates the *relative moving average* (**RMA**) of the *loss* values over the specified **lengthPeriod**.
- **relativeStrength = (averageGain / averageLoss)** it calculates the *relative strength* by *dividing* the **averageGain** by the **averageLoss**.
- **relativeStrengthIndex = (100 - (100 / (1 + relativeStrength)))** it calculates the **RSI** by *dividing* 100 by the *sum* of 1 and **relativeStrength**, then the result is *subtracted* from 100.
- **relativeStrengthIndex** *returns* the calculated **RSI** value.

&nbsp;

_Observation: This code is a basic implementation of an RSI, but it's worth noting that built-in function for calculating RSI is available in Pine Script, which is optimized and easier to use_.

_For RSI, can be used ta.rsi(source, length)_.
___
___
&nbsp;

## PSAR Lib

The **Parabolic Stop and Reverse** (PSAR) is a *trend following*, it aims to *identify* the *direction* of a *trend* and potentially providing *entry* and *exit* points. *PSAR* move with *price changes* and are used to *identify* and *follow* the *direction* of a *trend*.

File Script: [Parabolic Stop and Reverse Lib](psar_lib.pine)

- **psarFunction(startAccelerationFactor, incrementAccelerationFactor, maximumAccelerationFactor)** it defines the function *psarFunction* which takes an argument *startAccelerationFactor* which is the initial *acceleration factor (AF)*, it defines how *sensitive* the *PSAR* is at the beginning of a new *trend*. The *incrementAccelerationFactor* is the *increment* for the *AF* each time a new *extreme point (EP)* is recorded, an *EP* is the *highest high* of the *current uptrend* or the *lowest low* of the *current downtrend*. And *maximumAccelerationFactor* is the *maximum* value that the *AF* can reach.

> **CORE CONCEPTS**:

- **isLong** a *boolean* variable, indicates if the market is in an *uptrend* set the variable to *true*, or *downtrend* set the variable to *false*.
- **accelerationFactor (AF)** a *float* variable, is used to *accelerate* the *PSAR* towards the price. It starts at *specified input* value from **startAccelerationFactor** and *increases* every time a new *extreme point* (**EP**) is recorded by a *specified input* increment from **incrementAccelerationFactor**, up to a *specified input* maximum from **maximumAccelerationFactor**.
- **extremePoint (EP)** a *float* variable, it is the *highest* high in an *uptrend* or the *lowest* low in a *downtrend*.
- **stopAndReverse (SAR)** a *float* variable, is the *actual value* of the **Parabolic SAR**, which is plotted on the chart.
&nbsp;
```pinescript
var bool isLong = na
var float accelerationFactor = na
var float extremePoint = na
var float stopAndReverse = na
```
&nbsp;

> **CODE LOGIC**:

- At the beginning (**na(stopAndReverse[1])**), **stopAndReverse** is initialized to **close**, **extremePoint** to **high**, **accelerationFactor** to **startAccelerationFactor**, and **isLong** to *true*, assuming an *initial uptrend*.
&nbsp;
```pinescript
if (na(stopAndReverse[1]))
    isLong := true
    accelerationFactor := startAccelerationFactor
    extremePoint := high
    stopAndReverse := close
```
&nbsp;

- If the market was in an uptrend (**isLong[1]**), but the *low price of the current bar is below the previous SAR* (**low < stopAndReverse[1]**), it means the *trend has reversed*. So **isLong** is set to *false*, **accelerationFactor** is *reset* to **startAccelerationFactor**, **extremePoint** is *updated* to the *current low*, and **stopAndReverse** is *updated* to the last **extremePoint**.
&nbsp;
```pinescript
else if (isLong[1])
    if (low < stopAndReverse[1])
        isLong := false
        accelerationFactor := startAccelerationFactor
        extremePoint := low
        stopAndReverse := extremePoint[1]
```
&nbsp;

- If the market was in an *uptrend* and continues to be so, **isLong** remains *true* (**isLong := true**), **stopAndReverse** is *updated* by *adding* the product of the *previous* **accelerationFactor** and the *difference* between the *previous* **extremePoint** and **stopAndReverse** to the *previous* **stopAndReverse** (**stopAndReverse := stopAndReverse[1] + accelerationFactor[1] * (extremePoint[1] - stopAndReverse[1])**).
&nbsp;
```pinescript
else
    isLong := true
    stopAndReverse := stopAndReverse[1] + accelerationFactor[1] * (extremePoint[1] - stopAndReverse[1])
```
&nbsp;

- If a *new high* is *recorded* (**high > extremePoint[1]**), **accelerationFactor** is *increased* by **incrementAccelerationFactor**, up to the **maximumAccelerationFactor** (using **math.min()** function which _returns the smallest of multiple values._) and **extremePoint** is *updated* to the *current high*.
&nbsp;
```pinescript
if (high > extremePoint[1])
    accelerationFactor := math.min(maximumAccelerationFactor, accelerationFactor[1] + incrementAccelerationFactor)
    extremePoint := high
```
&nbsp;

- If *high price* of the *current bar* is *above* the *previous SAR* (**high > stopAndReverse[1]**), it means the trend has *reversed* to an *uptrend*. The **isLong** is set to *true* to *indicate* an *uptrend* (**isLong := true**). The **accelerationFactor** is *reset* to **startAccelerationFactor**, when *trend reversal occurs*, the acceleration factor is *reset* to its starting value (**accelerationFactor := startAccelerationFactor**). The **extremePoint** is *updated* to the *current high*, since it is a uptrend now, it is *tracking* the *highest price* (**extremePoint := high**). And the **stopAndReverse** is *updated* to the *last* **extremePoint**, the *highest price* reached in the *previous downtrend* and it is the *starting point* of the *SAR* in the *new uptrend* (**stopAndReverse := extremePoint[1]**).
&nbsp;
```pinescript
else if (high > stopAndReverse[1])
    isLong := true
    accelerationFactor := startAccelerationFactor
    extremePoint := high
    stopAndReverse := extremePoint[1]
```
&nbsp;

- If the market was in a *downtrend* and *continues* to be, **isLong** remains *false* to *indicate* that the market is still in a *downtrend* (**isLong := false**). The **stopAndReverse** is *updated* by *adding* the product of the *previous* **accelerationFactor** and the *difference* between the *previous* **extremePoint** and **stopAndReverse** to the *previous* **stopAndReverse**, to which moves the *SAR* closer to the price (**stopAndReverse := stopAndReverse[1] + accelerationFactor[1] * (extremePoint[1] - stopAndReverse[1])**).
&nbsp;
```pinescript
else
    isLong := false
    stopAndReverse := stopAndReverse[1] + accelerationFactor[1] * (extremePoint[1] - stopAndReverse[1])
```
&nbsp;

- If a new *low* is *recorded* (**low < extremePoint[1]**), it indicates the *downtrend* is *continuing* strongly. The **accelerationFactor** is *increased* by **incrementAccelerationFactor** (considering the *maximum* of **maximumAccelerationFactor**) (**accelerationFactor := math.min(maximumAccelerationFactor, accelerationFactor[1] + incrementAccelerationFactor)** (using **math.min()** function which _returns the smallest of multiple values._)), and **extremePoint** is updated to the *current low* (**extremePoint := low**). As result it does make the *SAR* more *sensitive* and causes it to *move closer to the price* faster.
&nbsp;
```pinescript
if (low < extremePoint[1])
    accelerationFactor := math.min(maximumAccelerationFactor, accelerationFactor[1] + incrementAccelerationFactor)
    extremePoint := low
```
&nbsp;

- **startAccelerationFactor** is the initial *acceleration factor (AF)*, it defines how *sensitive* the *PSAR* is at the beginning of a new *trend*.
- **incrementAccelerationFactor** is the *increment* for the *AF* each time a new *extreme point (EP)* is recorded, an *EP* is the *highest high* of the *current uptrend* or the *lowest low* of the *current downtrend*. Each time a new *EP* is recorded, the *AF* increases by the *increment* value.
- **maximumAccelerationFactor** is the *maximum* value that the *AF* can reach. Even if new *EPs* continue to be recorded, the *AF* will not increase beyond this maximum value. It acts as a *control* on the *AF* to *prevent* it from becoming too *large*, which would cause the *PSAR* to become too *sensitive* to minor price changes and flip too often.
- **plot()** function plots (*represents*) the *PSAR* on the chart.

&nbsp;

_Observation: The use of **math.min()** it's used to ensure that the **accelerationFactor** does not exceed the **maximumAccelerationFactor**. The acceleration factor (AF) is increased each time a new extreme point is made, but there is usually an upper limit to prevent the AF from becoming too large. So the line **accelerationFactor := math.min(maximumAccelerationFactor, accelerationFactor[1] + incrementAccelerationFactor)** sets **accelerationFactor** to the smaller of either **maximumAccelerationFactor** or **accelerationFactor[1] + incrementAccelerationFactor**_.

_If the calculated new **accelerationFactor** is greater than the **maximumAccelerationFactor**, then the **accelerationFactor** is set to the **maximumAccelerationFactor**. If it's less, then the **accelerationFactor** is set to **accelerationFactor[1] + incrementAccelerationFactor**_.

_This prevents the AF from becoming too large, which would make the **Parabolic SAR** overly sensitive to price changes, causing it to flip direction too frequently. The **math.min()** function is used to implement this upper limit or control on the AF_.

_This code is a basic implementation of an PSAR, but it's worth noting that built-in function for calculating PSAR is available in Pine Script, which is optimized and easier to use_.

_For PSAR, can be used ta.sar(start, inc, max)_.
___
___
&nbsp;
# Indicators
&nbsp;

## Relative Strength Index (RSI)

The **Relative Strength Index** (RSI) is a *momentum oscillator* that measures the speed and change of price movements. Can be used to identify **overbought** and/or **oversold** conditions in a market.

File Script: [Relative Strength Index](rsi.pine)

- **lengthPeriod** creates an *input* in the settings of the indicator that allows you to set the *length* of the RSI calculation *period*. Default is set to 14.
- **source** defines the source of the data for the RSI calculation as the *closing price* of each *period*. Setting the **source** as the **close** variable from pinescript.
- **differenceSource** calculates the *change* in *price* from one period to the next. Using the **ta.change()** function from pinescript, which _compares the current source value to its value length bars ago and returns the difference_.
- **gain** and **loss** it calculates the *gain* if the price increased (*differenceSource* > 0), then *gain* is equal to *differenceSource*, if the price decreased or stayed the same, then *gain* is 0. And calculates the *loss* if the price decreased (differenceSource < 0), then *loss* is the absolute value of *differenceSource*, if the price increased or stayed the same, then *loss* is 0. Using **na()** function which _returns true if something is na, false otherwise._ Using **math.max()** function which _returns the greatest of multiple values._ And using **math.min()** function which _returns the smallest of multiple values._
- **averageGain** it calculates the *average gain* over the specified period (**lengthPeriod**) and **averageLoss** calculates the *average loss* over the specified period (**lengthPeriod**). Using the **ta.rma()** function which is a _moving average used in RSI. It is the exponentially weighted moving average with alpha = 1 / length. Returns exponential moving average of source with alpha = 1 / length._
- **relativeStrength** calculates the *relative strength* (RS), which is the ratio of the *average gain* to the *average loss*.
- **relativeStrengthIndex** it calculates the *RSI*, which is _(100 - (100 / (1 + **relativeStrength**)))_.
- **plot()** function plots (*represents*) the *RSI* on the chart.
- **hline()** function plots (*represents*) a horizontal line at the level *specified*.

_Observation: The reason to use the minus symbol (-) at the line where is setting the loss variable (-math.min(differenceSource, 0)) is to flip the negative price change to a positive value, so will give the magnitude of the price drop as a positive number, or 0 if the price did not drop, this value is then used to calculate the average loss for the RSI calculation._
_It may be used the minus symbol (-) for setting the loss variable or the gain variable (-math.min(differenceSource, 0)) or (-math.max(differenceSource, 0)), but one of them has to have the minus symbol for negative price flip change to positive to calculate the average for the RSI calculation._
___
___
&nbsp;

## Exponential Moving Average (EMA)

The **Exponential Moving Average** (EMA) is a *trend following*, when the price of a asset is above the EMA, it could indicate an *upward* trend, while prices below of the asset it could indicate a *downward* trend.

File Script: [Exponential Moving Average](ema.pine)

- **lengthPeriod** creates an *input* in the settings of the indicator that allows you to set the *length* of the EMA calculation *period*. Default is set to 21.
- **source** defines the source of the data for the EMA calculation as the *closing price* of each *period*. Setting the **source** as the **close** variable from pinescript.
- **alpha** calculates the *weighting* multiplier for EMA.
- **sma** calculates the *SMA* for the *first length bars*.
- **ema** it calculates the *EMA*. If the *previous* EMA is not available, it uses the *manually* calculated *SMA* for the *first* EMA value. For *subsequent* periods, it uses the EMA *formula*.
- **plot()** function plots (*represents*) the *EMA* on the chart.

_Observation: The common practice is to use the Simple Moving Average (SMA) for the first value. The reason is that the SMA is a straightforward average calculation that equally weights all data points in its calculation period and by using the SMA as the initial value, it ensures that the EMA is not overly influenced by the initial seed value but instead reflects the average market behavior over the lookback period._
___
___
&nbsp;

## Parabolic Stop and Reverse (PSAR)

The **Parabolic Stop and Reverse** (PSAR) is a *trend following*, it aims to *identify* the *direction* of a *trend* and potentially providing *entry* and *exit* points. *PSAR* move with *price changes* and are used to *identify* and *follow* the *direction* of a *trend*.

File Script: [Parabolic Stop and Reverse](psar.pine)

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
___
___
&nbsp;

## Stochastic Slow (Stoch Slow)

The **Stochastic Slow** (Stoch Slow) is a *momentum oscillator*, it compares a *particular* closing price of a security to a *range* of its prices over a *certain period* of time. The *sensitivity* of the indicator is reducible by adjusting *time period* or by taking a *moving average* of the *result*.

File Script: [Stochastic Slow](stochastic_slow.pine)

- **smaCalculation** is a *function* for *simple moving average* (**SMA**). It *sums* up the last *length* bars of *source* and then *divides* by *length* to *calculate* the *average*. In the *context* of the *stochastic*, the *source* would be the *%K* values and the len*gth would be the *smoothing* period.
- **lengthPeriod**, **smoothK**, and **smoothD** are *input* (int) *parameters*, which represents the *calculation period* for the *stochastic* and the *smoothing periods* for the **%K** and **%D** lines.
- **lowestLow** and **highestHigh** representS the *lowest low* and *highest high* over the **lengthPeriod**. Using the **ta.lowest()** function to return _lowest value for a given number of bars back_, and **ta.highest()** function to return _highest value in the series_, both from pinescript.
- **K** uses the SMA function **smaCalculation** to calculate the *%K* line by taking the *close* of the *current* bar, *subtracting* the *lowest low*, and *dividing* by the *range* (*highest high - lowest low*), the result is *multiplied* by 100 and then *smoothed* using the *SMA function* with **smoothK** as the *smoothing period*.
- **D** is a SMA of the *K* line *over* the **smoothD** *period*.
- **plot()** function plots (*represents*) the *PSAR* on the chart.
- **hline()** function _renders a horizontal line at a given fixed price level_, in this case the *levels* used in this indicator is to identify *overbought* and *oversold* conditions.
- **fill()** function _fills background between two horizontal lines, two plots or hlines with a given color_, in this case fills the area *between* the *upper* and *lower* horizontal lines.

&nbsp;

_Observation: This Stochastic indicator smoothes **%K** and **%D** calculating %K as a moving average and %D as a moving average of %K_.

_%K is the **main** or **fast line** based on the current price level relative to the highest high and the lowest low over a specified period_.

_%D is the **slow line** or **signal line**, it is a **SMA** of the %K values over a certain period_.
___
___
&nbsp;


### references:

- Pinescript Documentation.
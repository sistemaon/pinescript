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
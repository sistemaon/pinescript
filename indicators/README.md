# Indicators

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

## Exponential Moving Average (EMA)

The **Exponential Moving Average** (EMA) is a *trend following*, when the price of a asset is above the EMA, it could indicate an *upward* trend, while prices below of the asset it could indicate a *downward* trend.

File Script: [Exponential Moving Average](ema.pine)

- **lengthPeriod** creates an *input* in the settings of the indicator that allows you to set the *length* of the EMA calculation *period*. Default is set to 21.
- **source** defines the source of the data for the EMA calculation as the *closing price* of each *period*. Setting the **source** as the **close** variable from pinescript.
- **alpha** calculates the *weighting* multiplier for EMA.
- **sma** calculates the *SMA* for the *first length bars*.
- **ema** it calculates the *EMA*. If the *previous* EMA is not available, it uses the *manually* calculated *SMA* for the *first* EMA value. For *subsequent* periods, it uses the EMA *formula*.
- **plot()** function plots (*represents*) the *EMA* on the chart.
___
___


### references:

- Pinescript Documentation.
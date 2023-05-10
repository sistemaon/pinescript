# Indicators

## Relative Strength Index 

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
___
___


### references:

- Pinescript Documentation.
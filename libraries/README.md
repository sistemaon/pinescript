# Library

## Uptrend Downtrend Loopback Candle Identification Lib

file script: [Uptrend Downtrend Loopback Candle Identification Lib](uptrend_downtrend_loopback_candle_identification_lib.pine)

link script: [Uptrend Downtrend Loopback Candle Identification Lib](https://www.tradingview.com/v/7XVVW2CK/)

### uptrendLoopbackCandleIdentification:
- The function starts by declaring a variable *isUptrend* and initializes it as *false*. This variable will be used to store the result of the loop analysis.
- A *for loop* is used to iterate through the specified *lookbackPeriod* with the loop index variable *i*.
- Inside the loop, the code checks if the *low* of the current bar (low) is greater than or equal to the *low* of the bar *i* bars ago (low).
- If the condition is met, the variable *isUptrend* is set to *true*.
- After the loop has finished, the function returns the final value of *isUptrend*.

### downtrendLoopbackCandleIdentification:
- The function starts by declaring a variable *isDowntrend* and initializes it as *false*. This variable will be used to store the result of the loop analysis.
- A *for loop* is used to iterate through the specified *lookbackPeriod* with the loop index variable *i*.
- Inside the loop, the code checks if the *low* of the current bar (low) is less than or equal to the *low* of the bar *i* bars ago (low).
- If the condition is met, the variable *isDowntrend* is set to *true*.
- After the loop has finished, the function returns the final value of *isDowntrend*.

_Each function will return true if the respective condition is met for any bar within the lookback period. The loops will not terminate early if the conditions are met._
___
___


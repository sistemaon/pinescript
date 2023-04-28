# Candlestick Patterns

## Spinning Top
The Spinning Top is a candlestick pattern representing market indecision. This pattern occurs when the real body (the difference between the open and close prices) of the candle is small compared to its overall range (the difference between the high and low prices). The Spinning Top pattern can be either bullish or bearish, depending on the direction of the real body.
They have small upper and lower shadows, but the size of the shadows are not important.
It is the diminutive size of the real body that makes this a spinning top.

file script: [spinning_top_candlestick_pattern.pine](spinning_top_candlestick_pattern.pine)

*maxSpinningTopThreshold* and *minSpinningTopThreshold* are input variables that allow you to set the maximum and minimum body size thresholds as a percentage of the overall range of the candle. The default values are 0.55 (55%) for the max threshold and 0.21 (21%) for the min threshold.
*showBullishSpinningTop* and *showBearishSpinningTop* are input boolean variables that allow you to toggle the display of bullish and bearish Spinning Top patterns on the chart.
*isBullishCandle* and *isBearishCandle* are variables that check if the current candle is bullish (close > open) or bearish (close < open).
*isSpinningTop* checks if the current candle is a Spinning Top by comparing the size of the real body to the min and max thresholds. If the size of the real body is less than the max threshold and greater than the min threshold, the candle is considered a Spinning Top.
*isSpinningTopBullish* and *isSpinningTopBearish* are variables that determine if the Spinning Top is bullish or bearish based on the direction of the candle and the positions of the high and low prices.
*barcolor()* function is used to color the Spinning Top candles white for bullish and black for bearish.
*plotshape()* functions are used to display an arrow below or above the Spinning Top candles, depending on whether they are bullish or bearish. The arrows are labeled "Spinning Top" and are colored green for bullish Spinning Tops and red for bearish Spinning Tops.


## Doji
The Doji candlestick pattern represents market indecision, and it occurs when the open and close prices of a candle are the same or very close to each other. Doji reveals no real bodies. They have horizontal lines. A doji occurs when the open and close for that session are the same or very close to being the same. The lengths of the shadows can vary.

file script: [doji.pine](doji.pine)

*dojiThreshold* is an input variable that allows you to set the maximum allowed difference between the open and close prices as a fraction of the price. The default value is 0.0005 or 0.05%.
*showBullishDoji* and *showBearishDoji* are input boolean variables that allow you to toggle the display of bullish and bearish Doji patterns on the chart.
*isBullishCandle* and isBearishCandle are variables that check if the current candle is bullish (close > open) or bearish (close < open).
*isDoji* checks if the absolute difference between the close and open prices is less than or equal to the dojiThreshold multiplied by the close price. If the condition is met, it means the current candle is a Doji.
*isDojiBullish* and isDojiBearish are variables that determine if the Doji is bullish or bearish based on the direction of the candle.
*barcolor()* function is used to color the Doji candles white for bullish and black for bearish.
*plotshape()* functions are used to display an arrow below or above the Doji candles, depending on whether they are bullish or bearish. The arrows are labeled "Doji" and are colored green for bullish Dojis and red for bearish Dojis.


### references:
- Japanese Candlestick Charting Techniques.
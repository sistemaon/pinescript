
# Candlestick Patterns
&nbsp;

## Spinning Top

The Spinning Top is a candlestick pattern representing market indecision.
This pattern occurs when the real body (the difference between the open and close prices) of the candle is small compared to its overall range (the difference between the high and low prices).
The Spinning Top pattern can be either bullish or bearish, depending on the direction of the real body.
They have small upper and lower shadows, but the size of the shadows are not important.
It is the diminutive size of the real body that makes this a spinning top.

File Script: [Spinning Top Candlestick Pattern](spinning_top_candlestick_pattern.pine)

- **maxSpinningTopThreshold** and **minSpinningTopThreshold** are input variables that allow you to set the maximum and minimum body size thresholds as a percentage of the overall range of the candle. The default values are 0.55 (55%) for the max threshold and 0.21 (21%) for the min threshold.
- **showBullishSpinningTop** and **showBearishSpinningTop** are input boolean variables that allow you to toggle the display of bullish and bearish Spinning Top patterns on the chart.
- **isBullishCandle** and **isBearishCandle** are variables that check if the current candle is bullish (close > open) or bearish (close < open).
- **isSpinningTop** checks if the current candle is a Spinning Top by comparing the size of the real body to the min and max thresholds. If the size of the real body is less than the max threshold and greater than the min threshold, the candle is considered a Spinning Top.
- **isSpinningTopBullish** and **isSpinningTopBearish** are variables that determine if the Spinning Top is bullish or bearish based on the direction of the candle and the positions of the high and low prices.
- **barcolor()** function is used to color the Spinning Top candles white for bullish and black for bearish.
- **plotshape()** functions are used to display an arrow below or above the Spinning Top candles, depending on whether they are bullish or bearish. The arrows are labeled "Spinning Top" and are colored green for bullish Spinning Tops and red for bearish Spinning Tops.
___
___
&nbsp;

## Doji

The Doji candlestick pattern represents market indecision, and it occurs when the open and close prices of a candle are the same or very close to each other.
Doji reveals no real bodies.
They have horizontal lines.
A doji occurs when the open and close for that session are the same or very close to being the same.
The lengths of the shadows can vary.

File Script: [Doji Candlestick Pattern](doji.pine)

- **dojiThreshold** is an input variable that allows you to set the maximum allowed difference between the open and close prices as a fraction of the price. The default value is 0.0005 or 0.05%.
- **showBullishDoji** and **showBearishDoji** are input boolean variables that allow you to toggle the display of bullish and bearish Doji patterns on the chart.
- **isBullishCandle** and isBearishCandle are variables that check if the current candle is bullish (close > open) or bearish (close < open).
- **isDoji** checks if the absolute difference between the close and open prices is less than or equal to the dojiThreshold multiplied by the close price. If the condition is met, it means the current candle is a Doji.
- **isDojiBullish** and isDojiBearish are variables that determine if the Doji is bullish or bearish based on the direction of the candle.
- **barcolor()** function is used to color the Doji candles white for bullish and black for bearish.
- **plotshape()** functions are used to display an arrow below or above the Doji candles, depending on whether they are bullish or bearish. The arrows are labeled "Doji" and are colored green for bullish Dojis and red for bearish Dojis.
___
___
&nbsp;

## Hammer and Hanging Man

The Hammer is a bullish candlestick pattern that occurs during a downtrend. It signals a potential reversal or the end of the downtrend. 
The Hanging Man is a bearish candlestick pattern that occurs during an uptrend. It signals a potential reversal or the end of the uptrend.

File Script: [Hammer and Hanging Man Candlestick Pattern](hammer_hanging_man.pine)

- The script imports the **uptrend_downtrend_loopback_candle_identification_lib** library, which contains functions to identify uptrends and downtrends based on a specified lookback period.
- The real body of the Hammer pattern should be small relative to the overall range of the candle. In the code, this is checked using the **isSmallRealBody** variable. The **hammerHangingManBodySizeThreshold** input parameter determines the maximum body size as a percentage of the overall range.
- The lower shadow of the Hammer pattern should be longer than the real body. In the code, this is checked using the **isLongLowerShadow** variable. The **hammerHangingManLowerShadowMultiplier** input parameter determines the minimum lower shadow length as a multiple of the real body size.
- The Hammer pattern should have no, or a very short, upper shadow. In the code, this is checked using the **isShortUpperShadow** variable. The **hammerHangingManUpperShadowThreshold** input parameter determines the maximum upper shadow length as a percentage of the overall range.
- If these conditions are met and the current candle is bullish **isBullishCandle**, the script identifies the pattern as a Hammer and plots an arrow below the candle labeled "Hammer" in green color. 
Or If these conditions are met and the current candle is bearish **isBearishCandle**, the script identifies the pattern as a Hanging Man and plots an arrow above the candle labeled "Hanging Man" in red color.
- The **isHammer** and **isHangingMan** are boolean variables that store the conditions for identifying a Hammer or a Hanging Man pattern.
- They also include additional conditions to ensure the current candle's low is lower or higher than the previous candles for Hammer and for Hanging Man patterns thru the **lookbackPeriod** to identify uptrend/downtrend analysis from candles.
- The **showHammer** and **showHangingMan** input boolean variables allow you to toggle the display of Hammer and Hanging Man patterns on the chart, respectively.
- The **barcolor()** function is used to color the Hammer candles white and the Hanging Man candles black.
- The **plotshape()** function is used to display an arrow below or above the identified Hammer or Hanging Man pattern, with the text "Hammer" or "Hanging Man" and their respective colors (green for Hammer and red for Hanging Man).
___
___
&nbsp;

## Engulfing Bullish/Bearish

Bullish Engulfing occurs in a downtrend when a small bearish (red) candle is followed by a larger bullish (green) candle that engulfs the body of the previous candle. This pattern indicates the possibility of a bullish trend reversal.

Bearish Engulfing occurs in an uptrend when a small bullish (green) candle is followed by a larger bearish (red) candle that engulfs the body of the previous candle. This pattern indicates the possibility of a bearish trend reversal.

File Script: [Engulfing Bullish/Bearish Candlestick Pattern](engulfing.pine)

- The **showBullishEngulfing** and **showBearishEngulfing** are boolean inputs to choose whether to display bullish engulfing patterns and/or bearish engulfing patterns.
- The **isBullishCandle** and **isBearishCandle** are boolean variables to identify if the current candle is bullish (close > open) and if the current candle is bearish (open > close).
- The **isPreviousBullishCandle** and **isPreviousBearishCandle** are boolean variables to identify if the previous candle is bullish (close[1] > open[1]) and if the previous candle is bearish (open[1] > close[1]).
- **isShadowEngulfing** a boolean variable to check if the current candle's low is lower than the previous candle's low and the current candle's high is higher than the previous candle's high.
- **isBullishBodyEngulfing** a boolean variable to check if the current candle engulfs the previous candle's body in a bullish manner (current open <= previous close, and current close >= previous open).
- **isBearishBodyEngulfing** a boolean variable to check if the current candle engulfs the previous candle's body in a bearish manner (current open >= previous close, and current close <= previous open).
- **isBullishEngulfing** a boolean variable to identify bullish engulfing patterns by combining the engulfing criteria with the previous and current candle types.
- **isBearishEngulfing** a boolean variable to identify bearish engulfing patterns by combining the engulfing criteria with the previous and current candle types.
- The **barcolor()** function is used to color the bars white if there's a bullish engulfing pattern or black if there's a bearish engulfing pattern.
- The **plotshape()** function is used to display an arrow and text for bullish engulfing and for bearish engulfing patterns if **showBullishEngulfing** and/or **showBearishEngulfing** are true.

_This indicator helps to visually and identify bullish and bearish engulfing patterns, which are often used as potential trend reversal signals in technical analysis._
___
___
&nbsp;

## Morning Star

The Morning Star pattern is a bullish reversal candlestick pattern that indicates a potential turnaround from a downtrend to an uptrend.

It consists of three candles. A long bearish candle, which represents a continuation of the downtrend. A small candle or doji, indicating indecision or weakening of the selling pressure. A long bullish candle, showing a shift in momentum from sellers to buyers, closing above the midpoint of the first candle.

File Script: [Morning Start Candlestick Pattern](morning_star.pine)

- The **showMorningStar** is a boolean, flag to show or hide the Morning Star pattern.
- The **dojiThreshold** and **weakCandleThreshold** are float, the threshold for identifying a doji candle (0.1 = 10%) and the threshold for identifying a weak candle (0.3 = 30%).
- The **isDoji** identifies if a candle is a doji based on the doji threshold, and **isWeakCandle** identifies if a candle is weak based on the weak candle threshold.
- **isBullishCandle** identifies if a candle is bullish (close > open) and **isBearishCandle** identifies if a candle is bearish (open > close).
- **isConditionFirstCandle** the *first candle* should be a *bearish candle*, and its open should be greater than the high of the *second candle* and should not be *isDoji* and should not be *isWeakCandle*.
- **isConditionSecondCandle** the *second candle* should be a *isDoji* or *isWeakCandle*.
- **isConditionThirdCandle** the *third candle* should have a close price higher than the high of the *second candle* and should not be *isDoji* and should not be *isWeakCandle*.
- **isMorningStar** *true* if all *three conditions* are met, *false* otherwise.
- The **barcolor()** function is used to color the bars, the *third candle* is colored white, *second candle* is colored white if *bullish* or black if *bearish*, *first candle* is colored black, colors are applied only when the **isMorningStar** condition is *true*.
- The **plotshape()** function is used to display an arrow and label below the *Morning Star* pattern if it is detected and if **showMorningStar** input is *true*, the label reads **Morning Star** with a green text color.

_This indicator is for detecting the Morning Star pattern in candlestick charts. The Morning Star pattern is a bullish reversal pattern that occurs at the end of a downtrend. It consists of three candles: a bearish candle, a small candle or doji, and a bullish candle. The pattern suggests that after a downtrend, buyers have started to take control, potentially leading to a change in trend direction._
___
___
&nbsp;

## Evening Star

The Evening Star pattern is a bearish reversal candlestick pattern that indicates a potential turnaround from a uptrend to an downtrend.

It consists of three candles. A large bullish candle, indicating an uptrend. A small or doji candle, either bullish or bearish, above the first candle's close, indicating indecision. A large bearish candle, closing below the midpoint of the first candle, indicating a potential trend reversal.

File Script: [Evening Start Candlestick Pattern](evening_star.pine)

- The **showEveningStar** is a boolean, flag to show or hide the Evening Star pattern.
- The **dojiThreshold** and **weakCandleThreshold** are float, the threshold for identifying a doji candle (0.1 = 10%) and the threshold for identifying a weak candle (0.3 = 30%).
- The **isDoji** identifies if a candle is a doji based on the doji threshold, and **isWeakCandle** identifies if a candle is weak based on the weak candle threshold.
- **isBullishCandle** identifies if a candle is bullish (close > open) and **isBearishCandle** identifies if a candle is bearish (open > close).
- **isConditionFirstCandle** the *first candle* should be a *bullish candle*, and its low should be greater than the open of the *second candle* and should not be *isDoji* and should not be *isWeakCandle*.
- **isConditionSecondCandle** the *second candle* should be a *isDoji* or *isWeakCandle*.
- **isConditionThirdCandle** the *third candle* should have a close price lower than the low of the *second candle* and should not be *isDoji* and should not be *isWeakCandle*.
- **isEveningStar** *true* if all *three conditions* are met, *false* otherwise.
- The **barcolor()** function is used to color the bars, the *third candle* is colored black, *second candle* is colored white if *bullish* or black if *bearish*, *first candle* is colored white, colors are applied only when the **isEveningStar** condition is *true*.
- The **plotshape()** function is used to display an arrow and label above the *Evening Star* pattern if it is detected and if **showEveningStar** input is *true*, the label reads **Evening Star** with a red text color.

_This indicator is for detecting the Evening Star pattern in candlestick charts. The Evening Star pattern is a bearish reversal pattern that occurs at the end of a uptrend. It consists of three candles: a bullish candle, a small candle or doji, and a bearish candle. The pattern suggests that after uptrend, sellers have started to take control, potentially leading to a change in trend direction._
___
___
&nbsp;

## Shooting Star and Inverted Hammer

> ### Shooting Star

The Shooting Star pattern is a bearish reversal pattern that signals a potential change in trend. This pattern is identified by a small real body near the top of the price range and a long upper shadow that is at least twice the size of the real body. Additionally, there should be little or no lower shadow.

File Script: [Shooting Start Candlestick Pattern](shooting_star_inverted_hammer.pine)

- The **showShootingStar** is a boolean, option allows the user to show or hide the Shooting Star pattern on the chart.
- The **weakCandleThreshold** is a float, defines the threshold for what is considered a *weak candle*. The default value is 0.45, which means a candle with a body that is less than 45% of the total price range is considered weak.
- The **upperShadowThreshold** is a float, defines the threshold for the size of the upper shadow. The default value is 1, which means the upper shadow must be at least as *long as the body of the candle*.
- **noLowerShadowThreshold** is a float, defines the threshold for the size of the lower shadow. The default value is 0.05, which means there should be *little or no lower shadow*.
- **isBullishCandle** identifies if a candle is bullish (close > open) and **isBearishCandle** identifies if a candle is bearish (open > close).
- **isWeakCandle** checks whether the candle is considered weak based on the customizable weak candle threshold.
- **isLongUpperShadow** checks whether the upper shadow is at least as long as the body of the candle multiplied by the customizable upper shadow threshold.
- **isNoLowerShadow** checks whether there is little or no lower shadow based on the customizable no/lower shadow threshold.
- **isShootingStar** *true* if all *three conditions* are met, *false* otherwise.
- The **barcolor()** function is used to color the bars, the *candle* is colored black if *isShootingStar* and *isBearishCandle* met criteria, or the *candle* is colored white if *isShootingStar* and *isBullishCandle* criteria meet.
- The **plotshape()** function is used to display an arrow and label above the *Shooting Star* pattern if it is detected and if **showShootingStar** input is *true*, the label reads **Shooting Star** with a red text color.

_The script identifies the Shooting Star pattern based on the customizable parameters and highlight the pattern on the chart with an arrow and text. The pattern indicates that the buyers attempted to push the price up but failed, and the sellers took control to push the price down, resulting in a bearish reversal_.

&nbsp;

> ### Inverted Hammer

The Inverted Hammer pattern is a bullish reversal pattern that signals a potential change in trend. This pattern is identified by a small real body near the bottom of the price range and a long lower shadow that is at least twice the size of the real body. Additionally, there should be little or no upper shadow.

File Script: [Inverted Hammer Candlestick Pattern](shooting_star_inverted_hammer.pine)

- The **showInvertedHammer** is a boolean, option allows the user to show or hide the Inverted Hammer pattern on the chart.
- The **weakCandleThreshold** is a float, defines the threshold for what is considered a *weak candle*. The default value is 0.45, which means a candle with a body that is less than 45% of the total price range is considered weak.
- The **upperShadowThreshold** is a float, defines the threshold for the size of the upper shadow. The default value is 1, which means there should be *little or no upper shadow*.
- **noLowerShadowThreshold** is a float, defines the threshold for the size of the lower shadow. The default value is 0.05, which means the lower shadow must be at least as long as the body of the candle multiplied by the customizable no/lower shadow threshold.
- **isBullishCandle** identifies if a candle is *bullish* (close > open) and **isBearishCandle** identifies if a candle is *bearish* (open > close).
- **isWeakCandle** checks whether the candle is considered weak based on the customizable weak candle threshold.
- **isLongLowerShadow** checks whether the lower shadow is at least as long as the body of the candle multiplied by the customizable lower shadow threshold.
- **isNoUpperShadow** checks whether there is *little or no upper shadow* based on the customizable upper shadow threshold.
- **isInvertedHammer** *true* if all *three conditions* are met, *false* otherwise.
- The **barcolor()** function is used to color the *bars*, the *candle* is colored black if *isInvertedHammer* and *isBearishCandle* criteria are met, or the candle is colored white if *isInvertedHammer* and *isBullishCandle* criteria are met.
- The **plotshape()** function is used to display an arrow and label below the *Inverted Hammer* pattern if it is detected and if **showInvertedHammer** input is *true*, the label reads *Inverted Hammer* with a green text color.

_The script identifies the Inverted Hammer pattern based on the customizable parameters and highlights the pattern on the chart with an arrow and text. The pattern indicates that the sellers attempted to push the price down but failed, and the buyers took control to push the price up, resulting in a bullish reversal._
___
___
&nbsp;

## Harami

The Harami is a two-candle pattern that signals a potential trend reversal. It is a Japanese term that means "pregnant", which describes the shape of the pattern.

The pattern consists of a large candle followed by a smaller candle that is completely contained within the range of the previous candle.

File Script: [Harami](harami.pine)

- The **showBullishHarami** and **showBearishHarami** are boolean, which allow the user to choose whether to display or hide the bullish or bearish Harami pattern.
- **lookbackPeriod** input is an integer (int), which represents the number of periods to look back for the purpose of identifying the current *trend* from candles.
- The **dojiThreshold**, **weakCandleThreshold** and **bodyThreshold** are float, the threshold for identifying a *doji* candle (0.1 = 10%), the threshold for identifying a *weak* candle (0.3 = 30%) and the threshold for identifying a long *body* candle (0.4 = 40%).
- The **isDoji** identifies if a candle is a doji based on the *doji threshold*, **isWeakCandle** identifies if a candle is weak based on the *weak candle threshold*, and **isLongCandle** identifies if a candle body is long based on the *body threshold*.
- **isBullishCandle** identifies if a candle is bullish (close > open) and **isBearishCandle** identifies if a candle is bearish (open > close).
- The **isLookbackUptrendCandle** and **isLookbackDowntrendCandle** variables identify the current candle's trend based on the *lookback period* from *candles*.
- The **isFirstCandleCondition** variable identifies whether the first candle meets the condition of a *long* body candle.
- The **isCurrentCandleCondition** variable identifies whether the current candle meets the condition of a *doji* or a *weak* candle.
- The **isBullishHarami** and **isBearishHarami** variables are used to check whether the *Harami* pattern has formed based on the *two conditions* met for *bullish* or *bearish* patterns.
- The **barcolor()** function is used to color the bars depending on the *bullish* or *bearish* pattern identified.
- The **plotshape()** function is used to display an arrow and label above or below the *Harami* pattern if it is detected, depending on whether it is a *bullish* or *bearish* pattern, and if the user has *enabled* display of the pattern.

_This indicator is for detecting the Harami pattern in candlestick charts. A bullish Harami pattern forms when a large bearish candle is followed by a smaller bullish candle that is completely contained within the range of the previous candle. This pattern suggests that the selling pressure is weakening, and a reversal to the upside may occur. A bearish Harami pattern forms when a large bullish candle is followed by a smaller bearish candle that is completely contained within the range of the previous candle. This pattern suggests that the buying pressure is weakening, and a reversal to the downside may occur._
___
___
&nbsp;

## Tweezer Top Bottom

The Tweezers are candlestick patterns that consist of two or more candlesticks with the same high or low price level, which are seen as a potential reversal signal.

The Tweezer Top is formed when two or more candlesticks have the same high price. Tweezer Bottom is formed when two or more candlesticks have the same low price.

File Script: [Tweezer Top/Bottom](tweezer_top_bottom.pine)

- The **showTweezerTop** and **showTweezerBottom** are boolean, which allow the user to choose whether to display or hide the bullish or bearish Tweezers pattern.
- **lookbackPeriod** input is an integer (int), which represents the number of periods to look back for the purpose of identifying the current *trend* from candles.
- The **dojiThreshold** and **weakCandleThreshold** are float, the threshold for identifying a *doji* candle (0.1 = 10%), the threshold for identifying a *weak* candle (0.3 = 30%).
- The **isDoji** identifies if a candle is a doji based on the *doji threshold* and **isWeakCandle** identifies if a candle is weak based on the *weak candle threshold*.
- **isBullishCandle** identifies if a candle is bullish (close > open) and **isBearishCandle** identifies if a candle is bearish (open > close).
- The **isLookbackUptrendCandle** and **isLookbackDowntrendCandle** variables identify the current candle's trend based on the *lookback period* from *candles*.
- **tweezerTopConditionCandle** and **twezzerBottomConditionCandle** check whether the *previous candle's high or low* matches the *current candle's high or low* respectively.
- **firstConditionCandle** checks that the *current* and *previous* candles are not *doji* or *weak* candles.
- **isTwezzerTop** and **isTwezzerBottom** variables combine the conditions to determine whether the *current* candle is a *Tweezer Top* or *Bottom* pattern.
- The **barcolor()** function is used to color the bars depending on the *bullish* or *bearish* pattern identified.
- The **plotshape()** function is used to display an arrow and label above or below the *Tweezers* pattern if it is detected, depending on whether it is a *bullish* or *bearish* pattern, and if the user has *enabled* display of the pattern.

_This indicator is for detecting the Tweezers pattern in candlestick charts. A Tweezer Top is a bearish reversal pattern that typically forms during an uptrend. The Tweezer Bottom is a bullish reversal pattern that usually appears during a downtrend. It can indicate that the trend may be reversing._
___
___
&nbsp;

### references:

- Japanese Candlestick Charting Techniques.
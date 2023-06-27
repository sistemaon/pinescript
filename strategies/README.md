# Strategies

## Simple Alert Strategy
* The strategy will alert you when the price crosses above or below a moving average.
* The sole purpose of this strategy is to test the alert system as fast as possible.

File Script: [Simple Alert Strategy](simple_alert_strategy.pine)

---
---

## Bollinger Bands Modified (Stormer) Strategy

> This **Strategy** is **based** on the **Setup** given by **Alexandre Wolwacz**, AKA **Stormer**.\
> **Stormer** is a professional trader and investor from Brazil.\
> More about **Stormer**'s work: [Stormer YT](https://www.youtube.com/@StormerOficial).\
> More about the **Setup**: [How to use bollinger bands to operate cryptocurrencies .P1](https://www.youtube.com/watch?v=iP9Iu6AVCNk) and [How to use bollinger bands to operate cryptocurrencies .P2](https://www.youtube.com/watch?v=1P9-NZu1wVI).

> Observations: This _strategy_ executes both long and short positions using _Bollinger Bands and Exponential Moving Average (EMA)_, or using _Bollinger Bands_ alone.\
The **detection** and **display** of the _inside bar pattern_ on the chart is optional, it **does not** impact the trades themselves.\
Please **read** the explanation carefully to better understand what this strategy **does**.


#### Strategy Settings
* Length of Bollinger Bands (**bbL**): Determines the **length** of the **Bollinger Bands**. _Default value is 20_.
* Standard Deviation of Bollinger Bands (**mult**): Sets the **standard deviation** of the **Bollinger Bands**. _Default value is 0.38_.
* Length of Exponential Moving Average (**emaL**): Specifies the **length** of the **Exponential Moving Average**. _Default value is 80_.
* Length of Highest High (**highestHighL**): Defines the **length** used to fetch the **highest high** candle. _Default value is 7_.
* Length of Lowest Low (**lowestLowL**): Defines the **length** used to fetch the **lowest low** candle. _Default value is 7_.
* Target Take Profit Factor (**targetFactor**): Determines the **factor** to _calculate_ the **take profit** level. _Default value is 1.6_.
* Check Trend EMA (**emaTrend**): If enabled, uses **EMA** as a **trend** _verification_ for opening positions. _Default value is true_.
* Add Another Crossover Check (**crossoverCheck**): If enabled, adds an **additional** _verification_ for price **crossing above** the **upper** Bollinger Band. _Default value is false_.
* Add Another Crossunder Check (**crossunderCheck**): If enabled, adds an **additional** _verification_ for price **crossing below** the **lower** Bollinger Band. _Default value is false_.
* Show Inside Bar Pattern (**insideBarPatternCheck**): If enabled, **displays** the **inside bar** pattern. _Default value is true_.

#### Calculation and Condition Variables
* middle, upper, lower: Calculates the Bollinger Bands based on the input parameters and returns **middle band**, **upper band** and **lower band**.
* ema: Calculates the **Exponential Moving Average** based on the input length.
* highestHigh: Fetches the **highest high candle** based on the specified length.
* lowestLow: Fetches the **lowest low candle** based on the specified length.
* isCrossover, isCrossunder: Checks if the price **crosses above** the _upper Bollinger Band_ or **below** the _lower Bollinger Band_.
* isPrevBarHighGreaterCurBarHigh, isPrevBarLowLesserCurBarLow: Compares the **previous** _bar's high_ with the **current** _bar's high_ and the **previous** _bar's low_ with the **current** _bar's low_, respectively.
* isInsideBar: Checks if the **current** bar forms an **inside bar** pattern.
* isBarLong, isBarShort: Identifies if the **current** bar is a **long** or **short** bar based on the **close** and **open** prices.
* isLongCross, isShortCross: Determines if the **conditions** for **entering** a **long** or **short** _position_ are met.
* isCandleAboveEma, isCandleBelowEma: Checks if the **current candle** is **above** or **below** the _Exponential Moving Average_.
* isLongCondition, isShortCondition: **Combines the condition** _variables_ to determine if it's suitable to **enter** a _long_ or _short_ position.

#### Position Entry and Exit
* If there is **no current** position and the conditions for a **long position** are met, the strategy **enters** a _long position_.
* If there is **no current** position and the conditions for a **short position** are met, the strategy **enters** a _short position_.
* The strategy sets the **stop loss** and **take profit** levels for the _entered positions_.
* The strategy also _sends an alert_ with the **entry**, **stop loss**, and **target** levels.
* The **Bollinger Bands**, **Exponential Moving Average**, and **inside bar** pattern are _plotted_ on the chart.

#### Position Management
* isPositionNone, isPositionLong, isPositionShort: Checks the **current** position **status**.
* enterLong, stopLossLong, targetLong, enterShort, stopLossShort, targetShort: **Stores** the _entry_ price, _stop loss_ level, and _target take_ profit level for **long** and **short** positions.
* isLongEntry, isShortEntry: **Flags** indicating whether a **long** or **short** position has been **entered**.

#### Trade Information Table
* The strategy **displays** a _trade information_ **table** in the bottom right corner of the chart.
* The table **shows** the **current** position's _information_ (**Entry Side**, **Entry Price**, **Stop Loss** and **Take Profit**).
* When _position is opened_, the **Entry Side** it is **Long** or **Short**, the **Entry Price**, **Stop Loss** and **Take Profit** are **Prices** shown. When there is no position, all are **None**.

File Script: [Bollinger Bands Modified (Stormer)](bollinger_bands_modified_stormer.pine)

---
---

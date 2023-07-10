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

Trading View: [Bollinger Bands Modified (Stormer)](https://www.tradingview.com/script/aksPa3e7-Bollinger-Bands-Modified-Stormer/)

---
---

## Moving Average Rainbow (Stormer) Strategy

> This **Strategy** is **based** on the **Setup** given by **Alexandre Wolwacz**, AKA **Stormer**.\
> **Stormer** is a professional trader and investor from Brazil.\
> More about **Stormer**'s work: [Stormer YT](https://www.youtube.com/@StormerOficial).\
> More about the **Setup**: [Setup to operate cryptocurrencies](https://www.youtube.com/watch?v=Dzncfb_Fmgs).

> Observations: This _strategy_ executes both long and short positions using _Moving Averages_.\
Please **read** the explanation carefully to better understand what this strategy **does**.

#### Strategy Settings
* Moving Average Type (**maType**): Determines the **type** of the **MAs**. _Default value is EMA_.
* Length of Moving Averages (**maLengthFirst**, **maLengthSecond**, **maLengthThird**, **maLengthFourth**, **maLengthFifth**, **maLengthSixth**, **maLengthSeventh**, **maLengthEighth**, **maLengthNineth**, **maLengthTenth**, **maLengthEleventh**, **maLengthTwelveth**, ): Sets the **length** of each of the **Moving Average**. _Default values are 3, 5, 8, 13, 20, 25, 30, 35, 40, 45, 50, 55_.
* Target Take Profit Factor (**targetFactor**): Determines the **factor** to _calculate_ the **take profit** level. _Default value is 1.6_.
* Check Turnover Trend (**verifyTurnoverTrend**): If enabled, checks for a _supposedly turnover trend_ and setup new target (for **long** is the _highest high_ and for **short** is the _lowest low_ identified). _Default value is true_.
* Check Turnover Signal (**verifyTurnoverSignal**): If enabled, checks for a _supposedly turnover signal_, closing the _current position_ and _opening_ a new one (for **long** it will _close_ and _open_ a _new_ for _short_, for **short** it will _close_ and _open_ a _new_ for _long_)._Default value is false_.
* Check Price Exit Turnover Signal (**verifyTurnoverSignalPriceExit**): It is a _complement_ option for **verifyTurnoverSignal**, if enabled, veryfies the _price_ if _profitable_ before _exiting_ the _current position_._Default value is false_.

#### Calculation and Condition Variables
* ma1, ma2, ma3, ma4, ma5, ma6, ma7, ma8, ma9, ma10, ma11, ma12: Calculates the **Moving Averages** from **mas** _function_ based on the _input parameters_ _maType_ (type of moving average) and the _maLengths_ (first untill twelveth) and returns _12_ **moving averages** with its supposedly **moving average** type.
* maMean: Calculates the _average_ of the _twelves_ **Moving Averages** by _summing_ the _MAs_ up and _dividing_ it by the quantity (in this case 12), returning the result of the calculation.
* isMa1To4Above: Calculates the **ma1** _untill_ **ma4** if it _one line_ is **greater** than the _other line_, if _one line_ is **above** the _other line_. Determining whether the _moving averages_ **ma1**, **ma2**, **ma3**, and **ma4** are in a strictly _increasing order_.
* isMa1To4Below: Calculates the **ma1** _untill_ **ma4** if it _one line_ is **lesser** than the _other line_, if _one line_ is **below** the _other line_. Determining whether the _moving averages_ **ma1**, **ma2**, **ma3**, and **ma4** are in a strictly _decreasing order_.
* isMa5To8Above: Calculates the **ma5** _untill_ **ma8** if it _one line_ is **greater** than the _other line_, if _one line_ is **above** the _other line_. Determining whether the _moving averages_ **ma5**, **ma6**, **ma7**, and **ma8** are in a strictly _increasing order_.
* isMa5To8Below: Calculates the **ma5** _untill_ **ma8** if it _one line_ is **lesser** than the _other line_, if _one line_ is **below** the _other line_. Determining whether the _moving averages_ **ma5**, **ma6**, **ma7**, and **ma8** are in a strictly _decreasing order_.
* isCloseGreaterMaMean: If **close** _price_ is **greater** than the _moving average mean_.
* isCloseLesserMaMean: If **close** _price_ is **lesser** than the _moving average mean_.
* isCurHighGreaterPrevHigh: If _current_ **high** _price_ is **greater** than the _previous_ **high** _price_.
* isCurLowLesserPrevLow: If _current_ **Low** _price_ is **lesser** than the _previous_ **Low** _price_.
* isMaUptrend: This defines _moving average_ **uptrend** when _close price_ is **greater** than the _moving average mean_ and the moving averages lines _5 to 8_ is in _increasing order_.
* isMaDowntrend: This defines _moving average_ **downtrend** when _close price_ is **lesser** than the _moving average mean_ and the moving averages lines _5 to 8_ is in _decreasing order_.
* isUptrend: Simply checks if is _moving average_ **uptrend**.
* isDowntrend: Simply checks if is _moving average_ **downtrend**.
* curTouchPriceUptrend and curTouchPriceDowntrend: Both of them _fetches_ the _price_ where the **low price** supposedly _touched_ the _ma line_ (for **uptrend**) and the **high price** supposedly _touched_ the _ma line_ (for **downtrend**). It is calculated from **maTouchPriceTrend** _function_ based on the _parameters_ which are the _12 moving averages lines_ from **mas** _function_ and the _trending_ if it is **uptrend** or **downtrend**.
* prevTouchPriceUptrend and prevTouchPriceDowntrend: They are simply the **previous** _result_ from **curTouchPriceUptrend** and **curTouchPriceDowntrend**.
* isPrevTouchPriceUptrendTouched: This checks if **prevTouchPriceUptrend** really _touched_ the price, meaning it has to be _greater than 0 and not a not available_ (**na**) variable.
* isPrevTouchPriceDowntrendTouched: This checks if **prevTouchPriceDowntrend** really _touched_ the price, meaning it has to be _greater than 0 and not a not available_ (**na**) variable.
* isPrevTouchedPriceUptrend: Checks for **isPrevTouchPriceUptrendTouched** _touched_ the price and **isMaUptrend** is **uptrend**.
* isPrevTouchedPriceDowntrend: Checks for **isPrevTouchPriceDowntrendTouched** _touched_ the price and **isMaDowntrend** is **downtrend**.
* isPositionFlat: This tests if the **strategy.position_size** is _0_, meaning that there is no open position.
* isPositionLongClose: Checks **positionEntryPrice** if there is _no_ entry price and _position_ **positionIsEntryLong** have not gone _long_.
* isPositionShortClose: Checks **positionEntryPrice** if there is _no_ entry price and _position_ **positionIsEntryShort** have not gone _short_.
* isLongCondition: If has _condition_ to go **long**, checking **isMaUptrend** if is _moving average_ is **uptrend** and **isCurHighGreaterPrevHigh** the _current_ high price is **greater** than the _previous_ high price and **isPrevTouchedPriceUptrend** is _previous_ low price has touched one of _moving average_ lines.
* isShortCondition: If has _condition_ to go **short**, checking **isMaDowntrend** if is _moving average_ is **downtrend** and **isCurLowLesserPrevLow** the _current_ low price is **lesser** than the _previous_ low price and **isPrevTouchedPriceDowntrend** is _previous_ high price has touched one of _moving average_ lines.
* longTurnoverExit: This condition checks if **verifyTurnoverSignal** is enabled with **verifyTurnoverSignalPriceExit** so to speak it may _exit_ the _current position_ **profitable** if there is a _potential_ turn over market _long going short_.
* shortTurnoverExit: This condition checks if **verifyTurnoverSignal** is enabled with **verifyTurnoverSignalPriceExit** so to speak it may _exit_ the _current position_ **profitable** if there is a _potential_ turn over market _short going long_.

#### Position Entry and Exit

###### Long Conditions
* If conditions **isLongCondition** for a _long position_ are met and **isPositionClose** have no _current position_ open or if **verifyTurnoverSignal** checking for a supposedly _turnover signal_ setting is _true_ and conditions **isLongCondition** for a _long position_ are met and **isPositionShort** _current position_ is on _short_, the strategy **enters** a _long position_ or **exits** the current _short position_ to **enter** _long position_.

###### Short Conditions
* If conditions **isShortCondition** for a _short position_ are met and **isPositionClose** have no _current position_ open or if **verifyTurnoverSignal** checking for a supposedly _turnover signal_ setting is _true_ and conditions **isShortCondition** for a _short position_ are met and **isPositionLong** _current position_ is on _long_, the strategy **enters** a _short position_ or **exits** the current _long position_ to **enter** _short position_.

#### Position Management

##### Long Conditions
###### Stop Loss
* Stop loss **stopLossLong** is defined by fetching the **prevTouchPriceUptrend** _previous touched moving average low price_ touched one of the twelve _moving averages_.\
If **isCurLowLesserPrevLow** _current low price_ is lesser than the _previous low price_, then it will check **curTouchPriceUptrend** _current touched moving average low price_ is not available (**na**) variable, to return the _current low price_ or the _low price_ from **curTouchPriceUptrend**, redefining then the stop loss **stopLossLong** checking if the _low price_ from **curTouchPriceUptrend** is not available (**na**), to recalculate the new value of the stop loss by summing _previous_ **stopLossLong** value with **curLowToucedPrice** _current touched moving average low price_ divided by 2 or returning only the **curLowToucedPrice** _current touched moving average low price_.

```pinescript
stopLossLong := prevTouchPriceUptrend
    if (isCurLowLesserPrevLow)
        curLowToucedPrice = na(curTouchPriceUptrend) ? low : curTouchPriceUptrend
        stopLossLong      := na(curTouchPriceUptrend) ? ((stopLossLong + curLowToucedPrice) / 2) : curLowToucedPrice
```

###### Take Profit
* The take profit **targetLong** is simply calculated by having the **close** price _summed_ with **close** price _minus_ the stop loss **stopLossLong** _multiplied_ by the target factor **targetFactor** defined.

```pinescript
targetLong := (close + (math.abs(close - stopLossLong) * targetFactor))
```

> Observations: Both **stopLossLong** and **targetLong** has to be greater than 0 to enter position.

###### Recalculate Take Profit
* When the **verifyTurnoverTrend** setting is set to **true** and **isPositionLong** position is **long**, it recalculates the new take profit target **longPositionHighestHigh**.\
First it gets the highest high during the long position opened.

```pinescript
curHighestHigh = high
    if (curHighestHigh > longPositionHighestHigh or na(longPositionHighestHigh))
        longPositionHighestHigh := curHighestHigh
```

Then it veryfies a supposedly _turn over trend_ by checking the _first four lines_ of the **moving averages** **isMa1To4Below** if it is _decreasing order_ and is **isCloseLesserMaMean** the _current close_ price is **lesser** than the _moving averages_ average and is **longPositionHighestHigh** the highest high is **greater** than the **strategy.position_avg_price** price opened position.

```pinescript
if (isMa1To4Below and isCloseLesserMaMean and longPositionHighestHigh > strategy.position_avg_price)
        isTurnoverTrendLongTrigger := true
        strategy.exit('Exit Long', 'Long', stop=stopLossLong, limit=longPositionHighestHigh)
```

> Observations: It keeps **stopLossLong** the same stop loss defined when opened position.

##### Short Conditions
###### Stop Loss
* Stop loss **stopLossShort** is defined by fetching the **prevTouchPriceDowntrend** _previous touched moving average high price_ touched one of the twelve _moving averages_.\
If **isCurHighGreaterPrevHigh** _current high price_ is greater than the _previous high price_, then it will check **curTouchPriceDowntrend** _current touched moving average high price_ is not available (**na**) variable, to return the _current high price_ or the _high price_ from **curTouchPriceDowntrend**, redefining then the stop loss **stopLossShort** checking if the _high price_ from **curTouchPriceDowntrend** is not available (**na**), to recalculate the new value of the stop loss by summing _previous_ **stopLossShort** value with **curHighToucedPrice** _current touched moving average high price_ divided by 2 or returning only the **curHighToucedPrice** _current touched moving average high price_.

```pinescript
stopLossShort := prevTouchPriceDowntrend
    if (isCurHighGreaterPrevHigh)
        curHighToucedPrice = na(curTouchPriceDowntrend) ? high : curTouchPriceDowntrend
        stopLossShort      := na(curTouchPriceDowntrend) ? ((stopLossShort + curHighToucedPrice) / 2) : curHighToucedPrice
```

###### Take Profit
* The take profit **targetShort** is simply calculated by having the **close** price _subtract_ with **close** price _minus_ the stop loss **stopLossShort** _multiplied_ by the target factor **targetFactor** defined.

```pinescript
targetShort := (close - (math.abs(close - stopLossShort) * targetFactor))
```

> Observations: Both **stopLossShort** and **targetShort** has to be greater than 0 to enter position.

###### Recalculate Take Profit
* When the **verifyTurnoverTrend** setting is set to **true** and **isPositionShort** position is **short**, it recalculates the new take profit target **shortPositionLowestLow**.\
First it gets the lowest low during the short position opened.

```pinescript
curLowestLow = low
    if (curLowestLow < shortPositionLowestLow or na(shortPositionLowestLow))
        shortPositionLowestLow := curLowestLow
```

Then it veryfies a supposedly _turn over trend_ by checking the _first four lines_ of the **moving averages** **isMa1To4Above** if it is _increasing order_ and is **isCloseGreaterMaMean** the _current close_ price is **greater** than the _moving averages_ average and is **shortPositionLowestLow** the lowest low is **lower** than the **strategy.position_avg_price** price opened position.

```pinescript
if (isMa1To4Above and isCloseGreaterMaMean and shortPositionLowestLow < strategy.position_avg_price)
        isTurnoverTrendShortTrigger := true
        strategy.exit('Exit Short', 'Short', stop=stopLossShort, limit=shortPositionLowestLow)
```

> Observations: It keeps **stopLossShort** the same stop loss defined when opened position.

#### Trade Information Table
* The strategy **displays** a _trade information_ **table** in the bottom right corner of the chart.
* The table **shows** the **current** position's _information_ (**Entry Side**, **Entry Price**, **Stop Loss**, **Take Profit**, **New Target** and **Possible Market Turnover**).
* When _position is opened_, the **Entry Side** it is **Long** or **Short**, the **Entry Price**, **Stop Loss** and **Take Profit** are **Prices** shown. When there is no position, all are **None**.
* When _during in position_, the **New Target** it is the _new price target_ and the **Possible Market Turnover** will trigger to show **YES**. When not triggered, both shows **None**.

File Script: [Moving Average Rainbow (Stormer)](ma_rainbow_stormer.pine)

Trading View: [Moving Average Rainbow (Stormer)](https://www.tradingview.com/script/y3jOCBh1-Moving-Average-Rainbow-Stormer/)

---
---
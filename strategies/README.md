# Strategies

## Simple Alert Strategy
* The strategy will alert you when the price crosses above or below a moving average.
* The sole purpose of this strategy is to test the alert system as fast as possible.

File Script: [Simple Alert Strategy](simple_alert_strategy.pine)

## Bollinger Bands Modified (Stormer)
* Bollinger Bands using 20 in Length and 0.38 in StdDev. Its purpose to enter the operation and indicate valid tendency.
* Prices above superior bands is an possible long tendency.
* Prices below inferior bands is an possible short tendency.
* Long position is taken when price closes above the bands and enters the operation right in the next opening candle.
* Short position is taken when price closes below the bands and enters the operation right in the next opening candle.
* Stop short is at the last top.
* Stop long is at the last bottom.
* Take profit is at 1.6x its risk.

File Script: [Bollinger Bands Modified Stormer](bollinger_bands_modified_stormer.pine)
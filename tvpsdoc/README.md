
# TradingView PineScript Documentação

## [TradingView PineScript Documentação Versão 5](./v5/README.md)



---

NOTES:
In LANGUAGE > Time Series = The same looping logic on all bars is applied to function calls such as plot(open) which will repeat on each bar, successively plotting on the chart the value of [open] for each bar. + O [open] tá referenciando o link high, e não open.

In LANGUAGE > TYPE SYSTEM = Pine Script™ also has built-in color constants, including `color.green, color.red, color.orange, color.blue` (the default color in plot*() functions and many of the default color-related properties in drawing types), etc.
`color.green, color.red, color.orange, color.blue` => tá referenciando var nos links, deveria ser const.
antes(https://www.tradingview.com/pine-script-reference/v5/#var_color{dot}green), depois(https://www.tradingview.com/pine-script-reference/v5/#const_color{dot}green)
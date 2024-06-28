
# TradingView PineScript Documentação

## [TradingView PineScript Documentação Versão 5](./v5/README.md)

---

NOTES:
In LANGUAGE > Time Series = The same looping logic on all bars is applied to function calls such as plot(open) which will repeat on each bar, successively plotting on the chart the value of [open] for each bar. + O [open] tá referenciando o link high, e não open.

In LANGUAGE > TYPE SYSTEM = Pine Script™ also has built-in color constants, including `color.green, color.red, color.orange, color.blue` (the default color in plot*() functions and many of the default color-related properties in drawing types), etc.
`color.green, color.red, color.orange, color.blue` => tá referenciando var nos links, deveria ser const.
antes(https://www.tradingview.com/pine-script-reference/v5/#var_color{dot}green), depois(https://www.tradingview.com/pine-script-reference/v5/#const_color{dot}green)

In LANGUAGE > TYPE SYSTEM > Collections = but the more generic `array.new<type>` form is preferred,
O `array.new<type>` tá referenciando type_array, deveria ser array.new_type.
antes (https://www.tradingview.com/pine-script-reference/v5/#type_array) depois (https://www.tradingview.com/pine-script-reference/v5/#fun_array.new%3Ctype%3E)

In LANGUAGE > BUILT-INS > BUILT-IN FUNCTIONS = Functions used to manipulate colors in the color namespace: color.from_gradient(), color.new(), color.rgb(), etc.
O color.new() tá referenciando .rgb, e o color.rgb() tá referenciando .new, ou seja, estão com links invertidos.

In ARRAYS > Looping through array elements = An alternative method to loop through an array is to use a for…in loop.
O "for…in" tá referenciando o link (https://www.tradingview.com/pine-script-reference/v5/#op_for{dot}{dot}{dot}in) sendo que deveria ser (https://www.tradingview.com/pine-script-reference/v5/#kw_for...in).

In ARRAYS > Looping through array elements = for…in loops can return a tuple containing each index and corresponding element.
O "for…in" tá referenciando o link (https://www.tradingview.com/pine-script-reference/v5/#op_for{dot}{dot}{dot}in) sendo que deveria ser (https://www.tradingview.com/pine-script-reference/v5/#kw_for...in).

In MATRICES > FOR...IN = When a script needs to iterate over and retrieve the rows of a matrix, using the for…in structure is often preferred over the standard for loop.
O "for…in" tá referenciando o link (https://www.tradingview.com/pine-script-reference/v5/#op_for{dot}{dot}{dot}in) sendo que deveria ser (https://www.tradingview.com/pine-script-reference/v5/#kw_for...in).

In MATRICES > Operation not available for non-square matrices. = matrix.inv(), matrix.det(), matrix.eigenvalues(), and matrix.eigenvectors().
matrix.eigenvectors() tá referenciando o link (https://www.tradingview.com/pine-script-reference/v5/#fun_matrix.inv) sendo que deveria ser (https://www.tradingview.com/pine-script-reference/v5/#fun_matrix.eigenvectors).

In INPUTS > INPUT FUNCTIONS = An input*.() call being just another function call in Pine Script™, its result can be combined with [arithmetic], _comparison_, [logical] or [ternary] operators to form an expression to be assigned to the variable.
O "_comparison_" não está sendo referenciado pela página específica sobre comparison em "./04_05_operadores.md#operadores-de-comparação", exemplo [comparison](./v5/04_05_operadores.md#operadores-de-comparação)

In LIBRARIES > QUALIFIED TYPE CONTROL = One can also use the [series](https://www.tradingview.com/pine-script-reference/v5/#type_simple) keyword to prefix the type of a library function parameter. However, because arguments are qualified as "series" by default, using the [series](https://www.tradingview.com/pine-script-reference/v5/#type_simple) modifier is redundant.
Os dois series que estão referenciando o link externo, deveria ser sobre o type series, este [link](https://www.tradingview.com/pine-script-reference/v5/#type_series).

In CONCEPTS > Other timeframes and data > Introduction = [request.security()](https://www.tradingview.com/pine-script-docs/concepts/other-timeframes-and-data#request-security) retrieves data from another symbol, timeframe, or other context.
[request.security_lower_tf()](https://www.tradingview.com/pine-script-docs/concepts/other-timeframes-and-data#request-security-lower-tf) retrieves intrabar data, i.e., data from a timeframe lower than the chart timeframe.
[request.currency_rate()](https://www.tradingview.com/pine-script-docs/concepts/other-timeframes-and-data#request-currency-rate) requests a daily rate to convert a value expressed in one currency to another.
[request.dividends(), request.splits(), and request.earnings()](https://www.tradingview.com/pine-script-docs/concepts/other-timeframes-and-data#request-dividends-request-splits-and-request-earnings) respectively retrieve information about an issuing company’s dividends, splits, and earnings.
[request.quandl()](https://www.tradingview.com/pine-script-docs/concepts/other-timeframes-and-data#request-quandl) retrieves information from NASDAQ Data Link (formerly Quandl).
[request.financial()](https://www.tradingview.com/pine-script-docs/concepts/other-timeframes-and-data#request-financial) retrieves financial data from FactSet.
[request.economic()](https://www.tradingview.com/pine-script-docs/concepts/other-timeframes-and-data#request-economic) retrieves economic and industry data.
[request.seed()](https://www.tradingview.com/pine-script-docs/concepts/other-timeframes-and-data#request-seed) retrieves data from a user-maintained GitHub repository.
Os links de referencia para os 'request.*' estão redirecionando para a mesma página.
[request.security()](https://www.tradingview.com/pine-script-reference/v5/#fun_request{dot}security)
[request.security_lower_tf()](https://www.tradingview.com/pine-script-reference/v5/#fun_request{dot}security_lower_tf)
[request.currency_rate()](https://www.tradingview.com/pine-script-reference/v5/#fun_request{dot}currency_rate)
[request.dividends(), request.splits(), e request.earnings()](https://www.tradingview.com/pine-script-reference/v5/#fun_request{dot}dividends-request{dot}splits-request{dot}earnings)
[request.quandl()](https://www.tradingview.com/pine-script-reference/v5/#fun_request{dot}quandl)
[request.financial()](https://www.tradingview.com/pine-script-reference/v5/#fun_request{dot}financial)
[request.economic()](https://www.tradingview.com/pine-script-reference/v5/#fun_request{dot}economic)
[request.seed()](https://www.tradingview.com/pine-script-reference/v5/#fun_request{dot}seed)

In CONCEPTS > `ignore_invalid_symbol` =
Not all issuing companies publish quarterly financial reports. If the symbol’s issuing company doesn’t report on a quarterly basis, change the “FQ” value in this script to the company’s minimum reporting period. See the [request.financial()](https://www.tradingview.com/pine-script-docs/concepts/other-timeframes-and-data#request-financial) section for more information.
Os links de referencia para os 'request.financial()' pode estar sendo redirecionado para uma página desconhecida.
[request.financial()](https://www.tradingview.com/pine-script-reference/v5/#fun_request{dot}financial)

In Other timeframes and data > TIMEFRAMES> LOWER TIMEFRAMES = Notice!While scripts can use request.security() to retrieve the values from a single intrabar on each chart bar, which might provide utility in some unique cases, we recommend using the request.security_lower_tf() function for intrabar analysis when possible, as it returns an array containing data from all available intrabars within a chart bar. See [this section](https://www.tradingview.com/pine-script-docs/concepts/other-timeframes-and-data/#request-security-lower-tf) to learn more.
O link para o "See [this section]" não encaminha para o destino correto. O link pode ser supostamente para "See [this section](https://www.tradingview.com/pine-script-docs/concepts/other-timeframes-and-data/#requestsecurity_lower_tf)"

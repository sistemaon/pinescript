
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

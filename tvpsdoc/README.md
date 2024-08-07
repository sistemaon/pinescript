
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

IN Other timeframes and data > Calculated variables = The script colors the plot with a gradient based on the correlation value. To learn more about color gradients in Pine, see [this](https://www.tradingview.com/pine-script-docs/concepts/colors/#colors) section of our User Manual’s page on colors.
O link do [this] deveria referenciar a parte do link sobre cores de gradiente.
[this](https://www.tradingview.com/pine-script-docs/concepts/colors/#colorfrom_gradient)

IN  Other timeframes and data > request.economic() = See the [Country/region codes]() section for a complete list of codes this function supports.
O [Country/region codes]() deveria referenciar a parte do "Country/region codes" para o link "https://www.tradingview.com/pine-script-docs/concepts/other-timeframes-and-data/#countryregion-codes" e não para o "https://www.tradingview.com/pine-script-docs/concepts/other-timeframes-and-data/#other-timeframes-and-data"
[Country/region codes](https://www.tradingview.com/pine-script-docs/concepts/other-timeframes-and-data/#countryregion-codes)

IN Plots > Introduction > Calls to plot() can, however, be designed to plot conditionally in two ways, which we cover in the [Conditional plots]() section of this page.
Deveria referenciar com (https://www.tradingview.com/pine-script-docs/concepts/plots/#plotting-conditionally)
e não com (https://www.tradingview.com/pine-script-docs/concepts/plots/#plots).

IN Strategies > strategy.order() > The same script would behave differently using strategy.entry(), as per the example shown in the [section above]().
[section above]() is referencing (https://www.tradingview.com/pine-script-docs/concepts/strategies/#strategy-entry)
in which is not redirecting to the reference but to the initial introduction.
It should be (https://www.tradingview.com/pine-script-docs/concepts/strategies/#strategyentry)

IN Text and shapes > plotchar() > "As explained in the [When the script’s scale must be preserved] section of our page on Debugging,"
[When the script’s scale must be preserved] is referencing (https://www.tradingview.com/pine-script-docs/writing/debugging/#when-the-scripts-scale-must-be-preserved)
shouldnt be referencing (https://www.tradingview.com/pine-script-docs/writing/debugging/#without-affecting-the-scale)


IN Text and shapes > Realtime behavior >= See the page on Pine Script™‘s [Execution model](https://www.tradingview.com/pine-script-docs/#execution-model). Deveria ser esse link [Execution model](https://www.tradingview.com/pine-script-docs/language/execution-model/#execution-model)


IN Time > Calendar dates and times > This will plot the day of the opening of the bar where the January 1st, 2021 at 00:00 time falls between its [time]() and time_close values:
Deveria ser: [time](https://www.tradingview.com/pine-script-reference/v5/#var_time)
Atual: [time](https://www.tradingview.com/pine-script-reference/v5/#var_time_close)

IN Timeframes > Introduction > The [input.timeframe()]() function provides a way to allow script users to define a timeframe
Deveria ser: [input.timeframe()](https://www.tradingview.com/pine-script-reference/v5/#fun_input.timeframe)
Atual: [input.timeframe()](https://www.tradingview.com/pine-script-reference/v5/#fun_input%7Bdot%7Dsession)

IN Timeframes > Comparing timeframes > the [input.timeframe()]() function into seconds, then divide by 60 to convert into minutes.
Deveria ser: [input.timeframe()](https://www.tradingview.com/pine-script-reference/v5/#fun_input.timeframe)
Atual: [input.timeframe()](https://www.tradingview.com/pine-script-reference/v5/#fun_input%7Bdot%7Dsession)

IN Timeframes > Comparing timeframes > we supply the timeframe selected by the script’s user through the call to [input.timeframe()]().
Deveria ser: [input.timeframe()](https://www.tradingview.com/pine-script-reference/v5/#fun_input.timeframe)
Atual: [input.timeframe()](https://www.tradingview.com/pine-script-reference/v5/#fun_input%7Bdot%7Dsession)

In Profiling and optimization > Profiling a script > Pine Profiler wraps a script and its significant code with [extra calculations]() required to collect performance data.
Deveria ser: [extra calculations](https://www.tradingview.com/pine-script-docs/writing/profiling-and-optimization/#a-look-into-the-profilers-inner-workings)
Atual: [extra calculations](https://www.tradingview.com/pine-script-docs/writing/profiling-and-optimization/#alook-into-the-profilers-inner-workings)

In Profiling and optimization > Code block results > This format is necessary for these structures due to the Profiler’s calculation and display constraints. See [this section]() for more information.
Deveria ser: [this section](https://www.tradingview.com/pine-script-docs/writing/profiling-and-optimization/#a-look-into-the-profilers-inner-workings)
Atual: [this section](https://www.tradingview.com/pine-script-docs/writing/profiling-and-optimization/#alook-into-the-profilers-inner-workings)

In Profiling and optimization > Repetitive profiling > For example, this script [queues]() pseudorandom values with a constant seed
Deveria ser:[queues](https://www.tradingview.com/pine-script-docs/language/arrays/#using-an-array-as-a-queue)
Atual: [queues](https://www.tradingview.com/pine-script-docs/language/arrays/#using-an-array-as-aqueue)

In Profiling and optimization > Using built-ins > 20 to 200 and [profiled the script]() again to observe the changes in performance.
Deveria ser: [profiled the script](https://www.tradingview.com/pine-script-docs/writing/profiling-and-optimization/#profiling-a-script)
Atual: [profiled the script](https://www.tradingview.com/pine-script-docs/writing/profiling-and-optimization/#profiling-ascript)

In Profiling and optimization > Using built-ins > In any case, one should always [profile their scripts](),
Deveria ser: [profile their scripts](https://www.tradingview.com/pine-script-docs/writing/profiling-and-optimization/#profiling-a-script)
Atual: [profile their scripts](https://www.tradingview.com/pine-script-docs/writing/profiling-and-optimization/#profiling-ascript)

In Profiling and optimization > Minimizing request.*() calls > The results from [profiling the script]() show that it took the script 340.8 milliseconds
Deveria ser: [profiling the script](https://www.tradingview.com/pine-script-docs/writing/profiling-and-optimization/#profiling-a-script)
Atual: [profiling the script](https://www.tradingview.com/pine-script-docs/writing/profiling-and-optimization/#profiling-ascript)

In Profiling and optimization > Avoiding redrawing > The results from [profiling this script]() show that line 24,
Deveria ser: [profiling this script](https://www.tradingview.com/pine-script-docs/writing/profiling-and-optimization/#profiling-a-script)
Atual: [profiling this script](https://www.tradingview.com/pine-script-docs/writing/profiling-and-optimization/#profiling-ascript)

In Profiling and optimization > Reducing drawing updates > After [profiling]() the script, we see that the code
Deveria ser: [profiling](https://www.tradingview.com/pine-script-docs/writing/profiling-and-optimization/#profiling-a-script)
Atual: [profiling](https://www.tradingview.com/pine-script-docs/writing/profiling-and-optimization/#profiling-ascript)

In Profiling and optimization > Storing calculated values > After [profiling]() the script’s performance over our chart’s data,
Deveria ser: [profiling](https://www.tradingview.com/pine-script-docs/writing/profiling-and-optimization/#profiling-a-script)
Atual: [profiling](https://www.tradingview.com/pine-script-docs/writing/profiling-and-optimization/#profiling-ascript)

In Profiling and optimization > Reducing loop calculations > After [profiling the script](), we see it took about two seconds to execute 21,778 times.
Deveria ser: [profiling the script](https://www.tradingview.com/pine-script-docs/writing/profiling-and-optimization/#profiling-a-script)
Atual: [profiling the script](https://www.tradingview.com/pine-script-docs/writing/profiling-and-optimization/#profiling-ascript)

In Profiling and optimization > Tips > Working around Profiler overhead > as explained in [this section](), the time
Deveria ser: [this section](https://www.tradingview.com/pine-script-docs/writing/profiling-and-optimization/#a-look-into-the-profilers-inner-workings)
Atual: [this section](https://www.tradingview.com/pine-script-docs/writing/profiling-and-optimization/#alook-into-the-profilers-inner-workings)

In Profiling and optimization > Tips > Working around Profiler overhead > know where to start without being able to profile the code.
Deveria ser: [profile the code](https://www.tradingview.com/pine-script-docs/writing/profiling-and-optimization/#profiling-a-script)
Atual: [profile the code](https://www.tradingview.com/pine-script-docs/writing/profiling-and-optimization/#profiling-ascript)

In Limitations > Time > Script execution > See the [Events triggering the execution of a script]() for a list of
Deveria ser: [Events triggering the execution of a script](https://www.tradingview.com/pine-script-docs/language/execution-model/#events-triggering-the-execution-of-a-script)
Atual: [Events triggering the execution of a script](https://www.tradingview.com/pine-script-docs/language/execution-model/#events)

In Limitations > Other limitations > Maximum bars back > References to past values using the [[]]() history-referencing
Deveria ser: [[]](https://www.tradingview.com/pine-script-reference/v5/#op_%5B%5D)
Atual: [[]](https://www.tradingview.com/pine-script-reference/v5/#op_op_%5B%5D)


In > Pine Script™ cannot determine the referencing length of a series. Try using max_bars_back in the indicator or strategy function > This behavior is described in more detail in the section about [drawings]().
Deveria ser: [drawings](https://www.tradingview.com/pine-script-docs/concepts/lines-and-boxes/#historical-buffer-and-max_bars_back)
Atual: [drawings](https://www.tradingview.com/pine-script-docs/concepts/lines-and-boxes/#historical-buffer-and-max-bars-back)


__No link sobre a constante CURRENCY.KRW https://br.tradingview.com/pine-script-reference/v5/#var_currency.KRW (é para referir a moeda sul coreana won, e não venceu)__




__FALTA TRADUZIR DO SITE:__

- no link [chart.right_visible_bar_time](https://br.tradingview.com/pine-script-reference/v5/#var_chart.right_visible_bar_time) A descrição do título não foi traduzido.
- no link [backadjustment.on](https://br.tradingview.com/pine-script-reference/v5/#const_backadjustment.on)
- no link [backadjustment.off](https://br.tradingview.com/pine-script-reference/v5/#const_backadjustment.off)
- no link [backadjustment.inherit](https://br.tradingview.com/pine-script-reference/v5/#const_backadjustment.inherit)
- no link [settlement_as_close.inherit](https://br.tradingview.com/pine-script-reference/v5/#const_settlement_as_close.inherit)
- no link [settlement_as_close.off](https://br.tradingview.com/pine-script-reference/v5/#const_settlement_as_close.off)
- no link [settlement_as_close.on](https://br.tradingview.com/pine-script-reference/v5/#const_settlement_as_close.on)


# Estruturas Condicionais

Estruturas condicionais no Pine Script são [if](https://br.tradingview.com/pine-script-reference/v5/#op_if) e [switch](https://br.tradingview.com/pine-script-reference/v5/#op_switch).

Podem ser usadas:

- Pelos seus efeitos colaterais (um impacto adicional no código, além de apenas retornar um valor), ou seja, quando não retornam um valor mas realizam ações, como reatribuir valores a variáveis ou chamar funções.
- Para retornar um valor ou uma tupla que então pode ser atribuída a uma (ou mais, no caso de tuplas) variável.

Estruturas condicionais, assim como as estruturas [for](https://br.tradingview.com/pine-script-reference/v5/#op_for) e [while](https://br.tradingview.com/pine-script-reference/v5/#op_while), podem ser incorporadas; pode-se utilizar um [if](https://br.tradingview.com/pine-script-reference/v5/#op_if) ou [switch](https://br.tradingview.com/pine-script-reference/v5/#op_switch) dentro de outra estrutura.

Algumas funções integradas do Pine Script não podem ser chamadas de dentro dos blocos locais de estruturas condicionais.

Estes são: [alertcondition()](https://br.tradingview.com/pine-script-reference/v5/#fun_alertcondition), [barcolor()](https://br.tradingview.com/pine-script-reference/v5/#fun_barcolor), [fill()](https://br.tradingview.com/pine-script-reference/v5/#fun_fill), [hline()](https://br.tradingview.com/pine-script-reference/v5/#fun_hline), [indicator()](https://br.tradingview.com/pine-script-reference/v5/#fun_indicator), [library()](https://br.tradingview.com/pine-script-reference/v5/#fun_library), [plot()](https://br.tradingview.com/pine-script-reference/v5/#fun_plot), [plotbar()](https://br.tradingview.com/pine-script-reference/v5/#fun_plotbar), [plotcandle()](https://br.tradingview.com/pine-script-reference/v5/#fun_plotcandle), [plotchar()](https://br.tradingview.com/pine-script-reference/v5/#fun_plotchar), [plotshape()](https://br.tradingview.com/pine-script-reference/v5/#fun_plotshape), [strategy()](https://br.tradingview.com/pine-script-reference/v5/#fun_strategy).

Isso não significa que sua funcionalidade não pode ser controlada por condições avaliadas pelo script — apenas que não podem ser incluídos dentro das estruturas condicionais. Observe que, embora chamadas de funções `input*.()` sejam permitidas em blocos locais, sua funcionalidade é a mesma que se estivessem no escopo global do script.

Os blocos locais em estruturas condicionais devem ser indentados por quatro espaços ou um tabulação.

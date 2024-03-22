
# Estruturas Condicionais

Estruturas condicionais no Pine Script são [if](https://br.tradingview.com/pine-script-reference/v5/#op_if) e [switch](https://br.tradingview.com/pine-script-reference/v5/#op_switch).

Podem ser usadas:

- Pelos seus efeitos colaterais/adicionais (um impacto adicional no código, além de apenas retornar um valor), ou seja, quando não retornam um valor mas realizam ações, como reatribuir valores a variáveis ou chamar funções.
- Para retornar um valor ou uma tupla que então pode ser atribuída a uma (ou mais, no caso de tuplas) variável.

Estruturas condicionais, assim como as estruturas [for](https://br.tradingview.com/pine-script-reference/v5/#op_for) e [while](https://br.tradingview.com/pine-script-reference/v5/#op_while), podem ser incorporadas; pode-se utilizar um [if](https://br.tradingview.com/pine-script-reference/v5/#op_if) ou [switch](https://br.tradingview.com/pine-script-reference/v5/#op_switch) dentro de outra estrutura.

Algumas funções integradas do Pine Script não podem ser chamadas de dentro dos blocos locais de estruturas condicionais.

Estes são: [alertcondition()](https://br.tradingview.com/pine-script-reference/v5/#fun_alertcondition), [barcolor()](https://br.tradingview.com/pine-script-reference/v5/#fun_barcolor), [fill()](https://br.tradingview.com/pine-script-reference/v5/#fun_fill), [hline()](https://br.tradingview.com/pine-script-reference/v5/#fun_hline), [indicator()](https://br.tradingview.com/pine-script-reference/v5/#fun_indicator), [library()](https://br.tradingview.com/pine-script-reference/v5/#fun_library), [plot()](https://br.tradingview.com/pine-script-reference/v5/#fun_plot), [plotbar()](https://br.tradingview.com/pine-script-reference/v5/#fun_plotbar), [plotcandle()](https://br.tradingview.com/pine-script-reference/v5/#fun_plotcandle), [plotchar()](https://br.tradingview.com/pine-script-reference/v5/#fun_plotchar), [plotshape()](https://br.tradingview.com/pine-script-reference/v5/#fun_plotshape), [strategy()](https://br.tradingview.com/pine-script-reference/v5/#fun_strategy).

Isso não significa que sua funcionalidade não pode ser controlada por condições avaliadas pelo script — apenas que não podem ser incluídos dentro das estruturas condicionais. Observe que, embora chamadas de funções `input*.()` sejam permitidas em blocos locais, sua funcionalidade é a mesma que se estivessem no escopo global do script.

Os blocos locais em estruturas condicionais devem ser indentados por quatro espaços ou um tabulação (_tab_).


# Estrutura `if`

## `if` usado por seus efeitos colaterais/adicionais

Uma estrutura [if](https://br.tradingview.com/pine-script-reference/v5/#op_if) utilizada por seus efeitos adicionais tem a seguinte sintaxe:

```c
if <expression>
    <local_block>
{else if <expression>
    <local_block>}
[else
    <local_block>]
```

Onde:

- As partes entre colchetes (`[]`) podem aparecer nenhuma ou uma vez, ou seja, é opcional e pode estar presente no código uma única vez ou não estar presente. E aquelas entre chaves (`{}`) podem aparecer nenhuma ou mais vezes, ou seja, é opcional e pode estar presente no código nenhuma vez, uma vez ou várias vezes.
- `<expression>` deve ser do tipo [bool](https://br.tradingview.com/pine-script-reference/v5/#type_bool) ou ser autoconvertível para esse tipo, o que é possível apenas para valores [int](https://br.tradingview.com/pine-script-reference/v5/#type_int) ou [float](https://br.tradingview.com/pine-script-reference/v5/#type_float) (consulte o [sistema de tipos](./000_type_system.md#tipos) para maiores informações).
- `<local_block>` consiste em nenhuma ou mais instruções seguidas de um valor de retorno, que pode ser uma tupla de valores. Deve ser indentado por quatro espaços ou uma tabulação (_tab_).
- Pode _não haver_ ou _haver mais cláusulas_ `else if`.
- Pode _não haver_ ou _haver uma cláusula_ `else`.

Quando a `<expressão>` seguinte ao [if](https://br.tradingview.com/pine-script-reference/v5/#op_if) avalia-se como [true](https://br.tradingview.com/pine-script-reference/v5/#const_true), o primeiro bloco local é executado, a execução da estrutura [if](https://br.tradingview.com/pine-script-reference/v5/#op_if) termina, e o(s) valor(es) avaliado(s) ao final do bloco local são retornados.

Quando a `<expressão>` seguinte ao [if](https://br.tradingview.com/pine-script-reference/v5/#op_if) avalia-se como [false](https://br.tradingview.com/pine-script-reference/v5/#const_false), as cláusulas `else if` sucessivas são avaliadas, se houver alguma presente. Quando a `<expressão>` de uma delas avalia-se como [true](https://br.tradingview.com/pine-script-reference/v5/#const_true), seu bloco local é executado, a execução da estrutura [if](https://br.tradingview.com/pine-script-reference/v5/#op_if) termina, e o(s) valor(es) avaliado(s) ao final do bloco local são retornados.

Quando nenhuma `<expressão>` avaliou-se como [true](https://br.tradingview.com/pine-script-reference/v5/#const_true) e existe uma cláusula `else`, seu bloco local é executado, a execução da estrutura [if](https://br.tradingview.com/pine-script-reference/v5/#op_if) termina, e o(s) valor(es) avaliado(s) ao final do bloco local são retornados.

Quando nenhuma `<expressão>` avaliou-se como [true](https://br.tradingview.com/pine-script-reference/v5/#const_true) e não existe uma cláusula `else`, é retornado [na](https://br.tradingview.com/pine-script-reference/v5/#var_na).

Usar estruturas [if](https://br.tradingview.com/pine-script-reference/v5/#op_if) por seus adicionais efeitos pode ser útil para gerenciar o fluxo de ordens em estratégias, por exemplo. Embora a mesma funcionalidade muitas vezes possa ser alcançada usando o parâmetro `when` em chamadas de `strategy.*()`, o código que usa estruturas [if](https://br.tradingview.com/pine-script-reference/v5/#op_if) é mais limpo e fácil leitura:

```c
if (ta.crossover(source, lower))
    strategy.entry("BBandLE", strategy.long, stop=lower,
                   oca_name="BollingerBands",
                   oca_type=strategy.oca.cancel, comment="BBandLE")
else
    strategy.cancel(id="BBandLE")
```

Restringir a execução do código a barras específicas pode ser feito usando estruturas [if](https://br.tradingview.com/pine-script-reference/v5/#op_if), como feito aqui para restringir as atualizações para rótulo (_label_) à última barra do gráfico:

```c
//@version=5
indicator("", "", true)
var ourLabel = label.new(bar_index, na, na, color = color(na), textcolor = color.orange)
if barstate.islast
    label.set_xy(ourLabel, bar_index + 2, hl2[1])
    label.set_text(ourLabel, str.tostring(bar_index + 1, "# bars in chart"))
```

Note que:

- Inicializado a variável `ourLabel` apenas na primeira barra do script, usando o modo de declaração [var](https://br.tradingview.com/pine-script-reference/v5/#op_var). O valor usado para inicializar a variável é fornecido pela chamada da função [label.new()](https://br.tradingview.com/pine-script-reference/v5/#fun_label{dot}new), que retorna um ID do _label_ apontando para o _label_ que ela cria. Utiliza-se essa chamada para definir as propriedades do _label_, pois uma vez definidas, elas persistirão até que as altere.
- O que acontece em seguida é que em cada barra sucessiva, o tempo de execução do Pine Script irá pular a inicialização de `ourLabel`, e a condição da estrutura [if](https://br.tradingview.com/pine-script-reference/v5/#op_if) ([barstate.islast](https://br.tradingview.com/pine-script-reference/v5/#var_barstate{dot}islast)) é avaliada. Ela retorna `false` em todas as barras até a última, então o script nada faz na maioria das barras históricas após a barra zero.
- Na última barra, [barstate.islast](https://br.tradingview.com/pine-script-reference/v5/#var_barstate{dot}islast) torna-se `true` e o bloco local da estrutura é executado, modificando em cada atualização do gráfico as propriedades do rótulo (_label_), que exibe o número de barras no conjunto de dados.
- Ao exibir o texto do _label_ sem um fundo, então define-se o fundo de _label_ como [na](https://br.tradingview.com/pine-script-reference/v5/#var_na) em chamada da função [label.new()](https://br.tradingview.com/pine-script-reference/v5/#fun_label{dot}new), e usa-se `hl2[1]` para a _posição y_ do _label_ para que ele não se mova o tempo todo. Ao usar a média dos valores [high](https://br.tradingview.com/pine-script-reference/v5/#var_high) e [low](https://br.tradingview.com/pine-script-reference/v5/#var_low) da barra __anterior__, o _label_ não se move até o momento em que a próxima barra em tempo real se abre.
- Usa-se `bar_index + 2` na chamada de [label.set_xy()](https://br.tradingview.com/pine-script-reference/v5/#fun_label{dot}set_xy) para deslocar o _label_ para a direita em duas barras.

## `if` usado pra retorno de valor

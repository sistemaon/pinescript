
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

## `if` Usado por seus Efeitos Colaterais/Adicionais

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
- `<expression>` deve ser do tipo [bool](https://br.tradingview.com/pine-script-reference/v5/#type_bool) ou ser autoconvertível para esse tipo, o que é possível apenas para valores [int](https://br.tradingview.com/pine-script-reference/v5/#type_int) ou [float](https://br.tradingview.com/pine-script-reference/v5/#type_float) (consulte o [sistema de tipos](./04_09_tipagem_do_sistema.md#tipos) para maiores informações).
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

## `if` Usado para Retorno de Valor

Uma estrutura [if](https://br.tradingview.com/pine-script-reference/v5/#op_if) usada para retornar um ou mais valores tem a seguinte sintaxe:

```c
[<declaration_mode>] [<type>] <identifier> = if <expression>
    <local_block>
{else if <expression>
    <local_block>}
[else
    <local_block>]
```

Onde:

- Elementos entre colchetes (`[]`) podem não aparecer ou aparecer uma vez, e elementos entre chaves (`{}`) podem não aparecer ou aparecer mais vezes.
- `<declaration_mode>` é o [modo de declaração](./04_06_declaracoes_de_variavel.md#modos-de-declaração) da variável.
- `<type>` é opcional, como em quase todas as declarações de variáveis do Pine Script (veja [tipos](./04_09_tipagem_do_sistema.md#tipos)).
- `<identifier>` é o [nome](./04_04_identificadores.md) da variável.
- `<expression>` pode ser um literal, uma variável, uma expressão ou uma chamada de função.
- `<local_block>` consiste em nenhum ou mais instruções seguidas de um valor de retorno, que pode ser uma tupla de valores. Deve ser indentado por quatro espaços ou uma tabulação (_tab_).
- O valor atribuído à variável é o valor de retorno do `<local_block>`, ou [na](https://br.tradingview.com/pine-script-reference/v5/#var_na) se nenhum bloco local for executado.

Exemplo:

```c
//@version=5
indicator("", "", true)
string barState = if barstate.islastconfirmedhistory
    "islastconfirmedhistory"
else if barstate.isnew
    "isnew"
else if barstate.isrealtime
    "isrealtime"
else
    "other"

f_print(_text) =>
    var table _t = table.new(position.middle_right, 1, 1)
    table.cell(_t, 0, 0, _text, bgcolor = color.yellow)
f_print(barState)
```

É possível omitir o bloco `else`. Nesse caso, se a `condition` for [false](https://br.tradingview.com/pine-script-reference/v5/#const_false), um valor _vazio_ (`na`, `false` ou `""`) será atribuído à variável `var_declarationX`.

Este exemplo mostra como [na](https://br.tradingview.com/pine-script-reference/v5/#var_na) é retornado quando nenhum bloco local é executado. Se `close > open` for `false` aqui, [na](https://br.tradingview.com/pine-script-reference/v5/#var_na) é retornado.

```c
x = if close > open
    close
```

Scripts podem conter estruturas `if` com `if` aninhados e outras estruturas condicionais.

Por exemplo:

```c
if condition1
    if condition2
        if condition3
            expression
```

No entanto, aninhar essas estruturas não é recomendado do ponto de vista de desempenho. Quando possível, geralmente é mais otimizador compor uma única instrução `if` com múltiplos [operadores lógicos](./04_05_operadores.md#operadores-lógicos) em vez de vários blocos `if` aninhados.

Exemplo:

```c
if condition1 and condition2 and condition3
    expression
```


# Estrutura `switch`

A estrutura [switch](https://br.tradingview.com/pine-script-reference/v5/#op_switch) existe em duas formas. Uma delas alterna entre os diferentes valores de uma expressão chave:

```c
[[<declaration_mode>] [<type>] <identifier> = ]switch <expression>
    {<expression> => <local_block>}
    => <local_block>
```

A outra forma não utiliza uma expressão como chave; ela alterna com base na avaliação de diferentes expressões:

```c
[[<declaration_mode>] [<type>] <identifier> = ]switch
    {<expression> => <local_block>}
    => <local_block>
```

Onde:

- Elementos entre colchetes (`[]`) podem não aparecer ou aparecer uma vez, e elementos entre chaves (`{}`) podem não aparecer ou aparecer mais vezes.
- `<declaration_mode>` é o [modo de declaração](./04_06_declaracoes_de_variavel.md#modos-de-declaração) da variável.
- `<type>` é opcional, como em quase todas as declarações de variáveis do Pine Script (veja [tipos](./04_09_tipagem_do_sistema.md#tipos)).
- `<identifier>` é o [nome](./04_04_identificadores.md) da variável.
- `<expression>` pode ser um literal, uma variável, uma expressão ou uma chamada de função.
- `<local_block>` consiste em nenhum ou mais instruções seguidas de um valor de retorno, que pode ser uma tupla de valores. Deve ser indentado por quatro espaços ou uma tabulação (_tab_).
- O `=> <local_block>` no final permite que especifique um valor de retorno que atua como padrão para ser usado quando nenhum outro caso na estrutura é executado.

Apenas um bloco local de uma estrutura [switch](https://br.tradingview.com/pine-script-reference/v5/#op_switch) é executado. Portanto, é uma _estruturada switch_ que _não passa pelos_ casos. Consequentemente, as instruções de `break` são desnecessárias.

Ambas as formas são permitidas como o valor usado para inicializar uma variável.

Assim como na estrutura [if](https://br.tradingview.com/pine-script-reference/v5/#op_if), se nenhum bloco local for executado, é retornado [na](https://br.tradingview.com/pine-script-reference/v5/#var_na).

## `switch` com Expressão 

Veja um exemplo de um [switch](https://br.tradingview.com/pine-script-reference/v5/#op_switch) usando uma expressão:

```c
//@version=5
indicator("Switch using an expression", "", true)

string maType = input.string("EMA", "MA type", options = ["EMA", "SMA", "RMA", "WMA"])
int maLength = input.int(10, "MA length", minval = 2)

float ma = switch maType
    "EMA" => ta.ema(close, maLength)
    "SMA" => ta.sma(close, maLength)
    "RMA" => ta.rma(close, maLength)
    "WMA" => ta.wma(close, maLength)
    =>
        runtime.error("No matching MA type found.")
        float(na)

plot(ma)
```

Note que:

- A expressão pela qual está alternando é a variável `maType`, que é do tipo "input int" (veja mais sobre o qualificador ["input"](./04_09_tipagem_do_sistema.md#input)). Como ela não pode ser alterada durante a execução do script, isso garante que qualquer tipo de _MA_ que o usuário selecione será executado em cada barra, o que é um requisito para funções como [ta.ema()](https://br.tradingview.com/pine-script-reference/v5/#fun_ta{dot}ema), que exigem um argumento "int simples" ("_simple int_") para seu parâmetro de `length`.
- Se nenhum valor correspondente for encontrado para `maType`, o [switch](https://br.tradingview.com/pine-script-reference/v5/#op_switch) executa o último bloco local introduzido por` =>`, que age como um _capturador geral_. Gerado um erro em tempo de execução nesse bloco. Também encerrado-o com `float(na)` para que o bloco local retorne um valor cujo tipo seja compatível com o dos outros blocos locais na estrutura, evitando um erro de compilação.

## `switch` sem Expressão

Veja um exemplo de uma estrutura [switch](https://br.tradingview.com/pine-script-reference/v5/#op_switch) que não utiliza expressão:

```c
//@version=5
strategy("Switch without an expression", "", true)

bool longCondition  = ta.crossover( ta.sma(close, 14), ta.sma(close, 28))
bool shortCondition = ta.crossunder(ta.sma(close, 14), ta.sma(close, 28))

switch
    longCondition  => strategy.entry("Long ID", strategy.long)
    shortCondition => strategy.entry("Short ID", strategy.short)
```

- Usando o [switch](https://br.tradingview.com/pine-script-reference/v5/#op_switch) para selecionar apropriadamente a ordem estratégica para emitir, dependendo se as condições `longCondition` ou `shortCondition` do tipo "bool" são `true`.
- As condições de formação de `longCondition` e `shortCondition` são exclusivas. Embora ambas possam ser `false` simultaneamente, não podem ser `true` ao mesmo tempo. O fato de apenas __um único__ bloco local da estrutura [switch](https://br.tradingview.com/pine-script-reference/v5/#op_switch) ser executado nunca é, portanto, um problema.
- Avaliando-as as chamadas para [ta.crossover()](https://br.tradingview.com/pine-script-reference/v5/#fun_ta{dot}crossover) e [ta.crossunder()](https://br.tradingview.com/pine-script-reference/v5/#fun_ta{dot}crossunder) __antes__ da entrada na estrutura de [switch](https://br.tradingview.com/pine-script-reference/v5/#op_switch). Não fazê-lo, como no exemplo a seguir, impediria que as funções fossem executadas em cada barra, o que resultaria em um aviso do compilador e em um comportamento errático.

Exemplo:

```c
//@version=5
strategy("Switch without an expression", "", true)

switch
    // Compiler warning! Will not calculate correctly!
    ta.crossover( ta.sma(close, 14), ta.sma(close, 28)) => strategy.entry("Long ID", strategy.long)
    ta.crossunder(ta.sma(close, 14), ta.sma(close, 28)) => strategy.entry("Short ID", strategy.short)
```


# Correspondência com o Requisito de Tipo do Bloco Local

Quando múltiplos blocos locais são usados em estruturas, o tipo do valor de retorno de todos os seus blocos locais deve corresponder. Isso se aplica apenas se a estrutura for usada para atribuir um valor a uma variável em uma declaração, porque uma variável só pode ter um tipo, e se a instrução retornar dois tipos incompatíveis em seus ramos (_branches_), o tipo da variável não pode ser determinado adequadamente. Se a estrutura não for atribuída em nenhum lugar, seus ramos podem retornar valores diferentes.

O exemplo a seguir compila sem problemas porque tanto [close](https://br.tradingview.com/pine-script-reference/v5/#var_close) quanto [open](https://br.tradingview.com/pine-script-reference/v5/#var_open) são do tipo [float](https://br.tradingview.com/pine-script-reference/v5/#type_float):

```c
x = if close > open
    close
else
    open
```

O exemplo a seguir não compila porque o primeiro bloco local retorna um valor `float`, enquanto o segundo retorna uma `string`, e o resultado da instrução `if` é atribuído à variável `x`:

```c
// Compilation error!
x = if close > open
    close
else
    "open"
```

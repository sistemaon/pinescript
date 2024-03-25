
# Loops

## Quando Loops são Desnecessários

O tempo de execução do Pine Script e suas funções integradas tornam os loops desnecessários em muitas situações. Programadores de Pine Script que possa não estar familiarizados com o tempo de execução do Pine Script e suas funções integradas e que desejam calcular a média dos últimos 10 valores de [close](https://br.tradingview.com/pine-script-reference/v5/#var_close), talvez criarão o código como:

```c
//@version=5
indicator("Inefficient MA", "", true)
MA_LENGTH = 10
sumOfCloses = 0.0
for offset = 0 to MA_LENGTH - 1
    sumOfCloses := sumOfCloses + close[offset]
inefficientMA = sumOfCloses / MA_LENGTH
plot(inefficientMA)
```

Um loop [for](https://br.tradingview.com/pine-script-reference/v5/#op_for) é desnecessário e ineficiente para realizar tarefas como esta no Pine Script.

Assim é como deve ser feito. Este código a seguir é mais curto e será executado muito mais rápido porque não utiliza um loop e usa a função embutida [ta.sma()](https://br.tradingview.com/pine-script-reference/v5/#fun_ta{dot}sma) para realizar a tarefa:

```c
//@version=5
indicator("Efficient MA", "", true)
thePineMA = ta.sma(close, 10)
plot(thePineMA)
```

Contar as ocorrências de uma condição nos últimos candles também é uma tarefa que os programadores do Pine Script possam pensar que deve ser feita com um loop. Para contar o número de candles de alta nos últimos 10 candles, usarão:

```c
//@version=5
indicator("Inefficient sum")
MA_LENGTH = 10
upBars = 0.0
for offset = 0 to MA_LENGTH - 1
    if close[offset] > open[offset]
        upBars := upBars + 1
plot(upBars)
```

A forma eficiente de escrever isso no Pine Script (para o programador, pois economiza tempo, para obter os gráficos que carregam mais rápido e para compartilhar recursos comuns de forma mais equitativa), é usar a função integrada [math.sum()](https://br.tradingview.com/pine-script-reference/v5/#fun_math{dot}sum), para realizar a tarefa:

```c
//@version=5
indicator("Efficient sum")
upBars = math.sum(close > open ? 1 : 0, 10)
plot(upBars)
```

O que passa no código é:

- Usa-se o operador ternário [?:](https://br.tradingview.com/pine-script-reference/v5/#op_{question}{colon}) para construir uma expressão que retorna 1 em barras de alta e 0 em outras barras.
- Usa-se a função embutida [math.sum()](https://br.tradingview.com/pine-script-reference/v5/#fun_math{dot}sum) para manter uma soma acumulada desse valor para as últimas 10 barras.

## Quando Loops são Necessários

Os loops existem por uma boa razão, pois mesmo no Pine Script, são necessários em alguns casos. Esses casos geralmente incluem:

- Manipulação de coleções ([arrays](./000_arrays.md), [matrizes](./000_matrices.md) e [mapas](./000_maps.md)).
- Analisar históricamente barras anteriores usando um valor de referência que só pode ser conhecido na barra atual, por exemplo, para descobrir quantos highs anteriores são maiores do que o [high](https://br.tradingview.com/pine-script-reference/v5/#var_high) da barra atual. Como o [high](https://br.tradingview.com/pine-script-reference/v5/#var_high) da barra atual só é conhecido na barra em que o script está sendo executado, um loop é necessário para voltar no tempo e analisar barras anteriores.
- Realizar cálculos em barras anteriores que não podem ser realizados usando funções integradas.

## `for`

A estrutura [for](https://br.tradingview.com/pine-script-reference/v5/#op_for) permite a execução repetitiva de instruções usando um contador.

Sua sintaxe é:

```c
[[<declaration_mode>] [<type>] <identifier> = ]for <identifier> = <expression> to <expression>[ by <expression>]
    <local_block_loop>
```

Onde:

- Elementos entre colchetes (`[]`) podem não aparecer ou aparecer uma vez, e elementos entre chaves (`{}`) podem não aparecer ou aparecer mais vezes.
- `<declaration_mode>` é o [modo de declaração](./04_06_declaracoes_de_variavel.md#modos-de-declaração) da variável.
- `<type>` é opcional, como em quase todas as declarações de variáveis do Pine Script (veja [tipos](./04_09_tipagem_do_sistema.md#tipos)).
- `<identifier>` é o [nome](./04_04_identificadores.md) da variável.
- `<expression>` pode ser um literal, uma variável, uma expressão ou uma chamada de função.
- `<local_block_loop>` consiste em nenhum ou mais instruções seguidas de um valor de retorno, que pode ser uma tupla de valores. Deve ser indentado por quatro espaços ou uma tabulação (_tab_). Pode conter a instrução `break` para sair do loop, ou a instrução `continue` para sair da iteração atual e continuar com a próxima.
- O valor atribuído à variável é o valor de retorno do `<local_block_loop>`, ou seja, o último valor calculado na última iteração do loop, ou [na](https://br.tradingview.com/pine-script-reference/v5/#var_na) se o loop não for executado.
- O identificador em `for <identifier>` é o _valor inicial_ do contador do loop.
- A expressão em `= <expression>` é o _valor de início_ do contador.
- A expressão em `to <expression>` é o _valor final_ do contador. __É avaliado somente ao entrar no loop__.
- A expressão em `by <expression>` é opcional. É o passo pelo qual o contador do loop é incrementado ou decrementado a cada iteração do loop. Seu valor padrão é 1 quando `start value < end value`. É -1 quando `start value > end value`. O passo (+1 ou -1) usado como padrão é determinado pelos valores inicial e final.

O exemplo a seguir usa um comando [for](https://br.tradingview.com/pine-script-reference/v5/#op_for) para retroceder uma quantidade de barras definida pelo usuário para determinar quantas barras têm um [high](https://br.tradingview.com/pine-script-reference/v5/#var_high) maior ou menor que o [high](https://br.tradingview.com/pine-script-reference/v5/#var_high) da última barra no gráfico. Um loop [for](https://br.tradingview.com/pine-script-reference/v5/#op_for) é necessário, pois o script só tem acesso ao valor de referência na última barra do gráfico. O tempo de execução do Pine Script não pode, neste caso, ser usado para calcular dinamicamente, já que o script está executando barra a barra.

```c
//@version=5
indicator("`for` loop")
lookbackInput = input.int(50, "Lookback in bars", minval = 1, maxval = 4999)
higherBars = 0
lowerBars = 0
if barstate.islast
    var label lbl = label.new(na, na, "", style = label.style_label_left)
    for i = 1 to lookbackInput
        if high[i] > high
            higherBars += 1
        else if high[i] < high
            lowerBars += 1
    label.set_xy(lbl, bar_index, high)
    label.set_text(lbl, str.tostring(higherBars, "# higher bars\n") + str.tostring(lowerBars, "# lower bars"))
```

O seguinte exemplo utiliza um loop em sua função `checkLinesForBreaches()` para percorrer uma array de linhas de pivô e excluí-las quando o preço as cruza. Um loop é necessário aqui porque todas as linhas em cada uma dos arrays `hiPivotLines` e `loPivotLines` precisam ser verificadas em cada barra, e não há uma função incorporada que possa fazer isso:

```c
//@version=5
MAX_LINES_COUNT = 100
indicator("Pivot line breaches", "", true, max_lines_count = MAX_LINES_COUNT)

color hiPivotColorInput  = input(color.new(color.lime, 0), "High pivots")
color loPivotColorInput  = input(color.new(color.fuchsia, 0), "Low pivots")
int   pivotLegsInput     = input.int(5, "Pivot legs")
int   qtyOfPivotsInput   = input.int(50, "Quantity of last pivots to remember", minval = 0, maxval = MAX_LINES_COUNT / 2)
int   maxLineLengthInput = input.int(400, "Maximum line length in bars", minval = 2)

// ————— Queues a new element in an array and de-queues its first element.
qDq(array, qtyOfElements, arrayElement) =>
    array.push(array, arrayElement)
    if array.size(array) > qtyOfElements
        // Only deqeue if array has reached capacity.
        array.shift(array)

// —————— Loop through an array of lines, extending those that price has not crossed and deleting those crossed.
checkLinesForBreaches(arrayOfLines) =>
    int qtyOfLines = array.size(arrayOfLines)
    // Don't loop in case there are no lines to check because "to" value will be `na` then`.
    for lineNo = 0 to (qtyOfLines > 0 ? qtyOfLines - 1 : na)
        // Need to check that array size still warrants a loop because we may have deleted array elements in the loop.
        if lineNo < array.size(arrayOfLines)
            line  currentLine    = array.get(arrayOfLines, lineNo)
            float lineLevel      = line.get_price(currentLine, bar_index)
            bool  lineWasCrossed = math.sign(close[1] - lineLevel) != math.sign(close - lineLevel)
            bool  lineIsTooLong  = bar_index - line.get_x1(currentLine) > maxLineLengthInput
            if lineWasCrossed or lineIsTooLong
                // Line stays on the chart but will no longer be extend on further bars.
                array.remove(arrayOfLines, lineNo)
                // Force type of both local blocks to same type.
                int(na)
            else
                line.set_x2(currentLine, bar_index)
                int(na)

// Arrays of lines containing non-crossed pivot lines.
var array<line> hiPivotLines = array.new_line(qtyOfPivotsInput)
var array<line> loPivotLines = array.new_line(qtyOfPivotsInput)

// Detect new pivots.
float hiPivot = ta.pivothigh(pivotLegsInput, pivotLegsInput)
float loPivot = ta.pivotlow(pivotLegsInput, pivotLegsInput)

// Create new lines on new pivots.
if not na(hiPivot)
    line newLine = line.new(bar_index[pivotLegsInput], hiPivot, bar_index, hiPivot, color = hiPivotColorInput)
    line.delete(qDq(hiPivotLines, qtyOfPivotsInput, newLine))
else if not na(loPivot)
    line newLine = line.new(bar_index[pivotLegsInput], loPivot, bar_index, loPivot, color = loPivotColorInput)
    line.delete(qDq(loPivotLines, qtyOfPivotsInput, newLine))

// Extend lines if they haven't been crossed by price.
checkLinesForBreaches(hiPivotLines)
checkLinesForBreaches(loPivotLines)
```

## `while`

A estrutura [while](https://br.tradingview.com/pine-script-reference/v5/#op_while) permite a execução repetitiva de declarações até que uma condição seja falsa. Sua sintaxe é:

```c
[[<declaration_mode>] [<type>] <identifier> = ]while <expression>
    <local_block_loop>
```

Onde:

- Elementos entre colchetes (`[]`) podem não aparecer ou aparecer uma vez.
- `<declaration_mode>` é o [modo de declaração](./04_06_declaracoes_de_variavel.md#modos-de-declaração) da variável.
- `<type>` é opcional, como em quase todas as declarações de variáveis do Pine Script (veja [tipos](./04_09_tipagem_do_sistema.md#tipos)).
- `<identifier>` é o [nome](./04_04_identificadores.md) da variável.
- `<expression>` pode ser um literal, uma variável, uma expressão ou uma chamada de função. É avaliado a cada iteração do loop. Quando ele avalia como `true`, o loop é executado. Quando ele avalia como `false`, o loop para. Observe que a avaliação da expressão é apenas feita antes de cada iteração. As alterações no valor da expressão dentro do loop só terão impacto na próxima iteração.
- `<local_block_loop>` consiste em nenhum ou mais declarações seguidas por um valor de retorno, que pode ser uma tupla de valores. Deve ser indentado por quatro espaços ou uma tabulação (_tab_). Pode conter o comando `break` para sair do loop, ou o comando `continue` para sair da iteração atual e continuar com a próxima.
- O valor atribuído à variável `<identifier>` é o valor de retorno do `<local_block_loop>`, isto é, o último valor calculado na última iteração do loop, ou [na](https://br.tradingview.com/pine-script-reference/v5/#var_na) se o loop não for executado.

Este é o primeiro exemplo de código da seção [for](./04_08_loops.md#for) escrito usando uma estrutura [while](https://br.tradingview.com/pine-script-reference/v5/#op_while) em vez de uma estrutura [for](https://br.tradingview.com/pine-script-reference/v5/#op_for):

```c
//@version=5
indicator("`for` loop")
lookbackInput = input.int(50, "Lookback in bars", minval = 1, maxval = 4999)
higherBars = 0
lowerBars = 0
if barstate.islast
    var label lbl = label.new(na, na, "", style = label.style_label_left)
    // Initialize the loop counter to its start value.
    i = 1
    // Loop until the `i` counter's value is <= the `lookbackInput` value.
    while i <= lookbackInput
        if high[i] > high
            higherBars += 1
        else if high[i] < high
            lowerBars += 1
        // Counter must be managed "manually".
        i += 1
    label.set_xy(lbl, bar_index, high)
    label.set_text(lbl, str.tostring(higherBars, "# higher bars\n") + str.tostring(lowerBars, "# lower bars"))
```

Perceba que:

- O contador `i` deve ser incrementado explicitamente em um dentro do bloco local do [while](https://br.tradingview.com/pine-script-reference/v5/#op_while).
- Usa-se o operador [+=](https://br.tradingview.com/pine-script-reference/v5/#op_{plus}=) para adicionar um ao contador. `lowerBars += 1` é equivalente a `lowerBars := lowerBars + 1`.

Calculando a função fatorial usando uma estrutura de repetição [while](https://br.tradingview.com/pine-script-reference/v5/#op_while):

```c
//@version=5
indicator("")
int n = input.int(10, "Factorial of", minval=0)

factorial(int val = na) =>
    int counter = val
    int fact = 1
    result = while counter > 0
        fact := fact * counter
        counter := counter - 1
        fact

// Only evaluate the function on the first bar.
var answer = factorial(n)
plot(answer)
```

Repare que:

- Utiliza-se [input.int()](https://br.tradingview.com/pine-script-reference/v5/#fun_input{dot}int) para a entrada porque precisa especificar um valor `minval` para proteger o código. Embora [input()](https://br.tradingview.com/pine-script-reference/v5/#fun_input) também suporta a entrada de valores do tipo "int", ele não suporta o parâmetro `minval`.
- Encapsulando a funcionalidade do script em uma função `factorial()` que aceita como argumento o valor cujo fatorial deve calcular. Usando `int val = na` para declarar o parâmetro da função, o que significa que se a função for chamada sem um argumento, como em `factorial()`, então o parâmetro `val` será inicializado como [na](https://br.tradingview.com/pine-script-reference/v5/#var_na), o que impedirá a execução do loop [while](https://br.tradingview.com/pine-script-reference/v5/#op_while) porque sua expressão `counter > 0` retornará [na](https://br.tradingview.com/pine-script-reference/v5/#var_na). A estrutura [while](https://br.tradingview.com/pine-script-reference/v5/#op_while) inicializará a variável de `result` como [na](https://br.tradingview.com/pine-script-reference/v5/#var_na). Por sua vez, como a inicialização do `result` é o valor de retorno do bloco local da função, a função retornará [na](https://br.tradingview.com/pine-script-reference/v5/#var_na).
- Note a última linha do bloco local do [while](https://br.tradingview.com/pine-script-reference/v5/#op_while): `fact`. É o valor de retorno do bloco local, portanto, o valor que ele tinha na última iteração da estrutura [while](https://br.tradingview.com/pine-script-reference/v5/#op_while).
- A inicialização de `result` não é necessária; foi feito isso para legibilidade. Pode-se muito bem ter usado:

```c
while counter > 0
    fact := fact * counter
    counter := counter - 1
    fact
```

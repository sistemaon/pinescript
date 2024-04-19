
# Tipagem do Sistema

O sistema de tipos do Pine Script determina a compatibilidade dos valores de um script com várias funções e operações. Embora seja possível criar scripts simples sabendo nada sobre o sistema de tipagem, um entendimento razoável dele é necessário para alcançar qualquer grau de proficiência da linguagem, e um conhecimento aprofundado de suas sutilezas permitirá que aproveite todo o seu potencial.

O Pine Script utiliza [tipos](./04_09_tipagem_do_sistema.md#tipos) para classificar todos os valores, e utiliza [qualificadores](./04_09_tipagem_do_sistema.md#qualificadores) para determinar se os valores são constantes, estabelecidos na primeira iteração do script, ou dinâmicos ao longo da execução do script. Este sistema se aplica a todos os valores do Pine, incluindo os de literais, variáveis, expressões, retornos de função e argumentos de função.

A tipagem do Sistema está intimamente entrelaçado com o [modelo de execução](./04_01_modelo_de_execucao.md) do Pine e os conceitos de [séries temporais](./04_02_series_temporais.md). Compreender os três é essencial para aproveitar ao máximo o poder do Pine Script.

> ## Note
> Para simplificar, muitas vezes usamos "tipo" para nos referirmos a um "tipo qualificado".


# Qualificadores

Os _qualificadores_ do Pine Script identificam quando um valor é acessível na execução do script:

- Os valores qualificados como [const](./04_09_tipagem_do_sistema.md#const) são estabelecidos em tempo de compilação (ou seja, ao salvar o script no Editor Pine ou adicioná-lo ao gráfico).
- Os valores qualificados como [input](./04_09_tipagem_do_sistema.md#input) estão disponíveis no momento da entrada (ou seja, ao alterar valores na aba "Configurações/Entradas" ("_Settings/Inputs_") do script).
- Os valores qualificados como [simple](./04_09_tipagem_do_sistema.md#simple) são estabelecidos na barra zero (o primeiro candle da execução do script).
- Os valores qualificados como [series](./04_09_tipagem_do_sistema.md#series) podem mudar ao longo da execução do script.

O Pine Script baseia a predominância dos qualificadores de tipo na seguinte hierarquia: __const < input < simple < series__, onde "const" é o qualificador _mais fraco_ (menos restritivo) e "series" é o _mais forte_ (mais restritivo). A hierarquia de qualificadores se traduz na seguinte regra: sempre que uma variável, função ou operação é compatível com um qualificador de tipo específico, valores com qualificadores mais _fracos_ também são permitidos.

Os scripts sempre qualificam os valores retornados de suas expressões com base no qualificador dominante em seus cálculos. Por exemplo, avaliar uma expressão que envolve valores "const" e "series" retornará um valor qualificado como "series". Além disso, os scripts não podem alterar o qualificador de um valor para um que esteja abaixo na hierarquia. Se um valor adquire um qualificador mais forte (por exemplo, um valor inicialmente inferido como "simple" se torna "series" mais tarde na execução do script), esse estado é irreversível.

Note que apenas os valores qualificados como "series" podem mudar ao longo da execução de um script, na qual inclui aos valores provenientes de várias funções integradas, como [close](https://br.tradingview.com/pine-script-reference/v5/#var_close) e [volume](https://br.tradingview.com/pine-script-reference/v5/#var_volume), e também resultados de quaisquer operações que envolvam valores "series". Valores qualificados como "const", "input" ou "simple" são consistentes ao longo da execução de um script.

## `const`

Os valores qualificados como "const" são estabelecidos durante o _tempo de compilação_, antes que o script inicie sua execução. A compilação ocorre inicialmente ao salvar um script no Editor Pine, sem a necessidade de executá-lo em um gráfico. Valores com o qualificador "const" nunca mudam entre as iterações do script, nem mesmo na barra inicial de sua execução.

Os scripts podem qualificar valores como "const" usando um valor _literal_ ou calculando valores a partir de expressões que usam apenas valores literais ou outras variáveis qualificadas como "const".

Exemplos de valores literais:

- _literal int_: `1`, `-1`, `42` .
- _literal float_: `1.`, `1.0`, `3.14`, `6.02E-23`, `3e8` .
- _literal bool_: `true`, `false` .
- _literal color_: `#FF55C6`, `#FF55C6ff` .
- _literal string_: `"A text literal"`, `"Embedded single quotes 'text'"`, `'Embedded double quotes "text"'` .

Os usuários podem definir explicitamente variáveis e parâmetros que aceitam apenas valores "const" incluindo a palavra-chave `const` na declaração.

O [guia de estilo](./000_style_guide.md) recomenda usar __SNAKE_CASE__ em maiúsculas para nomear variáveis "const" para melhor legibilidade. Embora não seja um requisito, também é possível usar a palavra-chave [var](https://br.tradingview.com/pine-script-reference/v5/#kw_var) ao declarar variáveis "const" para que o script as inicialize apenas no _primeiro candle_ do conjunto de dados. Consulte [esta seção](./04_06_declaracoes_de_variavel.md#var-var) do _Manual do Usuário_ para obter mais informações.

Abaixo está um exemplo que utiliza valores "const" dentro das funções [indicator()](https://br.tradingview.com/pine-script-reference/v5/#fun_indicator) e [plot()](https://br.tradingview.com/pine-script-reference/v5/#fun_plot), que ambas requerem um valor do tipo qualificado "const string" como seu argumento `title`:

```c
//@version=5

// The following global variables are all of the "const string" qualified type:

//@variable The title of the indicator.
INDICATOR_TITLE = "const demo"
//@variable The title of the first plot.
var PLOT1_TITLE = "High"
//@variable The title of the second plot.
const string PLOT2_TITLE = "Low"
//@variable The title of the third plot.
PLOT3_TITLE = "Midpoint between " + PLOT1_TITLE + " and " + PLOT2_TITLE

indicator(INDICATOR_TITLE, overlay = true)

plot(high, PLOT1_TITLE)
plot(low, PLOT2_TITLE)
plot(hl2, PLOT3_TITLE)
```

O exemplo a seguir gera um erro de compilação, pois utiliza [syminfo.ticker](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo.ticker), que retorna um valor "simple" porque depende de informações do gráfico que só são acessíveis após o início da execução do script:

```c
//@version=5

//@variable The title in the `indicator()` call.
var NAME = "My indicator for " + syminfo.ticker

indicator(NAME, "", true) // Causes an error because `NAME` is qualified as a "simple string".
plot(close)
```

## `input`

Valores qualificados como "input" são estabelecidos após a inicialização por meio das funções `input.*()`. Essas funções produzem valores que os usuários podem modificar na aba "Inputs" das configurações do script. Quando se altera qualquer um dos valores nesta guia, o script é re-executado desde o início do histórico do gráfico para garantir que seus valores de entrada sejam consistentes ao longo de sua execução.

> ## Note
> A função [input.source()](https://br.tradingview.com/pine-script-reference/v5/#fun_input.source) é uma exceção no espaço de nomes (_namespace_) `input.*()`, pois ela retorna valores qualificados como "series" em vez de "input", uma vez que variáveis integradas como [open](https://br.tradingview.com/pine-script-reference/v5/#var_open), [close](https://br.tradingview.com/pine-script-reference/v5/#var_close), etc., assim como os valores de outros plots de script, são qualificados como "series".

O script a seguir plota o valor de `sourceInput` do `symbolInput` e contexto do `timeframeInput`. A chamada [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security) é válida neste script, pois seus parâmetros `symbol` e `timeframe` permitem argumentos do tipo "simple string", o que significa que também podem aceitar valores do tipo "input string", pois o qualificador "input" é _inferior_ na hierarquia:

```c
//@version=5
indicator("input demo", overlay = true)

//@variable The symbol to request data from. Qualified as "input string".
symbolInput = input.symbol("AAPL", "Symbol")
//@variable The timeframe of the data request. Qualified as "input string".
timeframeInput = input.timeframe("D", "Timeframe")
//@variable The source of the calculation. Qualified as "series float".
sourceInput = input.source(close, "Source")

//@variable The `sourceInput` value from the requested context. Qualified as "series float".
requestedSource = request.security(symbolInput, timeframeInput, sourceInput)

plot(requestedSource)
```

## `simple`

Valores qualificados como "simple" estão disponíveis apenas quando o script inicia a execução na _primeira_ barra do histórico do gráfico, e permanecem consistentes durante a execução do script.

Os usuários podem definir explicitamente variáveis e parâmetros que aceitam valores "simple" incluindo a palavra-chave `simple` em sua declaração.

Muitas variáveis integradas retornam valores qualificados como "simple" porque dependem de informações que um script só pode obter uma vez que ele começa sua execução. Além disso, muitas funções embutidas requerem argumentos "simple" que não alteram ao longo do tempo. Onde quer que um script permita valores "simple", ele também pode aceitar valores qualificados como "input" ou "const".

O script a seguir, destaca o plano de fundo para alertar os usuários de que estão usando um tipo de gráfico não padrão. Ele usa o valor de [chart.is_standard](https://br.tradingview.com/pine-script-reference/v5/#var_chart.is_standard) para calcular a variável `isNonStandard` e, em seguida, usa o valor dessa variável para calcular `warningColor` que também referencia um valor "simple". O parâmetro `color` do [bgcolor()](https://br.tradingview.com/pine-script-reference/v5/#fun_bgcolor) permite um argumento de "series color", o que significa que também pode aceitar um valor de "simple color", já que "simple" está abaixo na hierarquia.

```c
//@version=5
indicator("simple demo", overlay = true)

//@variable Is `true` when the current chart is non-standard. Qualified as "simple bool".
isNonStandard = not chart.is_standard
//@variable Is orange when the the current chart is non-standard. Qualified as "simple color".
simple color warningColor = isNonStandard ? color.new(color.orange, 70) : na

// Colors the chart's background to warn that it's a non-standard chart type.
bgcolor(warningColor, title = "Non-standard chart color")
```

## `series`

Valores qualificados como "series" oferecem a maior flexibilidade em scripts, pois podem mudar em qualquer barra, até mesmo várias vezes na mesma barra.

Os usuários podem definir explicitamente variáveis e parâmetros que aceitam valores "series" ao incluir a palavra-chave `series` em sua declaração.

Variáveis integradas como [open](https://br.tradingview.com/pine-script-reference/v5/#var_open), [high](https://br.tradingview.com/pine-script-reference/v5/#var_high), [low](https://br.tradingview.com/pine-script-reference/v5/#var_low), [close](https://br.tradingview.com/pine-script-reference/v5/#var_close), [volume](https://br.tradingview.com/pine-script-reference/v5/#var_volume), [time](https://br.tradingview.com/pine-script-reference/v5/#var_time) e [bar_index](https://br.tradingview.com/pine-script-reference/v5/#var_bar_index), e o resultado de qualquer expressão que use essas variáveis embutidas, são qualificadas como "series". O resultado de qualquer função ou operação que retorna um valor dinâmico sempre será uma "series", assim como os resultados ao usar o operador de referência histórica [[]](https://br.tradingview.com/pine-script-reference/v5/#op_[]) para acessar valores históricos. Onde quer que um script aceite valores "series", ele também aceitará valores com qualquer outro qualificador, pois "series" é o _maior_ qualificador na hierarquia.

O script abaixo exibe o valor [highest](https://br.tradingview.com/pine-script-reference/v5/#fun_ta.highest) e [lowest](https://br.tradingview.com/pine-script-reference/v5/#fun_ta.lowest) de `sourceInput` ao longo de `lengthInput` barras. Os valores atribuídos às variáveis `highest` e `lowest` são do tipo qualificado "series float", podendo mudar ao longo da execução do script:

```c
//@version=5
indicator("series demo", overlay = true)

//@variable The source value to calculate on. Qualified as "series float".
series float sourceInput = input.source(close, "Source")
//@variable The number of bars in the calculation. Qualified as "input int".
lengthInput = input.int(20, "Length")

//@variable The highest `sourceInput` value over `lengthInput` bars. Qualified as "series float".
series float highest = ta.highest(sourceInput, lengthInput)
//@variable The lowest `sourceInput` value over `lengthInput` bars. Qualified as "series float".
lowest = ta.lowest(sourceInput, lengthInput)

plot(highest, "Highest source", color.green)
plot(lowest, "Lowest source", color.red)
```


# Tipos

Os _tipos_ do Pine Script classificam os valores e determinam as funções e operações com as quais eles são compatíveis. Isto incluem:

- Os tipos fundamentais: [int](./04_09_tipagem_do_sistema.md#int), [float](./04_09_tipagem_do_sistema.md#float), [bool](./04_09_tipagem_do_sistema.md#bool), [color](./04_09_tipagem_do_sistema.md#color) e [string](./04_09_tipagem_do_sistema.md#string).
- Os tipos especiais: [plot](./04_09_tipagem_do_sistema.md#plot-e-hline), [hline](./04_09_tipagem_do_sistema.md#plot-e-hline), [line](./04_09_tipagem_do_sistema.md#tipos-de-desenho), [linefill](./04_09_tipagem_do_sistema.md#tipos-de-desenho), [box](./04_09_tipagem_do_sistema.md#tipos-de-desenho), [polyline](./04_09_tipagem_do_sistema.md#tipos-de-desenho), [label](./04_09_tipagem_do_sistema.md#tipos-de-desenho), [table](./04_09_tipagem_do_sistema.md#tipos-de-desenho), [chart.point](./04_09_tipagem_do_sistema.md#pontos-do-gráfico), [array](./04_09_tipagem_do_sistema.md#coleções), [matrix](./04_09_tipagem_do_sistema.md#coleções) e [map](./04_09_tipagem_do_sistema.md#coleções).
- [Tipos definidos pelo usuário](./04_09_tipagem_do_sistema.md#tipos-definidos-pelo-usuário) (_User-defined types (UDTs)_).
- [vazio](./04_09_tipagem_do_sistema.md#void) (_void_).

Tipos fundamentais se referem à natureza subjacente de um valor, por exemplo, um valor de 1 é do tipo "int", 1.0 é do tipo "float", "AAPL" é do tipo "string", etc. Tipos especiais e tipos definidos pelo usuário utilizam _IDs_ que se referem a objetos de uma classe específica. Por exemplo, um valor do tipo "label" contém um ID que age como um _ponteiro_ referente a um objeto "label". O tipo "void" se refere à saída de uma função ou [método](./04_13_metodos.md) que não retorna um valor utilizável.

Pine Script pode converter automaticamente valores de alguns tipos para outros. As regras de conversão automática são: __int → float → bool__. Veja a seção de [Conversão de Tipo](./04_09_tipagem_do_sistema.md#conversão-de-tipo) para maiores informações.

Na maioria dos casos, o Pine Script pode determinar automaticamente o tipo de um valor. No entanto, também pode-se usar palavras-chave de tipo para especificar _explicitamente_ os tipos para legibilidade e para código que requer definições explícitas (por exemplo, declarar uma variável atribuída a [na](https://br.tradingview.com/pine-script-reference/v5/#var_na)).

Por exemplo:

```c
//@version=5
indicator("Types demo", overlay = true)

//@variable A value of the "const string" type for the `ma` plot's title.
string MA_TITLE = "MA"

//@variable A value of the "input int" type. Controls the length of the average.
int lengthInput = input.int(100, "Length", minval = 2)

//@variable A "series float" value representing the last `close` that crossed over the `ma`.
var float crossValue = na

//@variable A "series float" value representing the moving average of `close`.
float ma = ta.sma(close, lengthInput)
//@variable A "series bool" value that's `true` when the `close` crosses over the `ma`.
bool crossUp = ta.crossover(close, ma)
//@variable A "series color" value based on whether `close` is above or below its `ma`.
color maColor = close > ma ? color.lime : color.fuchsia

// Update the `crossValue`.
if crossUp
    crossValue := close

plot(ma, MA_TITLE, maColor)
plot(crossValue, "Cross value", style = plot.style_circles)
plotchar(crossUp, "Cross Up", "▲", location.belowbar, size = size.small)
```

## `int`

Valores do tipo "int" representam inteiros, isto é, números inteiros sem quaisquer quantidades fracionárias.

Literais inteiros são valores numéricos escritos em notação _decimal_.

Por exemplo:

```c
1
-1
750
```

Variáveis embutidas como [bar_index](https://br.tradingview.com/pine-script-reference/v5/#var_bar_index), [time](https://br.tradingview.com/pine-script-reference/v5/#var_time), [timenow](https://br.tradingview.com/pine-script-reference/v5/#var_timenow), [dayofmonth](https://br.tradingview.com/pine-script-reference/v5/#var_dayofmonth) e [strategy.wintrades](https://br.tradingview.com/pine-script-reference/v5/#var_strategy.wintrades) retornam valores do tipo "int".

## `float`

Valores do tipo "float" representam números de ponto flutuante, ou seja, números que podem conter quantidades inteiras e fracionárias.

Literais de ponto flutuante são valores numéricos escritos com um `.` de delimitador. Eles também podem conter o símbolo `e` ou `E` (que significa "10 elevado à potência de X", onde X é o número após o símbolo `e` ou `E`).

Por exemplo:

```c
3.14159    // Rounded value of Pi (π)
- 3.0
6.02e23    // 6.02 * 10^23 (a very large value)
1.6e-19    // 1.6 * 10^-19 (a very small value)
```

A precisão interna dos valores "float" no Pine Script é de 1e-16.

As variáveis internas como [close](https://br.tradingview.com/pine-script-reference/v5/#var_close), [hlcc4](https://br.tradingview.com/pine-script-reference/v5/#var_hlcc4), [volume](https://br.tradingview.com/pine-script-reference/v5/#var_volume), [ta.vwap](https://br.tradingview.com/pine-script-reference/v5/#var_ta.vwap) e [strategy.position_size](https://br.tradingview.com/pine-script-reference/v5/#var_strategy.position_size) retornam valores do tipo "float".

## `bool`

Valores do tipo "bool" representam o valor de verdade de uma comparação ou condição, os quais os scripts podem usar em [estruturas condicionais](./04_07_estruturas_condicionais.md) e outras expressões.

Existem apenas dois literais que representam valores booleanos:

```c
true    // true value
false   // false value
```

Quando uma expressão do tipo "bool" retorna [na](https://br.tradingview.com/pine-script-reference/v5/#var_na), os scripts tratam seu valor como `false` ao avaliar instruções condicionais e operadores.

Variáveis embutidas como [barstate.isfirst](https://br.tradingview.com/pine-script-reference/v5/#var_barstate.isfirst), [chart.is_heikinashi](https://br.tradingview.com/pine-script-reference/v5/#var_chart.is_heikinashi), [session.ismarket](https://br.tradingview.com/pine-script-reference/v5/#var_session.ismarket) e [timeframe.isdaily](https://br.tradingview.com/pine-script-reference/v5/#var_timeframe.isdaily) retornam valores do tipo "bool".

## `color`

Literais de cor têm o seguinte formato: `#RRGGBB` ou `#RRGGBBAA`. Os pares de letras representam valores _hexadecimais_ entre `00` e `FF` (0 a 255 em decimal).

Onde:

- Os pares `RR`, `GG` e `BB` representam, respectivamente, os valores para os componentes de cor vermelha, verde e azul.
- `AA` é um valor opcional para a opacidade da cor (ou componente _alfa_), onde `00` é invisível e `FF` é opaco. Quando o literal não inclui um par `AA`, o script o trata como totalmente opaco (o mesmo que usar `FF`).
- As letras hexadecimais nos literais podem ser maiúsculas ou minúsculas.

Exemplos de literais de "cor":

```c
#000000      // black color
#FF0000      // red color
#00FF00      // green color
#0000FF      // blue color
#FFFFFF      // white color
#808080      // gray color
#3ff7a0      // some custom color
#FF000080    // 50% transparent red color
#FF0000ff    // same as #FF0000, fully opaque red color
#FF000000    // completely transparent red color
```

Pine Script também possui [constantes de cores integradas](./000_colors.md#constante-de-cores), incluindo [color.green](https://br.tradingview.com/pine-script-reference/v5/#const_color{dot}green), [color.red](https://br.tradingview.com/pine-script-reference/v5/#const_color.red), [color.orange](https://br.tradingview.com/pine-script-reference/v5/#const_color.orange), [color.blue](https://br.tradingview.com/pine-script-reference/v5/#const_color.blue) (a cor padrão nas funções `plot*()` e muitas das propriedades padrão relacionadas à cor nos [tipos de desenho](./04_09_tipagem_do_sistema.md#tipos-de-desenho)), etc.

Ao usar constantes de cores embutidas, é possível adicionar transparência a elas por meio da função [color.new()](https://br.tradingview.com/pine-script-reference/v5/#fun_color.new).

Observe que ao especificar os componentes de cor vermelha, verde ou azul nas funções `color.*()`, usa-se argumentos "int" ou "float" com valores entre 0 e 255. Ao especificar a transparência, utiliza-se um valor entre 0 e 100, onde 0 significa totalmente opaco e 100 significa completamente transparente.

Por exemplo:

```c
//@version=5
indicator("Shading the chart's background", overlay = true)

//@variable A "const color" value representing the base for each day's color.
color BASE_COLOR = color.rgb(0, 99, 165)

//@variable A "series int" value that modifies the transparency of the `BASE_COLOR` in `color.new()`.
int transparency = 50 + int(40 * dayofweek / 7)

// Color the background using the modified `BASE_COLOR`.
bgcolor(color.new(BASE_COLOR, transparency))
```

Para mais informações veja [cores](./000_colors.md) e como usá-los em scripts.

## `string`

Valores do tipo "string" representam sequências de letras, números, símbolos, espaços e outros caracteres.

Literais de string no Pine são caracteres contidos entre aspas simples ou duplas.

Por exemplo:

```c
"This is a string literal using double quotes."
'This is a string literal using single quotes.'
```

Aspas simples e duplas são equivalentes em função no Pine Script. Uma "string" entre aspas duplas pode conter qualquer quantidade de aspas simples e vice-versa:

```c
"It's an example"
'The "Star" indicator'
```

Os scripts podem usar a barra invertida (`\`) para incluir o delimitador de fechamento em uma "string".

Por exemplo:

```c
'It\'s an example'
"The \"Star\" indicator"
```

Pode-se criar valores de "string" contendo o caractere de _escape_ de nova linha (`\n`) para exibir texto em várias linhas com funções `plot*()` e `log.*()` e objetos de [tipos de desenho](./04_09_tipagem_do_sistema.md#tipos-de-desenho).

Por exemplo:

```c
"This\nString\nHas\nOne\nWord\nPer\nLine"
```

Pode-se usar o operador [+](https://br.tradingview.com/pine-script-reference/v5/#op_+) para concatenar valores de "string":

```c
"This is a " + "concatenated string."
```

Os elementos incorporados no espaço de nomes (_namespace_) `str.*()` criam valores de "string" usando operações especializadas. Por exemplo, o script a seguir cria uma _string formatada_ para representar valores de preço "float" e exibe o resultado usando um rótulo:

```c
//@version=5
indicator("Formatted string demo", overlay = true)

//@variable A "series string" value representing the bar's OHLC data.
string ohlcString = str.format("Open: {0}\nHigh: {1}\nLow: {2}\nClose: {3}", open, high, low, close)

// Draw a label containing the `ohlcString`.
label.new(bar_index, high, ohlcString, textcolor = color.white)
```

Para mais informações sobre exibir valores "string" a partir de um script veja [Texto e Formas](./000_text_and_shapes.md).

As variáveis internas como [syminfo.tickerid](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo.tickerid), [syminfo.currency](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo.currency) e [timeframe.period](https://br.tradingview.com/pine-script-reference/v5/#var_timeframe.period) retornam valores do tipo "string".

## `plot` e `hline`

As funções [plot()](https://br.tradingview.com/pine-script-reference/v5/#fun_plot) e [hline()](https://br.tradingview.com/pine-script-reference/v5/#fun_hline) do Pine Script retornam IDs que referenciam respectivamente instâncias dos tipos "plot" e "hline". Esses tipos exibem valores calculados e níveis horizontais no gráfico, e pode-se atribuir seus IDs a variáveis para uso com a função [fill()](https://br.tradingview.com/pine-script-reference/v5/#fun_fill) integrada.

Por exemplo, o script a seguir plota duas _EMAs_ no gráfico e preenche o espaço entre elas usando a função [fill()](https://br.tradingview.com/pine-script-reference/v5/#fun_fill):

```c
//@version=5
indicator("plot fill demo", overlay = true)

//@variable A "series float" value representing a 10-bar EMA of `close`.
float emaFast = ta.ema(close, 10)
//@variable A "series float" value representing a 20-bar EMA of `close`.
float emaSlow = ta.ema(close, 20)

//@variable The plot of the `emaFast` value.
emaFastPlot = plot(emaFast, "Fast EMA", color.orange, 3)
//@variable The plot of the `emaSlow` value.
emaSlowPlot = plot(emaSlow, "Slow EMA", color.gray, 3)

// Fill the space between the `emaFastPlot` and `emaSlowPlot`.
fill(emaFastPlot, emaSlowPlot, color.new(color.purple, 50), "EMA Fill")
```

É importante notar que, ao contrário de outros tipos especiais, não há palavras-chave `plot` ou `hline` em Pine para declarar explicitamente o tipo de variável como "plot" ou "hline".

Os usuários podem controlar onde os gráficos de seus scripts são exibidos por meio das variáveis no `display.*`. Além disso, um script pode usar os valores dos gráficos de outro script como _entradas externas_ por meio da função [input.source()](https://br.tradingview.com/pine-script-reference/v5/#fun_input.source) (consulte [entradas de origem](./000_inputs.md#input-de-origem) para maiores informações).

## Tipos de Desenho

Os tipos de desenho do Pine Script permitem que os scripts criem desenhos personalizados nos gráficos. Isto incluem: [line](https://br.tradingview.com/pine-script-reference/v5/#type_line), [linefill](https://br.tradingview.com/pine-script-reference/v5/#type_linefill), [box](https://br.tradingview.com/pine-script-reference/v5/#type_box), [polyline](https://br.tradingview.com/pine-script-reference/v5/#type_polyline), [label](https://br.tradingview.com/pine-script-reference/v5/#type_label) e [table](https://br.tradingview.com/pine-script-reference/v5/#type_table).

Cada tipo também possui um espaço de nomes que contém todos os construtores embutidos que criam e gerenciam instâncias de desenhos. Por exemplo, os seguintes construtores `*.new()` criam novos objetos desses tipos em um script: [line.new()](https://br.tradingview.com/pine-script-reference/v5/#fun_line.new), [linefill.new()](https://br.tradingview.com/pine-script-reference/v5/#fun_linefill.new), [box.new()](https://br.tradingview.com/pine-script-reference/v5/#fun_box.new), [polyline.new()](https://br.tradingview.com/pine-script-reference/v5/#fun_polyline.new), [label.new()](https://br.tradingview.com/pine-script-reference/v5/#fun_label.new) e [table.new()](https://br.tradingview.com/pine-script-reference/v5/#fun_table.new).

Cada uma dessas funções retorna um _ID_ que é uma referência que identifica unicamente um objeto de desenho. Os IDs são sempre qualificados como "series", significando que seus tipos qualificados são "series line", "series label", etc. Os IDs de desenho agem como ponteiros, pois cada ID referencia uma instância específica de um desenho em todas as funções do espaço de nomes desse desenho. Por exemplo, o ID de uma linha retornado por uma chamada [line.new()](https://br.tradingview.com/pine-script-reference/v5/#fun_line.new) é usado posteriormente para se referir a esse objeto específico quando é hora de excluí-lo com [line.delete()](https://br.tradingview.com/pine-script-reference/v5/#fun_line.delete).

## Pontos do Gráfico

Pontos de gráfico são tipos especiais que representam coordenadas no gráfico. Os scripts usam as informações de objetos [chart.point](https://br.tradingview.com/pine-script-reference/v5/#type_chart.point) para determinar as localizações no gráfico de [lines](./000_lines_and_boxes.md#lines), [boxes](./000_lines_and_boxes.md#boxes), [polylines](./000_lines_and_boxes.md#polylines), e [labels](./000_text_and_shapes.md#labels).

Objetos deste tipo contêm três _campos_: `time`, `index` e `price` (_tempo, índice e preço_). Se uma instância de desenho usa o campo `time` ou `price` de um [chart.point](https://br.tradingview.com/pine-script-reference/v5/#type_chart.point) como _coordenada-x_ depende da propriedade `xloc` do desenho.

Pode-se usar qualquer uma das seguintes funções para criar pontos de gráfico em um script:

- [chart.point.new()](https://br.tradingview.com/pine-script-reference/v5/#fun_chart.point.new) -> Cria um novo [chart.point](https://br.tradingview.com/pine-script-reference/v5/#type_chart.point) com `time`, `index` e `price` especificados.
- [chart.point.now()](https://br.tradingview.com/pine-script-reference/v5/#fun_chart.point.now) -> Cria um novo [chart.point](https://br.tradingview.com/pine-script-reference/v5/#type_chart.point) com uma _coordenada-y_ de `price` especificada. Os campos de `time` e `index` contêm o [time](https://br.tradingview.com/pine-script-reference/v5/#var_time) e o [bar_index](https://br.tradingview.com/pine-script-reference/v5/#var_bar_index) da barra em que a função é executada.
- [chart.point_from_index()](https://br.tradingview.com/pine-script-reference/v5/#fun_chart.point.from_index) -> Cria um novo [chart.point](https://br.tradingview.com/pine-script-reference/v5/#type_chart.point) com uma _coordenada-x_ de `index` e uma _coordenada-y_ de `price` especificadas. O campo de `time` da instância resultante é [na](https://br.tradingview.com/pine-script-reference/v5/#var_na), o que significa que não funcionará com objetos de desenho que usam um valor `xloc` de [xloc.bar_time](https://br.tradingview.com/pine-script-reference/v5/#var_xloc.bar_time).
- [chart.point.from_time()](https://br.tradingview.com/pine-script-reference/v5/#fun_chart.point.from_time) -> Cria um novo [chart.point](https://br.tradingview.com/pine-script-reference/v5/#type_chart.point) com uma _coordenada-x_ de `time` e uma _coordenada-y_ de `price` especificadas. O campo de `index` da instância resultante é [na](https://br.tradingview.com/pine-script-reference/v5/#var_na), o que significa que não funcionará com objetos de desenho que usam um valor `xloc` de [xloc.bar_index](https://br.tradingview.com/pine-script-reference/v5/#var_xloc.bar_index).
- [chart.point.copy()](https://br.tradingview.com/pine-script-reference/v5/#fun_chart.point.copy) -> Cria um novo [chart.point](https://br.tradingview.com/pine-script-reference/v5/#type_chart.point) contendo as mesmas informações de `time`, `index` e `price` que o id na chamada da função.

O exemplo seguinte, desenha linhas conectando o [high](https://br.tradingview.com/pine-script-reference/v5/#var_high) da barra anterior com o [low](https://br.tradingview.com/pine-script-reference/v5/#var_low) da barra atual em cada barra do gráfico. Também exibe labels (_rótulos_) em ambos os pontos de cada linha. A linha e os labels obtêm informações das variáveis `firstPoint` e `secondPoint`, que referenciam pontos de gráfico criados usando [chart.point_from_index()](https://br.tradingview.com/pine-script-reference/v5/#fun_chart.point.from_index) e [chart.point.now()](https://br.tradingview.com/pine-script-reference/v5/#fun_chart.point.now):

```c
//@version=5
indicator("Chart points demo", overlay = true)

//@variable A new `chart.point` at the previous `bar_index` and `high`.
firstPoint = chart.point.from_index(bar_index - 1, high[1])
//@variable A new `chart.point` at the current bar's `low`.
secondPoint = chart.point.now(low)

// Draw a new line connecting coordinates from the `firstPoint` and `secondPoint`.
// This line uses the `index` fields from the points as x-coordinates.
line.new(firstPoint, secondPoint, color = color.purple, width = 3)
// Draw a label at the `firstPoint`. Uses the point's `index` field as its x-coordinate.
label.new(
     firstPoint, str.tostring(firstPoint.price), color = color.green,
     style = label.style_label_down, textcolor = color.white
 )
// Draw a label at the `secondPoint`. Uses the point's `index` field as its x-coordinate.
label.new(
     secondPoint, str.tostring(secondPoint.price), color = color.red,
     style = label.style_label_up, textcolor = color.white
 )
```

## Coleções

Coleções no Pine Script ([arrays](./04_14_arrays.md), [matrices](./04_15_matrices.md), e [maps](./04_16_mapas.md)) (_arrays, matrizes e mapas_) utilizam IDs de referência, assim como outros tipos especiais (por exemplo, labels). O tipo do ID define o tipo de _elementos_ que a coleção conterá. No Pine, especifica-se os tipos de array, matrix, e map (_array, matriz e mapa_) acrescentando um [modelo de tipo](./04_09_tipagem_do_sistema.md#templates-de-tipo) às palavras-chave [array](https://br.tradingview.com/pine-script-reference/v5/#type_array), [matrix](https://br.tradingview.com/pine-script-reference/v5/#type_matrix) ou [map](https://br.tradingview.com/pine-script-reference/v5/#type_map):

- `array<int>` define um array contendo elementos do tipo “int”.
- `array<label>` define um array contendo IDs de “label”.
- `array<UDT>` define um array contendo IDs referenciando objetos de um [tipo definido pelo usuário (UDT)](./04_09_tipagem_do_sistema.md#tipos-definidos-pelo-usuário).
- `matrix<float>` define uma _matrix_ contendo elementos do tipo “float”.
- `matrix<UDT>` define uma _matrix_ contendo IDs referenciando objetos de um [tipo definido pelo usuário (UDT)](./04_09_tipagem_do_sistema.md#tipos-definidos-pelo-usuário).
- `map<string, float>` define um mapa contendo chaves do tipo “string” e valores do tipo “float”.
- `map<int, UDT>` define um mapa contendo chaves do tipo “int” e IDs de instâncias de [tipo definido pelo usuário (UDT)](./04_09_tipagem_do_sistema.md#tipos-definidos-pelo-usuário) como valores.

Pode-se declarar um array de "int" com um único elemento de valor 10 de qualquer uma das seguintes formas, por exemplo:

```c
a1 = array.new<int>(1, 10)
array<int> a2 = array.new<int>(1, 10)
a3 = array.from(10)
array<int> a4 = array.from(10)
```

Note que:

- A sintaxe `int[]` também pode especificar um array de elementos "int", mas seu uso é desencorajado. Não existe equivalente para especificar os tipos de _matrix_ ou mapas dessa maneira.
- Existe tipo específico integrado para [arrays](https://br.tradingview.com/pine-script-reference/v5/#type_array), como [array.new_int()](https://br.tradingview.com/pine-script-reference/v5/#fun_array{dot}new_int), mas a forma mais genérica [array.new<type>](https://br.tradingview.com/pine-script-reference/v5/#fun_array.new%3Ctype%3E) é preferível, o que seria `array.new<int>()` para criar um array de elementos "int".

## Tipos Definidos pelo Usuário

A palavra-chave [type](https://br.tradingview.com/pine-script-reference/v5/#op_type) permite a criação de _tipos definidos pelo usuário (UDTs (User-Defined Types))_ na qual scripts podem criar [objetos](./04_12_objetos.md). _UDTs_ são tipos compostos; eles contêm um número arbitrário de _campos_ que podem ser de qualquer tipo, incluindo outros _tipos definidos pelo usuário_.

A sintaxe para definir um tipo definido pelo usuário é:

```c
[export] type <UDT_identifier>
    <field_type> <field_name> [= <value>]
    ...
```

Onde:

- `export` é a palavra-chave que um script de [biblioteca](https://br.tradingview.com/pine-script-reference/v5/#fun_library) usa para exportar o tipo definido pelo usuário. Para saber mais sobre a exportação de _UDTs_, veja [Bibliotecas](./000_library.md#tipos-e-objetos-definidos-pelo-usuário).
- `<UDT_identifier>` é o nome do tipo definido pelo usuário.
- `<field_type>` é o tipo do campo.
- `<field_name>` é o nome do campo.
- `<value>` é um valor padrão opcional para o campo, em que o script atribuirá a ele ao criar novos objetos desse _UDT_. Se não fornecer um valor, o padrão do campo será [na](https://br.tradingview.com/pine-script-reference/v5/#var_na). As mesmas regras que regem os valores padrão dos parâmetros nas assinaturas de funções se aplicam aos valores padrão dos campos. Por exemplo, os valores padrão de um _UDT_ não podem usar resultados do operador de referência histórica [[]](https://br.tradingview.com/pine-script-reference/v5/#op_[]) ou expressões.

O exemplo seguinte, declara um _UDT (User-Defined Type)_ chamado `pivotPoint` com um campo `pivotTime` do tipo "int" e um campo `priceLevel` do tipo "float", que irão armazenar, respectivamente, informações de tempo e preço sobre um pivot calculado:

```c
//@type             A user-defined type containing pivot information.
//@field pivotTime  Contains time information about the pivot.
//@field priceLevel Contains price information about the pivot.
type pivotPoint
    int   pivotTime
    float priceLevel
```

Os tipos definidos pelo usuário suportam `recursão de tipo`, ou seja, os campos de um _UDT_ podem referenciar objetos do mesmo tipo de _UDT_.

No exemplo abaixo, foi adicionado um campo `nextPivot` ao nosso tipo `pivotPoint` anterior que faz referência a outra instância de `pivotPoint`:

```c
//@type             A user-defined type containing pivot information.
//@field pivotTime  Contains time information about the pivot.
//@field priceLevel Contains price information about the pivot.
//@field nextPivot  A `pivotPoint` instance containing additional pivot information.
type pivotPoint
    int        pivotTime
    float      priceLevel
    pivotPoint nextPivot
```

Os scripts podem usar dois métodos integrados para criar e copiar _UDTs_: `new()` e `copy()`. Veja [Objetos](./04_12_objetos.md) para saber mais sobre _UDTs_.

## void

Existe um tipo "void" no Pine Script. Funções que têm apenas efeitos adicionais e não retornam nenhum resultado utilizável retornam o tipo "void". Um exemplo de é a função [alert()](https://br.tradingview.com/pine-script-reference/v5/#fun_alert); fazendo algo (dispara um evento de alerta), mas não retorna valor utilizável.

Scripts não podem usar resultados "void" em expressões ou atribuí-los a variáveis. Não existe a palavra-chave `void` no Pine Script, já que não é possível declarar uma variável do tipo "void".

## valor `na`

Existe um valor especial no Pine Script chamado [na](https://br.tradingview.com/pine-script-reference/v5/#var_na), que é um acrônimo para _not available_ (_não disponível_). Usa-se [na](https://br.tradingview.com/pine-script-reference/v5/#var_na) para representar um valor undefined (_indefinido_) de uma variável ou expressão. É semelhante a `null` em Java e `None` em Python.

Os scripts podem automaticamente converter valores [na](https://br.tradingview.com/pine-script-reference/v5/#var_na) para quase qualquer tipo. No entanto, em alguns casos, o compilador não pode inferir o tipo associado a um valor [na](https://br.tradingview.com/pine-script-reference/v5/#var_na) porque mais de uma regra de conversão de tipo pode ser aplicável.

Por exemplo:

```c
// Compilation error!
myVar = na
```

A linha de código acima causa um erro de compilação porque o compilador não pode determinar a natureza da variável `myVar`, ou seja, se a variável fará referência a valores numéricos para plotagem, valores de string para definição de texto em uma label ou outros valores para algum outro propósito posterior na execução do script.

Para resolver tais erros, devemos declarar explicitamente o tipo associado à variável. Suponha que a variável `myVar` fará referência a valores "float" em iterações de script subsequentes. Podemos resolver o erro declarando a variável com a palavra-chave [float](https://br.tradingview.com/pine-script-reference/v5/#type_float):

```c
float myVar = na
```

ou por conversão explícito do valor [na](https://br.tradingview.com/pine-script-reference/v5/#var_na) para o tipo "float" através da função [float()](https://br.tradingview.com/pine-script-reference/v5/#fun_float):

```c
myVar = float(na)
```

Para testar se o valor de uma variável ou expressão é [na](https://br.tradingview.com/pine-script-reference/v5/#var_na), pode chamar a função [na()](https://br.tradingview.com/pine-script-reference/v5/#fun_na), que retorna `true` se o valor for indefinido.

Por exemplo:

```c
//@variable Is 0 if the `myVar` is `na`, `close` otherwise.
float myClose = na(myVar) ? 0 : close
```

Não utilize o operador de comparação `==` para testar valores [na](https://br.tradingview.com/pine-script-reference/v5/#var_na), pois os scripts não podem determinar a igualdade de um valor indefinido:

```c
//@variable Returns the `close` value. The script cannot compare the equality of `na` values, as they're undefined.
float myClose = myVar == na ? 0 : close
```

As melhores práticas de programação frequentemente envolvem o tratamento de valores [na](https://br.tradingview.com/pine-script-reference/v5/#var_na) para evitar valores indefinidos em cálculos.

Por exemplo, a linha de código seguinte verifica se o valor de [close](https://br.tradingview.com/pine-script-reference/v5/#var_close) na barra atual é maior que o valor da barra anterior:

```c
//@variable Is `true` when the `close` exceeds the last bar's `close`, `false` otherwise.
bool risingClose = close > close[1]
```

Na primeira barra do gráfico, o valor de `risingClose` é [na](https://br.tradingview.com/pine-script-reference/v5/#var_na), já que não há nenhum valor de [close](https://br.tradingview.com/pine-script-reference/v5/#var_close) anterior para referência.

Pode-se garantir que a expressão também retorne um valor utilizável na primeira barra substituindo o valor passado indefinido por um valor da barra atual. Esta linha de código seguinte, utiliza a função [nz()](https://br.tradingview.com/pine-script-reference/v5/#fun_nz) para substituir o [close](https://br.tradingview.com/pine-script-reference/v5/#var_close) da barra passada pelo [open](https://br.tradingview.com/pine-script-reference/v5/#var_open) da barra atual quando o valor for [na](https://br.tradingview.com/pine-script-reference/v5/#var_na):

```c
//@variable Is `true` when the `close` exceeds the last bar's `close` (or the current `open` if the value is `na`).
bool risingClose = close > nz(close[1], open)
```

Proteger scripts contra instâncias [na](https://br.tradingview.com/pine-script-reference/v5/#var_na) ajuda a evitar que valores indefinidos se propaguem nos resultados de um cálculo.

Por exemplo, o script a seguir, declara uma variável `allTimeHigh` na primeira barra. Em seguida, utiliza o [math.max()](https://br.tradingview.com/pine-script-reference/v5/#fun_math.max) entre `allTimeHigh` e o [high](https://br.tradingview.com/pine-script-reference/v5/#var_high) da barra para atualizar o `allTimeHigh` ao longo de sua execução:

```c
//@version=5
indicator("na protection demo", overlay = true)

//@variable The result of calculating the all-time high price with an initial value of `na`.
var float allTimeHigh = na

// Reassign the value of the `allTimeHigh`.
// Returns `na` on all bars because `math.max()` can't compare the `high` to an undefined value.
allTimeHigh := math.max(allTimeHigh, high)

plot(allTimeHigh) // Plots `na` on all bars.
```

O script abaixo, plota um valor de [na](https://br.tradingview.com/pine-script-reference/v5/#var_na) em todas as barras, pois não foi incluido alguma proteção contra [na](https://br.tradingview.com/pine-script-reference/v5/#var_na) no código. Para corrigir o comportamento e plotar o resultado pretendido (ou seja, o máximo de todos os tempos dos preços do gráfico), pode-se usar [nz()](https://br.tradingview.com/pine-script-reference/v5/#fun_nz) para substituir os valores [na](https://br.tradingview.com/pine-script-reference/v5/#var_na) na série `allTimeHigh`:

```c
//@version=5
indicator("na protection demo", overlay = true)

//@variable The result of calculating the all-time high price with an initial value of `na`.
var float allTimeHigh = na

// Reassign the value of the `allTimeHigh`.
// We've used `nz()` to prevent the initial `na` value from persisting throughout the calculation.
allTimeHigh := math.max(nz(allTimeHigh), high)

plot(allTimeHigh)
```


# Templates de Tipo

Os _modelos de tipo_ especificam os tipos de dados que as coleções ([arrays](./04_14_arrays.md), [matrices](./04_15_matrices.md), e [maps](./04_16_mapas.md)) (_arrays, matrizes e mapas_) podem conter.

Modelos para [arrays](./04_14_arrays.md) e [matrices](./04_15_matrices.md) consistem de _um único_ tipo de identificador especificado entre os símbolos `<` e `>`, por exemplo, `<int>`, `<label>` e `<PivotPoint>` (onde `PivotPoint` é um [tipo definido pelo usuário (UDT)](./04_09_tipagem_do_sistema.md#tipos-definidos-pelo-usuário)).

Templates para [maps](./04_16_mapas.md) consistem em _dois_ tipos de identificadores de especificado entre os símbolos `<` e `>`, onde o primeiro especifica o tipo das _chaves_ em cada par de chave-valor, e o segundo especifica o tipo do _valor_. Por exemplo, `<string, float>` é um modelo de tipo para um mapa que contém chaves de `string` e valores `float`.

Os usuários podem construir templates de tipo a partir de:

- Tipos fundamentais: [int](./04_09_tipagem_do_sistema.md#int), [float](./04_09_tipagem_do_sistema.md#float), [bool](./04_09_tipagem_do_sistema.md#bool), [color](./04_09_tipagem_do_sistema.md#color) e [string](./04_09_tipagem_do_sistema.md#string).
- Os seguintes tipos especiais: [line](./04_09_tipagem_do_sistema.md#tipos-de-desenho), [linefill](./04_09_tipagem_do_sistema.md#tipos-de-desenho), [box](./04_09_tipagem_do_sistema.md#tipos-de-desenho), [polyline](./04_09_tipagem_do_sistema.md#tipos-de-desenho), [label](./04_09_tipagem_do_sistema.md#tipos-de-desenho), [table](./04_09_tipagem_do_sistema.md#tipos-de-desenho) e [chart.point](./04_09_tipagem_do_sistema.md#pontos-do-gráfico).
- [Tipos definidos pelo usuário (UDTs)](./04_09_tipagem_do_sistema.md#tipos-definidos-pelo-usuário).

Note que:

- [Maps](./04_16_mapas.md) podem usar qualquer um desses tipos como _valores_, mas só podem aceitar tipos fundamentais como _chaves_.

Os scripts usam templates de tipo para declarar variáveis que apontam para coleções, e ao criar novas instâncias de coleções.

Por exemplo:

```c
//@version=5
indicator("Type templates demo")

//@variable A variable initially assigned to `na` that accepts arrays of "int" values.
array<int> intArray = na
//@variable An empty matrix that holds "float" values.
floatMatrix = matrix.new<float>()
//@variable An empty map that holds "string" keys and "color" values.
stringColorMap = map.new<string, color>()
```


# Conversão de Tipo

O Pine Script inclui um mecanismo automático de conversão de tipo que _converte_ valores "__int__" para "__float__" quando necessário. Variáveis ou expressões que requerem valores "float" também podem usar valores "int" porque qualquer número inteiro pode ser representado como um número de ponto flutuante com sua parte fracionária igual a 0.

Por questões de compatibilidade com versões anteriores, o Pine Script também converte automaticamente valores "__int__" e "__float__" para "__bool__" quando necessário. Ao passar valores numéricos para os parâmetros de funções e operações que esperam tipos "bool", o Pine os converte automaticamente para "bool". No entanto, não é recomendado depender desse comportamento. A maioria dos scripts que convertem automaticamente valores numéricos para o tipo "bool" irá gerar um _compiler warning_ (_aviso do compilador_). É possível evitar o aviso do compilador e promover a legibilidade do código usando a função [bool()](https://br.tradingview.com/pine-script-reference/v5/#fun_bool), que converte explicitamente um valor numérico para o tipo "bool".

Ao converter um valor "int" ou "float" para "bool", um valor de 0 é convertido para `false` e qualquer outro valor numérico sempre é convertido para `true`.

O código abaixo demonstra o comportamento de autoconversão obsoleto no Pine. Ele cria uma variável `randomValue` com um valor "series float" em cada barra, que é passada para o parâmetro de `condition` (_condição_) em uma estrutura [if](https://br.tradingview.com/pine-script-reference/v5/#kw_if) e o parâmetro `series` na chamada de função [plotchar()](https://br.tradingview.com/pine-script-reference/v5/#fun_plotchar). Como ambos os parâmetros aceitam valores "bool", o script converte automaticamente o `randomValue` para "bool" ao avaliá-los:

```c
//@version=5
indicator("Auto-casting demo", overlay = true)

//@variable A random rounded value between -1 and 1.
float randomValue = math.round(math.random(-1, 1))
//@variable The color of the chart background.
color bgColor = na

// This raises a compiler warning since `randomValue` is a "float", but `if` expects a "bool".
if randomValue
    bgColor := color.new(color.blue, 60)
// This does not raise a warning, as the `bool()` function explicitly casts the `randomValue` to "bool".
if bool(randomValue)
    bgColor := color.new(color.blue, 60)

// Display unicode characters on the chart based on the `randomValue`.
// Whenever `math.random()` returns 0, no character will appear on the chart because 0 converts to `false`.
plotchar(randomValue)
// We recommend explicitly casting the number with the `bool()` function to make the type transformation more obvious.
plotchar(bool(randomValue))

// Highlight the background with the `bgColor`.
bgcolor(bgColor)
```

Às vezes, é necessário converter um tipo para outro quando as regras de autoconversão não são suficientes. Para esses casos, as seguintes funções de conversão de tipo estão disponíveis: [int()](https://br.tradingview.com/pine-script-reference/v5/#fun_int), [float()](https://br.tradingview.com/pine-script-reference/v5/#fun_float), [bool()](https://br.tradingview.com/pine-script-reference/v5/#fun_bool), [color()](https://br.tradingview.com/pine-script-reference/v5/#fun_color), [string()](https://br.tradingview.com/pine-script-reference/v5/#fun_string), [line()](https://br.tradingview.com/pine-script-reference/v5/#fun_line), [linefill()](https://br.tradingview.com/pine-script-reference/v5/#fun_linefill), [label()](https://br.tradingview.com/pine-script-reference/v5/#fun_label), [box()](https://br.tradingview.com/pine-script-reference/v5/#fun_box), e [table()](https://br.tradingview.com/pine-script-reference/v5/#fun_table).

O exemplo abaixo mostra um código que tenta usar um valor "const float" como argumento `length` na chamada da função [ta.sma()](https://br.tradingview.com/pine-script-reference/v5/#fun_ta.sma). O script falhará na compilação, pois não é possível converter automaticamente o valor "float" para o tipo "int" necessário:

```c
//@version=5
indicator("Explicit casting demo", overlay = true)

//@variable The length of the SMA calculation. Qualified as "const float".
float LENGTH = 10.0

float sma = ta.sma(close, LENGTH) // Compilation error. The `length` parameter requires an "int" value.

plot(sma)
```

O código gera o seguinte erro: "_Cannot call ‘ta.sma’ with argument ‘length’=’LENGTH’. An argument of ‘const float’ type was used but a ‘series int’ is expected._" ("_Não é possível chamar 'ta.sma' com o argumento 'length'='LENGTH'. Foi usado um argumento do tipo 'const float', mas é esperado um 'series int'._").

O compilador está informando que o código está usando um valor "float" onde é necessário um "int". Não há uma regra de auto-conversão para converter um "float" em um "int", então deve-se fazer o trabalho manualmente.

Nesta versão do código, usa-se a função [int()](https://br.tradingview.com/pine-script-reference/v5/#fun_int) para converter explicitamente o valor `LENGTH` "float" para o tipo "int" dentro da chamada de [ta.sma()](https://br.tradingview.com/pine-script-reference/v5/#fun_ta.sma):

```c
//@version=5
indicator("explicit casting demo")

//@variable The length of the SMA calculation. Qualified as "const float".
float LENGTH = 10.0

float sma = ta.sma(close, int(LENGTH)) // Compiles successfully since we've converted the `LENGTH` to "int".

plot(sma)
```

A conversão explícita de tipo também é útil ao declarar variáveis atribuídas a [na](https://br.tradingview.com/pine-script-reference/v5/#var_na), como explicado na [seção anterior](./04_09_tipagem_do_sistema.md#valor-na).

Por exemplo, poderia-se declarar explicitamente uma variável com um valor de [na](https://br.tradingview.com/pine-script-reference/v5/#var_na) como tipo "label" de qualquer uma das seguintes maneiras, equivalentes:

```c
// Explicitly specify that the variable references "label" objects:
label myLabel = na

// Explicitly cast the `na` value to the "label" type:
myLabel = label(na)
```


# Tuples

Uma _tupla_ é um conjunto de expressões separadas por vírgulas, envolvidas por colchetes. Quando uma função, [método](./04_13_metodos.md) ou outro bloco local retorna mais de um valor, os scripts retornam esses valores na forma de uma tupla.

Por exemplo, a seguinte [função definida pelo usuário](./04_11_funcoes_definida_pelo_usuario.md) retorna a soma e o produto de dois valores "float":

```c
//@function Calculates the sum and product of two values.
calcSumAndProduct(float a, float b) =>
    //@variable The sum of `a` and `b`.
    float sum = a + b
    //@variable The product of `a` and `b`.
    float product = a * b
    // Return a tuple containing the `sum` and `product`.
    [sum, product]
```

Ao chamar esta função posteriormente no script, usa-se uma _declaração de tupla_ para declarar várias variáveis correspondentes aos valores retornados pela chamada da função:

```c
// Declare a tuple containing the sum and product of the `high` and `low`, respectively.
[hlSum, hlProduct] = calcSumAndProduct(high, low)
```

Tenha em mente que, ao contrário da declaração de variáveis individuais, não pode-se definir explicitamente os tipos das variáveis da tupla (neste caso; `hlSum` e `hlProduct`). O compilador automaticamente infere os tipos associados às variáveis em uma tupla.

No exemplo acima, o resultado da tupla contém valores do mesmo tipo ("float"). No entanto, é importante observar que as tuplas podem conter valores de _tipos múltiplos_.

Por exemplo, a função `chartInfo()` abaixo retorna uma tupla contendo valores do tipo "int", "float", "bool", "color" e "string":

```c
//@function Returns information about the current chart.
chartInfo() =>
    //@variable The first visible bar's UNIX time value.
    int firstVisibleTime = chart.left_visible_bar_time
    //@variable The `close` value at the `firstVisibleTime`.
    float firstVisibleClose = ta.valuewhen(ta.cross(time, firstVisibleTime), close, 0)
    //@variable Is `true` when using a standard chart type, `false` otherwise.
    bool isStandard = chart.is_standard
    //@variable The foreground color of the chart.
    color fgColor = chart.fg_color
    //@variable The ticker ID of the current chart.
    string symbol = syminfo.tickerid
    // Return a tuple containing the values.
    [firstVisibleTime, firstVisibleClose, isStandard, fgColor, symbol]
```

Tuplas são especialmente úteis para solicitar múltiplos valores em uma única chamada [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security).

Por exemplo, esta função `roundedOHLC()` retorna uma tupla contendo valores __OHLC__ arredondados para os preços mais próximos que são divisíveis pelo valor [mintick](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo.mintick) (_mínimo do tick_) do símbolo. Chamando esta função como o argumento de `expression` em [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security) para solicitar uma tupla contendo valores __OHLC__ diários:

```c
//@function Returns a tuple of OHLC values, rounded to the nearest tick.
roundedOHLC() =>
    [math.round_to_mintick(open), math.round_to_mintick(high), math.round_to_mintick(low), math.round_to_mintick(close)]

[op, hi, lo, cl] = request.security(syminfo.tickerid, "D", roundedOHLC())
```

Também consegue-se obter o mesmo resultado passando diretamente uma tupla de valores arredondados como a `expression` na chamada de [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security).

```c
[op, hi, lo, cl] = request.security(
     syminfo.tickerid, "D",
     [math.round_to_mintick(open), math.round_to_mintick(high), math.round_to_mintick(low), math.round_to_mintick(close)]
 )
```

Blocos locais de [estruturas condicionais](./04_07_estruturas_condicionais.md), incluindo instruções [if](https://br.tradingview.com/pine-script-reference/v5/#kw_if) e [switch](https://br.tradingview.com/pine-script-reference/v5/#kw_switch), podem retornar tuplas.

Por exemplo:

```c
[v1, v2] = if close > open
    [high, close]
else
    [close, low]
```

e:

```c
[v1, v2] = switch
close > open => [high, close]
=>              [close, low]
```

No entanto, ternários não podem conter tuplas, pois os valores de retorno em uma declaração ternária não são considerados blocos locais:

```c
// Not allowed.
[v1, v2] = close > open ? [high, close] : [close, low]
```

Observe que todos os itens dentro de uma tupla retornada de uma função são qualificados como "simple" ou "series", dependendo de seu conteúdo. Se uma tupla contiver um valor "series", todos os outros elementos dentro da tupla também adotarão o qualificador "series".

Por exemplo:

```c
//@version=5
indicator("Qualified types in tuples demo")

makeTicker(simple string prefix, simple string ticker) =>
    tId = prefix + ":" + ticker // simple string
    source = close  // series float
    [tId, source]

// Both variables are series now.
[tId, source] = makeTicker("BATS", "AAPL")

// Error cannot call 'request.security' with 'series string' tId.
r = request.security(tId, "", source)

plot(r)
```

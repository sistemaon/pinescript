
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

## Pontos do gráfico

## Coleções

## Tipos Definidos pelo Usuário

## Tipos Definidos pelo Usuário

## void

## valor `na`

# Templates de Tipo

# Conversão de Tipo

# Tuples

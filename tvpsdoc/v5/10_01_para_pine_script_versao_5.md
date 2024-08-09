
# Para Pine Script Versão 5

Este guia documenta as __mudanças__ feitas no Pine Script da versão v4 para a v5. Ele guiará na adaptação de scripts Pine existentes para o Pine Script v5. Consulte as [Notas de lançamento](./09_notas_de_versao.md) para uma lista dos __novos__ recursos do Pine Script v5.

As adaptações mais frequentes necessárias para converter scripts antigos para a v5 são:

- Substituir [study()](https://br.tradingview.com/pine-script-reference/v4/#fun_study) por [indicator()](https://br.tradingview.com/pine-script-reference/v5/#fun_indicator) (a assinatura da função não mudou).
- Renomear chamadas de funções incorporadas para incluir seus novos namespaces (por exemplo, [highest()](https://br.tradingview.com/pine-script-reference/v4/#fun_highest) na v4 torna-se [ta.highest()](https://br.tradingview.com/pine-script-reference/v5/#fun_ta%7Bdot%7Dhighest) na v5).
- Reestruturar entradas para usar as funções `input.*()` mais especializadas.
- Eliminar o uso do parâmetro obsoleto `transp` usando [color.new()](https://br.tradingview.com/pine-script-reference/v5/#fun_color%7Bdot%7Dnew) para definir simultaneamente a cor e a transparência para uso com o parâmetro `color`.
- Se foram usados os parâmetros `resolution` e `resolution_gaps` na função [study()](https://br.tradingview.com/pine-script-reference/v4/#fun_study) da v4, será necessário mudá-los para `timeframe` e `timeframe_gaps` na função [indicator()](https://br.tradingview.com/pine-script-reference/v5/#fun_indicator) da v5.

## Conversor de v4 para v5

O Pine Editor inclui uma ferramenta para converter automaticamente scripts da v4 para v5. Para acessá-la, abra um script que contenha `//@version=4` e selecione a opção "Converter para v5" "_Convert to v5_" no menu "Mais" "_More_" identificado por três pontos no canto superior direito do painel do Editor:

![Conversor de v4 para v5](https://br.tradingview.com/pine-script-docs/_astro/v4_to_v5_convert_button.B6QYFNWm_1b9vBk.webp)

Nem todos os scripts podem ser convertidos automaticamente de v4 para v5. Se for necessário converter o script manualmente ou se o indicador retornar um erro de compilação após a conversão, use as seções a seguir para determinar como completar a conversão. Uma lista de alguns erros que podem ser encontrados durante a conversão automática e como corrigi-los pode ser encontrada na seção [Erros comuns de conversão de script](./10_01_para_pine_script_versao_5.md#erros-comuns-de-conversão-de-scripts) deste guia.

## Funções e Variáveis Renomeadas

Para maior clareza e consistência, muitas funções e variáveis incorporadas foram renomeadas na v5. A inclusão dos nomes de funções da v4 em um novo namespace é a causa da maioria das mudanças. Por exemplo, a função [sma()](https://br.tradingview.com/pine-script-reference/v4/#fun_sma) na v4 foi movida para o namespace `ta.` na v5: [ta.sma()](https://br.tradingview.com/pine-script-reference/v5/#fun_ta%7Bdot%7Dsma). Não é necessário memorizar os novos namespaces; se o nome antigo de uma função for digitado sem o namespace no Editor e a tecla de atalho 'Auto-completar' (`Ctrl` + `Space`, ou `Cmd` no MacOS) for pressionada, aparecerá um popup mostrando sugestões correspondentes:

![Funções e variáveis renomeadas](https://br.tradingview.com/pine-script-docs/_astro/v5_autocomplete.R-4HP09V_Z1R3GAs.webp)

Desconsiderando as funções movidas para novos namespaces, apenas duas funções foram renomeadas:

- `study()` agora é [indicator()](https://br.tradingview.com/pine-script-reference/v5/#fun_indicator).
- `tickerid()` agora é [ticker.new()](https://br.tradingview.com/pine-script-reference/v5/#fun_ticker%7Bdot%7Dnew).

A lista completa de funções e variáveis renomeadas pode ser encontrada na seção [Todas as alterações de nomes de variáveis, funções e parâmetros](./10_01_para_pine_script_versao_5.md#fun) deste guia.

## Parâmetros de função renomeados

Os nomes dos parâmetros de algumas funções incorporadas foram alterados para melhorar a nomenclatura. Isso não afeta a maioria dos scripts, mas se esses nomes de parâmetros foram usados ao chamar funções, será necessário adaptá-los. Por exemplo, todas as menções foram padronizadas:

```c
// Valid in v4. Not valid in v5.
timev4 = time(resolution = "1D")
// Valid in v5.
timev5 = time(timeframe = "1D")
// Valid in v4 and v5.
timeBoth = time("1D")
```

A lista completa de parâmetros de função renomeados pode ser encontrada na seção [Todas as alterações de nomes de variáveis, funções e parâmetros](./10_01_para_pine_script_versao_5.md#todas-as-mudanças-de-nomes-de-variáveis-funções-e-parâmetros) deste guia.

## Sobrecarga Removida do `rsi()`

Na v4, a função [rsi()](https://br.tradingview.com/pine-script-reference/v4/#fun_rsi) tinha duas sobrecargas diferentes:

- `rsi(series float, simple int)` para o cálculo normal do RSI, e;
- `rsi(series float, series float)` para uma sobrecarga usada no indicador MFI, que realizava um cálculo equivalente a `100.0 - (100.0 / (1.0 + arg1 / arg2))`.

Isso fazia com que uma única função incorporada se comportasse de duas maneiras muito diferentes, e era difícil distinguir qual se aplicava porque dependia do tipo do segundo argumento. Como resultado, vários indicadores usavam a função de forma incorreta e exibiam resultados incorretos. Para evitar isso, a segunda sobrecarga foi removida na v5.

A função [ta.rsi()](https://br.tradingview.com/pine-script-reference/v5/#fun_ta%7Bdot%7Drsi) na v5 aceita apenas um argumento "simple int" para seu parâmetro `length`. Se o código v4 utilizava a sobrecarga agora obsoleta da função com um segundo argumento `float`, é possível substituir toda a chamada `rsi()` pela fórmula a seguir, que é equivalente:

```c
100.0 - (100.0 / (1.0 + arg1 / arg2))
```

Observe que quando o código v4 utilizava um valor "series int" como segundo argumento para [rsi()](https://br.tradingview.com/pine-script-reference/v4/#fun_rsi), ele era automaticamente convertido para "series float" e a segunda sobrecarga da função era usada. Embora isso fosse sintaticamente correto, provavelmente __não__ produzia o resultado esperado. Na v5, [ta.rsi()](https://br.tradingview.com/pine-script-reference/v5/#fun_ta%7Bdot%7Drsi) requer um "simple int" para o argumento `length`, o que exclui comprimentos dinâmicos (ou "series"). Isso ocorre porque os cálculos do RSI usam a média móvel [ta.rma()](https://br.tradingview.com/pine-script-reference/v5/#fun_ta%7Bdot%7Drma), que é semelhante a [ta.ema()](https://br.tradingview.com/pine-script-reference/v5/#fun_ta%7Bdot%7Dema) no sentido de que depende de um processo recursivo baseado no comprimento usando os valores de barras anteriores. Isso torna impossível obter resultados corretos com um comprimento "series" que possa variar de barra para barra.

Se o código v4 usava um comprimento que era "const int", "input int" ou "simple int", nenhuma mudança é necessária.

## Palavras-Chave Reservadas

Várias palavras são reservadas e não podem ser usadas para nomes de variáveis ou funções. Elas são: `catch`, `class`, `do`, `ellipse`, `in`, `is`, `polygon`, `range`, `return`, `struct`, `text`, `throw`, `try`. Se o indicador v4 usar alguma delas, renomeie sua variável ou função para que o script funcione na v5.

## Remoção de `iff()` e `offset()`

As funções [iff()](https://br.tradingview.com/pine-script-reference/v4/#fun_iff) e [offset()](https://br.tradingview.com/pine-script-reference/v4/#fun_offset) foram removidas. O código que usava a função [iff()](https://br.tradingview.com/pine-script-reference/v4/#fun_iff) pode ser reescrito usando o operador ternário:

```c
// iff(<condition>, <return_when_true>, <return_when_false>)
// Valid in v4, not valid in v5
barColorIff = iff(close >= open, color.green, color.red)
// <condition> ? <return_when_true> : <return_when_false>
// Valid in v4 and v5
barColorTernary = close >= open ? color.green : color.red
```

Observe que o operador ternário é avaliado de forma "preguiçosa"; apenas o valor necessário é calculado (dependendo da avaliação da condição como `true` ou `false`). Isso é diferente de [iff()](https://br.tradingview.com/pine-script-reference/v4/#fun_iff), que sempre avaliava ambos os valores, mas retornava apenas o relevante.

Algumas funções requerem avaliação em todas as barras para calcular corretamente, então será necessário tomar providências especiais para essas, pré-avaliando-as antes do uso do operador ternário:

```c
// `iff()` in v4: `highest()` and `lowest()` are calculated on every bar
v1 = iff(close > open, highest(10), lowest(10)) 
plot(v1)
// In v5: forced evaluation on every bar prior to the ternary statement.
h1 = ta.highest(10)
l1 = ta.lowest(10)
v1 = close > open ? h1 : l1
plot(v1)
```

A função [offset()](https://br.tradingview.com/pine-script-reference/v4/#fun_offset) foi depreciada porque o operador [[]](https://br.tradingview.com/pine-script-reference/v5/#op_%5B%5D), mais legível, é equivalente:

```c
// Valid in v4. Not valid in v5.
prevClosev4 = offset(close, 1)
// Valid in v4 and v5.
prevClosev5 = close[1]
```

## Divisão de `input()` em Várias Funções

A função [input()](https://br.tradingview.com/pine-script-reference/v4/#fun_input) da v4 estava se tornando sobrecarregada com uma infinidade de sobrecargas e parâmetros. Foi dividido sua funcionalidade em diferentes funções para liberar espaço e fornecer uma estrutura mais robusta para acomodar as adições planejadas para entradas. Cada nova função usa o nome do tipo `input.*` da chamada `input()` da v4 que substitui. Por exemplo, agora há uma função especializada [input.float()](https://br.tradingview.com/pine-script-reference/v5/#fun_input%7Bdot%7Dfloat) substituindo a chamada `input(1.0, type = input.float)` da v4. Observe que ainda é possível usar `input(1.0)` na v5, mas como apenas [input.float()](https://br.tradingview.com/pine-script-reference/v5/#fun_input%7Bdot%7Dfloat) permite parâmetros como `minval`, `maxval`, etc., ela é mais poderosa. Também observe que [input.int()](https://br.tradingview.com/pine-script-reference/v5/#fun_input%7Bdot%7Dint) é a única função de entrada especializada que não usa o nome `input.integer` equivalente da v4. As constantes `input.*` foram removidas porque eram usadas como argumentos para o parâmetro `type`, que foi depreciado.

Para converter, por exemplo, um script v4 que usa uma entrada do tipo `input.symbol`, a função [input.symbol()](https://br.tradingview.com/pine-script-reference/v5/#fun_input%7Bdot%7Dsymbol) deve ser usada na v5:

```c
// Valid in v4. Not valid in v5.
aaplTicker = input("AAPL", type = input.symbol)
// Valid in v5
aaplTicker = input.symbol("AAPL")
```

A função [input()](https://br.tradingview.com/pine-script-reference/v5/#fun_input) persiste na v5, mas em uma forma mais simples, com menos parâmetros. Ela tem a vantagem de detectar automaticamente os tipos de entrada "bool/color/int/float/string/source" a partir do argumento usado para `defval`:

```c
// Valid in v4 and v5.
// While "AAPL" is a valid symbol, it is only a string here because `input.symbol()` is not used.
tickerString = input("AAPL", title = "Ticker string")
```

## Alguns Parâmetros de Função Agora Exigem Argumentos Incorporados

Na v4, constantes embutidas como `plot.style_area` usadas como argumentos ao chamar funções do Pine Script™ correspondiam a valores predefinidos de um tipo específico. Por exemplo, o valor de `barmerge.lookahead_on` era `true`, então era possível usar `true` em vez da constante nomeada ao fornecer um argumento para o parâmetro `lookahead` em uma chamada da função [security()](https://br.tradingview.com/pine-script-reference/v4/#fun_security). Descoberto que isso era uma fonte comum de confusão, o que levava programadores desavisados a produzir código que gerava resultados não intencionais.

Na v5, o uso de constantes nomeadas corretas como argumentos para parâmetros de função que exigem é obrigatório:

```c
// Not valid in v5: `true` is used as an argument for `lookahead`.
request.security(syminfo.tickerid, "1D", close, lookahead = true)
// Valid in v5: uses a named constant instead of `true`.
request.security(syminfo.tickerid, "1D", close, lookahead = barmerge.lookahead_on)

// Would compile in v4 because `plot.style_columns` was equal to 5.
// Won't compile in v5.
a = 2 * plot.style_columns
plot(a)
```

Para converter o script da v4 para a v5, certifique-se de usar as constantes embutidas nomeadas corretas como argumentos de função.

## Parâmetro `transp` Depreciado

O parâmetro `transp=` usado na assinatura de muitas funções de plotagem da v4 foi depreciado porque interferia na funcionalidade RGB. Agora, a transparência deve ser especificada junto com a cor como argumento para parâmetros como `color`, `textcolor`, etc. As funções [color.new()](https://br.tradingview.com/pine-script-reference/v5/#fun_color%7Bdot%7Dnew) ou [color.rgb()](https://br.tradingview.com/pine-script-reference/v5/#fun_color%7Bdot%7Drgb) serão necessárias nesses casos para combinar uma cor e sua transparência.

Observe que na v4, as funções [bgcolor()](https://br.tradingview.com/pine-script-reference/v5/#fun_bgcolor) e [fill()](https://br.tradingview.com/pine-script-reference/v5/#fun_fill) tinham um parâmetro `transp` opcional que usava um valor padrão de 90. Isso significava que o código abaixo podia exibir Bandas de Bollinger com um preenchimento semitransparente entre duas bandas e uma cor de fundo semitransparente onde as bandas cruzam o preço, mesmo sem nenhum argumento para o parâmetro `transp` em suas chamadas de [bgcolor()](https://br.tradingview.com/pine-script-reference/v5/#fun_bgcolor) e [fill()](https://br.tradingview.com/pine-script-reference/v5/#fun_fill):

```c
//@version=4
study("Bollinger Bands", overlay = true)
[middle, upper, lower] = bb(close, 5, 4)
plot(middle, color=color.blue)
p1PlotID = plot(upper, color=color.green)
p2PlotID = plot(lower, color=color.green)
crossUp = crossover(high, upper)
crossDn = crossunder(low, lower)
// Both `fill()` and `bgcolor()` have a default `transp` of 90
fill(p1PlotID, p2PlotID, color = color.green)
bgcolor(crossUp ? color.green : crossDn ? color.red : na)  
```

Na v5, é preciso mencionar explicitamente a transparência 90 com a cor, resultando em:

```c
//@version=5
indicator("Bollinger Bands", overlay = true)
[middle, upper, lower] = ta.bb(close, 5, 4)
plot(middle, color=color.blue)
p1PlotID = plot(upper, color=color.green)
p2PlotID = plot(lower, color=color.green)
crossUp = ta.crossover(high, upper)
crossDn = ta.crossunder(low, lower)
var TRANSP = 90
// We use `color.new()` to explicitly pass transparency to both functions
fill(p1PlotID, p2PlotID, color = color.new(color.green, TRANSP))
bgcolor(crossUp ? color.new(color.green, TRANSP) : crossDn ? color.new(color.red, TRANSP) : na)
```

## Mudança nos Dias de Sessão Padrão para `time()` e `time_close()`

O conjunto padrão de dias para strings de `session` usadas nas funções [time()](https://br.tradingview.com/pine-script-reference/v5/#fun_time) e [time_close()](https://br.tradingview.com/pine-script-reference/v5/#fun_time_close), e retornadas por [input.session()](https://br.tradingview.com/pine-script-reference/v5/#fun_input%7Bdot%7Dsession), foi alterado de `"23456"` (segunda a sexta) para `"1234567"` (domingo a sábado):

```c
// On symbols that are traded during weekends, this will behave differently in v4 and v5.
t0 = time("1D", "1000-1200")
// v5 equivalent of the behavior of `t0` in v4.
t1 = time("1D", "1000-1200:23456")
// v5 equivalent of the behavior of `t0` in v5.
t2 = time("1D", "1000-1200:1234567")
```

Essa mudança de comportamento não deve ter muito impacto em scripts executados em mercados convencionais que estão fechados nos finais de semana. Se for importante garantir que suas definições de sessão preservem o comportamento da v4 no código v5, adicione `":23456"` às suas strings de sessão. Veja a página [Sessões](./05_17_sessoes.md) deste manual para mais informações.

## `strategy.exit()` Agora Deve Fazer Algo

Os dias em que a função [strategy.exit()](https://br.tradingview.com/pine-script-reference/v5/#fun_strategy%7Bdot%7Dexit) podia ficar ociosa se foram. Agora ela deve realmente ter um efeito na estratégia, usando pelo menos um dos seguintes parâmetros: `profit`, `limit`, `loss`, `stop`, ou um dos seguintes pares: `trail_offset` combinado com `trail_price` ou `trail_points`. Quando usos de [strategy.exit()](https://br.tradingview.com/pine-script-reference/v5/#fun_strategy%7Bdot%7Dexit) que não atendem a esses critérios gerarem um erro ao converter uma estratégia para v5, você pode eliminar essas linhas com segurança, pois elas não faziam nada em seu código de qualquer maneira.

## Erros Comuns de Conversão de Scripts

### Argumento Inválido 'style'/'linestyle' na Chamada de 'plot'/'hline'

Para corrigir esse problema, é necessário substituir os argumentos "int" usados para os parâmetros `style` e `linestyle` nas funções [plot()](https://br.tradingview.com/pine-script-reference/v5/#fun_plot) e [hline()](https://br.tradingview.com/pine-script-reference/v5/#fun_hline) por constantes embutidas:

```c
// Will cause an error during conversion
plotStyle = input(1)
hlineStyle = input(1)
plot(close, style = plotStyle)
hline(100, linestyle = hlineStyle)

// Will work in v5
//@version=5
indicator("")
plotStyleInput = input.string("Line", options = ["Line", "Stepline", "Histogram", "Cross", "Area", "Columns", "Circles"])
hlineStyleInput = input.string("Solid", options = ["Solid", "Dashed", "Dotted"])

plotStyle = plotStyleInput == "Line" ? plot.style_line : 
      plotStyleInput == "Stepline" ? plot.style_stepline :
      plotStyleInput == "Histogram" ? plot.style_histogram :
      plotStyleInput == "Cross" ? plot.style_cross :
      plotStyleInput == "Area" ? plot.style_area :
      plotStyleInput == "Columns" ? plot.style_columns :
      plot.style_circles

hlineStyle = hlineStyleInput == "Solid" ? hline.style_solid :
      hlineStyleInput == "Dashed" ? hline.style_dashed :
      hline.style_dotted

plot(close, style = plotStyle)
hline(100, linestyle = hlineStyle) 
```

Veja a seção [Alguns parâmetros de função agora exigem argumentos embutidos](./10_01_para_pine_script_versao_5.md#alguns-parâmetros-de-função-agora-exigem-argumentos-incorporados) deste guia para mais informações.

### Identificador não Declarado 'input.%input_name%'

Para corrigir esse problema, remova as constantes `input.*` do seu código:

```c
// Will cause an error during conversion
_integer = input.integer
_bool = input.bool
i1 = input(1, "Integer", _integer)
i2 = input(true, "Boolean", _bool)

// Will work in v5
i1 = input.int(1, "Integer")
i2 = input.bool(true, "Boolean")
```

Veja a página do Manual do Usuário sobre [Entradas](./05_09_inputs.md) e a seção [Alguns parâmetros de função agora exigem argumentos embutidos](./10_01_para_pine_script_versao_5.md#alguns-parâmetros-de-função-agora-exigem-argumentos-incorporados) deste guia para mais informações.

### Argumento Inválido 'when' na Chamada de 'strategy.close'

Este problema é causado por uma confusão entre [strategy.entry()](https://br.tradingview.com/pine-script-reference/v5/#fun_strategy%7Bdot%7Dentry) e [strategy.close()](https://br.tradingview.com/pine-script-reference/v5/#fun_strategy%7Bdot%7Dclose).

O segundo parâmetro de [strategy.close()](https://br.tradingview.com/pine-script-reference/v5/#fun_strategy%7Bdot%7Dclose) é `when`, que espera um argumento "bool". Na v4, era permitido usar `strategy.long` como argumento porque era um "bool". No entanto, na v5, constantes embutidas nomeadas devem ser usadas como argumentos, então `strategy.long` não é mais permitido como argumento para o parâmetro `when`.

A chamada `strategy.close("Short", strategy.long)` neste código é equivalente a `strategy.close("Short")`, que é o que deve ser usado na v5:

```c
// Will cause an error during conversion
if (longCondition)
    strategy.close("Short", strategy.long)
    strategy.entry("Long", strategy.long)

// Will work in v5:
if (longCondition)
    strategy.close("Short")
    strategy.entry("Long", strategy.long)  
```

Veja a seção [Alguns parâmetros de função agora exigem argumentos embutidos](./10_01_para_pine_script_versao_5.md#alguns-parâmetros-de-função-agora-exigem-argumentos-incorporados) deste guia para mais informações.

### Não é Possível Chamar 'input.int' com o Argumento 'minval'='%value%'. Foi Usado um Argumento do Tipo 'literal float', mas um 'const int' é Esperado

Na v4, era possível passar um argumento "float" para `minval` ao inserir um valor "int". Isso não é mais possível na v5; valores "int" são exigidos para entradas "int":

```c
// Works in v4, will break on conversion because minval is a 'float' value
int_input = input(1, "Integer", input.integer, minval = 1.0)

// Works in v5
int_input = input.int(1, "Integer", minval = 1)  
```

Veja a página do Manual do Usuário sobre [Entradas](./05_09_inputs.md) e a seção [Alguns parâmetros de função agora exigem argumentos embutidos](./10_01_para_pine_script_versao_5.md#alguns-parâmetros-de-função-agora-exigem-argumentos-incorporados) deste guia para mais informações.

## Todas as Mudanças de Nomes de Variáveis, Funções e Parâmetros

### Funções e Variáveis Removidas

| v4                                       | v5                                  |
|------------------------------------------|-------------------------------------|
| `input.bool` input                       | Substituído por `input.bool()`      |
| `input.color` input                      | Substituído por `input.color()`     |
| `input.float` input                      | Substituído por `input.float()`     |
| `input.integer` input                    | Substituído por `input.int()`       |
| `input.resolution` input                 | Substituído por `input.timeframe()` |
| `input.session` input                    | Substituído por `input.session()`   |
| `input.source` input                     | Substituído por `input.source()`    |
| `input.string` input                     | Substituído por `input.string()`    |
| `input.symbol` input                     | Substituído por `input.symbol()`    |
| `input.time` input                       | Substituído por `input.time()`      |
| `iff()`                                  | Use o operador `?:`                 |
| `offset()`                               | Use o operador `[]`                 |
|                                          |                                     |

### Funções e Parâmetros Renomeados

#### Sem Mudança de Namespace

| v4 | v5 |
| --- | --- |
| `study(<...>, resolution, resolution_gaps, <...>)` | `indicator(<...>, timeframe, timeframe_gaps, <...>)` |
| `strategy.entry(long)` | `strategy.entry(direction)` |
| `strategy.order(long)` | `strategy.order(direction)` |
| `time(resolution)` | `time(timeframe)` |
| `time_close(resolution)` | `time_close(timeframe)` |
| `nz(x, y)` | `nz(source, replacement)` |
|            |                           |

#### Namespace "ta" para Funções e Variáveis de Análise Técnica

##### Funções e Variáveis de Indicadores

| v4 | v5 |
| --- | --- |
| `accdist` | `ta.accdist` |
| `alma()` | `ta.alma()` |
| `atr()` | `ta.atr()` |
| `bb()` | `ta.bb()` |
| `bbw()` | `ta.bbw()` |
| `cci()` | `ta.cci()` |
| `cmo()` | `ta.cmo()` |
| `cog()` | `ta.cog()` |
| `dmi()` | `ta.dmi()` |
| `ema()` | `ta.ema()` |
| `hma()` | `ta.hma()` |
| `iii` | `ta.iii` |
| `kc()` | `ta.kc()` |
| `kcw()` | `ta.kcw()` |
| `linreg()` | `ta.linreg()` |
| `macd()` | `ta.macd()` |
| `mfi()` | `ta.mfi()` |
| `mom()` | `ta.mom()` |
| `nvi` | `ta.nvi` |
| `obv` | `ta.obv` |
| `pvi` | `ta.pvi` |
| `pvt` | `ta.pvt` |
| `rma()` | `ta.rma()` |
| `roc()` | `ta.roc()` |
| `rsi(x, y)` | `ta.rsi(source, length)` |
| `sar()` | `ta.sar()` |
| `sma()` | `ta.sma()` |
| `stoch()` | `ta.stoch()` |
| `supertrend()` | `ta.supertrend()` |
| `swma(x)` | `ta.swma(source)` |
| `tr` | `ta.tr` |
| `tr()` | `ta.tr()` |
| `tsi()` | `ta.tsi()` |
| `vwap` | `ta.vwap` |
| `vwap(x)` | `ta.vwap(source)` |
| `vwma()` | `ta.vwma()` |
| `wad` | `ta.wad` |
| `wma()` | `ta.wma()` |
| `wpr()` | `ta.wpr()` |
| `wvad` | `ta.wvad` |
|        |           |

##### Funções de Suporte

| v4 | v5 |
| --- | --- |
| `barsince()` | `ta.barsince()` |
| `change()` | `ta.change()` |
| `correlation(source_a, source_b, length)` | `ta.correlation(source1, source2, length)` |
| `cross(x, y)` | `ta.cross(source1, source2)` |
| `crossover(x, y)` | `ta.crossover(source1, source2)` |
| `crossunder(x, y)` | `ta.crossunder(source1, source2)` |
| `cum(x)` | `ta.cum(source)` |
| `dev()` | `ta.dev()` |
| `falling()` | `ta.falling()` |
| `highest()` | `ta.highest()` |
| `highestbars()` | `ta.highestbars()` |
| `lowest()` | `ta.lowest()` |
| `lowestbars()` | `ta.lowestbars()` |
| `median()` | `ta.median()` |
| `mode()` | `ta.mode()` |
| `percentile_linear_interpolation()` | `ta.percentile_linear_interpolation()` |
| `percentile_nearest_rank()` | `ta.percentile_nearest_rank()` |
| `percentrank()` | `ta.percentrank()` |
| `pivothigh()` | `ta.pivothigh()` |
| `pivotlow()` | `ta.pivotlow()` |
| `range()` | `ta.range()` |
| `rising()` | `ta.rising()` |
| `stdev()` | `ta.stdev()` |
| `valuewhen()` | `ta.valuewhen()` |
| `variance()` | `ta.variance()` |
|              |                 |

#### Namespace "math" para Funções e Variáveis Relacionadas à Matemática

| v4 | v5 |
| --- | --- |
| `abs(x)` | `math.abs(number)` |
| `acos(x)` | `math.acos(number)` |
| `asin(x)` | `math.asin(number)` |
| `atan(x)` | `math.atan(number)` |
| `avg()` | `math.avg()` |
| `ceil(x)` | `math.ceil(number)` |
| `cos(x)` | `math.cos(angle)` |
| `exp(x)` | `math.exp(number)` |
| `floor(x)` | `math.floor(number)` |
| `log(x)` | `math.log(number)` |
| `log10(x)` | `math.log10(number)` |
| `max()` | `math.max()` |
| `min()` | `math.min()` |
| `pow()` | `math.pow()` |
| `random()` | `math.random()` |
| `round(x, precision)` | `math.round(number, precision)` |
| `round_to_mintick(x)` | `math.round_to_mintick(number)` |
| `sign(x)` | `math.sign(number)` |
| `sin(x)` | `math.sin(angle)` |
| `sqrt(x)` | `math.sqrt(number)` |
| `sum()` | `math.sum()` |
| `tan(x)` | `math.tan(angle)` |
| `todegrees()` | `math.todegrees()` |
| `toradians()` | `math.toradians()` |
|               |                    |

#### Namespace "request" para Funções que Solicitam Dados Externos

| v4 | v5 |
| --- | --- |
| `financial()` | `request.financial()` |
| `quandl()` | `request.quandl()` |
| `security(<...>, resolution, <...>)` | `request.security(<...>, timeframe, <...>)` |
| `splits()` | `request.splits()` |
| `dividends()` | `request.dividends()` |
| `earnings()` | `request.earnings()` |
|              |                      |

#### Namespace "ticker" para Funções que Ajudam a Criar Tickers

| v4 | v5 |
| --- | --- |
| `heikinashi()` | `ticker.heikinashi()` |
| `kagi()` | `ticker.kagi()` |
| `linebreak()` | `ticker.linebreak()` |
| `pointfigure()` | `ticker.pointfigure()` |
| `renko()` | `ticker.renko()` |
| `tickerid()` | `ticker.new()` |
|              |                |

#### Namespace "str" para Funções que Manipulam Strings

| v4 | v5 |
| --- | --- |
| `tostring(x, y)` | `str.tostring(value, format)` |
| `tonumber(x)` | `str.tonumber(string)` |
|               |                        |

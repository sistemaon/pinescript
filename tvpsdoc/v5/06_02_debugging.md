
# Debugging (_Depuração_)

A integração próxima entre o Pine Editor e a interface de gráficos do TradingView facilita a depuração interativa e eficiente do código Pine Script, permitindo que scripts produzam resultados dinâmicos em vários locais, dentro e fora do gráfico. Programadores podem utilizar esses resultados para refinar os comportamentos do script e garantir que tudo funcione conforme esperado.

Quando um programador entende as técnicas apropriadas para inspecionar os diversos comportamentos que podem ser encontrados ao escrever um script, é possível identificar e resolver rapidamente potenciais problemas no código, proporcionando uma experiência de codificação mais tranquila. Esta página demonstra algumas das formas mais úteis de depurar código ao trabalhar com Pine Script.

> __Observação!__\
> Antes de prosseguir, recomenda-se familiarizar-se com o [Modelo de Execução](./04_01_modelo_de_execucao.md) e o [Sistema de Tipos](./04_09_tipagem_do_sistema.md) do Pine Script, pois é crucial entender esses detalhes ao depurar no ambiente Pine Script.

## A Disposição do Terreno

Scripts Pine podem gerar seus resultados de várias maneiras diferentes, que os programadores podem utilizar para depuração.

As funções `plot*()` podem exibir resultados em um painel de gráfico, na linha de status do script, na escala de preços (eixo-y) e na "Janela de Dados" "_Data Window_", fornecendo maneiras simples e convenientes de depurar valores numéricos e condicionais:

![A disposição do terreno 01](./imgs/Debugging-The-lay-of-the-land-1.BMFtSmLt_rnRYW.webp)

```c
//@version=5
indicator("The lay of the land - Plots")

// Plot the `bar_index` in all available locations.
plot(bar_index, "bar_index", color.teal, 3)
```

__Note que:__

- As saídas da linha de status de um script só serão exibidas ao habilitar a caixa de seleção "Valores" "_Values_" na seção "Indicadores" "_Indicators_" das configurações da "Linha de Status" "_Status line_" do gráfico.
- As escalas de preços só exibirão valores ou nomes de plots ao habilitar as opções no menu suspenso "Indicadores e Financeiros" nas configurações de "Escalas e Linhas" do gráfico.

A função [bgcolor()](https://br.tradingview.com/pine-script-reference/v5/#fun_bgcolor) exibe cores no fundo do painel do script, e a função [barcolor()](https://br.tradingview.com/pine-script-reference/v5/#fun_barcolor) altera as cores das barras ou velas do gráfico principal. Ambas as funções fornecem uma maneira simples de visualizar condições:

![A disposição do terreno 02](./imgs/Debugging-The-lay-of-the-land-2.Dt_T7jt8_Zx7tBv.webp)

```c
//@version=5
indicator("The lay of the land - Background and bar colors")

//@variable Is `true` if the `close` is rising over 2 bars.
bool risingPrice = ta.rising(close, 2)

// Highlight the chart background and color the main chart bars based on `risingPrice`.
bgcolor(risingPrice ? color.new(color.green, 70) : na, title= "`risingPrice` highlight")
barcolor(risingPrice ? color.aqua : chart.bg_color, title = "`risingPrice` bar color")
```

Os [tipos de desenho](./04_09_tipagem_do_sistema.md#tipos-de-desenho) do Pine ([line](./05_12_lines_e_boxes.md#lines-linhas), [box](./05_12_lines_e_boxes.md#boxes-caixas), [polyline](./05_12_lines_e_boxes.md#polylines-polilinhas), [label](./05_20_text_e_shapes.md#labels)) produzem desenhos no painel do script. Embora não retornem resultados em outros locais, como a linha de status ou a Janela de Dados, fornecem soluções alternativas e flexíveis para inspecionar valores numéricos, condições e strings diretamente no gráfico:

![A disposição do terreno 03](./imgs/Debugging-The-lay-of-the-land-3.BR7JNYrd_Z2s1rX.webp)

```c
//@version=5
indicator("The lay of the land - Drawings", overlay = true)

//@variable Is `true` when the time changes on the "1D" timeframe.
bool newDailyBar = timeframe.change("1D")
//@variable The previous bar's `bar_index` from when `newDailyBar` last occurred.
int closedIndex = ta.valuewhen(newDailyBar, bar_index - 1, 0)
//@variable The previous bar's `close` from when `newDailyBar` last occurred.
float closedPrice = ta.valuewhen(newDailyBar, close[1], 0)

if newDailyBar
    //@variable Draws a line from the previous `closedIndex` and `closedPrice` to the current values.
    line debugLine = line.new(closedIndex[1], closedPrice[1], closedIndex, closedPrice, width = 2)
    //@variable Variable info to display in a label.
    string debugText = "'1D' bar closed at: \n(" + str.tostring(closedIndex) + ", " + str.tostring(closedPrice) + ")"
    //@variable Draws a label at the current `closedIndex` and `closedPrice`.
    label.new(closedIndex, closedPrice, debugText, color = color.purple, textcolor = color.white)
```

As funções `log.*()` produzem resultados nos [Pine Logs](./06_02_debugging.md#pine-logs). Sempre que um script chama qualquer uma dessas funções, uma mensagem é registrada no painel de [Pine Logs](./06_02_debugging.md#pine-logs), juntamente com um carimbo de data/hora e opções de navegação para identificar os horários específicos, barras do gráfico e linhas de código que acionaram o log:

![A disposição do terreno 04](./imgs/Debugging-The-lay-of-the-land-4.fSt39wX6_2urrNy.webp)

```c
//@version=5
indicator("The lay of the land - Pine Logs")

//@variable The natural logarithm of the current `high - low` range.
float logRange = math.log(high - low)

// Plot the `logRange`.
plot(logRange, "logRange")

if barstate.isconfirmed
    // Generate an "error" or "info" message on the confirmed bar, depending on whether `logRange` is defined.
    switch 
        na(logRange) => log.error("Undefined `logRange` value.")
        =>              log.info("`logRange` value: " + str.tostring(logRange))
else
    // Generate a "warning" message for unconfirmed values.
    log.warning("Unconfirmed `logRange` value: " + str.tostring(logRange))
```

Pode-se aplicar qualquer uma das opções acima, ou uma combinação delas, para estabelecer rotinas de depuração que atendam às suas necessidades e preferências, dependendo dos tipos de dados e estruturas com os quais estão trabalhando. Veja as seções abaixo para explicações detalhadas de várias técnicas de depuração.

## Valores Numéricos

Ao criar código no Pine Script, trabalhar com números é inevitável. Portanto, para garantir que um script funcione conforme o esperado, é crucial entender como inspecionar os valores numéricos ([int](./04_09_tipagem_do_sistema.md#int) e [float](./04_09_tipagem_do_sistema.md#float)) que ele recebe e calcula.

> __Observação!__\
> Esta seção discute abordagens fundamentais _baseadas em gráficos_ para depuração de números. Scripts também podem converter números em [strings](./04_09_tipagem_do_sistema.md#string), permitindo inspecionar números usando técnicas relacionadas a strings. Para mais informações, veja as seções [Strings](./06_02_debugging.md#strings) e [Pine Logs](./06_02_debugging.md#pine-logs).

## Plotando Números

Uma das maneiras mais simples de inspecionar os valores numéricos de um script é usar as funções `plot*()`, que podem exibir resultados graficamente no gráfico e mostrar números formatados na linha de status do script, na escala de preços e na Janela de Dados. Os locais onde uma função `plot*()` exibe seus resultados dependem do parâmetro `display`. Por padrão, seu valor é [display.all](https://br.tradingview.com/pine-script-reference/v5/#const_display.all).

> __Observação!__\
> Apenas o _escopo global_ de um script pode conter chamadas `plot*()`, o que significa que essas funções só podem aceitar variáveis globais e literais. Elas não podem usar variáveis declaradas nos escopos locais de [loops](./04_08_loops.md), [estruturas condicionais](./04_07_estruturas_condicionais.md) ou [funções definidas pelo usuário](./04_11_funcoes_definidas_pelo_usuario.md) e [métodos](./04_13_metodos.md#métodos-definidos-pelo-usuário).

O exemplo a seguir utiliza a função [plot()](https://br.tradingview.com/pine-script-reference/v5/#fun_plot) para exibir a variação de 1 barra no valor da variável incorporada [time](https://br.tradingview.com/pine-script-reference/v5/#var_time) medida em timeframes do gráfico (por exemplo, um valor plotado de 1 no gráfico "1D" significa que há uma diferença de um dia entre os horários de abertura das barras atual e anterior). Inspecionar essa série pode ajudar a identificar lacunas de tempo nos dados do gráfico, o que é uma informação útil ao projetar indicadores baseados em tempo.

Como não há especificação de argumento `display`, a função usa [display.all](https://br.tradingview.com/pine-script-reference/v5/#const_display.all), o que significa que mostrará dados em _todos_ os locais possíveis, como mostrado abaixo:

![Plotando números](./imgs/Debugging-Numeric-values-Plotting-numbers-1.CPWsJyU1_26SArq.webp)

```c
//@version=5
indicator("Plotting numbers demo", "Time changes")

//@variable The one-bar change in the chart symbol's `time` value, measured in units of the chart timeframe.
float timeChange = ta.change(time) / (1000.0 * timeframe.in_seconds())

// Display the `timeChange` in all possible locations.
plot(timeChange, "Time difference (in chart bar units)", color.purple, 3) 
```

__Note que:__

- Os números exibidos na linha de status do script e na Janela de Dados refletem os valores plotados na localização do cursor do gráfico. Essas áreas mostrarão o valor da barra mais recente quando o ponteiro do mouse não estiver no gráfico.
- O número na escala de preços reflete o valor mais recente disponível no gráfico visível.

### Sem Afetar a Escala

Ao depurar vários valores numéricos em um script, pode ser necessário inspecioná-los sem interferir nas escalas de preços ou sobrecarregar as saídas visuais no painel do gráfico, pois escalas distorcidas e plots sobrepostos podem dificultar a avaliação dos resultados.

Uma maneira simples de inspecionar números sem adicionar mais visuais ao painel do gráfico é alterar os valores `display` nas chamadas `plot*()` do script para outras variáveis ou expressões `display.*`.

Um exemplo prático envolve a criação de um script que calcula uma média móvel ponderada personalizada dividindo a [soma](https://br.tradingview.com/pine-script-reference/v5/#fun_math.sum) dos valores `weight * close` pela [soma](https://br.tradingview.com/pine-script-reference/v5/#fun_math.sum) da série `weight`:

![Sem afetar a escala 01](./imgs/Debugging-Numeric-values-Plotting-numbers-Without-affecting-the-scale-1.D9lHNsXL_mUjuG.webp)

```c
//@version=5
indicator("Plotting without affecting the scale demo", "Weighted Average", true)

//@variable The number of bars in the average.
int lengthInput = input.int(20, "Length", 1)

//@variable The weight applied to the price on each bar.
float weight = math.pow(close - open, 2)

//@variable The numerator of the average.
float numerator = math.sum(weight * close, lengthInput)
//@variable The denominator of the average.
float denominator = math.sum(weight, lengthInput)

//@variable The `lengthInput`-bar weighted average.
float average = numerator / denominator

// Plot the `average`.
plot(average, "Weighted Average", linewidth = 3)
```

Para inspecionar as variáveis usadas no cálculo do `average` e ajustar o resultado, a exibição dos valores `weight`, `numerator` e `denominator` com a função [plot()](https://br.tradingview.com/pine-script-reference/v5/#fun_plot) em todos os locais pode dificultar a identificação da linha `average` no gráfico, devido às diferentes escalas de cada variável:

![Sem afetar a escala 02](./imgs/Debugging-Numeric-values-Plotting-numbers-Without-affecting-the-scale-2.CNBE_Mtb_2rATc4.webp)

```c
//@version=5
indicator("Plotting without affecting the scale demo", "Weighted Average", true)

//@variable The number of bars in the average.
int lengthInput = input.int(20, "Length", 1)

//@variable The weight applied to the price on each bar.
float weight = math.pow(close - open, 2)

//@variable The numerator of the average.
float numerator = math.sum(close * weight, lengthInput)
//@variable The denominator of the average.
float denominator = math.sum(weight, lengthInput)

//@variable The `lengthInput`-bar weighted average.
float average = numerator / denominator

// Plot the `average`.
plot(average, "Weighted Average", linewidth = 3)

// Create debug plots for the `weight`, `numerator`, and `denominator`.
plot(weight, "weight", color.purple)
plot(numerator, "numerator", color.teal)
plot(denominator, "denominator", color.maroon)
```

Embora seja possível ocultar plots individuais na aba "Estilo" "_Style_" das configurações do script, isso também impede a inspeção dos resultados em qualquer outro local. Para visualizar simultaneamente os valores das variáveis e preservar a escala do gráfico, é possível alterar os valores `display` nos plots de depuração.

A versão abaixo inclui uma variável `debugLocations` nas chamadas de depuração [plot()](https://br.tradingview.com/pine-script-reference/v5/#fun_plot) com um valor de `display.all - display.pane` para especificar que todos os locais _exceto_ o painel do gráfico mostrarão os resultados. Agora é possível inspecionar os valores do cálculo sem a desordem extra:

![Sem afetar a escala 03](./imgs/Debugging-Numeric-values-Plotting-numbers-Without-affecting-the-scale-3.iomfmsC__26l9R3.webp)

```c
//@version=5
indicator("Plotting without affecting the scale demo", "Weighted Average", true)

//@variable The number of bars in the average.
int lengthInput = input.int(20, "Length", 1)

//@variable The weight applied to the price on each bar.
float weight = math.pow(close - open, 2)

//@variable The numerator of the average.
float numerator = math.sum(close * weight, lengthInput)
//@variable The denominator of the average.
float denominator = math.sum(weight, lengthInput)

//@variable The `lengthInput`-bar weighted average.
float average = numerator / denominator

// Plot the `average`.
plot(average, "Weighted Average", linewidth = 3)

//@variable The display locations of all debug plots.
debugLocations = display.all - display.pane
// Create debug plots for the `weight`, `numerator`, and `denominator`.
plot(weight, "weight", color.purple, display = debugLocations)
plot(numerator, "numerator", color.teal, display = debugLocations)
plot(denominator, "denominator", color.maroon, display = debugLocations)
```

### A Partir de Escopos Locais

Os _escopos locais_ de um script são seções de código indentado dentro de [estruturas condicionais](./04_07_estruturas_condicionais.md), [funções](./04_11_funcoes_definidas_pelo_usuario.md) e [métodos](./04_13_metodos.md#métodos-definidos-pelo-usuário). Ao trabalhar com variáveis declaradas nesses escopos, utilizar as funções `plot*()` para exibir seus valores diretamente _não funcionará_, pois os plots só funcionam com literais e variáveis _globais_.

Para exibir os valores de uma variável local usando plots, é possível atribuir seus resultados a uma variável global e passar essa variável para a chamada `plot*()`.

> __Observação!__\
> A abordagem descrita abaixo funciona para variáveis locais declaradas dentro de [estruturas condicionais](./04_07_estruturas_condicionais.md). Empregar um processo semelhante para [funções](./04_11_funcoes_definidas_pelo_usuario.md) e [métodos](./04_13_metodos.md#métodos-definidos-pelo-usuário) requer [coleções](./04_09_tipagem_do_sistema.md#coleções), [tipos definidos pelo usuário](./04_09_tipagem_do_sistema.md#tipos-definidos-pelo-usuário) ou outros tipos de referência incorporados. Veja a seção [Depuração de funções](./06_02_debugging.md#depuração-de-funções) para mais informações.

Por exemplo, este script calcula a variação máxima e mínima de todos os tempos no preço de [close](https://br.tradingview.com/pine-script-reference/v5/#var_close) em um período `lengthInput`. Ele usa uma estrutura [if](https://br.tradingview.com/pine-script-reference/v5/#kw_if) para declarar uma variável local `change` e atualizar as variáveis globais `maxChange` e `minChange` uma vez a cada `lengthInput` barras:

![A partir de escopos locais 01](./imgs/Debugging-Numeric-values-Plotting-numbers-From-local-scopes-1.DEllI76K_2tIueB.webp)

```c
//@version=5
indicator("Plotting numbers from local scopes demo", "Periodic changes")

//@variable The number of chart bars in each period.
int lengthInput = input.int(20, "Period length", 1)

//@variable The maximum `close` change over each `lengthInput` period on the chart.
var float maxChange = na
//@variable The minimum `close` change over each `lengthInput` period on the chart.
var float minChange = na

//@variable Is `true` once every `lengthInput` bars.
bool periodClose = bar_index % lengthInput == 0

if periodClose
    //@variable The change in `close` prices over `lengthInput` bars.
    float change = close - close[lengthInput]
    // Update the global `maxChange` and `minChange`.
    maxChange := math.max(nz(maxChange, change), change)
    minChange := math.min(nz(minChange, change), change)

// Plot the `maxChange` and `minChange`.
plot(maxChange, "Max periodic change", color.teal, 3)
plot(minChange, "Min periodic change", color.maroon, 3)
hline(0.0, color = color.gray, linestyle = hline.style_solid)
```

Para inspecionar o histórico da variável `change` usando um plot, é necessário atribuir seu valor a outra variável _global_ para uso em uma função `plot*()`.

Abaixo, foi adicionada uma variável `debugChange` com um valor inicial de [na](https://br.tradingview.com/pine-script-reference/v5/#var_na) no escopo global, e o script reatribui seu valor dentro da estrutura [if](https://br.tradingview.com/pine-script-reference/v5/#kw_if) usando a variável local `change`. Agora, é possível usar [plot()](https://br.tradingview.com/pine-script-reference/v5/#fun_plot) com a variável `debugChange` para visualizar o histórico de valores disponíveis em `change`:

![A partir de escopos locais 02](./imgs/Debugging-Numeric-values-Plotting-numbers-From-local-scopes-2.CPaEqcWa_i20qC.webp)

```c
//@version=5
indicator("Plotting numbers from local scopes demo", "Periodic changes")

//@variable The number of chart bars in each period.
int lengthInput = input.int(20, "Period length", 1)

//@variable The maximum `close` change over each `lengthInput` period on the chart.
var float maxChange = na
//@variable The minimum `close` change over each `lengthInput` period on the chart.
var float minChange = na

//@variable Is `true` once every `lengthInput` bars.
bool periodClose = bar_index % lengthInput == 0

//@variable Tracks the history of the local `change` variable.
float debugChange = na

if periodClose
    //@variable The change in `close` prices over `lengthInput` bars.
    float change = close - close[lengthInput]
    // Update the global `maxChange` and `minChange`.
    maxChange := math.max(nz(maxChange, change), change)
    minChange := math.min(nz(minChange, change), change)
    // Assign the `change` value to the `debugChange` variable.
    debugChange := change

// Plot the `maxChange` and `minChange`.
plot(maxChange, "Max periodic change", color.teal, 3)
plot(minChange, "Min periodic change", color.maroon, 3)
hline(0.0, color = color.gray, linestyle = hline.style_solid)

// Create a debug plot to visualize the `change` history.
plot(debugChange, "Extracted change", color.purple, 15, plot.style_areabr)
```

__Note que:__

- O script usa [plot.style_areabr](https://br.tradingview.com/pine-script-reference/v5/#const_plot.style_areabr) no plot de depuração, que não se conecta sobre valores [na](https://br.tradingview.com/pine-script-reference/v5/#var_na) como o estilo padrão faz.
- Quando o valor plotado da barra visível mais à direita é [na](https://br.tradingview.com/pine-script-reference/v5/#var_na), o número na escala de preços representa o último valor _não-na_ antes dessa barra, se existir.

## Com Desenhos

Uma abordagem alternativa para inspecionar graficamente o histórico de valores numéricos de um script é usar os [tipos de desenho](./04_09_tipagem_do_sistema.md#tipos-de-desenho) do Pine, incluindo [linhas](./05_12_lines_e_boxes.md#lines-linhas), [caixas](./05_12_lines_e_boxes.md#boxes-caixas), [polilinhas](./05_12_lines_e_boxes.md#polylines-polilinhas) e [labels](./05_20_text_e_shapes.md#labels).

Embora os desenhos do Pine não exibam resultados em nenhum outro lugar além do painel do gráfico, scripts podem criá-los a partir de _escopos locais_, incluindo os escopos de [funções](./04_11_funcoes_definidas_pelo_usuario.md) e [métodos](./04_13_metodos.md#métodos-definidos-pelo-usuário) (veja a seção [Depuração de funções](./06_02_debugging.md#depuração-de-funções) para saber mais). Além disso, scripts podem posicionar desenhos em _qualquer_ localização disponível no gráfico, independentemente do [bar_index](https://br.tradingview.com/pine-script-reference/v5/#var_bar_index) atual.

Por exemplo, ao revisitar o script "Periodic changes" da [seção anterior](./06_02_debugging.md#a-partir-de-escopos-locais), é possível inspecionar o histórico da variável local `change` _sem_ usar um plot. Neste caso, evita-se declarar uma variável global separada e, em vez disso, cria-se objetos de desenho diretamente do escopo local da estrutura [if](https://br.tradingview.com/pine-script-reference/v5/#kw_if).

O script abaixo é uma modificação do script anterior que usa [caixas](./05_12_lines_e_boxes.md#boxes-caixas) para visualizar o comportamento da variável `change`. Dentro do escopo da estrutura [if](https://br.tradingview.com/pine-script-reference/v5/#kw_if), ele chama [box.new()](https://br.tradingview.com/pine-script-reference/v5/#fun_box.new) para criar uma [caixa](https://br.tradingview.com/pine-script-reference/v5/#type_box) que se estende desde a barra `lengthInput` barras atrás até o [bar_index](https://br.tradingview.com/pine-script-reference/v5/#var_bar_index) atual:

![Com desenhos](./imgs/Debugging-Numeric-values-With-drawings-1.DsvQ-qM2_Z2i05an.webp)

```c
//@version=5
indicator("Drawing numbers from local scopes demo", "Periodic changes", max_boxes_count = 500)

//@variable The number of chart bars in each period.
int lengthInput = input.int(20, "Period length", 1)

//@variable The maximum `close` change over each `lengthInput` period on the chart.
var float maxChange = na
//@variable The minimum `close` change over each `lengthInput` period on the chart.
var float minChange = na

//@variable Is `true` once every `lengthInput` bars.
bool periodClose = bar_index % lengthInput == 0

if periodClose
    //@variable The change in `close` prices over `lengthInput` bars.
    float change = close - close[lengthInput]
    // Update the global `maxChange` and `minChange`.
    maxChange := math.max(nz(maxChange, change), change)
    minChange := math.min(nz(minChange, change), change)
    //@variable Draws a box on the chart to visualize the `change` value.
    box debugBox = box.new(
         bar_index - lengthInput, math.max(change, 0.0), bar_index, math.min(change, 0.0),
         color.purple, bgcolor = color.new(color.purple, 80), text = str.tostring(change)
     )

// Plot the `maxChange` and `minChange`.
plot(maxChange, "Max periodic change", color.teal, 3)
plot(minChange, "Min periodic change", color.maroon, 3)
hline(0.0, color = color.gray, linestyle = hline.style_solid)
```

__Note que:__

- O script inclui `max_boxes_count = 500` na função [indicator()](https://br.tradingview.com/pine-script-reference/v5/#fun_indicator), o que permite mostrar até 500 [caixas](./05_12_lines_e_boxes.md#boxes-caixas) no gráfico.
- Foram usados [math.max(change, 0.0)](https://br.tradingview.com/pine-script-reference/v5/#fun_math.max) e [math.min(change, 0.0)](https://br.tradingview.com/pine-script-reference/v5/#fun_math.min) na função [box.new()](https://br.tradingview.com/pine-script-reference/v5/#fun_box.new) como os valores `top` e `bottom`.
- A chamada [box.new()](https://br.tradingview.com/pine-script-reference/v5/#fun_box.new) inclui [str.tostring(change)](https://br.tradingview.com/pine-script-reference/v5/#fun_str.tostring) como argumento `text` para exibir uma _"representação em string"_ do valor "float" da variável `change` em cada desenho de [caixa](https://br.tradingview.com/pine-script-reference/v5/#type_box). Veja [esta seção](./06_02_debugging.md#representando-outros-tipos) para mais informações sobre como representar dados com strings.

Para mais informações sobre o uso de [caixas](./05_12_lines_e_boxes.md#boxes-caixas) e outros [tipos de desenho](./04_09_tipagem_do_sistema.md#tipos-de-desenho) relacionados, veja a página [Linhas e Caixas](./05_12_lines_e_boxes.md) do Manual do Usuário.

## Condições

Muitos scripts criados no Pine envolvem declarar e avaliar _condições_ para ditar ações específicas do script, como acionar diferentes padrões de cálculo, visuais, sinais, alertas, ordens de estratégia, etc. Portanto, é essencial entender como inspecionar as condições que um script usa para garantir a execução adequada.

> __Observação!__\
> Esta seção discute técnicas de depuração baseadas em visuais de gráficos. Para saber mais sobre _registro_ de condições, veja a seção [Pine Logs](./06_02_debugging.md#pine-logs).

<!-- ### Como Números

Uma maneira possível de depurar as condições de um script é definir _valores numéricos_ baseados nelas, o que permite inspecioná-las usando abordagens numéricas, como as descritas na [seção anterior](./06_02_debugging.md#valores-numéricos).

Um exemplo simples calcula a razão entre o preço [ohlc4](https://br.tradingview.com/pine-script-reference/v5/#var_ohlc4) e a [média móvel](https://br.tradingview.com/pine-script-reference/v5/#fun_ta.sma) de `lengthInput` barras. Ele atribui uma condição à variável `priceAbove` que retorna `true` sempre que o valor da razão excede 1 (ou seja, o preço está acima da média).

Para inspecionar as ocorrências da condição, foi criada uma variável `debugValue` atribuída ao resultado de uma expressão que usa o operador ternário [?:](https://br.tradingview.com/pine-script-reference/v5/#op_?:) para retornar 1 quando `priceAbove` é `true` e 0 caso contrário. O script plota o valor da variável em todos os locais disponíveis:

![Como números](./imgs/Debugging-Conditions-As-numbers-1.BoeptaNB_2nR2E8.webp)

```c
//@version=5
indicator("Conditions as numbers demo", "MA signal")

//@variable The number of bars in the moving average calculation.
int lengthInput = input.int(20, "Length", 1)

//@variable The ratio of the `ohlc4` price to its `lengthInput`-bar moving average.
float ratio = ohlc4 / ta.sma(ohlc4, lengthInput)

//@variable The condition to inspect. Is `true` when `ohlc4` is above its moving average, `false` otherwise.
bool priceAbove = ratio > 1.0
//@variable Returns 1 when the `priceAbove` condition is `true`, 0 otherwise.
int debugValue = priceAbove ? 1 : 0

// Plot the `debugValue.
plot(debugValue, "Conditional number", color.teal, 3)
```

__Note que:__

- Representar valores "bool" usando números também permite que scripts exibam formas ou caracteres condicionais em locais específicos do eixo-y com [plotshape()](https://br.tradingview.com/pine-script-reference/v5/#fun_plotshape) e [plotchar()](https://br.tradingview.com/pine-script-reference/v5/#fun_plotchar), além de facilitar a depuração condicional com [plotarrow()](https://br.tradingview.com/pine-script-reference/v5/#fun_plotarrow). Veja a [próxima seção](./06_02_debugging.md#plotando-formas-condicionais) para saber mais. -->







<!-- ### Plotando Formas Condicionais

As funções [plotshape()](https://br.tradingview.com/pine-script-reference/v5/#fun_plotshape) e [plotchar()](https://br.tradingview.com/pine-script-reference/v5/#fun_plotchar) são úteis para depurar condições, pois podem plotar formas ou caracteres em locais absolutos ou relativos do gráfico sempre que contiverem um argumento `series` `true` ou `não-na`.

Essas funções também podem exibir representações _numéricas_ da `series` na linha de status do script e na Janela de Dados, sendo também úteis para depurar [números](./06_02_debugging.md#valores-numéricos). Um modo simples e prático de depurar números com essas funções está demonstrado na seção [Dicas](./06_02_debugging.md#dicas).

Os locais dos plots no gráfico dependem do parâmetro `location`, que por padrão é [location.abovebar](https://br.tradingview.com/pine-script-reference/v5/#const_location.abovebar).

> __Observação!__\
> Ao usar [location.abovebar](https://br.tradingview.com/pine-script-reference/v5/#const_location.abovebar) ou [location.belowbar](https://br.tradingview.com/pine-script-reference/v5/#const_location.belowbar), a função posiciona as formas/caracteres em relação aos preços do _gráfico principal_. Se o script plota seus valores em um painel de gráfico separado, recomenda-se depurar com outras opções de `location` para evitar interferir na escala do painel.

Vamos inspecionar uma condição usando essas funções. O script a seguir calcula um [RSI](https://br.tradingview.com/pine-script-reference/v5/#fun_ta.rsi) com um comprimento `lengthInput` e uma variável `crossBelow`, cujo valor é o resultado de uma condição que retorna `true` quando o RSI cruza abaixo de 30. Ele chama [plotshape()](https://br.tradingview.com/pine-script-reference/v5/#fun_plotshape) para exibir um círculo próximo ao topo do painel sempre que a condição ocorre:

![Plotando formas condicionais 01](./imgs/Debugging-Conditions-Plotting-conditional-shapes-1.B_YUlhVQ_Z1ARRXr.webp)

```c
//@version=5
indicator("Conditional shapes demo", "RSI cross under 30")

//@variable The length of the RSI.
int lengthInput = input.int(14, "Length", 1)

//@variable The calculated RSI value.
float rsi = ta.rsi(close, lengthInput)

//@variable Is `true` when the `rsi` crosses below 30, `false` otherwise.
bool crossBelow = ta.crossunder(rsi, 30.0)

// Plot the `rsi`.
plot(rsi, "RSI", color.rgb(136, 76, 146), linewidth = 3)
// Plot the `crossBelow` condition as circles near the top of the pane.
plotshape(crossBelow, "RSI crossed below 30", shape.circle, location.top, color.red, size = size.small)
```

__Note que:__

- A linha de status e a Janela de Dados exibem um valor de 1 quando `crossBelow` é `true` e 0 quando é `false`.

Para exibir as formas em locais _precisos_ em vez de relativos ao painel do gráfico, é possível usar [números condicionais](./06_02_debugging.md#valores-numéricos) e [location.absolute](https://br.tradingview.com/pine-script-reference/v5/#const_location.absolute) na chamada de [plotshape()](https://br.tradingview.com/pine-script-reference/v5/#fun_plotshape).

Neste exemplo, o script anterior foi modificado criando uma variável `debugNumber` que retorna o valor do `rsi` quando `crossBelow` é `true` e [na](https://br.tradingview.com/pine-script-reference/v5/#var_na) caso contrário. A função [plotshape()](https://br.tradingview.com/pine-script-reference/v5/#fun_plotshape) usa essa nova variável como argumento `series` e [location.absolute](https://br.tradingview.com/pine-script-reference/v5/#const_location.absolute) como argumento `location`:

![Plotando formas condicionais 02](./imgs/Debugging-Conditions-Plotting-conditional-shapes-2.Daveblc1_ZgMwC0.webp)

```c
//@version=5
indicator("Conditional shapes demo", "RSI cross under 30")

//@variable The length of the RSI.
int lengthInput = input.int(14, "Length", 1)

//@variable The calculated RSI value.
float rsi = ta.rsi(close, lengthInput)

//@variable Is `true` when the `rsi` crosses below 30, `false` otherwise.
bool crossBelow = ta.crossunder(rsi, 30.0)
//@variable Returns the `rsi` when `crossBelow` is `true`, `na` otherwise.
float debugNumber = crossBelow ? rsi : na

// Plot the `rsi`.
plot(rsi, "RSI", color.rgb(136, 76, 146), linewidth = 3)
// Plot circles at the `debugNumber`.
plotshape(debugNumber, "RSI when it crossed below 30", shape.circle, location.absolute, color.red, size = size.small)
```

__Note que:__

- Como foi passada uma série _numérica_ para a função, o plot condicional agora mostra os valores do `debugNumber` na linha de status e na Janela de Dados em vez de 1 ou 0.

Outro modo prático de depurar condições é usar [plotarrow()](https://br.tradingview.com/pine-script-reference/v5/#fun_plotarrow). Essa função plota uma seta com localização relativa aos preços do _gráfico principal_ sempre que o argumento `series` não é zero e não é [na](https://br.tradingview.com/pine-script-reference/v5/#var_na). O comprimento de cada seta varia com o valor da `series` fornecida. Assim como [plotshape()](https://br.tradingview.com/pine-script-reference/v5/#fun_plotshape) e [plotchar()](https://br.tradingview.com/pine-script-reference/v5/#fun_plotchar), [plotarrow()](https://br.tradingview.com/pine-script-reference/v5/#fun_plotarrow) também pode exibir resultados numéricos na linha de status e na Janela de Dados.

> __Observação!__\
> Como essa função sempre posiciona setas em relação aos preços do gráfico principal, recomenda-se usá-la apenas se o script ocupar o painel do gráfico principal para evitar interferir na escala.

Este exemplo mostra um modo alternativo de inspecionar a condição `crossBelow` usando [plotarrow()](https://br.tradingview.com/pine-script-reference/v5/#fun_plotarrow). Nesta versão, `overlay` está definido como `true` na função [indicator()](https://br.tradingview.com/pine-script-reference/v5/#fun_indicator) e foi adicionada uma chamada de [plotarrow()](https://br.tradingview.com/pine-script-reference/v5/#fun_plotarrow) para visualizar os valores condicionais. O `debugNumber` neste exemplo mede o quanto o `rsi` caiu abaixo de 30 cada vez que a condição ocorre:

![Plotando formas condicionais 03](./imgs/Debugging-Conditions-Plotting-conditional-shapes-3.IZzMwtM0_Z1HvpVm.webp)

```c
//@version=5
indicator("Conditional shapes demo", "RSI cross under 30", true)

//@variable The length of the RSI.
int lengthInput = input.int(14, "Length", 1)

//@variable The calculated RSI value.
float rsi = ta.rsi(close, lengthInput)

//@variable Is `true` when the `rsi` crosses below 30, `false` otherwise.
bool crossBelow = ta.crossunder(rsi, 30.0)
//@variable Returns `rsi - 30.0` when `crossBelow` is `true`, `na` otherwise.
float debugNumber = crossBelow ? rsi - 30.0 : na

// Plot the `rsi`.
plot(rsi, "RSI", color.rgb(136, 76, 146), display = display.data_window)
// Plot arrows at the `debugNumber`.
plotarrow(debugNumber, "RSI cross below 30 distance")
```

__Note que:__

- O valor `display` na função [plot()](https://br.tradingview.com/pine-script-reference/v5/#fun_plot) do `rsi` foi definido como [display.data_window](https://br.tradingview.com/pine-script-reference/v5/#const_display.data_window) para [preservar a escala do gráfico](./06_02_debugging.md#sem-afetar-a-escala).

Para saber mais sobre [plotshape()](https://br.tradingview.com/pine-script-reference/v5/#fun_plotshape), [plotchar()](https://br.tradingview.com/pine-script-reference/v5/#fun_plotchar) e [plotarrow()](https://br.tradingview.com/pine-script-reference/v5/#fun_plotarrow), veja a página [Textos e Formas](./05_20_text_e_shapes.md) deste manual.

### Cores Condicionais

Uma maneira elegante de representar visualmente

 condições no Pine é criar expressões que retornem valores de [cor](./04_09_tipagem_do_sistema.md#color) com base nos estados `true` ou `false`, pois os scripts podem usá-los para controlar a aparência de [objetos de desenho](./04_09_tipagem_do_sistema.md#tipos-de-desenho) ou os resultados de chamadas `plot*()`, [fill()](https://br.tradingview.com/pine-script-reference/v5/#fun_fill), [bgcolor()](https://br.tradingview.com/pine-script-reference/v5/#fun_bgcolor) ou [barcolor()](https://br.tradingview.com/pine-script-reference/v5/#fun_barcolor).

> __Observação!__\
> Assim como as funções `plot*()`, scripts só podem chamar [fill()](https://br.tradingview.com/pine-script-reference/v5/#fun_fill), [bgcolor()](https://br.tradingview.com/pine-script-reference/v5/#fun_bgcolor) e [barcolor()](https://br.tradingview.com/pine-script-reference/v5/#fun_barcolor) a partir do _escopo global_, e as funções não podem aceitar variáveis locais.

Por exemplo, este script calcula a variação nos preços de [close](https://br.tradingview.com/pine-script-reference/v5/#var_close) ao longo de `lengthInput` barras e declara duas variáveis “bool” para identificar quando a variação de preço é positiva ou negativa.

O script usa esses valores “bool” como condições em expressões [ternárias](https://br.tradingview.com/pine-script-reference/v5/#op_?:) para atribuir os valores de três variáveis “color”, e depois usa essas variáveis como argumentos `color` em [plot()](https://br.tradingview.com/pine-script-reference/v5/#fun_plot), [bgcolor()](https://br.tradingview.com/pine-script-reference/v5/#fun_bgcolor) e [barcolor()](https://br.tradingview.com/pine-script-reference/v5/#fun_barcolor) para depurar os resultados:

![Cores condicionais 01](./imgs/Debugging-Conditions-Conditional-colors-1.B5Y6dhKf_Z1jtyTu.webp)

```c
//@version=5
indicator("Conditional colors demo", "Price change colors")

//@variable The number of bars in the price change calculation.
int lengthInput = input.int(10, "Length", 1)

//@variable The change in `close` prices over `lengthInput` bars.
float priceChange = ta.change(close, lengthInput)

//@variable Is `true` when the `priceChange` is a positive value, `false` otherwise.
bool isPositive = priceChange > 0
//@variable Is `true` when the `priceChange` is a negative value, `false` otherwise.
bool isNegative = priceChange < 0

//@variable Returns a color for the `priceChange` plot to show when `isPositive`, `isNegative`, or neither occurs.
color plotColor = isPositive ? color.teal : isNegative ? color.maroon : chart.fg_color
//@variable Returns an 80% transparent color for the background when `isPositive` or `isNegative`, `na` otherwise.
color bgColor = isPositive ? color.new(color.aqua, 80) : isNegative ? color.new(color.fuchsia, 80) : na
//@variable Returns a color to emphasize chart bars when `isPositive` occurs. Otherwise, returns the `chart.bg_color`.
color barColor = isPositive ? color.orange : chart.bg_color

// Plot the `priceChange` and color it with the `plotColor`.
plot(priceChange, "Price change", plotColor, style = plot.style_area)
// Highlight the pane's background with the `bgColor`.
bgcolor(bgColor, title = "Background highlight")
// Emphasize the chart bars with positive price change using the `barColor`.
barcolor(barColor, title = "Positive change bars")
```

__Note que:__

- A função [barcolor()](https://br.tradingview.com/pine-script-reference/v5/#fun_barcolor) sempre colore as barras do gráfico principal, independentemente de o script ocupar outro painel do gráfico, e o gráfico só exibirá os resultados se as barras estiverem visíveis.

Veja as páginas [Cores](./05_07_cores.md), [Preenchimentos](./05_16_fills.md), [Fundos](./05_08_fundos.md) e [Coloração de Barras](./05_15_coloração_de_barras.md) para mais informações sobre como trabalhar com cores, preencher plots, destacar fundos e colorir barras. -->

## Pine Logs

## Strings

## Depuração de Funções

## Representando Outros Tipos

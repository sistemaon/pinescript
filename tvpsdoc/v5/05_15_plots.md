
# Plots (_Plotagem_)

A função [plot()](https://br.tradingview.com/pine-script-reference/v5/#fun_plot) é a mais frequentemente usada para exibir informações calculadas usando scripts Pine. Ela é versátil e pode plotar diferentes estilos de linhas, histogramas, áreas, colunas (como colunas de volume), preenchimentos, círculos ou cruzes.

O uso de [plot()](https://br.tradingview.com/pine-script-reference/v5/#fun_plot) para criar preenchimentos é explicado na página sobre [Fills](./05_08_fills.md).

Este script demonstra alguns usos diferentes de [plot()](https://br.tradingview.com/pine-script-reference/v5/#fun_plot) em um script de sobreposição:

![Plots 01](./imgs/Plots-Introduction-01.Ctk4DlIO_24M1qs.webp)

```c
//@version=5
indicator("`plot()`", "", true)
plot(high, "Blue `high` line")
plot(math.avg(close, open), "Crosses in body center", close > open ? color.lime : color.purple, 6, plot.style_cross)
plot(math.min(open, close), "Navy step line on body low point", color.navy, 3, plot.style_stepline)
plot(low, "Gray dot on `low`", color.gray, 3, plot.style_circles)

color VIOLET = #AA00FF
color GOLD   = #CCCC00
ma = ta.alma(hl2, 40, 0.85, 6)
var almaColor = color.silver
almaColor := ma > ma[2] ? GOLD : ma < ma[2]  ? VIOLET : almaColor
plot(ma, "Two-color ALMA", almaColor, 2)
```

__Note que:__

- A primeira chamada de [plot()](https://br.tradingview.com/pine-script-reference/v5/#fun_plot) plota uma linha azul de 1 pixel ao longo das máximas das barras.
- A segunda plota cruzes no ponto médio dos corpos. As cruzes são coloridas de verde-limão quando a barra está em alta e roxas quando está em baixa. O argumento usado para `linewidth` é `6`, mas não é um valor de pixel; apenas um tamanho relativo.
- A terceira chamada plota uma linha escalonada de 3 pixels de largura seguindo o ponto baixo dos corpos.
- A quarta chamada plota um círculo cinza na [mínima](https://br.tradingview.com/pine-script-reference/v5/#var_low) das barras.
- A última plotagem requer alguma preparação. Primeiro, define as cores de alta/baixa, calcula uma [Média Móvel Arnaud Legoux](https://br.tradingview.com/support/solutions/43000594683), depois realiza cálculos de cor. Inicializa a variável de cor na barra zero apenas, usando [var](https://br.tradingview.com/pine-script-reference/v5/#kw_var). Inicializando com [color.silver](https://br.tradingview.com/pine-script-reference/v5/#const_color%7Bdot%7Dsilver), então, nas primeiras barras do conjunto de dados, até que uma das condições cause a mudança de cor, a linha será prata. As condições que mudam a cor da linha exigem que ela esteja mais alta/baixa do que seu valor de duas barras atrás. Isso faz com que as transições de cor sejam menos ruidosas do que se olhásse apenas para um valor mais alto/baixo do que o anterior.

Este script mostra outros usos de [plot()](https://br.tradingview.com/pine-script-reference/v5/#fun_plot) em um painel:

![Plots 02](./imgs/Plots-Introduction-02.CEeiodqC_Z1WHdqJ.webp)

```c
//@version=5
indicator("Volume change", format = format.volume)

color GREEN         = #008000
color GREEN_LIGHT   = color.new(GREEN, 50)
color GREEN_LIGHTER = color.new(GREEN, 85)
color PINK          = #FF0080
color PINK_LIGHT    = color.new(PINK, 50)
color PINK_LIGHTER  = color.new(PINK, 90)

bool  barUp = ta.rising(close, 1)
bool  barDn = ta.falling(close, 1)
float volumeChange = ta.change(volume)

volumeColor = barUp ? GREEN_LIGHTER : barDn ? PINK_LIGHTER : color.gray
plot(volume, "Volume columns", volumeColor, style = plot.style_columns)

volumeChangeColor = barUp ? volumeChange > 0 ? GREEN : GREEN_LIGHT : volumeChange > 0 ? PINK : PINK_LIGHT
plot(volumeChange, "Volume change columns", volumeChangeColor, 12, plot.style_histogram)

plot(0, "Zero line", color.gray)
```

__Note que:__

- É plotado valores normais de [volume](https://br.tradingview.com/pine-script-reference/v5/#var_volume) como colunas largas acima da linha zero (veja o `style = plot.style_columns` na nossa chamada de [plot()](https://br.tradingview.com/pine-script-reference/v5/#fun_plot)).
- Antes de plotar as colunas, calcula o `volumeColor` usando os valores das variáveis booleanas `barUp` e `barDn`. Elas se tornam respectivamente `true` quando o [fechamento](https://br.tradingview.com/pine-script-reference/v5/#var_close) da barra atual é maior/menor que o anterior. Note que o "Volume" não usa a mesma condição; ele identifica uma barra de alta com `close > open`. As cores `GREEN_LIGHTER` e `PINK_LIGHTER` são usadas para as colunas de volume.
- Porque a primeira plotagem plota colunas, não utiliza o parâmetro `linewidth`, pois ele não tem efeito sobre colunas.
- A segunda plotagem do script é a __mudança__ no volume, que foi calculado anteriormente usando `ta.change(volume)`. Este valor é plotado como um histograma, onde o parâmetro `linewidth` controla a largura da coluna. Foi definido essa largura como `12` para que os elementos do histograma sejam mais finos que as colunas da primeira plotagem. Valores positivos/negativos de `volumeChange` são plotados acima/abaixo da linha zero; nenhuma manipulação é necessária para obter este efeito.
- Antes de plotar o histograma de valores de `volumeChange`, calcula seu valor de cor, que pode ser uma de quatro cores diferentes. Usa-se as cores brilhantes `GREEN` ou `PINK` quando a barra está em alta/baixa E o volume aumentou desde a última barra (`volumeChange > 0`). Como `volumeChange` é positivo nesse caso, o elemento do histograma será plotado acima da linha zero. É usado as cores `GREEN_LIGHT` ou `PINK_LIGHT` quando a barra está em alta/baixa E o volume NÃO aumentou desde a última barra. Como `volumeChange` é negativo nesse caso, o elemento do histograma será plotado abaixo da linha zero.
- Finalmente, plotado uma linha zero. Poderia ter usado `hline(0)` ali.
- Usa `format = format.volume` na chamada de [indicator()](https://br.tradingview.com/pine-script-reference/v5/#fun_indicator) para que grandes valores exibidos por este script sejam abreviados como os do indicador incorporado "Volume".

Chamadas de [plot()](https://br.tradingview.com/pine-script-reference/v5/#fun_plot) devem sempre ser colocadas na primeira posição de uma linha, o que implica que estão sempre no escopo global do script. Elas não podem ser colocadas em funções definidas pelo usuário ou estruturas como [if](https://br.tradingview.com/pine-script-reference/v5/#kw_if), [for](https://br.tradingview.com/pine-script-reference/v5/#kw_for), etc. No entanto, chamadas para [plot()](https://br.tradingview.com/pine-script-reference/v5/#fun_plot) __podem__ ser projetadas para plotar condicionalmente de duas maneiras, abordado na seção [plots condicionais](./05_15_plots.md#plotagem-condicional) desta página.

Um script só pode plotar no seu próprio espaço visual, seja em um painel ou no gráfico como uma sobreposição. Scripts que rodam em um painel podem apenas [colorir barras](./05_03_coloracao_de_barras.md) na área do gráfico.

<!-- ## Parâmetros de `plot()`

A função [plot()](https://br.tradingview.com/pine-script-reference/v5/#fun_plot) tem a seguinte assinatura:

```c
plot(series, title, color, linewidth, style, trackprice, histbase, offset, join, editable, show_last, display, force_overlay) → plot
```

Os parâmetros de [plot()](https://br.tradingview.com/pine-script-reference/v5/#fun_plot) são:

`series`

É o único parâmetro obrigatório. Seu argumento deve ser do tipo "series int/float". Note que, devido às regras de auto-conversão no Pine Script converterem na direção int 🠆 float 🠆 bool, uma variável do tipo "bool" não pode ser usada como está; deve ser convertida para "int" ou "float" para ser usada como argumento. Por exemplo, se `newDay` é do tipo "bool", então `newDay ? 1 : 0` pode ser usado para plotar 1 quando a variável é `true`, e zero quando é `false`.

`title`

Requer um argumento do tipo "const string", então deve ser conhecido no tempo de compilação. A string aparece:

- Na escala do script quando o campo "Configurações do gráfico/Escalas/Rótulo do Nome do Indicador" ("_Chart settings/Scales/Indicator Name Label_") está marcado.
- Na Janela de Dados ("_Data Window_").
- Na aba "Configurações/Estilo" ("_Settings/Style_").
- No dropdown de campos [input.source()](https://br.tradingview.com/pine-script-reference/v5/#fun_input%7Bdot%7Dsource).
- No campo "Condição" ("_Condition_") da caixa de diálogo "Criar Alerta" ("_Create Alert_"), quando o script está selecionado.
- Como cabeçalho de coluna ao exportar dados do gráfico para um arquivo CSV.

`color`

Aceita "series color", então pode ser calculado dinamicamente, barra a barra. Plotar com [na](https://br.tradingview.com/pine-script-reference/v5/#var_na) como a cor, ou qualquer cor com uma transparência de 100, é uma maneira de esconder plotagens quando não são necessárias.

`linewidth`

Refere-se ao tamanho do elemento plotado, mas não se aplica a todos os estilos. Quando uma linha é plotada, a unidade é pixels. Não tem impacto quando [plot.style_columns](https://br.tradingview.com/pine-script-reference/v5/#const_plot%7Bdot%7Dstyle_columns) é utilizado.

`style`

Os argumentos disponíveis são:

- [plot.style_line](https://br.tradingview.com/pine-script-reference/v5/#const_plot%7Bdot%7Dstyle_line) (padrão): Plota uma linha contínua usando o argumento `linewidth` em pixels para sua largura. Valores [na](https://br.tradingview.com/pine-script-reference/v5/#var_na) não serão plotados como linha, mas serão conectados quando um valor que não é [na](https://br.tradingview.com/pine-script-reference/v5/#var_na) aparecer. Valores não-[na](https://br.tradingview.com/pine-script-reference/v5/#var_na) são conectados apenas se estiverem visíveis no gráfico.
- [plot.style_linebr](https://br.tradingview.com/pine-script-reference/v5/#const_plot%7Bdot%7Dstyle_linebr): Permite a plotagem de linhas descontínuas, não plotando em valores [na](https://br.tradingview.com/pine-script-reference/v5/#var_na) e não conectando lacunas, ou seja, não criando pontes sobre valores [na](https://br.tradingview.com/pine-script-reference/v5/#var_na).
- [plot.style_stepline](https://br.tradingview.com/pine-script-reference/v5/#const_plot%7Bdot%7Dstyle_stepline): Plota usando um efeito de escada. As transições entre mudanças de valores são feitas usando uma linha vertical desenhada no meio das barras, em vez de uma linha diagonal ponto a ponto conectando os pontos médios das barras. Também pode ser usado para alcançar um efeito semelhante ao de [plot.style_linebr](https://br.tradingview.com/pine-script-reference/v5/#const_plot%7Bdot%7Dstyle_linebr), mas apenas se o cuidado for tomado para não plotar cor em valores [na](https://br.tradingview.com/pine-script-reference/v5/#var_na).
- [plot.style_area](https://br.tradingview.com/pine-script-reference/v5/#const_plot%7Bdot%7Dstyle_area): Plota uma linha de largura `linewidth`, preenchendo a área entre a linha e o `histbase`. O argumento `color` é usado tanto para a linha quanto para o preenchimento. Pode-se fazer a linha de uma cor diferente usando outra chamada de [plot()](https://br.tradingview.com/pine-script-reference/v5/#fun_plot). Valores positivos são plotados acima do `histbase`, valores negativos abaixo dele.
- [plot.style_areabr](https://br.tradingview.com/pine-script-reference/v5/#const_plot%7Bdot%7Dstyle_areabr): Semelhante a [plot.style_area](https://br.tradingview.com/pine-script-reference/v5/#const_plot%7Bdot%7Dstyle_area), mas não cria pontes sobre valores [na](https://br.tradingview.com/pine-script-reference/v5/#var_na). Outra diferença é como a escala do indicador é calculada. Apenas os valores plotados são usados no cálculo da escala _y_ do espaço visual do script. Se apenas valores altos situados longe do `histbase` forem plotados, por exemplo, esses valores serão usados para calcular a escala _y_ do espaço visual do script. Valores positivos são plotados acima do `histbase`, valores negativos abaixo dele.
- [plot.style_columns](https://br.tradingview.com/pine-script-reference/v5/#const_plot%7Bdot%7Dstyle_columns): Plota colunas semelhantes às do indicador incorporado "Volume". O valor de `linewidth` __não__ afeta a largura das colunas. Valores positivos são plotados acima do `histbase`, valores negativos abaixo dele. Sempre inclui o valor de `histbase` na escala _y_ do espaço visual do script.
- [plot.style_histogram](https://br.tradingview.com/pine-script-reference/v5/#const_plot%7Bdot%7Dstyle_histogram): Plota colunas semelhantes às do indicador incorporado "Volume", exceto que o valor de `linewidth` é usado para determinar a largura das barras do histograma em pixels. Note que, como `linewidth` requer um valor "input int", a largura das barras do histograma não pode variar de barra para barra. Valores positivos são plotados acima do `histbase`, valores negativos abaixo dele. Sempre inclui o valor de `histbase` na escala _y_ do espaço visual do script.
- [plot.style_circles](https://br.tradingview.com/pine-script-reference/v5/#const_plot%7Bdot%7Dstyle_circles) e [plot.style_cross](https://br.tradingview.com/pine-script-reference/v5/#const_plot%7Bdot%7Dstyle_cross): Plotam uma forma que não é conectada entre as barras, a menos que `join = true` também seja usado. Para esses estilos, o argumento `linewidth` torna-se uma medida de tamanho relativa — suas unidades não são pixels.

`trackprice`

O valor padrão é `false`. Quando `true`, uma linha pontilhada composta de pequenos quadrados será plotada em toda a largura do espaço visual do script. Frequentemente usado em conjunto com `show_last = 1, offset = -99999` para ocultar a plotagem real e deixar apenas a linha pontilhada residual.

`histbase`

É o ponto de referência usado com [plot.style_area](https://br.tradingview.com/pine-script-reference/v5/#const_plot%7Bdot%7Dstyle_area), [plot.style_columns](https://br.tradingview.com/pine-script-reference/v5/#const_plot%7Bdot%7Dstyle_columns) e [plot.style_histogram](https://br.tradingview.com/pine-script-reference/v5/#const_plot%7Bdot%7Dstyle_histogram). Determina o nível que separa valores positivos e negativos do argumento `series`. Não pode ser calculado dinamicamente, pois requer um valor "input int/float".

`offset`

Permite deslocar a plotagem no passado/futuro usando um deslocamento negativo/positivo em barras. O valor não pode mudar durante a execução do script.

`join`

Afeta apenas os estilos [plot.style_circles](https://br.tradingview.com/pine-script-reference/v5/#const_plot%7Bdot%7Dstyle_circles) ou [plot.style_cross](https://br.tradingview.com/pine-script-reference/v5/#const_plot%7Bdot%7Dstyle_cross). Quando `true`, as formas são conectadas por uma linha de um pixel.

`editable`

Este parâmetro booleano controla se as propriedades da plotagem podem ser editadas na aba "Configurações/Estilo" ("_Settings/Style_"). O valor padrão é `true`.

`show_last`

Permite controlar quantas das últimas barras os valores plotados são visíveis. Um argumento "input int" é necessário, então não pode ser calculado dinamicamente.

`display`

O padrão é [display.all](https://br.tradingview.com/pine-script-reference/v5/#const_display%7Bdot%7Dall). Quando configurado para [display.none](https://br.tradingview.com/pine-script-reference/v5/#const_display%7Bdot%7Dnone), os valores plotados não afetarão a escala do espaço visual do script. A plotagem será invisível e não aparecerá nos valores do indicador ou na Janela de Dados. Pode ser útil em plotagens destinadas a serem usadas como entradas externas para outros scripts, ou para plotagens usadas com o espaço reservado `{{plot("[plot_title]")}}` em chamadas de [alertcondition()](https://br.tradingview.com/pine-script-reference/v5/#fun_alertcondition), por exemplo:

```c
//@version=5
indicator("")
r = ta.rsi(close, 14)
xUp = ta.crossover(r, 50)
plot(r, "RSI", display = display.none)
alertcondition(xUp, "xUp alert", message = 'RSI is bullish at: {{plot("RSI")}}')
```

`force_overlay`

Se `true`, os resultados plotados serão exibidos no painel principal do gráfico, mesmo quando o script ocupar um painel separado. Opcional. O valor padrão é `false`.

## Plotagem Condicional

Chamadas de [plot()](https://br.tradingview.com/pine-script-reference/v5/#fun_plot) não podem ser usadas em estruturas condicionais como [if](https://br.tradingview.com/pine-script-reference/v5/#kw_if), mas podem ser controladas variando seus valores plotados ou suas cores. Quando nenhuma plotagem é necessária, é possível plotar valores [na](https://br.tradingview.com/pine-script-reference/v5/#var_na) ou usar a cor [na](https://br.tradingview.com/pine-script-reference/v5/#var_na) ou qualquer cor com 100 de transparência (o que também a torna invisível).

### Controle de Valor

Uma maneira de controlar a exibição das plotagens é plotar valores [na](https://br.tradingview.com/pine-script-reference/v5/#var_na) quando nenhuma plotagem é necessária. Às vezes, valores retornados por funções como [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request%7Bdot%7Dsecurity) retornarão valores [na](https://br.tradingview.com/pine-script-reference/v5/#var_na), quando `gaps = barmerge.gaps_on` é usado, por exemplo. Em ambos os casos, às vezes é útil plotar linhas descontínuas. Este script mostra algumas maneiras de fazer isso:

![controle de valor](./imgs/Plots-PlottingConditionally-01.DCGRnCbX_1NjoOk.webp)

```c
//@version=5
indicator("Discontinuous plots", "", true)
bool plotValues = bar_index % 3 == 0
plot(plotValues ? high : na, color = color.fuchsia, linewidth = 6, style = plot.style_linebr)
plot(plotValues ? high : na)
plot(plotValues ? math.max(open, close) : na, color = color.navy, linewidth = 6, style = plot.style_cross)
plot(plotValues ? math.min(open, close) : na, color = color.navy, linewidth = 6, style = plot.style_circles)
plot(plotValues ? low : na, color = plotValues ? color.green : na, linewidth = 6, style = plot.style_stepline)
```

__Note que:__

- A condição que determina quando plotar é definida usando `bar_index % 3 == 0`, que se torna `true` quando o restante da divisão do índice da barra por 3 é zero. Isso acontecerá a cada três barras.
- Na primeira plotagem, é usado [plot.style_linebr](https://br.tradingview.com/pine-script-reference/v5/#const_plot%7Bdot%7Dstyle_linebr), que plota a linha fúcsia nos máximos. Está centrado no ponto médio horizontal da barra.
- A segunda plotagem mostra o resultado de plotar os mesmos valores, mas sem usar cuidado especial para quebrar a linha. O que acontece aqui é que a linha azul fina da chamada simples de [plot()](https://br.tradingview.com/pine-script-reference/v5/#fun_plot) é automaticamente conectada sobre valores [na](https://br.tradingview.com/pine-script-reference/v5/#var_na) (ou _gaps_), então a plotagem não é interrompida.
- Em seguida, são plotados cruzes e círculos azul-marinho nos topos e fundos dos corpos. Os estilos [plot.style_circles](https://br.tradingview.com/pine-script-reference/v5/#const_plot%7Bdot%7Dstyle_circles) e [plot.style_cross](https://br.tradingview.com/pine-script-reference/v5/#const_plot%7Bdot%7Dstyle_cross) são uma maneira simples de plotar valores descontínuos, por exemplo, para níveis de stop ou take profit, ou níveis de suporte e resistência.
- A última plotagem em verde nos mínimos das barras é feita usando [plot.style_stepline](https://br.tradingview.com/pine-script-reference/v5/#const_plot%7Bdot%7Dstyle_stepline). Note como seus segmentos são mais largos que os segmentos da linha fúcsia plotados com [plot.style_linebr](https://br.tradingview.com/pine-script-reference/v5/#const_plot%7Bdot%7Dstyle_linebr). Também observe como na última barra, ele plota apenas até a metade, até que a próxima barra entre.
- A ordem de plotagem de cada plotagem é controlada por sua ordem de aparição no script.

Este script mostra como restringir a plotagem para barras após uma data definida pelo usuário. A função [input.time()](https://br.tradingview.com/pine-script-reference/v5/#fun_input%7Bdot%7Dtime) é usada para criar um widget de entrada permitindo que os usuários do script selecionem uma data e hora, usando 1º de janeiro de 2021 como seu valor padrão:

```c
//@version=5
indicator("", "", true)
startInput = input.time(timestamp("2021-01-01"))
plot(time > startInput ? close : na)
``` -->

## Controle de Cor

# Levels (_Níveis_)

# Plotagem Condicional

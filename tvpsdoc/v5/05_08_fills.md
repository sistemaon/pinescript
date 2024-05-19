
# Fills (_Preenchimentos_)

Algumas das saídas visuais do Pine Script, incluindo [plots](./04_09_tipagem_do_sistema.md#plot-e-hline), [hlines](./04_09_tipagem_do_sistema.md#plot-e-hline), [lines](./05_12_lines_e_boxes.md#lines-linhas), [boxes](./05_12_lines_e_boxes.md#boxes-caixas) e [polylines](./05_12_lines_e_boxes.md#polylines), permitem preencher o espaço do gráfico que ocupam com cores. Três mecanismos diferentes facilitam o preenchimento do espaço entre essas saídas:

- A função [fill()](https://br.tradingview.com/pine-script-reference/v5/#fun_fill) preenche o espaço entre dois plots de chamadas [plot()](https://br.tradingview.com/pine-script-reference/v5/#fun_plot) ou duas linhas horizontais (hlines) de chamadas [hline()](https://br.tradingview.com/pine-script-reference/v5/#fun_hline) com uma cor especificada.
- Objetos do tipo [linefill](https://br.tradingview.com/pine-script-reference/v5/#type_linefill) preenchem o espaço entre instâncias de [line](https://br.tradingview.com/pine-script-reference/v5/#type_line) criadas com [line.new()](https://br.tradingview.com/pine-script-reference/v5/#fun_line.new).
- Outros tipos de desenho, nomeadamente [boxes](./05_12_lines_e_boxes.md#boxes-caixas) e [polylines](./05_12_lines_e_boxes.md#polylines), têm propriedades incorporadas que permitem preencher os espaços visuais que ocupam.


# Preenchimentos de `plot()` e `hline()`

A função [fill()](https://br.tradingview.com/pine-script-reference/v5/#fun_fill) preenche o espaço entre dois plots ou linhas horizontais. Ela possui as seguintes duas assinaturas:

```c
fill(plot1, plot2, color, title, editable, show_last, fillgaps) → void
fill(hline1, hline2, color, title, editable, fillgaps) → void
```

Os parâmetros `plot1`, `plot2`, `hline1` e `hline2` aceitam IDs de [plot](./04_09_tipagem_do_sistema.md#plot-e-hline) ou [hline](./04_09_tipagem_do_sistema.md#plot-e-hline) retornados pelas chamadas das funções [plot()](https://br.tradingview.com/pine-script-reference/v5/#fun_plot) e [hline()](https://br.tradingview.com/pine-script-reference/v5/#fun_hline). A função [fill()](https://br.tradingview.com/pine-script-reference/v5/#fun_fill) é a única embutida que pode usar esses IDs.

Este exemplo simples demonstra como a função [fill()](https://br.tradingview.com/pine-script-reference/v5/#fun_fill) funciona com IDs de [plot e hline](./04_09_tipagem_do_sistema.md#plot-e-hline). Ela chama [plot()](https://br.tradingview.com/pine-script-reference/v5/#fun_plot) e [hline()](https://br.tradingview.com/pine-script-reference/v5/#fun_hline) três vezes para exibir valores arbitrários no gráfico. Cada uma dessas chamadas retorna um ID, que o script atribui a variáveis para uso na função [fill()](https://br.tradingview.com/pine-script-reference/v5/#fun_fill). Os valores de `p1`, `p2` e `p3` são IDs de "plot", enquanto `h1`, `h2` e `h3` referenciam IDs de "hline":

![Preenchimentos de `plot()` e `hline()` 01](./imgs/Fills-Fill-1.png)

```c
//@version=5
indicator("Example 1")

// Assign "plot" IDs to the `p1`, `p2`, and `p3` variables.
p1 = plot(math.sin(high), "Sine of `high`")
p2 = plot(math.cos(low), "Cosine of `low`")
p3 = plot(math.sin(close), "Sine of `close`")
// Fill the space between `p1` and `p2` with 90% transparent red.
fill(p1, p3, color.new(color.red, 90), "`p1`-`p3` fill")
// Fill the space between `p2` and `p3` with 90% transparent blue.
fill(p2, p3, color.new(color.blue, 90), "`p2`-`p3` fill")

// Assign "hline" IDs to the `h1`, `h2`, and `h3` variables.
h1 = hline(0, "First level")
h2 = hline(1.0, "Second level")
h3 = hline(0.5, "Third level")
h4 = hline(1.5, "Fourth level")
// Fill the space between `h1` and `h2` with 90% transparent yellow.
fill(h1, h2, color.new(color.yellow, 90), "`h1`-`h2` fill")
// Fill the space between `h3` and `h4` with 90% transparent lime.
fill(h3, h4, color.new(color.lime, 90), "`h3`-`h4` fill")
```

É importante notar que a função [fill()](https://br.tradingview.com/pine-script-reference/v5/#fun_fill) requer _quaisqueres_ dois IDs de "plot" ou IDs de "hline". __Não é possível__ misturar esses tipos na chamada da função. Consequentemente, às vezes os programadores precisarão usar [plot()](https://br.tradingview.com/pine-script-reference/v5/#fun_plot) onde normalmente usariam [hline()](https://br.tradingview.com/pine-script-reference/v5/#fun_hline) se quiserem preencher o espaço entre um nível consistente e uma _series_ flutuante.

Por exemplo, este script calcula um `oscillator` ("_oscilador_") com base na distância percentual entre o preço de [close](https://br.tradingview.com/pine-script-reference/v5/#var_close) do gráfico e uma [SMA](https://br.tradingview.com/pine-script-reference/v5/#fun_ta.sma) de 10 barras, e então o plota no painel do gráfico. Neste caso, é desejado preencher a área entre o `oscillator` e zero. Embora o nível zero possa ser exibido com [hline()](https://br.tradingview.com/pine-script-reference/v5/#fun_hline) já que seu valor não muda, não é possível passar um ID de "plot" e um ID de "hline" para a função [fill()](https://br.tradingview.com/pine-script-reference/v5/#fun_fill). Portanto, é necessário usar uma chamada [plot()](https://br.tradingview.com/pine-script-reference/v5/#fun_plot) para o nível ser permitido que o script preencha o espaço:

![Preenchimentos de `plot()` e `hline()` 02](./imgs/Fills-Fill-2.png)

```c
//@version=5
indicator("Example 2")

//@variable The 10-bar moving average of `close` prices.
float ma = ta.sma(close, 10)
//@variable The distance from the `ma` to the `close` price, as a percentage of the `ma`.
float oscillator = 100 * (ma - close) / ma

//@variable The ID of the `oscillator` plot for use in the `fill()` function.
oscPlotID = plot(oscillator, "Oscillator")
//@variable The ID of the zero level plot for use in the `fill()` function.
//          Requires a "plot" ID since the `fill()` function can't use "plot" and "hline" IDs at the same time.
zeroPlotID = plot(0, "Zero level", color.silver, 1, plot.style_circles)

// Filll the space between the `oscPlotID` and `zeroPlotID` with 90% transparent blue.
fill(oscPlotID, zeroPlotID, color.new(color.blue, 90), "Oscillator fill")
```

O parâmetro `color` da função [fill()](https://br.tradingview.com/pine-script-reference/v5/#fun_fill) aceita um argumento de "series color", o que significa que a cor do preenchimento pode mudar ao longo das barras do gráfico.

Por exemplo, este código preenche o espaço entre dois plots de médias móveis com cores verdes ou vermelhas 90% transparentes, com base em se `ma1` está acima de `ma2`:

![Preenchimentos de `plot()` e `hline()` 03](./imgs/Fills-Fill-3.png)

```c
//@version=5
indicator("Example 3", overlay = true)

//@variable The 5-bar moving average of `close` prices.
float ma1 = ta.sma(close, 5)
//@variable The 20-bar moving average of `close` prices.
float ma2 = ta.sma(close, 20)

//@variable The 90% transparent color of the space between MA plots. Green if `ma1 > ma2`, red otherwise.
color fillColor = ma1 > ma2 ? color.new(color.green, 90) : color.new(color.red, 90)

//@variable The ID of the `ma1` plot for use in the `fill()` function.
ma1PlotID = plot(ma1, "5-bar SMA")
//@variable The ID of the `ma2` plot for use in the `fill()` function.
ma2PlotID = plot(ma2, "20-bar SMA")

// Fill the space between the `ma1PlotID` and `ma2PlotID` using the `fillColor`.
fill(ma1PlotID, ma2PlotID, fillColor, "SMA plot fill")
```

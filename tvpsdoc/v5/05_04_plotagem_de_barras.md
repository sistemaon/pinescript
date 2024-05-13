
# Plotagem de Barras

As funções incorporadas [plotcandle()](https://br.tradingview.com/pine-script-reference/v5/#fun_plotcandle) e [plotbar()](https://br.tradingview.com/pine-script-reference/v5/#fun_plotbar) são utilizadas para plotar velas e barras convencionais.

Ambas as funções exigem quatro argumentos que serão usados para os preços __OHLC__ ([open](https://br.tradingview.com/pine-script-reference/v5/#var_open), [high](https://br.tradingview.com/pine-script-reference/v5/#var_high), [low](https://br.tradingview.com/pine-script-reference/v5/#var_low), [close](https://br.tradingview.com/pine-script-reference/v5/#var_close)) (_abertura_, _máximo_, _mínimo_, _fechamento_) das barras que elas plotarão. Se algum desses valores for [na](https://br.tradingview.com/pine-script-reference/v5/#var_na), nenhuma barra será plotada.


<!-- # Plotando Velas com `plotcandle()`

A assinatura de [plotcandle()](https://br.tradingview.com/pine-script-reference/v5/#fun_plotcandle) é:

```c
plotcandle(open, high, low, close, title, color, wickcolor, editable, show_last, bordercolor, display) → void
```

Este comando plota velas simples, todos em azul, usando os valores habituais de __OHLC__, em um painel separado:

```c
//@version=5
indicator("Single-color candles")
plotcandle(open, high, low, close)
```

![Plotando velas com plotcandle() 01](./imgs/BarPlotting-Plotcandle-1.png)

Para colorir de verde ou vermelho, pode-se usar o seguinte código:

```c
//@version=5
indicator("Example 2")
paletteColor = close >= open ? color.lime : color.red
plotbar(open, high, low, close, color = paletteColor)
```

![Plotando velas com plotcandle() 02](./imgs/BarPlotting-Plotcandle-2.png) -->

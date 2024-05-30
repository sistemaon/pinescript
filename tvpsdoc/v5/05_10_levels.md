
# Levels (_Níveis_)

# Níveis `hline()`

Níveis são linhas plotadas usando a função [hline()](https://br.tradingview.com/pine-script-reference/v5/#fun_hline). Ela é projetada para plotar níveis __horizontais__ usando uma __única cor__, ou seja, a cor não muda em diferentes barras. Veja a seção [Níveis](./05_15_plots.md#levels-níveis) da página sobre [plot()](https://br.tradingview.com/pine-script-reference/v5/#plot) para maneiras alternativas de plotar níveis quando [hline()](https://br.tradingview.com/pine-script-reference/v5/#fun_hline) não faz o que é necessário.

A função tem a seguinte assinatura:

```c
hline(price, title, color, linestyle, linewidth, editable) → hline
```

[hline()](https://br.tradingview.com/pine-script-reference/v5/#fun_hline) tem algumas restrições quando comparada a [plot()](https://br.tradingview.com/pine-script-reference/v5/#fun_plot):

- Como o objetivo da função é plotar linhas horizontais, seu parâmetro de `price` requer um argumento "input int/float", o que significa que valores "series float", como [close](https://br.tradingview.com/pine-script-reference/v5/#var_close) ou valores calculados dinamicamente, não podem ser usados.
- Seu parâmetro de `color` requer um argumento "input int", o que impede o uso de cores dinâmicas, ou seja, cores calculadas em cada barra — ou valores "series color".
- Três estilos de linha diferentes são suportados através do parâmetro `linestyle`: `hline.style_solid`, `hline.style_dotted` e `hline.style_dashed`.

Veja [hline()](https://br.tradingview.com/pine-script-reference/v5/#fun_hline) em ação no indicador "True Strength Index":

```c
//@version=5
indicator("TSI")
myTSI = 100 * ta.tsi(close, 25, 13)

hline( 50, "+50",  color.lime)
hline( 25, "+25",  color.green)
hline(  0, "Zero", color.gray, linestyle = hline.style_dotted)
hline(-25, "-25",  color.maroon)
hline(-50, "-50",  color.red)

plot(myTSI)
```

![Níveis hline() 01](./imgs/Levels-HlineLevels-01.png)

![Níveis hline() 02](./imgs/Levels-HlineLevels-02.png)

__Note que:__

- Cinco níveis são exibidos, cada um de uma cor diferente.
- Um estilo de linha diferente é usado para a linha central zero.
- Cores são escolhidas para funcionar bem tanto em temas claros quanto escuros.
- O intervalo usual para os valores do indicador é de __+100__ a __-100__. Como a função embutida [ta.tsi()](https://br.tradingview.com/pine-script-reference/v5/#fun_ta{dot}tsi) retorna valores na faixa de __+1__ a __-1__, o ajuste é feito no código.


# Preenchimentos entre Níveis

O espaço entre dois níveis plotados com [hline()](https://br.tradingview.com/pine-script-reference/v5/#fun_hline) pode ser colorido usando [fill()](https://br.tradingview.com/pine-script-reference/v5/#fun_fill). Lembre-se de que __ambos__ os plots devem ter sido plotados com [hline()](https://br.tradingview.com/pine-script-reference/v5/#fun_hline).

Adicionando algumas cores de fundo ao indicador TSI:

```c
//@version=5
indicator("TSI")
myTSI = 100 * ta.tsi(close, 25, 13)

plus50Hline  = hline( 50, "+50",  color.lime)
plus25Hline  = hline( 25, "+25",  color.green)
zeroHline    = hline(  0, "Zero", color.gray, linestyle = hline.style_dotted)
minus25Hline = hline(-25, "-25",  color.maroon)
minus50Hline = hline(-50, "-50",  color.red)

// ————— Function returns a color in a light shade for use as a background.
fillColor(color col) =>
    color.new(col, 90)

fill(plus50Hline,  plus25Hline,  fillColor(color.lime))
fill(plus25Hline,  zeroHline,    fillColor(color.teal))
fill(zeroHline,    minus25Hline, fillColor(color.maroon))
fill(minus25Hline, minus50Hline, fillColor(color.red))

plot(myTSI)
```

![Preenchimentos entre níveis 01](./imgs/Levels-FillBetweenLevels-01.png)

![Preenchimentos entre níveis 02](./imgs/Levels-FillBetweenLevels-02.png)

__Note que:__

- Agora foi utilizado o valor de retorno das chamadas da função [hline()](https://br.tradingview.com/pine-script-reference/v5/#fun_hline), que é do tipo especial [hline](./04_09_tipagem_do_sistema.md#plot-e-hline). As variáveis `plus50Hline`, `plus25Hline`, `zeroHline`, `minus25Hline` e `minus50Hline` são usadas para armazenar esses IDs de "hline" porque serão necessários nas chamadas [fill()](https://br.tradingview.com/pine-script-reference/v5/#fun_fill) posteriormente.
- Para gerar tons de cores mais claros para as cores de fundo, é declarada uma função `fillColor()` que aceita uma cor e retorna sua transparência de 90. Chamadas para essa função são usadas para os argumentos de cor nas chamadas [fill()](https://br.tradingview.com/pine-script-reference/v5/#fun_fill).
- As chamadas [fill()](https://br.tradingview.com/pine-script-reference/v5/#fun_fill) são feitas para cada um dos quatro preenchimentos diferentes desejados, entre quatro pares diferentes de níveis.
- A cor `color.teal` é usada no segundo preenchimento porque ela produz um verde que se encaixa melhor no esquema de cores do que o `color.green` usado para o nível 25.

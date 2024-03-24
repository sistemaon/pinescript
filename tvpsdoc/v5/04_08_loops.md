
# Loops

## Quando Loops Não são Necessários

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

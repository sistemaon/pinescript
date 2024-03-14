
# Modelo de Execução

O modelo de execução do tempo de execução do Pine Script está intimamente ligado à [série temporal](./000_time_series.md) e a [tipagem do sistema](./000_type_system.md) do Pine Script. Entender os três é fundamental para aproveitar ao máximo o poder do Pine Script.

O modelo de execução determina como o script é executado nos gráficos e, portanto, como o código desenvolvido nos scripts funciona. O código nada faria se não fosse pelo tempo de execução do Pine Script, que entra em ação após ser compilado e, é executado no gráfico porque um dos [eventos que desencadeia a execução de um script](./04_01_modelo_de_execucao.md#eventos-desencadeiando-a-execução-do-script) ocorreu.

Quando um script Pine é carregado em um gráfico, o mesmo é executado uma vez em cada barra histórica usando os valores __OHLCV__ (open, high, low, close, volume) (_abertura, alta, baixa, fechamento, volume_) disponíveis para cada barra. Assim que a execução do script alcança última barra disponível nos dados do gráfico, se a negociação estiver atualmente ativa no símbolo do gráfico, então os indicadores Pine Script serão executados a cada vez que ocorrer uma atualização, ou por, mudanças no preço ou volume. Por padrão as estratégias do Pine Script só serão executadas quando a última barra se fechar, mas também podem ser configuradas para executar a cada atualização, como os indicadores.

Todos os pares de __símbolos/períodos de tempo__ têm um conjunto de dados composto por um número limitado de barras.
Ao explorar um gráfico para a esquerda para ver as barras anteriores do conjunto de dados, as barras correspondentes são carregadas no gráfico.
O processo de carregamento é interrompido quando não há mais barras para aquele par específico do __símbolo/período de tempo__ ou quando o [número máximo de barras](./000_chart_bars.md) permitido pelo tipo de conta foi carregado. Consegue-se navegar pelo gráfico para a esquerda até a primeira barra do conjunto de dados, que tem um valor de índice (_index_) de 0 (veja [bar_index](https://www.tradingview.com/pine-script-reference/v5/#var_bar_index)).

Quando o script é executado pela primeira vez num gráfico, todas as barras num conjunto de dados são _barras históricas_, exceto a mais à direita, ou seja, a barra atual se uma sessão de negociação estiver ativa. Quando a negociação está ativa na barra mais recente, essa é chamada de _barra em tempo real_. A barra em tempo real é atualizada quando uma mudança de preço ou volume é detectada. Quando a barra em tempo real se fecha, ela se torna uma barra em tempo real decorrida, ou seja, uma barra concluída e uma nova barra em tempo real se abre.


# Cálculo Baseado em Barras Históricas

Vamos usar este simples script e acompanhar sua execução em barras históricas:

```c
//@version=5
indicator("My Script", overlay = true)
src = close
a = ta.sma(src, 5)
b = ta.sma(src, 50)
c = ta.cross(a, b)
plot(a, color = color.blue)
plot(b, color = color.black)
plotshape(c, color = color.red)
```

Nas barras históricas, um script é executado equivalentemente no fechamento da barra, quando os valores __OHLCV__ são todos conhecidos para aquela barra. Antes da execução do script na barra, as variáveis integradas como __open__, __high__, __low__, __close__, __volume__ e __time__ (_abertura_, _máxima_, _mínima_, _fechamento_, _volume_ e _tempo_) são definidas com valores correspondentes aos daquela barra. O script é executado __uma vez por barra histórica__.

Nosso exemplo de script é executado pela primeira vez na primeira barra do conjunto de dados no índice 0 (_index 0_). Cada instrução é executada usando os valores da barra atual. Portanto, na primeira barra do conjunto de dados, a seguinte instrução:

```c
src = close
```

Inicializa a variável `src` com o valor `close` para aquela primeira barra, e cada uma das próximas linhas é executada sequencialmente. Como o script só é executado uma vez para cada barra histórica, o script sempre calculará usando o mesmo valor `close` para uma barra histórica específica.

A execução de cada linha no script produz cálculos que, por sua vez, geram os valores de saída do indicador, que podem então ser plotados no gráfico. No exemplo utiliza as funções `plot` e `plotshape` no final do script para produzir valores de saída. No caso de uma estratégia, o resultado dos cálculos pode ser usado para plotar valores ou ditar as ordens a serem colocadas.

Após a execução e a plotagem na primeira barra, o script é executado na segunda barra do conjunto de dados, que tem um índice de 1 (_index 1_). O processo então se repete até que todas as barras históricas no conjunto de dados sejam processadas e o script alcance a barra mais atual no gráfico.

![Cálculo baseado em barras históricas](./imgs/execution_model_calculation_on_history.png)


# Eventos Desencadeando a Execução do Script
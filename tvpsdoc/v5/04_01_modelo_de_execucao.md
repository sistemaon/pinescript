
# Modelo de Execução

O modelo de execução do tempo de execução do Pine Script está intimamente ligado à [série temporal](./000_time_series.md) e a [tipagem do sistema](./000_type_system.md) do Pine Script. Entender os três é fundamental para aproveitar ao máximo o poder do Pine Script.

O modelo de execução determina como o script é executado nos gráficos e, portanto, como o código desenvolvido nos scripts funciona. O código nada faria se não fosse pelo tempo de execução do Pine Script, que entra em ação após ser compilado e, é executado no gráfico porque um dos [eventos que desencadeia a execução de um script](./04_01_modelo_de_execucao.md#eventos-desencadeiando-a-execução-do-script) ocorreu.

Quando um script Pine é carregado em um gráfico, o mesmo é executado uma vez em cada barra histórica usando os valores __OHLCV__ (open, high, low, close, volume) (_abertura, alta, baixa, fechamento, volume_) disponíveis para cada barra. Assim que a execução do script alcança última barra disponível nos dados do gráfico, se a negociação estiver atualmente ativa no símbolo do gráfico, então os indicadores Pine Script serão executados a cada vez que ocorrer uma atualização, ou por, mudanças no preço ou volume. Por padrão as estratégias do Pine Script só serão executadas quando a última barra se fechar, mas também podem ser configuradas para executar a cada atualização, como os indicadores.

Todos os pares de __símbolos/períodos de tempo__ têm um conjunto de dados composto por um número limitado de barras.
Ao explorar um gráfico para a esquerda para ver as barras anteriores do conjunto de dados, as barras correspondentes são carregadas no gráfico.
O processo de carregamento é interrompido quando não há mais barras para aquele par específico do __símbolo/período de tempo__ ou quando o [número máximo de barras](./000_chart_bars.md) permitido pelo tipo de conta foi carregado. Consegue-se navegar pelo gráfico para a esquerda até a primeira barra do conjunto de dados, que tem um valor de índice (_index_) de 0 (veja [bar_index](https://www.tradingview.com/pine-script-reference/v5/#var_bar_index)).

Quando o script é executado pela primeira vez num gráfico, todas as barras num conjunto de dados são _barras históricas_, exceto a mais à direita, ou seja, a barra atual se uma sessão de negociação estiver ativa. Quando a negociação está ativa na barra mais recente, essa é chamada de _barra em tempo real_. A barra em tempo real é atualizada quando uma mudança de preço ou volume é detectada. Quando a barra em tempo real se fecha, ela se torna uma barra em tempo real decorrida, ou seja, uma barra concluída e uma nova barra em tempo real se abre.


# Eventos Desencadeando a Execução do Script
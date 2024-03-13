
# Modelo de Execução

O modelo de execução do tempo de execução do Pine Script está intimamente ligado à [série temporal](./000_time_series.md) e a [tipagem do sistema](./000_type_system.md) do Pine Script. Entender os três é fundamental para aproveitar ao máximo o poder do Pine Script.

O modelo de execução determina como o script é executado nos gráficos e, portanto, como o código desenvolvido nos scripts funciona. O código nada faria se não fosse pelo tempo de execução do Pine Script, que entra em ação após ser compilado e, é executado no gráfico porque um dos [eventos que desencadeia a execução de um script](./04_01_modelo_de_execucao.md#eventos-desencadeiando-a-execução-do-script) ocorreu.

Quando um script Pine é carregado em um gráfico, o mesmo é executado uma vez em cada barra histórica usando os valores __OHLCV__ (open, high, low, close, volume) (_abertura, alta, baixa, fechamento, volume_) disponíveis para cada barra. Assim que a execução do script alcança última barra disponível nos dados do gráfico, se a negociação estiver atualmente ativa no símbolo do gráfico, então os indicadores Pine Script serão executados a cada vez que ocorrer uma atualização, ou seja, mudanças no preço ou volume. Por padrão as estratégias do Pine Script só serão executadas quando a última barra se fechar, mas também podem ser configuradas para executar a cada atualização, assim como os indicadores.

# Eventos Desencadeando a Execução do Script
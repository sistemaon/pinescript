
# Próximos Passos

Após seus [primeiros passos](./01_primeiros_passos.md) e seu [primeiro indicador](./02_primeiro_indicador.md), iremos explorar um pouco mais do cenário do Pine Script compartilhando algumas dicas para orientá-lo em sua jornada de aprendizado do Pine Script.


# "Indicadores" e "Estratégias"

[Estratégias](./000_strategies.md) do Pine Script são usadas para realizar backtest em dados históricos e teste avançado em mercados abertos (_open markets_). Além dos cálculos de indicadores, contêm funções `strategy.*()` para enviar ordens de negociação para o emulador de corretora do Pine Script, que simula a execução. As estratégias exibem os resultados do backtest na aba "Teste de Estratégias" (_"Strategy Tester"_) na parte inferior do gráfico, ao lado da aba "Editor Pine" (_"Pine Editor"_).

Os indicadores do Pine Script também realizam cálculos, mas não podem ser usados em backtesting. Pois não exigem o emulador de corretora, consomem menos recursos e executam de maneira mais célere. Portanto, é vantajoso usar indicadores sempre que possível.

Tanto indicadores quanto estratégias podem ser executados em modo de sobreposição (_overlay mode_) (sobre as barras do gráfico (no gráfico)) ou em modo de painel (_pane mode_) (em uma seção separada abaixo ou acima do gráfico). Ambos também podem plotar informações em seu respectivo espaço, e ambos podem gerar [eventos de alerta](./000_alert_events.md).


# Como os Scripts são Executados

Um script em Pine __não__ é como programas em muitas das linguagens de programação que executam uma única vez e depois param. No ambiente de _execução_ do Pine Script, um script roda de formar semelhante a um loop invisível, onde é executado uma vez em cada barra do gráfico, da esquerda para a direita. Barras do gráfico que fecham enquanto o script executa-os são chamadas de _barras históricas_ (_historical bars_). Quando a execução alcança a última barra do gráfico e o mercado está aberto, estamos na _barra em tempo real_ (_realtime bar_). O script então é executado uma vez a cada vez em que uma mudança de preço ou volume é detectada, e uma última vez para aquela _barra em tempo real_ ao fechar. A barra em tempo real então se torna em uma _barra em tempo real decorrido_. Note que quando o script é executado em tempo real, ele não recalcula em todas as barras históricas do gráfico em cada atualização de preço/volume. Pois já foi calculado uma vez nessas barras, então não é necessário recalculá-las a cada movimento do gráfico. Para mais informações veja a página do [modelo de execução](000_execution_model.md).

Quando o script é executado em uma barra histórica, a variável [close](https://www.tradingview.com/pine-script-reference/v5/#var_close), embutida, contém o valor do _fechamento_ daquela barra. Quando o script é executado na barra em tempo real, o [close](https://www.tradingview.com/pine-script-reference/v5/#var_close) retorna o preço __corrente__/__atual__ do símbolo até que a barra se fecha.

Diferente dos indicadores, normalmente as estratégias são executadas apenas uma vez nas barras em tempo real, ao fecharem-se. Também podem ser configurados para executar em cada atualização de preço/volume, se for necessário. Para mais informações e para entender como as estratégias calculam diferentemente dos indicadores veja a página sobre [Estratégias](./000_strategies.md).

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


# Série Temporal

A principal estrutura de dados usada no Pine Script é chamada de [série temporal](./000_time_series.md) (_time series_). Séries temporais possui um valor para cada barra na qual o script é executado, então expandem-se continuamente à medida que o script é executado em demais barras. Valores passados da série temporal podem ser referenciados usando o operador de referência histórica: [[]](https://www.tradingview.com/pine-script-reference/v5/#op_[]).
Por exemplo, `close[1]` refere-se ao valor de fechamento ([close](https://www.tradingview.com/pine-script-reference/v5/#var_close)) da barra anterior no momento em que o script está sendo executado.

Embora este mecanismo de indexação possa lembrar sobre matrizes (_arrays_), uma série temporal é diferente e pensar em termos de arrays pode dificultar a compreensão desse conceito fundamental do Pine Script.
Um bom entendimento tanto do [modelo de execução](./000_execution_model.md) quanto das [séries temporais](./000_time_series.md) é essencial para compreender como os scripts Pine funcionam.
Se desconhece dados organizados em séries temporais, será necessário praticar para aprender a utilizá-los eficientemente. Assim que familiarizar com esses conceitos-chave, descobrirá que, ao combinar o uso de séries temporais com funções integradas intrinsecamente projetadas para manuseá-las de forma eficiente, bastante pode ser realizado em poucas linhas de código.


# Publicando Scripts

TradingView é o lar de uma vasta comunidade de programadores de Pine Script e milhões de traders do mundo todo. Uma vez que se torne proficiente o suficiente em Pine Script, pode optar por compartilhar seus scripts com outros traders. Antes de fazer isso, por favor, reserve um tempo para aprender Pine Script o suficiente para fornecer aos traders uma ferramenta original e confiável. Todos os scripts publicados, sem exceção, são analisados por nossa equipe de moderadores e devem cumprir com as nossas [Regras de Publicação de Scripts](https://www.tradingview.com/support/solutions/43000590599), que exigem que sejam originais e bem documentados.

Se deseja usar scripts em Pine para uso próprio, simplesmente escreva-os no Editor Pine e adicione-os ao gráfico a partir daí; não precisa publicá-los para usá-los. Se deseja compartilhar seus scripts com amigos, pode publicá-los de forma privada e enviar o link da sua publicação privada. Para mais informações veja a página [Publicando](./000_publishing.md).


# Próximos Passos

Após seus [primeiros passos](./01_primeiros_passos.md) e seu [primeiro indicador](./02_primeiro_indicador.md), iremos explorar um pouco mais do cenário do Pine Script compartilhando algumas dicas para orientá-lo em sua jornada de aprendizado do Pine Script.


# "Indicadores" e "Estratégias"

[Estratégias](./000_strategies.md) do Pine Script são usadas para realizar backtest em dados históricos e teste avançado em mercados abertos (_open markets_). Além dos cálculos de indicadores, contêm funções `strategy.*()` para enviar ordens de negociação para o emulador de corretora do Pine Script, que simula a execução. As estratégias exibem os resultados do backtest na aba "Teste de Estratégias" (_"Strategy Tester"_) na parte inferior do gráfico, ao lado da aba "Editor Pine" (_"Pine Editor"_).

Os indicadores do Pine Script também realizam cálculos, mas não podem ser usados em backtesting. Pois não exigem o emulador de corretora, consomem menos recursos e executam de maneira mais célere. Portanto, é vantajoso usar indicadores sempre que possível.

Tanto indicadores quanto estratégias podem ser executados em modo de sobreposição (_overlay mode_) (sobre as barras do gráfico (no gráfico)) ou em modo de painel (_pane mode_) (em uma seção separada abaixo ou acima do gráfico). Ambos também podem plotar informações em seu respectivo espaço, e ambos podem gerar [eventos de alerta](./000_alert_events.md).



# Alertas

Os alertas do TradingView funcionam 24 horas por dia, 7 dias por semana, (_24x7_) em nossos servidores e não exigem que as pessoas estejam logadas para serem executados. Os alertas são criados a partir da _User Interface (UI)_/_interface de usuário_  dos gráficos. Todas as informações necessárias para entender como os alertas funcionam e como criá-los a partir da __UI__ dos gráficos podem ser encontradas na página [Sobre os alertas do TradingView](https://br.tradingview.com/support/solutions/43000520149/) no _Centro de Ajuda_.

Os tipos de alerta disponíveis no TradingView (_generic alerts_, _drawing alerts_ e _script alerts on order fill events_)/(_alertas genéricos_, _alertas de desenho_ e _alertas de scripts em eventos de preenchimento de ordens_) são criados a partir de símbolos ou scripts carregados no gráfico e não exigem codificação específica. É possível criar esses tipos de alerta a partir da _UI_ interface do usuário dos gráficos.

Outros tipos de alertas (_script alerts triggering on alert() function calls_, e _alertcondition() alerts_)/(_alertas de script acionando chamadas da função alert()_ e _alertas alertcondition()_) exigem que um código específico em Pine Script esteja presente em um script para criar um _evento de alerta_ antes que os usuários possam criar alertas a partir dele usando a interface do usuário dos gráficos. Além disso, enquanto os usuários de script podem criar _alertas de script_ acionados em _eventos de preenchimento de ordens_ a partir da interface do usuário dos gráficos em qualquer estratégia carregada no seu gráfico. Programadores podem especificar mensagens de alerta de preenchimento de ordem explícitas em seu script para cada tipo de ordem preenchida pelo _broker emulator_ (_emulador de corretora_).

Esta página aborda as diferentes maneiras pelas quais os programadores em Pine Script podem codificar seus scripts para criar eventos de alerta, a partir dos quais os usuários do script, por sua vez, poderão criar alertas através da interface de usuário dos gráficos.

Serão explorados:

- Como utilizar a função [alert()](https://br.tradingview.com/pine-script-reference/v5/#fun_alert) para chamadas de _função alert()_ em indicadores ou estratégias, que podem então ser incluídas nos _alertas de script_ criados a partir da interface de usuário dos gráficos.
- Como adicionar mensagens de alerta personalizadas para serem incluídas nos _alertas de script_ que são acionados pelos _eventos de preenchimento de ordens_ em estratégias.
- Como utilizar a função [alertcondition()](https://br.tradingview.com/pine-script-reference/v5/#fun_alertcondition) para gerar, apenas em indicadores, _eventos de alertcondition()_ que podem, então, ser usados para criar _alertas de alertcondition()_ a partir da interface de usuário dos gráficos.

Tenha em mente que:

- Nenhum código relacionado a alertas no Pine Script pode criar um alerta em execução na interface de usuário dos gráficos; ele apenas cria eventos de alerta que podem então ser usados por usuários de script para criar alertas em execução a partir da interface de usuário dos gráficos.
- Alertas só são acionados na barra em tempo real. O escopo operacional do código Pine Script que lida com qualquer tipo de alerta é, portanto, restrito apenas a barras em tempo real.
- Quando um alerta é criado na interface de usuário dos gráficos, TradingView salva uma imagem espelho do script e suas entradas, junto com o símbolo principal do gráfico e o _timeframe_ para executar o alerta em seus servidores. Alterações subsequentes nas entradas do script ou no gráfico, portanto, não afetarão os alertas em execução previamente criados a partir deles. Se desejar que as alterações em seu contexto se reflitam no comportamento de um alerta em execução, será necessário excluir o alerta e criar um novo no novo contexto.


# Background

Os diferentes métodos que os programadores de Pine Script podem usar atualmente para criar eventos de alerta em seus scripts são o resultado de melhorias sucessivas implementadas ao longo da evolução do Pine Script. A função [alertcondition()](https://br.tradingview.com/pine-script-reference/v5/#fun_alertcondition), que funciona apenas em indicadores, foi o primeiro recurso que permitiu aos programadores do Pine Script criar eventos de alerta. Posteriormente, foram introduzidos alertas de preenchimento de ordens para estratégias, que são ativados quando o emulador de corretagem cria _eventos de preenchimento de ordens_. _Eventos de preenchimento de ordens_ não exigem um código especial para que os usuários do script criem alertas sobre eles, mas, por meio do parâmetro `alert_message` nas funções de `strategy.*()` que geram ordens, os programadores podem personalizar a mensagem dos alertas que são ativados em _eventos de preenchimento de ordens_, definindo uma mensagem de alerta distinta para qualquer número de eventos de cumprimento de ordens.

A função [alert()](https://br.tradingview.com/pine-script-reference/v5/#fun_alert) é a adição mais recente ao Pine Script. Ela substitui, mais ou menos, a função [alertcondition()](https://br.tradingview.com/pine-script-reference/v5/#fun_alertcondition), e, quando utilizada em estratégias, serve como um complemento útil para alertas em _eventos de preenchimento de ordens_.


# Qual é o Tipo de Alerta mais Adequado?

Para os programadores do Pine Script, a função [alert()](https://br.tradingview.com/pine-script-reference/v5/#fun_alert) geralmente será mais fácil e flexível para trabalhar. Diferentemente da [alertcondition()](https://br.tradingview.com/pine-script-reference/v5/#fun_alertcondition), ela permite mensagens de alerta dinâmicas, funciona tanto em indicadores quanto em estratégias, e o programador decide a frequência dos eventos de [alert()](https://br.tradingview.com/pine-script-reference/v5/#fun_alert).

Enquanto as chamadas de [alert()](https://br.tradingview.com/pine-script-reference/v5/#fun_alert) possam ser geradas com base em qualquer lógica programável no Pine Script, incluindo quando ordens são __enviadas__ ao emulador de corretagem em estratégias, elas não podem ser programadas para disparar quando as ordens são __executadas__ (ou _preenchidas_), pois após as ordens serem enviadas ao emulador de corretagem, o emulador controla a execução delas e não relata os eventos de preenchimento diretamente ao script.

Quando um usuário de script deseja gerar um alerta sobre eventos de preenchimento de ordem de uma estratégia, ele deve incluir esses eventos ao criar um _alerta de script_ sobre a estratégia na caixa de diálogo "_Create Alert_" ("_Criar Alerta_"). Não é necessário um código especial nos scripts para que os usuários possam fazer isso. No entanto, a mensagem enviada com eventos de preenchimento de ordens pode ser personalizada pelos programadores por meio do uso do parâmetro `alert_message` nas chamadas de função de `strategy.*()` que gera ordens. A combinação de chamadas de [alert()](https://br.tradingview.com/pine-script-reference/v5/#fun_alert) e o uso de argumentos de `alert_message` personalizados nas chamadas de função de `strategy.*()` que gera ordens deve permitir que os programadores gerem eventos de alerta sobre a maioria das condições que ocorrem durante a execução de seus scripts.

A função [alertcondition()](https://br.tradingview.com/pine-script-reference/v5/#fun_alertcondition) permanece no Pine Script por compatibilidade com versões anteriores, mas também pode ser utilizada vantajosamente para gerar alertas distintos disponíveis para seleção como itens individuais no campo "_Condition_" ("_Condição_") da caixa de diálogo "_Create Alert_" ("_Criar Alerta_").

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


# Alertas de Script

Quando um usuário de script cria um _alerta de script_ usando a caixa de diálogo "Create Alert" ("_Criar Alerta_"), os eventos que podem acionar o alerta variam dependendo se o alerta é criado a partir de um indicador ou de uma estratégia.

Um _alerta de script_ criado a partir de um __indicador__ será acionado quando:

- O indicador contém chamadas de [alert()](https://br.tradingview.com/pine-script-reference/v5/#fun_alert).
- A lógica do código permite que uma chamada específica de [alert()](https://br.tradingview.com/pine-script-reference/v5/#fun_alert) seja executada.
- A frequência especificada na chamada de [alert()](https://br.tradingview.com/pine-script-reference/v5/#fun_alert) permite que o alerta seja acionado.

Um _alerta de script_ criado a partir de uma __estratégia__ pode ser acionado por _chamadas da função alert()_, por _eventos de preenchimento de ordens_ ou ambos. O usuário do script que cria um alerta sobre uma estratégia decide quais tipos de eventos deseja incluir no _alerta de script_. Embora seja possível criar um _alerta de script_ sobre _eventos de preenchimento de ordens_ sem a necessidade de incluir código especial na estratégia, é necessário que ela contenha chamadas de [alert()](https://br.tradingview.com/pine-script-reference/v5/#fun_alert) para que os usuários possam incluir _chamadas da função alert()_ no _alerta de script_, ou seja, para que os usuários possam configurar alertas que se ativam com base na função _alert()_ em uma estratégia, é necessário primeiro que essa estratégia tenha chamadas da função _alert()_ em seu código.

## Eventos da Função `alert()`

A função [alert()](https://br.tradingview.com/pine-script-reference/v5/#fun_alert) possui a seguinte assinatura:

```c
alert(message, freq)
```

`message`

Uma "série de strings" representa o texto da mensagem enviada quando o alerta é acionado. Como esse argumento permite valores "_series_", ele pode ser gerado em tempo de execução e variar de barra a barra, tornando-o dinâmico.

`freq`

Uma "input string" especifica a frequência de acionamento do alerta. Os argumentos válidos são:

- `alert.freq_once_per_bar`: Apenas a primeira chamada por barra em tempo real aciona o alerta (valor padrão).
- `alert.freq_once_per_bar_close`: Um alerta é acionado apenas quando a barra em tempo real fecha e uma chamada de [alert()](https://br.tradingview.com/pine-script-reference/v5/#fun_alert) é executada durante essa iteração do script.
- `alert.freq_all`: Todas as chamadas durante a barra em tempo real acionam o alerta.

A função [alert()](https://br.tradingview.com/pine-script-reference/v5/#fun_alert) pode ser utilizada tanto em indicadores quanto em estratégias. Para que uma chamada de [alert()](https://br.tradingview.com/pine-script-reference/v5/#fun_alert) acione um _alerta de script_ configurado com base em _chamadas da função alert()_, a lógica do script deve permitir que a chamada de [alert()](https://br.tradingview.com/pine-script-reference/v5/#fun_alert) seja executada, __e__ a frequência determinada pelo parâmetro `freq` deve permitir que o alerta seja acionado.

Observe que, por padrão, as estratégias são recalculadas no fechamento da barra, então se a função [alert()](https://br.tradingview.com/pine-script-reference/v5/#fun_alert) com a frequência `alert.freq_all` ou `alert.freq_once_per_bar` for usada em uma estratégia, ela será chamada no máximo uma vez no fechamento da barra. Para permitir que a função [alert()](https://br.tradingview.com/pine-script-reference/v5/#fun_alert) seja chamada durante o processo de construção da barra, é necessário ativar a opção [`calc_on_every_tick`](https://br.tradingview.com/pine-script-reference/v5/#fun_strategy).

## Usando Todas as Chamadas de `alert()`

Observa este exemplo de detecção de cruzamentos da linha central do RSI:

```c
//@version=5
indicator("All `alert()` calls")
r = ta.rsi(close, 20)

// Detect crosses.
xUp = ta.crossover( r, 50)
xDn = ta.crossunder(r, 50)
// Trigger an alert on crosses.
if xUp
    alert("Go long (RSI is " + str.tostring(r, "#.00)"))
else if xDn
    alert("Go short (RSI is " + str.tostring(r, "#.00)"))

plotchar(xUp, "Go Long",  "▲", location.bottom, color.lime, size = size.tiny)
plotchar(xDn, "Go Short", "▼", location.top,    color.red,  size = size.tiny)
hline(50)
plot(r)
```

Se um _alerta de script_ for criado a partir deste script:

- Quando o RSI cruzar a linha central para cima, o _alerta de script_ será acionado com a mensagem "Go long...". Quando o RSI cruzar a linha central para baixo, o _alerta de script_ será acionado com a mensagem "Go short...".
- Como nenhum argumento é especificado para o parâmetro `freq` na chamada de [alert()](https://br.tradingview.com/pine-script-reference/v5/#fun_alert), o valor padrão de `alert.freq_once_per_bar` será usado, então o alerta só será acionado na primeira vez que cada uma das chamadas de [alert()](https://br.tradingview.com/pine-script-reference/v5/#fun_alert) for executada durante a barra em tempo real.
- A mensagem enviada com o alerta é composta por duas partes: uma string constante e, em seguida, o resultado da chamada [str.tostring()](https://br.tradingview.com/pine-script-reference/v5/#fun_str{dot}tostring), que incluirá o valor do RSI no momento em que a chamada de [alert()](https://br.tradingview.com/pine-script-reference/v5/#fun_alert) é executada pelo script. Uma mensagem de alerta para um cruzamento para cima seria, por exemplo: "Go long (RSI is 53.41)".
- Como um _alerta de script_ sempre é acionado em qualquer ocorrência de uma chamada para [alert()](https://br.tradingview.com/pine-script-reference/v5/#fun_alert), desde que a frequência usada na chamada permita, este script específico não permite que um usuário de script restrinja seu _alerta de script_ apenas para posições longas, por exemplo.

__Note que:__

- Ao contrário de uma chamada de [alertcondition()](https://br.tradingview.com/pine-script-reference/v5/#fun_alertcondition) que sempre é colocada na coluna 0 (no escopo global do script), a chamada de [alert()](https://br.tradingview.com/pine-script-reference/v5/#fun_alert) é colocada no escopo local de uma condição [if](https://br.tradingview.com/pine-script-reference/v5/#kw_if), de modo que só é executada quando a condição _desencadeante_ é atendida. Se uma chamada de [alert()](https://br.tradingview.com/pine-script-reference/v5/#fun_alert) fosse colocada no escopo global do script na coluna 0, ela seria executada em todas as barras, o que provavelmente não seria o comportamento desejado.
- Um [alertcondition()](https://br.tradingview.com/pine-script-reference/v5/#fun_alertcondition) não poderia aceitar a mesma string que foi usada para a mensagem do alerta devido ao uso da chamada [str.tostring()](https://br.tradingview.com/pine-script-reference/v5/#fun_str{dot}tostring). As mensagens de [alertcondition()](https://br.tradingview.com/pine-script-reference/v5/#fun_alertcondition) devem ser strings constantes.

Por fim, como as mensagens de [alert()](https://br.tradingview.com/pine-script-reference/v5/#fun_alert) podem ser construídas dinamicamente em tempo de execução, seria possível utilizar o seguinte código para gerar os eventos de alerta:

```c
// Trigger an alert on crosses.
if xUp or xDn
    firstPart = (xUp ? "Go long" : "Go short") + " (RSI is "
    alert(firstPart + str.tostring(r, "#.00)"))
```

## Usando Chamadas Seletivas de `alert()`

Quando os usuários criam um __alerta de script com base em chamadas da função alert()__, o alerta será acionado em qualquer chamada que o script faça à função [alert()](https://br.tradingview.com/pine-script-reference/v5/#fun_alert), desde que as restrições de frequência sejam atendidas. Para permitir que os usuários do script selecionem qual chamada da função [alert()](https://br.tradingview.com/pine-script-reference/v5/#fun_alert) no script acionará um _alerta de script_, é necessário fornecer-lhes meios para indicar sua preferência nas entradas do script e codificar a lógica apropriada no script. Dessa forma, os usuários poderão criar múltiplos _alertas de script_ a partir de um único script, cada um se comportando de maneira diferente conforme as escolhas feitas nas entradas do script antes de criar o alerta na interface gráfica.

Suponha, para o próximo exemplo, que se deseja oferecer a opção de acionar alertas apenas para posições _long_, apenas para posições _short_ ou ambas. O script poderia ser codificado desta forma:

```c
//@version=5
indicator("Selective `alert()` calls")
detectLongsInput  = input.bool(true,  "Detect Longs")
detectShortsInput = input.bool(true,  "Detect Shorts")
repaintInput      = input.bool(false, "Allow Repainting")

r = ta.rsi(close, 20)
// Detect crosses.
xUp = ta.crossover( r, 50)
xDn = ta.crossunder(r, 50)
// Only generate entries when the trade's direction is allowed in inputs.
enterLong  = detectLongsInput  and xUp and (repaintInput or barstate.isconfirmed)
enterShort = detectShortsInput and xDn and (repaintInput or barstate.isconfirmed)
// Trigger the alerts only when the compound condition is met.
if enterLong
    alert("Go long (RSI is " + str.tostring(r, "#.00)"))
else if enterShort
    alert("Go short (RSI is " + str.tostring(r, "#.00)"))

plotchar(enterLong,  "Go Long",  "▲", location.bottom, color.lime, size = size.tiny)
plotchar(enterShort, "Go Short", "▼", location.top,    color.red,  size = size.tiny)
hline(50)
plot(r)
```

__Note como:__

- Foi criado uma condição composta que só é atendida quando a seleção do usuário permite uma entrada naquela direção. Uma entrada _long_ no cruzamento da linha central só dispara o alerta quando as entradas _longs_ estão habilitadas nos _Inputs_ (_Entradas_) do script.
- Foi oferecido ao usuário a opção de indicar sua preferência por repintura, input do script "_Allow Repainting_". Quando ele não permite que os cálculos sejam repintados, é esperado a confirmação da barra para acionar a condição composta. Dessa forma, o alerta e o marcador só aparecem no final da barra em tempo real.
- Se um usuário deste script quisesse criar dois alertas de script distintos a partir deste script, ou seja, um acionando apenas em posições _longs_ e outro apenas em posições _shorts_, então ele precisaria:
    - Selecionar apenas "_Detect Longs_" nos _inputs_ e criar um primeiro _alerta de script_ no script.
    - Selecionar apenas "_Detect Shorts_" nos _inputs_ e criar outro _alerta de script_ no script.

## Em Estratégias

Chamadas da função [alert()](https://br.tradingview.com/pine-script-reference/v5/#fun_alert) também podem ser utilizadas em estratégias, com a observação de que, por padrão, as estratégias executam apenas no [close](https://br.tradingview.com/pine-script-reference/v5/#var_close) (_fechamento_) das barras em tempo real. A menos que `calc_on_every_tick = true` seja usado na declaração [strategy()](https://br.tradingview.com/pine-script-reference/v5/#fun_strategy), todas as chamadas de [alert()](https://br.tradingview.com/pine-script-reference/v5/#fun_alert) utilizarão a frequência `alert.freq_once_per_bar_close`, independentemente do argumento usado para `freq`.

Enquanto os _alertas de script_ em estratégias utilizem _eventos de preenchimento de ordens_ para acionar alertas quando o emulador de corretagem executa ordens, a função [alert()](https://br.tradingview.com/pine-script-reference/v5/#fun_alert) pode ser usada vantajosamente para gerar outros eventos de alerta em estratégias.

Esta estratégia cria _chamadas da função alert()_ quando o RSI se move contra a operação por três barras consecutivas:

```c
//@version=5
strategy("Strategy with selective `alert()` calls")
r = ta.rsi(close, 20)

// Detect crosses.
xUp = ta.crossover( r, 50)
xDn = ta.crossunder(r, 50)
// Place orders on crosses.
if xUp
    strategy.entry("Long", strategy.long)
else if xDn
    strategy.entry("Short", strategy.short)

// Trigger an alert when RSI diverges from our trade's direction.
divInLongTrade  = strategy.position_size > 0 and ta.falling(r, 3)
divInShortTrade = strategy.position_size < 0 and ta.rising( r, 3)
if divInLongTrade
    alert("WARNING: Falling RSI", alert.freq_once_per_bar_close)
if divInShortTrade
    alert("WARNING: Rising RSI", alert.freq_once_per_bar_close)

plotchar(xUp, "Go Long",  "▲", location.bottom, color.lime, size = size.tiny)
plotchar(xDn, "Go Short", "▼", location.top,    color.red,  size = size.tiny)
plotchar(divInLongTrade,  "WARNING: Falling RSI", "•", location.top,    color.red,  size = size.tiny)
plotchar(divInShortTrade, "WARNING: Rising RSI",  "•", location.bottom, color.lime, size = size.tiny)
hline(50)
plot(r)
```

Se um usuário criar um _alerta de script_ a partir desta estratégia e incluir tanto _eventos de preenchimento de ordens_ quanto _chamadas da função alert()_ em seu alerta, o alerta será acionado sempre que uma ordem for executada ou quando uma das chamadas de [alert()](https://br.tradingview.com/pine-script-reference/v5/#fun_alert) for executada pelo script na iteração de fechamento da barra em tempo real, isto é, quando [barstate.isrealtime](https://br.tradingview.com/pine-script-reference/v5/#var_barstate{dot}isrealtime) e [barstate.isconfirmed](https://br.tradingview.com/pine-script-reference/v5/#var_barstate{dot}isconfirmed) forem ambos verdadeiros. Os _eventos da função alert()_ no script só acionarão o alerta quando a barra em tempo real fechar, pois `alert.freq_once_per_bar_close` é o argumento usado para o parâmetro `freq` nas chamadas de [alert()](https://br.tradingview.com/pine-script-reference/v5/#fun_alert).

## _Order Fill Events_ (_Eventos de Preenchimento de Ordens_)

Quando um _alerta de script_ é criado a partir de um indicador, ele só pode ser acionado por _chamadas da função alert()_. No entanto, quando um _alerta de script_ é criado a partir de uma estratégia, o usuário pode especificar que _eventos de preenchimento de ordens_ também acionem o _alerta de script_. Um _evento de preenchimento de ordem_ é qualquer evento gerado pelo emulador de corretagem que cause a execução de uma ordem simulada. Isso equivale ao preenchimento de uma ordem de negociação por uma corretora/bolsa. As ordens não são necessariamente executadas no momento em que são colocadas. Em uma estratégia, a execução das ordens só pode ser detectada indiretamente e após o fato, analisando mudanças em variáveis ​​incorporadas como [strategy.opentrades](https://br.tradingview.com/pine-script-reference/v5/#var_strategy{dot}opentrades) ou [strategy.position_size](https://br.tradingview.com/pine-script-reference/v5/#var_strategy{dot}position_size). _Alertas de script_ configurados em _eventos de preenchimento de ordens_ são úteis pois permitem o acionamento de alertas no momento preciso da execução de uma ordem, antes que a lógica do script possa detectá-la.

Programadores de Pine Script podem personalizar a mensagem de alerta enviada quando ordens específicas são executadas. Embora isso não seja um pré-requisito para que _eventos de preenchimento de ordens_ sejam acionados, mensagens de alerta personalizadas podem ser úteis porque permitem a inclusão de sintaxe personalizada nos alertas, a fim de encaminhar ordens reais para uma _engine_ de execução de terceiros, por exemplo. A especificação de mensagens de alerta personalizadas para _eventos de preenchimento de ordens_ específicos é realizada por meio do parâmetro `alert_message` em funções que podem gerar ordens: [strategy.close()](https://br.tradingview.com/pine-script-reference/v5/#fun_strategy{dot}close), [strategy.entry()](https://br.tradingview.com/pine-script-reference/v5/#fun_strategy{dot}entry), [strategy.exit()](https://br.tradingview.com/pine-script-reference/v5/#fun_strategy{dot}exit) e [strategy.order()](https://br.tradingview.com/pine-script-reference/v5/#fun_strategy{dot}order).

O argumento usado para o parâmetro `alert_message` é uma "series string", portanto, pode ser construído dinamicamente usando qualquer variável disponível para o script, desde que seja convertida para o formato de string.

Examinando uma estratégia onde utiliza-se o parâmetro `alert_message` nas chamadas [strategy.entry()](https://br.tradingview.com/pine-script-reference/v5/#fun_strategy{dot}entry):

```c
//@version=5
strategy("Strategy using `alert_message`")
r = ta.rsi(close, 20)

// Detect crosses.
xUp = ta.crossover( r, 50)
xDn = ta.crossunder(r, 50)
// Place order on crosses using a custom alert message for each.
if xUp
    strategy.entry("Long", strategy.long, stop = high, alert_message = "Stop-buy executed (stop was " + str.tostring(high) + ")")
else if xDn
    strategy.entry("Short", strategy.short, stop = low, alert_message = "Stop-sell executed (stop was " + str.tostring(low) + ")")

plotchar(xUp, "Go Long",  "▲", location.bottom, color.lime, size = size.tiny)
plotchar(xDn, "Go Short", "▼", location.top,    color.red,  size = size.tiny)
hline(50)
plot(r)
```

__Note que:__

- Utiliza-se o parâmetro `stop` nas chamadas de [strategy.entry()](https://br.tradingview.com/pine-script-reference/v5/#fun_strategy{dot}entry), o que cria ordens de compra e venda por stop. Isso implica que as ordens de compra só serão executadas quando o preço estiver acima do `high` (_máximo_) da barra onde a ordem foi colocada, e as ordens de venda só serão executadas quando o preço estiver abaixo do `low` (_mínimo_) da barra onde a ordem foi colocada.
- As setas para cima/para baixo que são plotadas com [plotchar()](https://br.tradingview.com/pine-script-reference/v5/#fun_plotchar), são plotadas quando as ordens são __colocadas__. Pode transcorrer um número variável de barras antes que a ordem seja efetivamente executada e, em alguns casos, a ordem nunca será executada porque o preço não atende à condição necessária.
- Devido ao uso do mesmo argumento de `id` para todas as ordens de compra, qualquer nova ordem de compra colocada antes da condição de uma ordem anterior ser atendida substituirá essa ordem. O mesmo se aplica às ordens de venda.
- As variáveis incluídas no argumento `alert_message` são avaliadas no momento em que a ordem é executada, ou seja, quando o alerta é acionado.

Quando o parâmetro `alert_message` é utilizado nas chamadas de função `strategy.*()` que geram ordens da estratégia, os usuários do script devem incluir o placeholder `{{strategy.order.alert_message}}` no campo "Message" ("_Mensagem_") da caixa de diálogo "Create Alert" ("_Criar Alerta_") ao criar _alertas de script_ sobre _eventos de preenchimento de ordens_. Isso é necessário para que o argumento `alert_message` usado nas chamadas de função `strategy.*()` que geram ordens da estratégia seja utilizado na mensagem dos alertas acionados em cada _evento de preenchimento de ordem_. Quando apenas o placeholder `{{strategy.order.alert_message}}` é usado no campo "Message" ("_Mensagem_") e o parâmetro `alert_message` está presente em apenas algumas das chamadas de função `strategy.*()` que geram ordens em sua estratégia, uma string vazia substituirá o placeholder na mensagem dos alertas acionados por qualquer chamada de função `strategy.*()` que gere ordens e não utilize o parâmetro `alert_message`.

Embora outros placeholders possam ser usados no campo "Message" ("_Mensagem_") da caixa de diálogo "Create Alert" ("_Criar Alerta_") por usuários que criam alertas sobre _eventos de preenchimento de ordens_, eles não podem ser usados no argumento de `alert_message`.


# Eventos `alertcondition()`

A função [alertcondition()](https://br.tradingview.com/pine-script-reference/v5/#fun_alertcondition) permite que programadores criem _eventos de condição de alerta_ individuais em seus indicadores. Um indicador pode conter mais de uma chamada à [alertcondition()](https://br.tradingview.com/pine-script-reference/v5/#fun_alertcondition). Cada chamada para [alertcondition()](https://br.tradingview.com/pine-script-reference/v5/#fun_alertcondition) em um script cria um alerta correspondente selecionável no menu suspenso "_Condition_" ("_Condição_") da caixa de diálogo "_Create Alert_" ("_Criar Alerta_").

Embora a presença de chamadas [alertcondition()](https://br.tradingview.com/pine-script-reference/v5/#fun_alertcondition) em um script de __strategy__ (__estratégia__) não cause um erro de compilação, alertas não podem ser criados a partir delas.

A função [alertcondition()](https://br.tradingview.com/pine-script-reference/v5/#fun_alertcondition) possui a seguinte assinatura:

```c
alertcondition(condition, title, message)
```

`condition`

Um valor de "series bool" (`true` ou `false`) que determina quando o alerta será acionado. É um argumento obrigatório. Quando o valor é `true`, o alerta é acionado. Quando é `false`, o alerta não é acionado. Ao contrário das chamadas da função [alert()](https://br.tradingview.com/pine-script-reference/v5/#fun_alert), as chamadas de [alertcondition()](https://br.tradingview.com/pine-script-reference/v5/#fun_alertcondition) devem iniciar na coluna zero de uma linha, portanto, não podem ser colocadas em blocos condicionais.

`title`

Um argumento opcional "const string" que define o nome da condição de alerta conforme ele aparecerá no campo "_Condition_" ("_Condição_") da caixa de diálogo "_Create Alert_" ("_Criar Alerta_") na interface gráfica. Se nenhum argumento for fornecido, "Alert" será usado.

`message`

Um argumento opcional "const string" que especifica a mensagem de texto a ser exibida quando o alerta é acionado. O texto aparecerá no campo "_Message_" ("_Mensagem_") da caixa de diálogo "_Create Alert_" ("_Criar Alerta_"), onde os usuários do script podem então modificá-lo ao criar um alerta. __Como esse argumento deve ser uma "const string", ele precisa ser conhecido no momento da compilação e, portanto, não pode variar de barra para barra__. No entanto, pode conter _placeholders_ que serão substituídos em tempo de execução por valores dinâmicos que podem mudar de barra para barra. Consulte a seção de [Placeholders](./05_01_alertas.md#placeholders) desta página para obter uma lista.

A função [alertcondition()](https://br.tradingview.com/pine-script-reference/v5/#fun_alertcondition) não inclui o parâmetro `freq`. A frequência dos _alertas de alertcondition()_ é determinada pelos usuários na caixa de diálogo "_Create Alert_" ("_Criar Alerta_").


## Utilizando uma Condição

Aqui está um exemplo de código que cria _eventos alertcondition()_:

```c
//@version=5
indicator("`alertcondition()` on single condition")
r = ta.rsi(close, 20)

xUp = ta.crossover( r, 50)
xDn = ta.crossunder(r, 50)

plot(r, "RSI")
hline(50)
plotchar(xUp, "Long",  "▲", location.bottom, color.lime, size = size.tiny)
plotchar(xDn, "Short", "▼", location.top,    color.red,  size = size.tiny)

alertcondition(xUp, "Long Alert",  "Go long")
alertcondition(xDn, "Short Alert", "Go short ")
```

Devido à presença de duas chamadas de [alertcondition()](https://br.tradingview.com/pine-script-reference/v5/#fun_alertcondition)` no script, dois alertas distintos estarão disponíveis no campo "Condição" da caixa de diálogo "Criar Alerta": _"Long Alert_" e "_Short Alert_".

Para incluir o valor do RSI no momento do cruzamento, não é possível adicionar diretamente seu valor à string de `message` utilizando `str.tostring(r)`, como seria possível em uma chamada de [alert()](https://br.tradingview.com/pine-script-reference/v5/#fun_alert) ou no argumento `alert_message` de uma estratégia. No entanto, é possível incluir esse valor utilizando um placeholder. 

Este método apresenta duas alternativas:

```c
alertcondition(xUp, "Long Alert",  "Go long. RSI is {{plot_0}}")
alertcondition(xDn, "Short Alert", "Go short. RSI is {{plot("RSI")}}")
```

__Note que:__

- A primeira linha utiliza o placeholder `{{plot_0}}`, onde o número do plot corresponde à ordem do plot no script.
- A segunda linha utiliza o tipo de placeholder `{{plot("[plot_title]")}}`, que deve incluir o `title` da chamada [plot()](https://br.tradingview.com/pine-script-reference/v5/#fun_plot) usada no script para plotar o RSI. Aspas duplas são utilizadas para envolver o título do plot dentro do placeholder `{{plot("RSI")}}`. Isso exige que aspas simples sejam usadas para envolver a string de `message`.
- Utilizando um desses métodos, pode incluir qualquer valor numérico que seja plotado pelo indicador, mas como strings não podem ser plotadas, nenhuma variável de string pode ser utilizada.

## Usando Condições Compostas:

Ao oferecer aos usuários do script a possibilidade de criar um único alerta a partir de um indicador usando múltiplas chamadas de [alertcondition()](https://br.tradingview.com/pine-script-reference/v5/#fun_alertcondition), é necessário fornecer opções nos _inputs_ do script. Por meio dessas opções, os usuários indicarão as condições que desejam acionar o alerta antes de criá-lo.

Este script demonstra uma maneira de fazer isso:

```c
//@version=5
indicator("`alertcondition()` on multiple conditions")
detectLongsInput  = input.bool(true, "Detect Longs")
detectShortsInput = input.bool(true, "Detect Shorts")

r = ta.rsi(close, 20)
// Detect crosses.
xUp = ta.crossover( r, 50)
xDn = ta.crossunder(r, 50)
// Only generate entries when the trade's direction is allowed in inputs.
enterLong  = detectLongsInput  and xUp
enterShort = detectShortsInput and xDn

plot(r)
plotchar(enterLong,  "Go Long",  "▲", location.bottom, color.lime, size = size.tiny)
plotchar(enterShort, "Go Short", "▼", location.top,    color.red,  size = size.tiny)
hline(50)
// Trigger the alert when one of the conditions is met.
alertcondition(enterLong or enterShort, "Compound alert", "Entry")
```

Observe como a chamada de [alertcondition()](https://br.tradingview.com/pine-script-reference/v5/#fun_alertcondition) pode ser acionada por uma de duas condições. Cada condição só pode acionar o alerta se o usuário a habilitar nos _inputs_ do script antes de criar o alerta.

## Placeholders

Esses placeholders podem ser usados no argumento de `message` das chamadas de [alertcondition()](https://br.tradingview.com/pine-script-reference/v5/#fun_alertcondition). Eles serão substituídos por valores dinâmicos quando o alerta for acionado. Esses são os únicos meios de incluir valores dinâmicos (valores que podem variar de barra para barra) nas mensagens de [alertcondition()](https://br.tradingview.com/pine-script-reference/v5/#fun_alertcondition).

Note-se que os usuários que criam _alertas de alertcondition()_ a partir da caixa de diálogo "_Criar Alerta_" na interface de gráficos também podem usar esses placeholders no campo "_Mensagem_" da caixa de diálogo.

`{{exchange}}`

Bolsa do símbolo usado no alerta (NASDAQ, NYSE, MOEX, etc.). Observe que, para símbolos com cotação atrasada, a bolsa terminará com "_DL" ou "_DLY". Por exemplo, "NYMEX_DL".

`{{interval}}`

Retorna o _timeframe_ do gráfico no qual o alerta é criado. Note que gráficos de _Range_ são calculados com base em dados de 1 minuto, portanto, o placeholder sempre retornará "1" em qualquer alerta criado em um gráfico de _Range_.

`{{open}}`, `{{high}}`, `{{low}}`, `{{close}}`, `{{volume}}`

Valores correspondentes da barra na qual o alerta foi acionado.

`{{plot_0}}`, `{{plot_1}}`, __[...]__, `{{plot_19}}`

Valor do número correspondente do plot. Os plots são numerados de zero a 19 na ordem de aparecimento no script, portanto, apenas um dos primeiros 20 plots pode ser usado. Por exemplo, o indicador incorporado "Volume" possui duas séries de saída: Volume e Volume MA, então poderia usar o seguinte:

```c
alertcondition(volume > ta.sma(volume,20), "Volume alert", "Volume ({{plot_0}}) > average ({{plot_1}})")
```

`{{plot("[plot_title]")}}`

Este placeholder pode ser usado quando se precisa referir a um plot utilizando o argumento de `title` usado em uma chamada de [plot()](https://br.tradingview.com/pine-script-reference/v5/#fun_plot). Observe que aspas duplas (`"`) __devem__ ser usadas dentro do placeholder para _wrap_ o argumento de `title`. Isso exige que aspas simples (`'`) sejam usadas para envolver a string de `message`:

```c
//@version=5
indicator("")
r = ta.rsi(close, 14)
xUp = ta.crossover(r, 50)
plot(r, "RSI", display = display.none)
alertcondition(xUp, "xUp alert", message = 'RSI is bullish at: {{plot("RSI")}}')
```

`{{ticker}}`

Ticker do símbolo usado no alerta (AAPL, BTCUSD, etc.).

`{{time}}`

Retorna o horário no início da barra. O horário é UTC, formatado como `yyyy-MM-ddTHH:mm:ssZ`, por exemplo: `2019-08-27T09:56:00Z`.

`{{timenow}}`

Horário atual quando o alerta é acionado, formatado da mesma forma que `{{time}}`. A precisão é até o segundo mais próximo, independentemente do _timeframe_ do gráfico.


# Evitando Repintura com Alertas

Os casos mais comuns de repintura que os traders desejam evitar com alertas são aqueles em que é necessário prevenir o acionamento de um alerta em algum momento durante a barra em tempo real quando ele __não__ teria sido acionado no fechamento da barra. Isso pode ocorrer quando essas condições são atendidas:

- Os cálculos usados na condição que aciona o alerta podem variar durante a barra em tempo real. Isso ocorrerá com qualquer cálculo que utilize valores de `high`, `low` ou `close`, por exemplo, o que inclui quase todos os indicadores integrados. Também será o caso com o resultado de qualquer chamada a [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request{dot}security) usando um _timeframe_ maior do que o do gráfico, quando a barra atual desse intervalo maior ainda não tiver fechado.
- O alerta pode ser acionado antes do fechamento da barra em tempo real, portanto, com qualquer frequência diferente de "_Once Per Bar Close_" ("_Uma Vez por Fechamento da Barra_").

A maneira mais simples de evitar esse tipo de repintura é configurar a frequência de acionamento dos alertas para que eles só sejam acionados no fechamento da barra em tempo real. Não existe solução única; evitar esse tipo de repintura __sempre__ envolve esperar por informações confirmadas, o que significa que o trader deve sacrificar a imediatez para alcançar confiabilidade.

Note-se que outros tipos de repintura, como os documentados na seção de [Repintura](./05_16_repintura.md), podem não ser evitáveis simplesmente acionando alertas no fechamento das barras em tempo real.
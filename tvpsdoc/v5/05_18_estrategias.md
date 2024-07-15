
# Estratégias

As estratégias do Pine Script simulam a execução de negociações em dados históricos e em tempo real para facilitar o backtesting e o forward testing de sistemas de negociação. Elas incluem muitas das mesmas capacidades dos indicadores do Pine Script, além de fornecer a capacidade de colocar, modificar e cancelar ordens hipotéticas e analisar os resultados.

Quando um script usa a função [strategy()](https://br.tradingview.com/pine-script-reference/v5/#fun_strategy) para sua declaração, ele ganha acesso ao namespace `strategy.*`, onde pode chamar funções e variáveis para simular ordens e acessar informações essenciais da estratégia. Além disso, o script exibirá informações e resultados simulados externamente na aba "Strategy Tester".

## Um Exemplo Simples de Estratégia

O script a seguir é uma estratégia simples que simula a entrada de posições longas ou curtas com base no cruzamento de duas médias móveis:

```c
//@version=5
strategy("test", overlay = true)

// Calculate two moving averages with different lengths.
float fastMA = ta.sma(close, 14)
float slowMA = ta.sma(close, 28)

// Enter a long position when `fastMA` crosses over `slowMA`.
if ta.crossover(fastMA, slowMA)
    strategy.entry("buy", strategy.long)

// Enter a short position when `fastMA` crosses under `slowMA`.
if ta.crossunder(fastMA, slowMA)
    strategy.entry("sell", strategy.short)

// Plot the moving averages.
plot(fastMA, "Fast MA", color.aqua)
plot(slowMA, "Slow MA", color.orange)
```

__Note que:__

- A linha `strategy("test", overlay = true)` declara que o script é uma estratégia chamada "test" com saídas visuais sobrepostas no painel principal do gráfico.
- [strategy.entry()](https://br.tradingview.com/pine-script-reference/v5/#fun_strategy%7Bdot%7Dentry) é o comando que o script usa para simular ordens de "compra" e "venda". Quando o script coloca uma ordem, ele também plota o `id` da ordem no gráfico e uma seta para indicar a direção.
- Duas funções [plot()](https://br.tradingview.com/pine-script-reference/v5/#fun_plot) plotam as médias móveis com duas cores diferentes para referência visual.

## Aplicando uma Estratégia a um Gráfico

Para testar uma estratégia, aplique-a ao gráfico. Você pode usar uma estratégia integrada a partir da caixa de diálogo "Indicadores e Estratégias" ou escrever a sua própria no Pine Editor. Clique em "Adicionar ao gráfico" "_Add to chart_" na aba "Pine Editor" para aplicar um script ao gráfico:

![Aplicando uma estratégia a um gráfico 01](./imgs/Strategies-Applying-a-strategy-to-a-chart-1.sh7yATus_1iF3oX.webp)

Depois que um script de estratégia é compilado e aplicado a um gráfico, ele plotará marcas de ordem no painel principal do gráfico e exibirá resultados de desempenho simulados na aba "Teste de Estratégia" "_Strategy Tester_" abaixo:

![Aplicando uma estratégia a um gráfico 02](./imgs/Strategies-Applying-a-strategy-to-a-chart-2.DSJCJN56_Z2kfkPT.webp)

__Note que:__ Os resultados de uma estratégia aplicada a gráficos não padronizados ([Heikin Ashi](https://br.tradingview.com/support/solutions/43000619436), [Renko](https://br.tradingview.com/support/solutions/43000502284), [Line Break](https://br.tradingview.com/support/solutions/43000502273), [Kagi](https://br.tradingview.com/support/solutions/43000502272), [Point & Figure](https://br.tradingview.com/support/solutions/43000502276) e [Range](https://br.tradingview.com/support/solutions/43000474007)) não refletem as condições reais do mercado por padrão. Scripts de estratégia usarão os valores de preços sintéticos desses gráficos durante a simulação, que muitas vezes não se alinham com os preços reais do mercado e, portanto, produzirão resultados de backtest irreais. Recomenda-se fortemente o uso de tipos de gráficos padrão para backtesting de estratégias. Alternativamente, em gráficos Heikin Ashi, os usuários podem simular ordens usando preços reais habilitando a opção "Preencher ordens usando OHLC padrão" "_Fill orders using standard OHLC_" nas [Propriedades das estratégias](https://br.tradingview.com/support/solutions/43000628599).

## Testador de Estratégia

O módulo Strategy Tester está disponível para todos os scripts declarados com a função [strategy()](https://br.tradingview.com/pine-script-reference/v5/#fun_strategy). Os usuários podem acessar esse módulo na aba "Testador de Estratégia" "_Strategy Tester_" abaixo de seus gráficos, onde podem visualizar convenientemente suas estratégias e analisar os resultados hipotéticos de desempenho.

### Visão Geral

A aba [Overview](https://br.tradingview.com/support/solutions/43000681733-overview-tab) do Testador de Estratégia apresenta métricas de desempenho essenciais e curvas de patrimônio e drawdown ao longo de uma sequência simulada de negociações, proporcionando uma visão rápida do desempenho da estratégia sem mergulhar em detalhes granulares. O gráfico nesta seção mostra a [curva de patrimônio](https://br.tradingview.com/support/solutions/43000681735) da estratégia como um gráfico de linha, a [curva de patrimônio de compra e manutenção](https://br.tradingview.com/support/solutions/43000681736) como uma linha contínua e a [curva de drawdown](https://br.tradingview.com/support/solutions/43000681734) como um histograma. Os usuários podem alternar essas plotagens e escalá-las como valores absolutos ou percentuais usando as opções abaixo do gráfico.

![Visão geral](./imgs/Strategies-Strategy-tester-Overview-1.Dl0kXpLm_Z2wYThl.webp)

__Note que:__

- O gráfico de visão geral usa duas escalas; a da esquerda é para as curvas de patrimônio e a da direita é para a curva de drawdown.
- Quando um usuário clica em um ponto dessas plotagens, isso direcionará a visualização do gráfico principal para o ponto onde a negociação foi fechada.

### Resumo de Desempenho

A aba [Performance Summary](https://br.tradingview.com/support/solutions/43000681683) do módulo apresenta uma visão abrangente das métricas de desempenho de uma estratégia. Ela exibe três colunas: uma para todas as negociações, uma para todas as longas e outra para todas as curtas, proporcionando aos traders insights mais detalhados sobre o desempenho simulado de negociações longas, curtas e gerais de uma estratégia.

![Resumo de desempenho](./imgs/Strategies-Strategy-tester-Performance-summary-1.DcoSYJi9_Zew5cK.webp)

### Lista de Negociações

A aba [List of Trades](https://br.tradingview.com/support/solutions/43000681737) fornece uma visão granular das negociações simuladas por uma estratégia com informações essenciais, incluindo a data e hora de execução, o tipo de ordem usada (entrada ou saída), o número de contratos/ações/lotes/unidades negociados e o preço, bem como algumas métricas de desempenho da negociação.

![Lista de negociações](./imgs/Strategies-Strategy-tester-List-of-trades-1.CqWLl3Uc_Tg2ti.webp)

__Note que:__

- Os usuários podem navegar pelos horários de negociações específicas em seus gráficos clicando nelas nesta lista.
- Clicando no campo "Trade #" acima da lista, os usuários podem organizar as negociações em ordem crescente a partir da primeira ou em ordem decrescente a partir da última.

### Propriedades

A aba "Propriedades" "_Properties_" fornece informações detalhadas sobre a configuração de uma estratégia e o conjunto de dados ao qual ela é aplicada. Inclui o intervalo de datas da estratégia, informações do símbolo, configurações do script e propriedades da estratégia.

- __Intervalo de Datas__ - Inclui o intervalo de datas com negociações simuladas e o intervalo total disponível para backtesting.
- __Informações do Símbolo__ - Contém o nome do símbolo e corretora/bolsa, o timeframe e tipo do gráfico, o tamanho do tick, o valor do ponto para o gráfico e a moeda base.
- __Entradas da Estratégia__ - Descreve os vários parâmetros e variáveis usadas no script da estratégia disponíveis na aba "Entradas" "_Inputs_" das configurações do script.
- __Propriedades da Estratégia__ - Fornece uma visão geral da configuração da estratégia de negociação. Inclui detalhes essenciais como o capital inicial, moeda base, tamanho da ordem, margem, pirâmide, comissão e deslizamento. Além disso, esta seção destaca quaisquer modificações feitas no comportamento de cálculo da estratégia.

![Propriedades](./imgs/Strategies-Strategy-tester-Properties-1.BJRt5efJ_DxF2g.webp)

## Emulador do Broker

TradingView utiliza um "_broker emulator_" para simular o desempenho das estratégias de negociação. Diferente da negociação na vida real, o emulador usa estritamente os preços disponíveis no gráfico para a simulação de ordens. Consequentemente, a simulação só pode realizar negociações históricas após o fechamento de uma barra e só pode realizar negociações em tempo real em um novo tick de preço. Para mais informações sobre esse comportamento, consulte o [Modelo de Execução](./04_01_modelo_de_execucao.md) do Pine Script.

Como o emulador só pode usar dados do gráfico, ele faz suposições sobre o movimento dos preços intrabar. Ele usa os preços de abertura, máxima e mínima de uma barra para inferir a atividade intrabar enquanto calcula preenchimentos de ordens com a seguinte lógica:

- Se o preço máximo estiver mais próximo do preço de abertura do que o preço mínimo, assume-se que o preço se moveu nesta ordem na barra: abertura → máxima → mínima → fechamento.
- Se o preço mínimo estiver mais próximo do preço de abertura do que o preço máximo, assume-se que o preço se moveu nesta ordem na barra: abertura → mínima → máxima → fechamento.
- O emulador do broker assume que não existem lacunas entre os preços dentro das barras, significando que toda a gama de preços intrabar está disponível para execução de ordens. No entanto, as lacunas entre o fechamento da barra anterior e a abertura da barra atual são levadas em consideração ao preencher ordens de preço fixo (todas as ordens, exceto ordens de mercado). Se o preço da ordem for cruzado durante a lacuna entre duas barras, a ordem é executada na abertura da próxima barra e não no preço especificado.

![Emulador do broker](./imgs/Strategies-Broker-emulator-1.TOD5mNOt_1q8drI.webp)

### Ampliador da Barra

Os assinantes da conta Premium podem substituir as suposições intrabar do emulador do broker através do parâmetro `use_bar_magnifier` da função [strategy()](https://br.tradingview.com/pine-script-reference/v5/#fun_strategy) ou da entrada "Usar amplificador de barras" "_Use bar magnifier_" na aba "Propriedades" "_Properties_" das configurações do script. A [Lupa de Barras/Ampliador](https://br.tradingview.com/support/solutions/43000669285) inspeciona dados em timeframes menores do que os do gráfico para obter informações mais granulares sobre a ação do preço dentro de uma barra, permitindo preenchimentos de ordens mais precisos durante a simulação.

Para demonstrar, o script a seguir coloca uma ordem limite de "Compra" "_Buy_" no `entryPrice` e uma ordem limite de "Saída" "_Exit_" no `exitPrice` quando o valor do [tempo](https://br.tradingview.com/pine-script-reference/v5/#var_time) cruza o `orderTime`, e desenha duas linhas horizontais para visualizar os preços das ordens. O script também destaca o fundo usando o `orderColor` para indicar quando a estratégia colocou as ordens:

![Ampliador da barra 01](./imgs/Strategies-Broker-emulator-Bar-magnifier-1.coZBXeS5_Z73KL6.webp)

```c
//@version=5
strategy("Bar Magnifier Demo", overlay = true, use_bar_magnifier = false)

//@variable The UNIX timestamp to place the order at.
int orderTime = timestamp("UTC", 2023, 3, 22, 18)

//@variable Returns `color.orange` when `time` crosses the `orderTime`, false otherwise.
color orderColor = na

// Entry and exit prices.
float entryPrice = hl2 - (high - low)
float exitPrice  = entryPrice + (high - low) * 0.25

// Entry and exit lines.
var line entryLine = na
var line exitLine  = na

if ta.cross(time, orderTime)
    // Draw new entry and exit lines.
    entryLine := line.new(bar_index, entryPrice, bar_index + 1, entryPrice, color = color.green, width = 2)
    exitLine  := line.new(bar_index, exitPrice, bar_index + 1, exitPrice, color = color.red, width = 2)

    // Update order highlight color.
    orderColor := color.new(color.orange, 80)

    // Place limit orders at the `entryPrice` and `exitPrice`.
    strategy.entry("Buy", strategy.long, limit = entryPrice)
    strategy.exit("Exit", "Buy", limit = exitPrice)

// Update lines while the position is open.
else if strategy.position_size > 0.0
    entryLine.set_x2(bar_index + 1)
    exitLine.set_x2(bar_index + 1)

bgcolor(orderColor)
```

Como visto no gráfico acima, o emulador do broker assumiu que os preços intrabar se moveram de abertura para máxima, depois máxima para mínima, depois mínima para fechamento na barra em que a ordem "Buy" foi preenchida, significando que o emulador assumiu que a ordem "Exit" não poderia ser preenchida na mesma barra. No entanto, após incluir `use_bar_magnifier = true` na declaração, se vê uma história diferente:

![Ampliador da barra 02](./imgs/Strategies-Broker-emulator-Bar-magnifier-2.BNRDP6RA_1rU9cu.webp)

> __Observação:__\
> O número máximo de intrabarras que um script pode solicitar é 200.000. Alguns símbolos com histórico mais longo podem não ter cobertura completa de intrabarras para suas barras iniciais do gráfico com essa limitação, significando que negociações simuladas nessas barras não serão afetadas pelo amplificador de barras.

## Ordens e Entradas

Assim como na negociação real, as estratégias do Pine usam ordens para gerenciar posições. Nesse contexto, uma _ordem_ é um comando para simular uma ação de mercado, e uma _negociação_ é o resultado após a ordem ser preenchida. Portanto, para entrar ou sair de posições usando Pine, os usuários devem criar ordens com parâmetros que especificam como elas se comportarão.

Para examinar mais de perto como as ordens funcionam e como elas se tornam negociações, aqui um script de estratégia simples:

```c
//@version=5
strategy("My strategy", overlay = true, margin_long = 100, margin_short = 100)

//@function Displays text passed to `txt` when called. 
debugLabel(txt) => 
    label.new(
         bar_index, high, text = txt, color=color.lime, style = label.style_label_lower_right, 
         textcolor = color.black, size = size.large
     )

longCondition = bar_index % 20 == 0 // true on every 20th bar
if (longCondition)
    debugLabel("Long entry order created")
    strategy.entry("My Long Entry Id", strategy.long)
strategy.close_all()
```

Neste script, é definido uma `longCondition` que é verdadeira sempre que o `bar_index` é divisível por 20, ou seja, a cada 20ª barra. A estratégia usa essa condição dentro de uma estrutura [if](https://br.tradingview.com/pine-script-reference/v5/#kw_if) para simular uma ordem de entrada com [strategy.entry()](https://br.tradingview.com/pine-script-reference/v5/#fun_strategy%7Bdot%7Dentry) e desenha um _label_ no preço de entrada com a função `debugLabel()` definida pelo usuário. O script chama [strategy.close_all()](https://br.tradingview.com/pine-script-reference/v5/#fun_strategy%7Bdot%7Dclose_all) do escopo global para simular uma ordem de mercado que fecha qualquer posição aberta. Veja o que acontece quando é adicionado o script ao gráfico:

![Ordens e entradas](./imgs/Strategies-Orders-and-entries-1.CX7qCrxg_Z2raUd5.webp)

As setas azuis no gráfico indicam os locais de entrada, e as roxas marcam os pontos onde a estratégia fechou posições. Observe que os _labels_ precedem o ponto de entrada real, em vez de ocorrer na mesma barra - isso são as ordens em ação. Por padrão, as estratégias do Pine aguardam o próximo tick de preço disponível antes de preencher ordens, pois preencher uma ordem no mesmo tick não é realista. Além disso, elas recalculam no fechamento de cada barra histórica, significando que o próximo tick disponível para preencher uma ordem é a abertura da próxima barra neste caso. Como resultado, por padrão, todas as ordens são atrasadas por uma barra do gráfico.

É importante notar que, embora o script chame [strategy.close_all()](https://br.tradingview.com/pine-script-reference/v5/#fun_strategy%7Bdot%7Dclose_all) do escopo global, forçando a execução em cada barra, a chamada da função não faz nada se a estratégia não estiver simulando uma posição aberta. Se houver uma posição aberta, o comando emite uma ordem de mercado para fechá-la, que é executada no próximo tick disponível. Por exemplo, quando a `longCondition` é verdadeira na barra 20, a estratégia coloca uma ordem de entrada para preencher no próximo tick, que é na abertura da barra 21. Uma vez que o script recalcula seus valores no fechamento dessa barra, a função coloca uma ordem para fechar a posição, que é preenchida na abertura da barra 22.

### Tipos de Ordens

As estratégias do Pine Script permitem aos usuários simular diferentes tipos de ordens para suas necessidades específicas. Os principais tipos notáveis são _market/mercado_, _limit/limite_, _stop_ e _stop-limit_.

#### Ordens de Market/Mercado 

Ordens de mercado são o tipo mais básico de ordens. Elas comandam uma estratégia para comprar ou vender um ativo o mais rápido possível, independentemente do preço. Consequentemente, elas sempre executam no próximo tick de preço disponível. Por padrão, todas as funções `strategy.*()` que geram ordens produzem especificamente ordens de mercado.

O script a seguir simula uma ordem de mercado longa quando o `bar_index` é divisível por `2 * cycleLength`. Caso contrário, simula uma ordem de mercado curta quando o `bar_index` é divisível por `cycleLength`, resultando em uma estratégia com negociações longas e curtas alternadas a cada `cycleLength` barras:

![Ordens de market/mercado](./imgs/Strategies-Orders-and-entries-Order-types-1.XLQDthDF_Z1YB5yU.webp)

```c
//@version=5
strategy("Market order demo", overlay = true, margin_long = 100, margin_short = 100)

//@variable Number of bars between long and short entries.
cycleLength = input.int(10, "Cycle length")

//@function Displays text passed to `txt` when called.
debugLabel(txt, lblColor) => label.new(
     bar_index, high, text = txt, color = lblColor, textcolor = color.white, 
     style = label.style_label_lower_right, size = size.large
 )

//@variable Returns `true` every `2 * cycleLength` bars.
longCondition = bar_index % (2 * cycleLength) == 0
//@variable Returns `true` every `cycleLength` bars.
shortCondition = bar_index % cycleLength == 0

// Generate a long market order with a `color.green` label on `longCondition`.
if longCondition
    debugLabel("Long market order created", color.green)
    strategy.entry("My Long Entry Id", strategy.long)
// Otherwise, generate a short market order with a `color.red` label on `shortCondition`.
else if shortCondition
    debugLabel("Short market order created", color.red)
    strategy.entry("My Short Entry Id", strategy.short)
```

#### Ordens de Limit/Limite

Ordens de limite comandam uma estratégia para entrar em uma posição a um preço específico ou melhor (inferior ao especificado para ordens longas e superior para ordens curtas). Quando o preço de mercado atual é melhor do que o parâmetro `limit` da ordem, a ordem será preenchida sem esperar que o preço de mercado atinja o nível limite.

Para simular ordens de limite em um script, passe um valor de preço para um comando de colocação de ordem com um parâmetro `limit`. O exemplo a seguir coloca uma ordem de limite 800 ticks abaixo do fechamento da barra 100 barras antes do `last_bar_index`:

![Ordens de limit/limite 01](./imgs/Strategies-Orders-and-entries-Order-types-2.Ma_eheTS_1mD31p.webp)

```c
//@version=5
strategy("Limit order demo", overlay = true, margin_long = 100, margin_short = 100)

//@function Displays text passed to `txt` and a horizontal line at `price` when called.
debugLabel(price, txt) =>
    label.new(
         bar_index, price, text = txt, color = color.teal, textcolor = color.white, 
         style = label.style_label_lower_right, size = size.large
     )
    line.new(
         bar_index, price, bar_index + 1, price, color = color.teal, extend = extend.right, 
         style = line.style_dashed
     )

// Generate a long limit order with a label and line 100 bars before the `last_bar_index`.
if last_bar_index - bar_index == 100
    limitPrice = close - syminfo.mintick * 800
    debugLabel(limitPrice, "Long Limit order created")
    strategy.entry("Long", strategy.long, limit = limitPrice)
```

Observe como o script colocou o _label_ e começou a linha várias barras antes da negociação. Enquanto o preço permanecesse acima do valor `limitPrice`, a ordem não poderia ser preenchida. Uma vez que o preço de mercado atingiu o limite, a estratégia executou a negociação no meio da barra. Se tivesse definido o `limitPrice` para 800 ticks _acima_ do fechamento da barra em vez de _abaixo_, a ordem seria preenchida imediatamente, pois o preço já estaria em um valor melhor:

![Ordens de limit/limite 02](./imgs/Strategies-Orders-and-entries-Order-types-3.D5qNWPW8_Ad7IE.webp)

```c
//@version=5
strategy("Limit order demo", overlay = true, margin_long = 100, margin_short = 100)

//@function Displays text passed to `txt` and a horizontal line at `price` when called.
debugLabel(price, txt) =>
    label.new(
        bar_index, price, text = txt, color = color.teal, textcolor = color.white, 
        style = label.style_label_lower_right, size = size.large
    )
    line.new(
        bar_index, price, bar_index + 1, price, color = color.teal, extend = extend.right, 
        style = line.style_dashed
    )

// Generate a long limit order with a label and line 100 bars before the `last_bar_index`.
if last_bar_index - bar_index == 100
    limitPrice = close + syminfo.mintick * 800
    debugLabel(limitPrice, "Long Limit order created")
    strategy.entry("Long", strategy.long, limit = limitPrice)
```

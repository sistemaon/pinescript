
# Outros Timeframes e Dados

Pine Script permite solicitar dados de fontes e contextos diferentes daqueles usados em seus gráficos. As funções apresentadas nesta página podem buscar dados de várias fontes alternativas:

- [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request{dot}security) recupera dados de outro símbolo, timeframe ou outro contexto.
- [request.security_lower_tf()](https://br.tradingview.com/pine-script-reference/v5/#fun_request{dot}security_lower_tf) recupera dados _intrabar_, ou seja, dados de um timeframe inferior ao do gráfico.
- [request.currency_rate()](https://br.tradingview.com/pine-script-reference/v5/#fun_request{dot}currency_rate) solicita uma _taxa diária_ para converter um valor expressado em uma moeda para outra.
- [request.dividends(), request.splits(), e request.earnings()](https://br.tradingview.com/pine-script-reference/v5/#fun_request{dot}dividends-request{dot}splits-request{dot}earnings) recuperam, respectivamente, informações sobre dividendos, desdobramentos e lucros de uma empresa emissora.
- [request.quandl()](https://br.tradingview.com/pine-script-reference/v5/#fun_request{dot}quandl) recupera informações do [NASDAQ Data Link](https://data.nasdaq.com).
- [request.financial()](https://br.tradingview.com/pine-script-reference/v5/#fun_request{dot}financial) recupera dados financeiros do [FactSet](https://www.factset.com/).
- [request.economic()](https://br.tradingview.com/pine-script-reference/v5/#fun_request{dot}economic) recupera dados econômicos e industriais.
- [request.seed()](https://br.tradingview.com/pine-script-reference/v5/#fun_request{dot}seed) recupera dados de um repositório GitHub _mantido pelo usuário_.

> __Observação!__\
> Ao longo desta página, e em outras partes da documentação que discutem as funções `request.*()`, é comum o uso do termo "_context_" para descrever o ID do ticker, timeframe e quaisquer modificações (ajustes de preço, configurações de sessão, tipos de gráficos não padronizados, etc.) que se aplicam a um gráfico ou aos dados recuperados por um script.

Estas são as assinaturas das funções no namespace `request.*`:

```c
request.security(symbol, timeframe, expression, gaps, lookahead, ignore_invalid_symbol, currency, calc_bars_count) → series <type>

request.security_lower_tf(symbol, timeframe, expression, ignore_invalid_symbol, currency, ignore_invalid_timeframe, calc_bars_count) → array<type>

request.currency_rate(from, to, ignore_invalid_currency) → series float

request.dividends(ticker, field, gaps, lookahead, ignore_invalid_symbol, currency) → series float

request.splits(ticker, field, gaps, lookahead, ignore_invalid_symbol) → series float

request.earnings(ticker, field, gaps, lookahead, ignore_invalid_symbol, currency) → series float

request.quandl(ticker, gaps, index, ignore_invalid_symbol) → series float

request.financial(symbol, financial_id, period, gaps, ignore_invalid_symbol, currency) → series float

request.economic(country_code, field, gaps, ignore_invalid_symbol) → series float

request.seed(source, symbol, expression, ignore_invalid_symbol, calc_bars_count) → series <type>
```

A família de funções `request.*()` possui diversas aplicações potenciais. Ao longo desta página, serão discutidas em detalhes essas funções e alguns de seus casos de uso típicos.

> __Observação!__\
> Também é possível permitir que scripts compatíveis avaliem seus escopos em outros contextos sem exigir funções `request.*()` usando o parâmetro `timeframe` da declaração [indicator()](https://br.tradingview.com/pine-script-reference/v5/#fun_indicator).


# Características Comuns

Muitas funções no namespace `request.*()` compartilham algumas propriedades e parâmetros comuns. Antes de explorar cada função em profundidade, é importante familiarizar-se com essas características.

## Uso

Todas as funções `request.*()` retornam resultados "series", o que significa que podem produzir valores diferentes em cada barra. No entanto, a maioria dos parâmetros das funções `request.*()` requer argumentos "const", "input" ou "simple".

Em essência, Pine Script deve determinar os valores da maioria dos argumentos passados para uma função `request.*()` na compilação do script ou na primeira barra do gráfico, dependendo do [tipo qualificado](./04_09_tipagem_do_sistema.md#qualificadores) que cada parâmetro aceita, e esses valores não podem mudar durante a execução do script. A única exceção é o parâmetro `expression` em [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security), [request.security_lower_tf()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security_lower_tf), e [request.seed()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.seed), que aceita argumentos "series".

Chamadas para funções `request.*()` são executadas em cada barra do gráfico, e scripts não podem desativá-las seletivamente durante sua execução. Scripts não podem chamar funções `request.*()` dentro dos escopos locais de [estruturas condicionais](./04_07_estruturas_condicionais.md), ou funções e métodos exportados por [bibliotecas](./05_11_libraries.md), mas podem usar essas chamadas de função dentro dos corpos de [funções definidas pelo usuário](./04_11_funcoes_definida_pelo_usuario.md) e [métodos](./04_13_metodos.md#métodos-definidos-pelo-usuário) não exportados.

Ao usar qualquer função `request.*()` dentro de um script, a performance de execução é uma consideração importante. Essas funções podem ter um impacto significativo na performance do script. Enquanto scripts podem conter um máximo de 40 chamadas para o namespace `request.*()`, é recomendável minimizar o número de chamadas nos scripts para manter o consumo de recursos o mais baixo possível. Para mais informações sobre as limitações dessas funções, veja [esta seção](./06_05_limitacoes.md#chamadas-request) da página do Manual do Usuário sobre as [limitações](./06_05_limitacoes.md) do Pine.

## `gaps`

Ao usar uma função `request.*()` para recuperar dados de outro contexto, os dados podem não ser recebidos em cada nova barra como seriam no gráfico atual. O parâmetro `gaps` de uma função `request.*()` permite controlar como a função responde a valores inexistentes na série solicitada.

> __Observação!__\
> Ao usar a função [indicator()](https://br.tradingview.com/pine-script-reference/v5/#fun_indicator) para avaliar um script em outro contexto, o parâmetro `timeframe_gaps` especifica como ele lida com valores inexistentes. O parâmetro é similar ao parâmetro `gaps` para funções `request.*()`.

Suponha que há um script que solicita dados por hora para o símbolo do gráfico com [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security) executando em um gráfico de 1 minuto. Nesse caso, a chamada da função retornará novos valores apenas nas barras de 1 minuto que cobrem os horários de abertura/fechamento das barras horárias do símbolo. Nas outras barras do gráfico, pode-se decidir se a função retornará valores [na](https://br.tradingview.com/pine-script-reference/v5/#var_na) ou os últimos valores disponíveis via o parâmetro `gaps`.

Quando o parâmetro `gaps` usa [barmerge.gaps_on](https://br.tradingview.com/pine-script-reference/v5/#var_barmerge.gaps_on), a função retornará resultados [na](https://br.tradingview.com/pine-script-reference/v5/#var_na) em todas as barras do gráfico onde novos dados ainda não foram confirmados do contexto solicitado. Caso contrário, quando o parâmetro usa [barmerge.gaps_off](https://br.tradingview.com/pine-script-reference/v5/#var_barmerge.gaps_off), a função preencherá as lacunas nos dados solicitados com os últimos valores confirmados em barras históricas e os valores em desenvolvimento mais recentes em barras em tempo real.

O script abaixo demonstra a diferença no comportamento ao [plotar](./05_15_plots.md) os resultados de duas chamadas para [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security) que buscam o preço de [fechamento](https://br.tradingview.com/pine-script-reference/v5/#var_close) do símbolo atual no timeframe por hora em um gráfico de 1 minuto. A primeira chamada usa `gaps = barmerge.gaps_off` e a segunda usa `gaps = barmerge.gaps_on`:

![gaps](./imgs/Other-timeframes-and-data-Common-characteristics-Gaps-1.DX6PixJ0_ZcmWMl.webp)

```c
//@version=5
indicator("gaps demo", overlay = true)

//@variable The `close` requested from the hourly timeframe without gaps.
float dataWithoutGaps = request.security(syminfo.tickerid, "60", close, gaps = barmerge.gaps_off)
//@variable The `close` requested from the hourly timeframe with gaps.
float dataWithGaps = request.security(syminfo.tickerid, "60", close, gaps = barmerge.gaps_on)

// Plot the requested data.
plot(dataWithoutGaps, "Data without gaps", color.blue, 3, plot.style_linebr)
plot(dataWithGaps, "Data with gaps", color.purple, 15, plot.style_linebr)

// Highlight the background for realtime bars.
bgcolor(barstate.isrealtime ? color.new(color.aqua, 70) : na, title = "Realtime bar highlight")
```

__Note que:__

- [barmerge.gaps_off](https://br.tradingview.com/pine-script-reference/v5/#var_barmerge.gaps_off) é o valor padrão para o parâmetro `gaps` em todas as funções `request.*()` aplicáveis.
- O script plota a série solicitada como linhas com quebras ([plot.style_linebr](https://br.tradingview.com/pine-script-reference/v5/#var_plot.style_linebr)), que não se conectam sobre valores [na](https://br.tradingview.com/pine-script-reference/v5/#var_na) como o estilo padrão ([plot.style_line](https://br.tradingview.com/pine-script-reference/v5/#var_plot.style_line)) faz.
- Ao usar [barmerge.gaps_off](https://br.tradingview.com/pine-script-reference/v5/#var_barmerge.gaps_off), a função [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security) retorna o último fechamento confirmado ([fechamento](https://br.tradingview.com/pine-script-reference/v5/#var_close)) do timeframe horário em todas as barras históricas. Ao rodar em barras em tempo real (as barras com o fundo [color.aqua](https://br.tradingview.com/pine-script-reference/v5/#var_color.aqua) neste exemplo), ela retorna o valor de fechamento atual do símbolo, independentemente da confirmação. Para mais informações, veja a seção [Comportamento histórico e em tempo real](./05_14_outros_timeframes_e_dados.md#comportamento-histórico-e-tempo-real) desta página.

## `ignore_invalid_symbol`

O parâmetro `ignore_invalid_symbol` das funções `request.*()` determina como uma função lidará com solicitações de dados inválidos, por exemplo:

- Usando uma função `request.*()` com um ID de ticker inexistente como o parâmetro `symbol/ticker`.
- Usando [request.financial()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.financial) para recuperar informações que não existem para o `symbol` ou `period` especificado.
- Usando [request.economic()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.economic) para solicitar um `field` que não existe para um `country_code`.

Uma chamada de função `request.*()` produzirá um _erro de tempo de execução_ e interromperá a execução do script ao fazer uma solicitação errônea se o parâmetro `ignore_invalid_symbol` for `false`. Quando o valor deste parâmetro é `true`, a função retornará valores [na](https://br.tradingview.com/pine-script-reference/v5/#var_na) nesse caso, em vez de gerar um erro.

Este exemplo usa chamadas `request.*()` dentro de uma [função definida pelo usuário](./04_11_funcoes_definida_pelo_usuario.md) para recuperar dados para estimar a capitalização de mercado (market cap) de um instrumento. A função definida pelo usuário `calcMarketCap()` chama [request.financial()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.financial) para recuperar o total de ações em circulação para um símbolo e [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security) para recuperar uma tupla contendo o preço de [fechamento](https://br.tradingview.com/pine-script-reference/v5/#var_close) do símbolo e o [currency](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo.currency) (_moeda_). Foi incluído `ignore_invalid_symbol = true` em ambas as chamadas `request.*()` para evitar erros de tempo de execução para solicitações inválidas.

O script exibe uma [string formatada](https://br.tradingview.com/pine-script-reference/v5/#fun_str.format) representando o valor estimado da capitalização de mercado do símbolo e a moeda em uma [tabela](https://br.tradingview.com/pine-script-reference/v5/#type_table) no gráfico e usa um [plot](https://br.tradingview.com/pine-script-reference/v5/#fun_plot) para visualizar o histórico de `marketCap`:

![ignore_invalid_symbol](./imgs/Other-timeframes-and-data-Common-characteristics-Ignore-invalid-symbol-1.DPSV2CB9_2wVvY7.webp)

```c
//@version=5
indicator("ignore_invalid_symbol demo", "Market cap estimate", format = format.volume)

//@variable The symbol to request data from.
string symbol = input.symbol("TSX:SHOP", "Symbol")

//@function Estimates the market capitalization of the specified `tickerID` if the data exists.
calcMarketCap(simple string tickerID) =>
    //@variable The quarterly total shares outstanding for the `tickerID`. Returns `na` when the data isn't available.
    float tso = request.financial(tickerID, "TOTAL_SHARES_OUTSTANDING", "FQ", ignore_invalid_symbol = true)
    //@variable The `close` price and currency for the `tickerID`. Returns `[na, na]` when the `tickerID` is invalid.
    [price, currency] = request.security(
         tickerID, timeframe.period, [close, syminfo.currency], ignore_invalid_symbol = true
     )
    // Return a tuple containing the market cap estimate and the quote currency.
    [tso * price, currency]

//@variable A `table` object with a single cell that displays the `marketCap` and `quoteCurrency`.
var table infoTable = table.new(position.top_right, 1, 1)
// Initialize the table's cell on the first bar.
if barstate.isfirst
    table.cell(infoTable, 0, 0, "", text_color = color.white, text_size = size.huge, bgcolor = color.teal)

// Get the market cap estimate and quote currency for the `symbol`.
[marketCap, quoteCurrency] = calcMarketCap(symbol)

//@variable The formatted text displayed inside the `infoTable`.
string tableText = str.format("Market cap:\n{0} {1}", str.tostring(marketCap, format.volume), quoteCurrency)
// Update the `infoTable`.
table.cell_set_text(infoTable, 0, 0, tableText)

// Plot the `marketCap` value.
plot(marketCap, "Market cap", color.new(color.purple, 60), style = plot.style_area)
```

__Note que:__

- A função `calcMarketCap()` retornará valores apenas para instrumentos válidos com dados de ações em circulação, como o selecionado para este exemplo. Ela retornará um valor de capitalização de mercado de [na](https://br.tradingview.com/pine-script-reference/v5/#var_na) em outros que não possuem dados financeiros, incluindo forex, cripto e derivativos.
- Nem todas as empresas emissoras publicam relatórios financeiros trimestrais. Se a empresa emissora do `symbol` não relatar trimestralmente, altere o valor "FQ" neste script para o timeframe mínimo de relatório da empresa. Veja a seção [request.financial()](https://br.tradingview.com/pine-script-reference/v5/#fun_request{dot}financial) para mais informações.
- Foi utilizado [format.volume](https://br.tradingview.com/pine-script-reference/v5/#var_format.volume) nas chamadas [indicator()](https://br.tradingview.com/pine-script-reference/v5/#fun_indicator) e [str.tostring()](https://br.tradingview.com/pine-script-reference/v5/#fun_str.tostring), que especificam que o eixo y do painel do gráfico representa valores formatados como volume e a representação "string" do valor de `marketCap` aparece como texto formatado como volume.
- Este script cria uma [tabela](https://br.tradingview.com/pine-script-reference/v5/#type_table) e inicializa sua célula na [primeira barra do gráfico](https://br.tradingview.com/pine-script-reference/v5/#var_barstate.isfirst), depois [atualiza o texto da célula](https://br.tradingview.com/pine-script-reference/v5/#fun_table.cell_set_text) nas barras subsequentes. Para saber mais sobre como trabalhar com tabelas, veja a página [Tabelas](./05_19_tables.md) do Manual do Usuário.

## `currency`

O parâmetro `currency` de uma função `request.*()` permite especificar a moeda dos dados solicitados. Quando o valor deste parâmetro difere do [syminfo.currency](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo.currency) do contexto solicitado, a função converterá os valores solicitados para expressá-los na moeda especificada. Este parâmetro pode aceitar uma variável embutida do namespace `currency.*`, como [currency.JPY](https://br.tradingview.com/pine-script-reference/v5/#var_currency.JPY), ou uma "string" representando o [código de moeda ISO 4217](https://pt.wikipedia.org/wiki/ISO_4217) (por exemplo, "JPY").

A taxa de conversão entre o [syminfo.currency](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo.currency) dos dados solicitados e a `currency` especificada depende da correspondente taxa diária "*FX_IDC*" do dia anterior. Se nenhum instrumento disponível fornecer a taxa de conversão diretamente, a função usará o valor de um [símbolo spread](https://br.tradingview.com/support/solutions/43000502298) para derivar a taxa.

> __Observação!__\
> Nem todas as chamadas de função `request.*()` retornam valores expressos como uma quantia em moeda. Portanto, a conversão de moeda não é sempre necessária. Por exemplo, algumas séries retornadas por [request.financial()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.financial) são expressas em unidades diferentes de moeda, como as métricas "PIOTROSKI_F_SCORE" e "NUMBER_OF_EMPLOYEES". Cabe aos programadores determinar quando a conversão de moeda é apropriada em suas solicitações de dados.

## `lookahead`

O parâmetro `lookahead` em [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security), [request.dividends()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.dividends), [request.splits()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.splits), e [request.earnings()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.earnings) especifica o comportamento do lookahead da chamada da função. Seu valor padrão é [barmerge.lookahead_off](https://br.tradingview.com/pine-script-reference/v5/#var_barmerge.lookahead_off).

Ao solicitar dados de um contexto do timeframe superior _higher-timeframe (HTF)_, o valor `lookahead` determina se a função pode solicitar valores de tempos _além_ daqueles das barras históricas em que é executada. Em outras palavras, o valor `lookahead` determina se os dados solicitados podem conter _viés de lookahead_ em barras históricas.

Ao solicitar dados de um contexto de timeframe inferior _lower-timeframe (LTF)_, o parâmetro `lookahead` determina se a função solicita valores da primeira ou última barra _intrabar_ (LTF) em cada barra do gráfico.

__Os programadores devem ter extrema cautela ao usar lookahead em seus scripts, especialmente ao solicitar dados de timeframes superiores.__ Ao usar [barmerge.lookahead_on](https://br.tradingview.com/pine-script-reference/v5/#var_barmerge.lookahead_on) como valor `lookahead`, é necessário garantir que isso não comprometa a integridade da lógica do script, vazando dados _futuros_ em barras históricas do gráfico.

Os seguintes cenários são casos em que habilitar lookahead é aceitável em uma chamada `request.*()`:

- A `expression` em [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security) referencia uma série com um "_deslocamento histórico_" ("_historical offset_") (por exemplo, `close[1]`), o que impede a função de solicitar valores futuros que não teria acesso em tempo real.
- O `timeframe` especificado na chamada é o mesmo do gráfico em que o script é executado, ou seja, [timeframe.period](https://br.tradingview.com/pine-script-reference/v5/#var_timeframe.period).
- A chamada da função solicita dados de um timeframe intrabar, ou seja, um timeframe menor que o [timeframe.period](https://br.tradingview.com/pine-script-reference/v5/#var_timeframe.period). Veja [esta seção](./05_14_outros_timeframes_e_dados.md#dados-lower-timeframe-ltf-timeframe-inferior) para mais informações.

> __Observação!__\
> Usar [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security) para vazar dados futuros para o passado é __enganoso__ e __não permitido__ em publicações de scripts. Embora os resultados do seu script em barras históricas possam parecer ótimos devido à sua aquisição aparentemente "mágico" de presciência (que não será capaz de reproduzir em barras em tempo real), isso enganará você e os usuários do seu script. Se [publicar seu script](./06_04_publicando_scripts.md) para compartilhá-lo com outros, certifique-se de __não enganar os usuários__ acessando informações futuras em barras históricas.

Este exemplo demonstra como o parâmetro `lookahead` afeta o comportamento das solicitações de dados de timeframes superiores e por que habilitar lookahead em [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security) sem deslocar a `expression` é enganoso. O script chama [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security) para obter o preço [máximo](https://br.tradingview.com/pine-script-reference/v5/#var_high) (HTF) do símbolo do gráfico atual de três maneiras diferentes e [plota](./05_15_plots.md) as séries resultantes no gráfico para comparação.

A primeira chamada usa [barmerge.lookahead_off](https://br.tradingview.com/pine-script-reference/v5/#var_barmerge.lookahead_off) (padrão), e as outras usam [barmerge.lookahead_on](https://br.tradingview.com/pine-script-reference/v5/#var_barmerge.lookahead_on). No entanto, a terceira chamada de [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security) também _desloca_ sua `expression` usando o operador de referência histórica [[]](https://br.tradingview.com/pine-script-reference/v5/#op_%5B%5D) para evitar vazamento de dados futuros no passado.

Como visto no gráfico, o [plot](https://br.tradingview.com/pine-script-reference/v5/#fun_plot) da série solicitada usando [barmerge.lookahead_on](https://br.tradingview.com/pine-script-reference/v5/#var_barmerge.lookahead_on) sem deslocamento ([fuchsia](https://br.tradingview.com/pine-script-reference/v5/#var_color.fuchsia)) mostra os preços [máximos](https://br.tradingview.com/pine-script-reference/v5/#var_high) (HTF) finais _antes_ de estarem realmente disponíveis em barras históricas, enquanto as outras duas chamadas não:

![lookahead](./imgs/Other-timeframes-and-data-Common-characteristics-Lookahead-1.DhbZxNLg_239Pup.webp)

```c
//@version=5
indicator("lookahead demo", overlay = true)

//@variable The timeframe to request the data from.
string timeframe = input.timeframe("30", "Timeframe")

//@variable The requested `high` price from the current symbol on the `timeframe` without lookahead bias.
//          On realtime bars, it returns the current `high` of the `timeframe`.
float lookaheadOff = request.security(syminfo.tickerid, timeframe, high, lookahead = barmerge.lookahead_off)

//@variable The requested `high` price from the current symbol on the `timeframe` with lookahead bias.
//          Returns values that should NOT be accessible yet on historical bars.
float lookaheadOn = request.security(syminfo.tickerid, timeframe, high, lookahead = barmerge.lookahead_on)

//@variable The requested `high` price from the current symbol on the `timeframe` without lookahead bias or repainting.
//          Behaves the same on historical and realtime bars.
float lookaheadOnOffset = request.security(syminfo.tickerid, timeframe, high[1], lookahead = barmerge.lookahead_on)

// Plot the values.
plot(lookaheadOff, "High, no lookahead bias", color.new(color.blue, 40), 5)
plot(lookaheadOn, "High with lookahead bias", color.fuchsia, 3)
plot(lookaheadOnOffset, "High, no lookahead bias or repaint", color.aqua, 3)
// Highlight the background on realtime bars.
bgcolor(barstate.isrealtime ? color.new(color.orange, 60) : na, title = "Realtime bar highlight")
```

__Note que:__

- A série solicitada usando [barmerge.lookahead_off](https://br.tradingview.com/pine-script-reference/v5/#var_barmerge.lookahead_off) tem um novo valor histórico no _final_ de cada timeframe HTF, e ambas as séries solicitadas usando [barmerge.lookahead_on](https://br.tradingview.com/pine-script-reference/v5/#var_barmerge.lookahead_on) têm novos dados históricos no _início_ de cada timeframe.
- Em barras em tempo real, o plot da série sem lookahead ([blue](https://br.tradingview.com/pine-script-reference/v5/#var_color.blue)) e a série com lookahead e sem deslocamento histórico ([fuchsia](https://br.tradingview.com/pine-script-reference/v5/#var_color.fuchsia)) mostram o _mesmo valor_ (ou seja, o preço máximo não confirmado do timeframe HTF), pois não existem dados além desses pontos para vazar para o passado. Ambos esses plots __*repintarão*__ seus resultados após reiniciar a execução do script, pois as barras em [tempo real](https://br.tradingview.com/pine-script-reference/v5/#var_barstate.isrealtime) se tornarão barras [históricas](https://br.tradingview.com/pine-script-reference/v5/#var_barstate.ishistory).
- A série que usa lookahead e um deslocamento histórico ([aqua](https://br.tradingview.com/pine-script-reference/v5/#var_color.aqua)) não repinta seus valores, pois sempre referencia o último valor _confirmado_ do timeframe superior. Veja a seção [Evitando Repintura](./05_14_outros_timeframes_e_dados.md#evitando-repintura) desta página para mais informações.

> __Observação!__\
> No Pine Script v1 e v2, a função `security()` não incluía um parâmetro `lookahead`, mas se comportava como nas versões posteriores do Pine com `lookahead = barmerge.lookahead_on`, o que significa que sistematicamente usava dados do contexto HTF futuro em barras históricas. Portanto, deve-se _ter cautela_ com scripts do Pine v1 ou v2 que usam chamadas `security()` HTF, a menos que as chamadas da função contenham deslocamentos históricos.


# Feeds de Dados

Os provedores de dados do TradingView fornecem diferentes feeds de dados que scripts podem acessar para recuperar informações sobre um instrumento, incluindo:

- Dados históricos intradiários (para timeframe < 1D).
- Dados históricos de fim de dia _End-of-day (EOD)_ (para timeframe >= 1D).
- Dados em tempo real (que podem estar atrasados, dependendo do tipo de conta e dos serviços de dados adicionais).
- Dados de horas estendidas.

Nem todos esses tipos de feeds de dados existem para todos os instrumentos. Por exemplo, o símbolo "BNC:BLX" possui apenas dados EOD disponíveis.

Para alguns instrumentos com feeds históricos intradiários e EOD, os dados de volume podem não ser os mesmos, pois algumas negociações (negociações em bloco, negociações OTC, etc.) podem estar disponíveis apenas no _fim_ do dia de negociação. Consequentemente, o feed EOD incluirá esses dados de volume, mas o feed intradiário não. As diferenças entre os feeds de volume EOD e intradiário são quase inexistentes para instrumentos como criptomoedas, mas são comuns em ações.

Pequenas discrepâncias de preço também podem ocorrer entre os feeds EOD e intradiário. Por exemplo, o valor máximo em uma barra EOD pode não corresponder a nenhum valor máximo intradiário fornecido pelo provedor de dados para aquele dia.

Outra distinção entre os feeds de dados EOD e intradiário é que os feeds EOD não contêm informações de __*horas estendidas*__.

Ao recuperar informações em barras em tempo real com funções `request.*()`, é importante notar que os dados históricos e em tempo real relatados para um instrumento frequentemente dependem de feeds de dados _diferentes_. Um corretor/bolsa pode modificar retroativamente os valores relatados em barras em tempo real, o que os dados refletirão apenas após atualizar o gráfico ou reiniciar a execução do script.

Outra consideração importante é que os feeds de dados do gráfico e os feeds solicitados de provedores pelo script são gerenciados por processos independentes e concorrentes. Consequentemente, em alguns casos _raros_, é possível que ocorram disputas onde os resultados solicitados temporariamente fiquem fora de sincronia com o gráfico em uma barra em tempo real, o que um script ajusta retroativamente após reiniciar sua execução.

Esses pontos podem explicar variações nos valores recuperados pelas funções `request.*()` ao solicitar dados de outros contextos. Também podem resultar em discrepâncias entre os dados recebidos em barras em tempo real e barras históricas. Não há regras rígidas sobre as variações que podem ser encontradas em seus feeds de dados solicitados.

> __Observação!__\
> Como regra, o TradingView _não_ gera dados; ele depende de seus provedores de dados para as informações exibidas nos gráficos e acessadas por scripts.

Ao usar feeds de dados solicitados de outros contextos, também é crucial considerar as diferenças de _eixo de tempo_ entre o gráfico em que o script é executado e os feeds solicitados, pois as funções `request.*()` adaptam a série retornada ao eixo de tempo do gráfico. Por exemplo, solicitar dados "BTCUSD" no gráfico "SPY" com [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security) mostrará novos valores apenas quando o gráfico "SPY" tiver novos dados também. Como "SPY" não é um símbolo 24 horas, os dados "BTCUSD" retornados conterão lacunas que não estão presentes ao visualizar seu gráfico diretamente.

## `request.security()`

A função [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security) permite que scripts solicitem dados de outros contextos além do gráfico no qual o script está sendo executado, como:

- Outros símbolos, incluindo [símbolos de spread](https://br.tradingview.com/support/solutions/43000502298).
- Outros timeframes (veja a página sobre [Timeframes](./05_22_timeframes.md) para aprender sobre especificações de timeframes no Pine Script).
- [Contextos Personalizados](./05_14_outros_timeframes_e_dados.md#contextos-personalizados), incluindo sessões alternativas, ajustes de preço, tipos de gráficos, etc..., usando funções `ticker.*()`.

Esta é a assinatura da função:

```c
request.security(symbol, timeframe, expression, gaps, lookahead, ignore_invalid_symbol, currency, calc_bars_count) → series <type>
```

O valor `symbol` é o identificador do ticker que representa o símbolo de onde buscar os dados. Este parâmetro aceita valores em qualquer um dos seguintes formatos:

- Uma "string" representando um símbolo (por exemplo, "IBM" ou "EURUSD") ou um _par "Exchange:Symbol"_ (por exemplo, "NYSE:IBM" ou "OANDA:EURUSD"). Quando o valor não contém um prefixo de bolsa, a função seleciona a bolsa automaticamente. Recomenda-se especificar o prefixo da bolsa sempre que possível para resultados consistentes. Pode-se também passar uma string vazia para este parâmetro, o que faz com que a função use o símbolo do gráfico atual.
- Uma "string" representando um [símbolo spread](https://br.tradingview.com/support/solutions/43000502298) (por exemplo, "AMD/INTC"). Note que o modo "Bar Replay" não funciona com esses símbolos.
- As variáveis embutidas [syminfo.ticker](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo.ticker) ou [syminfo.tickerid](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo.tickerid), que retornam o símbolo ou o _par "Exchange:Symbol"_ que o gráfico atual referencia. Recomenda-se usar [syminfo.tickerid](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo.tickerid) para evitar ambiguidades, a menos que a informação da bolsa não importe na solicitação de dados. Para mais informações sobre variáveis `syminfo.*`, veja [esta seção](./05_06_informacoes_do_grafico.md#informações-do-símbolo) da página de [Informações do Gráfico](./05_06_informacoes_do_grafico.md).
- Um identificador de ticker personalizado criado usando funções `ticker.*()`. IDs de ticker construídos a partir dessas funções podem conter configurações adicionais para solicitar dados usando cálculos de [gráficos não padronizados](./05_13_dados_de_graficos_nao_padronizados.md), sessões alternativas e outros contextos. Veja a seção de [Contextos Personalizados](./05_14_outros_timeframes_e_dados.md#contextos-personalizados) para mais informações.

O valor `timeframe` especifica o timeframe dos dados solicitados. Este parâmetro aceita valores "string" no formato de [especificação de timeframe](./05_22_timeframes.md#especificações-de-string-do-timeframe) (por exemplo, um valor de "1D" representa o timeframe diário). Para solicitar dados do mesmo timeframe do gráfico em que o script é executado, use a variável [timeframe.period](https://br.tradingview.com/pine-script-reference/v5/#var_timeframe.period) ou uma string vazia.

O parâmetro `expression` da função [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security) determina os dados que ela recupera do contexto especificado. Este parâmetro versátil aceita valores "series" dos tipos [int](./04_09_tipagem_do_sistema.md#int), [float](./04_09_tipagem_do_sistema.md#float), [bool](./04_09_tipagem_do_sistema.md#bool), [color](./04_09_tipagem_do_sistema.md#color), [string](./04_09_tipagem_do_sistema.md#string), e [chart.point](./04_09_tipagem_do_sistema.md#chart-points-pontos-do-gráfico). Também pode aceitar [tuplas](./04_09_tipagem_do_sistema.md#tuples-tuplas), [coleções](./04_09_tipagem_do_sistema.md#coleções), [tipos definidos pelo usuário](./04_09_tipagem_do_sistema.md#tipos-definidos-pelo-usuário), e as saídas de chamadas de funções e [métodos](./04_13_metodos.md). Para mais detalhes sobre os dados que podem ser recuperados, veja a seção de [Dados Solicitáveis](./05_14_outros_timeframes_e_dados.md#dados-solicitáveis) abaixo.

> __Observação!__\
> Ao usar o valor de uma chamada [input.source()](https://br.tradingview.com/pine-script-reference/v5/#fun_input.source) no argumento `expression` e a entrada referenciar uma série de outro indicador, as funções `request.*()` calculam os resultados desse valor usando o __símbolo do gráfico__, independentemente do argumento `symbol` fornecido, pois não podem avaliar os escopos exigidos por uma série externa. Portanto, não se recomenda tentar solicitar dados de entrada de fonte externa de outros contextos.


# Timeframes

A função [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security) pode solicitar dados de qualquer timeframe disponível, independentemente do gráfico em que o script está sendo executado. O timeframe dos dados recuperados depende do argumento `timeframe` na chamada da função, que pode representar um timeframe superior (por exemplo, usando "1D" como valor de `timeframe` enquanto o script é executado em um gráfico intradiário) ou o timeframe do gráfico (ou seja, usando [timeframe.period](https://br.tradingview.com/pine-script-reference/v5/#var_timeframe.period) ou uma string vazia como argumento `timeframe`).

Scripts também podem solicitar dados _limitados_ de timeframes inferiores com [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security) (por exemplo, usando "1" como argumento `timeframe` enquanto o script é executado em um gráfico de 60 minutos). No entanto, não é recomendado usar esta função para solicitações de dados LTF. A função [request.security_lower_tf()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security_lower_tf) é mais adequada para esses casos.

## Timeframes Superiores

A maioria dos casos de uso de [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security) envolve solicitar dados de um timeframe superior ou igual ao timeframe do gráfico. Por exemplo, este script recupera o preço [hl2](https://br.tradingview.com/pine-script-reference/v5/#var_hl2) de um `higherTimeframe` solicitado. Ele [plota](./05_15_plots.md) a série resultante no gráfico ao lado do [hl2](https://br.tradingview.com/pine-script-reference/v5/#var_hl2) do gráfico atual para comparação:

![Timeframes superiores 01](./imgs/Other-timeframes-and-data-Request-security-Timeframes-Higher-timeframes-1.Cfl6KncV_1tTnEJ.webp)

```c
//@version=5
indicator("Higher timeframe security demo", overlay = true)

//@variable The higher timeframe to request data from.
string higherTimeframe = input.timeframe("240", "Higher timeframe")

//@variable The `hl2` value from the `higherTimeframe`. Combines lookahead with an offset to avoid repainting.
float htfPrice = request.security(syminfo.tickerid, higherTimeframe, hl2[1], lookahead = barmerge.lookahead_on)

// Plot the `hl2` from the chart timeframe and the `higherTimeframe`.
plot(hl2, "Current timeframe HL2", color.teal, 2)
plot(htfPrice, "Higher timeframe HL2", color.purple, 3)
```

__Note que:__

- Foi incluído um "_deslocamento_" ("_offset_") no argumento `expression` e usado [barmerge.lookahead_on](https://br.tradingview.com/pine-script-reference/v5/#var_barmerge.lookahead_on) em [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security) para garantir que a série retornada se comporte da mesma forma em barras históricas e em tempo real. Veja a seção [Evitando Repintura](./05_14_outros_timeframes_e_dados.md#evitando-repintura) para mais informações.

Note que no exemplo acima, é possível selecionar um valor de `higherTimeframe` que na verdade representa um _timeframe inferior_ ao usado pelo gráfico, pois o código não impede isso. Ao projetar um script para funcionar especificamente com timeframes superiores, recomenda-se incluir condições para evitar o acesso a timeframes inferiores, especialmente se houver intenção de [publicar](./06_04_publicando_scripts.md) o script.

Abaixo, foi adicionada uma estrutura [if](https://br.tradingview.com/pine-script-reference/v5/#kw_if) ao exemplo anterior que gera um [runtime error](https://br.tradingview.com/pine-script-reference/v5/#fun_runtime.error) ("_erro de tempo de execução_") quando a entrada `higherTimeframe` representa um timeframe menor que o timeframe do gráfico, evitando efetivamente que o script solicite dados LTF:

![Timeframes superiores 02](./imgs/Other-timeframes-and-data-Request-security-Timeframes-Higher-timeframes-2.DLmdElJ0_Z1Wqvh6.webp)

```c
//@version=5
indicator("Higher timeframe security demo", overlay = true)

//@variable The higher timeframe to request data from.
string higherTimeframe = input.timeframe("240", "Higher timeframe")

// Raise a runtime error when the `higherTimeframe` is smaller than the chart's timeframe.
if timeframe.in_seconds() > timeframe.in_seconds(higherTimeframe)
    runtime.error("The requested timeframe is smaller than the chart's timeframe. Select a higher timeframe.")

//@variable The `hl2` value from the `higherTimeframe`. Combines lookahead with an offset to avoid repainting.
float htfPrice = request.security(syminfo.tickerid, higherTimeframe, hl2[1], lookahead = barmerge.lookahead_on)

// Plot the `hl2` from the chart timeframe and the `higherTimeframe`.
plot(hl2, "Current timeframe HL2", color.teal, 2)
plot(htfPrice, "Higher timeframe HL2", color.purple, 3)
```

## Timeframes Inferiores

Embora a função [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security) seja destinada a operar em timeframes maiores ou iguais ao timeframe do gráfico, ela _pode_ solicitar dados de timeframes inferiores também, com limitações. Ao chamar essa função para acessar um timeframe inferior, ela avaliará a `expression` a partir do contexto LTF. No entanto, ela só pode retornar os resultados de uma única intrabar (barra LTF) em cada barra do gráfico.

A intrabar da qual a função retorna dados em cada barra histórica do gráfico depende do valor `lookahead` na chamada da função. Ao usar [barmerge.lookahead_on](https://br.tradingview.com/pine-script-reference/v5/#var_barmerge.lookahead_on), ela retornará a _primeira_ intrabar disponível do timeframe gráfico. Ao usar [barmerge.lookahead_off](https://br.tradingview.com/pine-script-reference/v5/#var_barmerge.lookahead_off), ela retornará a _última_ intrabar do timeframe gráfico. Em barras em tempo real, ela retorna o último valor disponível da `expression` do timeframe, independentemente do valor `lookahead`, pois as informações da intrabar em tempo real recuperadas pela função ainda não estão ordenadas.

Este script recupera dados de [fechamento](https://br.tradingview.com/pine-script-reference/v5/#var_close) do timeframe válido mais próximo a um quarto do tamanho do timeframe do gráfico. Ele faz duas chamadas para [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security) com valores `lookahead` diferentes. A primeira chamada usa [barmerge.lookahead_on](https://br.tradingview.com/pine-script-reference/v5/#var_barmerge.lookahead_on) para acessar o primeiro valor intrabar em cada barra do gráfico. A segunda usa o valor padrão `lookahead` ([barmerge.lookahead_off](https://br.tradingview.com/pine-script-reference/v5/#var_barmerge.lookahead_off)), que solicita o último valor intrabar atribuído a cada barra do gráfico. O script [plota](./05_15_plots.md) as saídas de ambas as chamadas no gráfico para comparar a diferença:

![Timeframes inferiores](./imgs/Other-timeframes-and-data-Request-security-Timeframes-Lower-timeframes-1.CzbZyyC2_Z2o4fpJ.webp)

```c
//@version=5
indicator("Lower timeframe security demo", overlay = true)

//@variable The valid timeframe closest to 1/4 the size of the chart timeframe.
string lowerTimeframe = timeframe.from_seconds(int(timeframe.in_seconds() / 4))

//@variable The `close` value on the `lowerTimeframe`. Represents the first intrabar value on each chart bar.
float firstLTFClose = request.security(syminfo.tickerid, lowerTimeframe, close, lookahead = barmerge.lookahead_on)
//@variable The `close` value on the `lowerTimeframe`. Represents the last intrabar value on each chart bar.
float lastLTFClose = request.security(syminfo.tickerid, lowerTimeframe, close)

// Plot the values.
plot(firstLTFClose, "First intrabar close", color.teal, 3)
plot(lastLTFClose, "Last intrabar close", color.purple, 3)
// Highlight the background on realtime bars.
bgcolor(barstate.isrealtime ? color.new(color.orange, 70) : na, title = "Realtime background highlight")
```

__Note que:__

- O script determina o valor de `lowerTimeframe` calculando o número de segundos no timeframe do gráfico com [timeframe.in_seconds()](https://br.tradingview.com/pine-script-reference/v5/#fun_timeframe.in_seconds), depois dividindo por quatro e convertendo o resultado para uma [string de timeframe válida](./05_22_timeframes.md#especificações-de-string-do-timeframe) via [timeframe.from_seconds()](https://br.tradingview.com/pine-script-reference/v5/#fun_timeframe.from_seconds).
- O plot da série sem lookahead ([purple](https://br.tradingview.com/pine-script-reference/v5/#var_color.purple)) alinha-se com o valor de [fechamento](https://br.tradingview.com/pine-script-reference/v5/#var_close) no timeframe do gráfico, pois este é o último valor intrabar na barra do gráfico.
- Ambas as chamadas de [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security) retornam o _mesmo_ valor (o [fechamento](https://br.tradingview.com/pine-script-reference/v5/#var_close) atual) em cada barra [realtime](https://br.tradingview.com/pine-script-reference/v5/#var_barstate.isrealtime), como mostrado nas barras com o fundo [orange](https://br.tradingview.com/pine-script-reference/v5/#var_color.orange).
- Scripts podem recuperar até 100.000 intrabars de um contexto de timeframe inferior. Veja [esta](./06_05_limitacoes.md#intrabars) seção da página de [Limitações](./06_05_limitacoes.md).

> __Observação!__\
> Embora scripts possam usar [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security) para recuperar os valores de uma única intrabar em cada barra do gráfico, o que pode ser útil em alguns casos únicos, recomenda-se usar a função [request.security_lower_tf()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security_lower_tf) para análise intrabar sempre que possível, pois ela retorna um [array](https://br.tradingview.com/pine-script-reference/v5/#type_array) contendo dados de _todas_ as intrabars disponíveis dentro de uma barra do gráfico. Veja [esta seção](./05_14_outros_timeframes_e_dados.md#requestsecurity_lower_tf) para mais informações.


# Dados Solicitáveis

A função [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security) é bastante versátil, pois pode recuperar valores de qualquer tipo fundamental ([int](./04_09_tipagem_do_sistema.md#int), [float](./04_09_tipagem_do_sistema.md#float), [bool](./04_09_tipagem_do_sistema.md#bool), [color](./04_09_tipagem_do_sistema.md#color), ou [string](./04_09_tipagem_do_sistema.md#string)). Ela também pode solicitar IDs de estruturas de dados e tipos embutidos ou [definidos pelo usuário](./04_09_tipagem_do_sistema.md#tipos-definidos-pelo-usuário) que referenciam tipos fundamentais. Os dados que essa função solicita dependem do parâmetro `expression`, que aceita qualquer um dos seguintes argumentos:

- [Variáveis incorporadas e chamadas de função.](./05_14_outros_timeframes_e_dados.md#variáveis-e-funções-embutidas)
- [Variáveis calculadas pelo script.](./05_14_outros_timeframes_e_dados.md#variáveis-calculadas)
- [Tuplas.](./05_14_outros_timeframes_e_dados.md#tuples-tuplas)
- [Chamadas de funções definidas pelo usuário.](./05_14_outros_timeframes_e_dados.md#funções-definidas-pelo-usuário)
- [Pontos do gráfico.](./05_14_outros_timeframes_e_dados.md#pontos-do-gráfico)
- [Coleções.](./05_14_outros_timeframes_e_dados.md#coleções)
- [Tipos definidos pelo usuário.](./05_14_outros_timeframes_e_dados.md#tipos-definidos-pelo-usuário)

> __Observação!__\
> A função [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security) duplica os escopos e operações necessários para a `expression` calcular os valores solicitados em outro contexto, o que aumenta o consumo de memória em tempo de execução. Além disso, os escopos adicionais produzidos por cada chamada de [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security) contam para os _limites de compilação_ do script. Consulte a seção [Contagem de escopos](./06_05_limitacoes.md#contagem-de-escopos) da página de [Limitações](./06_05_limitacoes.md) para mais informações.

## Variáveis e Funções Embutidas

Um caso de uso frequente de [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security) é solicitar a saída de uma variável incorporada ou chamada de função/[método](./04_13_metodos.md) de outro símbolo ou timeframe.

Por exemplo, para calcular a [SMA](https://br.tradingview.com/pine-script-reference/v5/#fun_ta.sma) de 20 barras do preço [ohlc4](https://br.tradingview.com/pine-script-reference/v5/#var_ohlc4) de um símbolo do timeframe diário enquanto se está em um gráfico intradiário, pode-se realizar isso com uma única linha de código:

```c
float ma = request.security(syminfo.tickerid, "1D", ta.sma(ohlc4, 20))
```

A linha acima calcula o valor de [ta.sma(ohlc4, 20)](https://br.tradingview.com/pine-script-reference/v5/#fun_ta.sma) no símbolo atual a partir do timeframe diário.

É importante notar que iniciantes no Pine podem confundir a linha de código acima como equivalente à seguinte:

```c
float ma = ta.sma(request.security(syminfo.tickerid, "1D", ohlc4), 20)
```

No entanto, essa linha retornará um resultado completamente _diferente_. Em vez de solicitar uma SMA de 20 barras do timeframe diário, ela solicita o preço [ohlc4](https://br.tradingview.com/pine-script-reference/v5/#var_ohlc4) do timeframe diário e calcula a [ta.sma()](https://br.tradingview.com/pine-script-reference/v5/#fun_ta.sma) dos resultados ao longo de 20 __barras do gráfico__.

Em essência, quando a intenção é solicitar os resultados de uma expressão de outros contextos, passe a expressão _diretamente_ para o parâmetro `expression` na chamada [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security), conforme demonstrado no exemplo inicial.

Explicando esse conceito: o script abaixo calcula uma faixa de médias móveis em múltiplos timeframes (_multi-timeframe (MTF)_), onde cada média móvel na faixa é calculada sobre o mesmo número de barras em seu respectivo timeframe. Cada chamada [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security) usa [ta.sma(close, length)](https://br.tradingview.com/pine-script-reference/v5/#fun_ta.sma) como seu argumento `expression` para retornar uma SMA de `length` barras do timeframe especificado:

![Variáveis e funções embutidas](./imgs/Other-timeframes-and-data-Request-security-Requestable-data-Built-in-variables-and-functions-1.CPvZdzBd_Z1LKRRo.webp)

```c
//@version=5
indicator("Requesting built-ins demo", "MTF Ribbon", true)

//@variable The length of each moving average.
int length = input.int(20, "Length", 1)

//@variable The number of seconds in the chart timeframe.
int chartSeconds = timeframe.in_seconds()

// Calculate the higher timeframes closest to 2, 3, and 4 times the size of the chart timeframe.
string htf1 = timeframe.from_seconds(chartSeconds * 2)
string htf2 = timeframe.from_seconds(chartSeconds * 3)
string htf3 = timeframe.from_seconds(chartSeconds * 4)

// Calculate the `length`-bar moving averages from each timeframe.
float chartAvg = ta.sma(ohlc4, length)
float htfAvg1  = request.security(syminfo.tickerid, htf1, ta.sma(ohlc4, length))
float htfAvg2  = request.security(syminfo.tickerid, htf2, ta.sma(ohlc4, length))
float htfAvg3  = request.security(syminfo.tickerid, htf3, ta.sma(ohlc4, length))

// Plot the results.
plot(chartAvg, "Chart timeframe SMA", color.red, 3)
plot(htfAvg1, "Double timeframe SMA", color.orange, 3)
plot(htfAvg2, "Triple timeframe SMA", color.green, 3)
plot(htfAvg3, "Quadruple timeframe SMA", color.blue, 3)

// Highlight the background on realtime bars.
bgcolor(barstate.isrealtime ? color.new(color.aqua, 70) : na, title = "Realtime highlight")
```

__Note que:__

- O script calcula os timeframes mais altos da faixa multiplicando o valor de [timeframe.in_seconds()](https://br.tradingview.com/pine-script-reference/v5/#fun_timeframe.in_seconds) do gráfico por 2, 3 e 4, convertendo cada resultado em uma [string de timeframe válida](./05_22_timeframes.md#especificações-de-string-do-timeframe) usando [timeframe.from_seconds()](https://br.tradingview.com/pine-script-reference/v5/#fun_timeframe.from_seconds).
- Em vez de chamar [ta.sma()](https://br.tradingview.com/pine-script-reference/v5/#fun_ta.sma) dentro de cada chamada [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security), poderia-se usar a variável `chartAvg` como `expression` em cada chamada para obter o mesmo resultado. Veja a [próxima seção](./05_14_outros_timeframes_e_dados.md#variáveis-calculadas) para mais informações.
- Em barras em tempo real, este script também rastreia valores de SMA _não confirmados_ de cada timeframe mais alto. Veja a seção [Comportamento histórico e em tempo real](./05_14_outros_timeframes_e_dados.md#comportamento-histórico-e-tempo-real) para saber mais.

## Variáveis Calculadas

O parâmetro `expression` de [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security) aceita variáveis declaradas no escopo global, permitindo que scripts avaliem os cálculos de suas variáveis em outros contextos sem listar as operações de forma redundante em cada chamada de função.

Por exemplo, pode-se declarar a seguinte variável:

```c
priceReturn = (close - close[1]) / close[1]
```

E executar o cálculo da variável a partir de outro contexto com [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security):

```c
requestedReturn = request.security(symbol, timeframe.period, priceReturn)
```

A chamada da função na linha acima retornará o resultado do cálculo de `priceReturn` nos dados de outro `symbol` como uma série adaptada ao gráfico atual, que o script pode exibir diretamente no gráfico ou utilizar em operações adicionais.

O exemplo a seguir compara os retornos de preço do símbolo do gráfico atual e de outro `symbol` especificado. O script declara a variável `priceReturn` no contexto do gráfico, depois usa essa variável em [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security) para avaliar seu cálculo em outro `symbol`. Em seguida, calcula a [correlação](https://br.tradingview.com/pine-script-reference/v5/#fun_ta.correlation) entre `priceReturn` e `requestedReturn` e [plota](./05_15_plots.md) o resultado no gráfico:

![Variáveis calculadas](./imgs/Other-timeframes-and-data-Request-security-Requestable-data-Calculated-variables-1.DpMsOLKI_Z6c3MD.webp)

```c
//@version=5
indicator("Requesting calculated variables demo", "Price return correlation")

//@variable The symbol to compare to the chart symbol.
string symbol = input.symbol("SPY", "Symbol to compare")
//@variable The number of bars in the calculation window.
int length = input.int(60, "Length", 1)

//@variable The close-to-close price return.
float priceReturn = (close - close[1]) / close[1]
//@variable The close-to-close price return calculated on another `symbol`.
float requestedReturn = request.security(symbol, timeframe.period, priceReturn)

//@variable The correlation between the `priceReturn` and `requestedReturn` over `length` bars.
float correlation = ta.correlation(priceReturn, requestedReturn, length)
//@variable The color of the correlation plot.
color plotColor = color.from_gradient(correlation, -1, 1, color.purple, color.orange)

// Plot the correlation value.
plot(correlation, "Correlation", plotColor, style = plot.style_area)
```

__Note que:__

- A chamada [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security) executa o mesmo cálculo usado na declaração de `priceReturn`, exceto que utiliza os valores de [fechamento](https://br.tradingview.com/pine-script-reference/v5/#var_close) obtidos do `symbol` de entrada.
- O script colore o gráfico com um [gradiente](https://br.tradingview.com/pine-script-reference/v5/#fun_color.from_gradient) baseado no valor de `correlation`. Para saber mais sobre gradientes de cores no Pine, consulte [esta](./05_07_cores.md#colorfrom_gradient) seção da página do Manual do Usuário sobre [cores](./05_07_cores.md).

## Tuples (_Tuplas_)

[Tuplas](./04_09_tipagem_do_sistema.md#tuples-tuplas) no Pine Script são conjuntos de expressões separados por vírgula, delimitados por colchetes, que podem conter múltiplos valores de qualquer tipo disponível. Usam-se tuplas ao criar funções ou outros blocos locais que retornam mais de um valor.

A função [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security) pode aceitar uma tupla como seu argumento `expression`, permitindo que scripts solicitem múltiplas séries de diferentes tipos em uma única chamada de função. As expressões dentro das tuplas solicitadas podem ser de qualquer tipo descrito na seção [Dados Solicitáveis](./05_14_outros_timeframes_e_dados.md#dados-solicitáveis) desta página, excluindo outras tuplas.

> __Observação!__\
> O tamanho combinado de todas as tuplas retornadas pelas chamadas `request.*()` em um script não pode exceder 127 elementos. Veja [esta seção](./06_05_limitacoes.md#limite-de-elementos-da-tupla) da página de [Limitações](./06_05_limitacoes.md) para mais informações.

As tuplas são especialmente úteis quando um script precisa recuperar mais de um valor de um contexto específico.

Por exemplo, este script calcula o [percentual de ranking](https://br.tradingview.com/pine-script-reference/v5/#fun_ta.percentrank) do preço de [fechamento](https://br.tradingview.com/pine-script-reference/v5/#var_close) ao longo de `length` barras e atribui a expressão à variável `rank`. Em seguida, chama [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security) para solicitar uma tupla contendo os valores de `rank`, [ta.crossover(rank, 50)](https://br.tradingview.com/pine-script-reference/v5/#fun_ta.crossover) e [ta.crossunder(rank, 50)](https://br.tradingview.com/pine-script-reference/v5/#fun_ta.crossunder) do `timeframe` especificado. O script [plota](./05_15_plots.md) o `requestedRank` e usa os valores "bool" `crossOver` e `crossUnder` dentro de [bgcolor()](https://br.tradingview.com/pine-script-reference/v5/#fun_bgcolor) para destacar condicionalmente o fundo do painel do gráfico:

![Tuples tuplas](./imgs/Other-timeframes-and-data-Request-security-Requestable-data-Tuples-1.DfMFJD2A_22M4RL.webp)

```c
//@version=5
indicator("Requesting tuples demo", "Percent rank cross")

//@variable The timeframe of the request.
string timeframe = input.timeframe("240", "Timeframe")
//@variable The number of bars in the calculation.
int length = input.int(20, "Length")

//@variable The previous bar's percent rank of the `close` price over `length` bars.
float rank = ta.percentrank(close, length)[1]

// Request the `rank` value from another `timeframe`, and two "bool" values indicating the `rank` from the `timeframe`
// crossed over or under 50.
[requestedRank, crossOver, crossUnder] = request.security(
     syminfo.tickerid, timeframe, [rank, ta.crossover(rank, 50), ta.crossunder(rank, 50)],
     lookahead = barmerge.lookahead_on
 )

// Plot the `requestedRank` and create a horizontal line at 50.
plot(requestedRank, "Percent Rank", linewidth = 3)
hline(50, "Cross line", linewidth = 2)
// Highlight the background of all bars where the `timeframe`'s `crossOver` or `crossUnder` value is `true`.
bgcolor(crossOver ? color.new(color.green, 50) : crossUnder ? color.new(color.red, 50) : na)
```

__Note que:__

- A expressão da variável `rank` foi deslocada por uma barra usando o operador de referência de histórico [[]](https://br.tradingview.com/pine-script-reference/v5/#op_%5B%5D) e incluído [barmerge.lookahead_on](https://br.tradingview.com/pine-script-reference/v5/#var_barmerge.lookahead_on) na chamada [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security) para garantir que os valores nas barras em tempo real não sejam repintados após se tornarem barras históricas. Veja a seção [Evitando Repintura](./05_14_outros_timeframes_e_dados.md#evitando-repintura) para mais informações.
- A chamada [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security) retorna uma tupla, portanto, é usada uma _declaração de tupla_ para declarar as variáveis `requestedRank`, `crossOver` e `crossUnder`. Para saber mais sobre o uso de tuplas, veja [esta seção](./04_09_tipagem_do_sistema.md#tuples-tuplas) sobre o [sistema de tipos](./04_09_tipagem_do_sistema.md).

## Funções Definidas pelo Usuário

[Funções Definida pelo Usuário](./04_11_funcoes_definida_pelo_usuario.md) e [métodos](./04_13_metodos.md#métodos-definidos-pelo-usuário) são funções personalizadas escritas por usuários. Elas permitem definir sequências de operações associadas a um identificador que os scripts podem chamar convenientemente durante sua execução (por exemplo, `myUDF()`).

A função [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security) pode solicitar os resultados de [Funções Definida pelo Usuário](./04_11_funcoes_definida_pelo_usuario.md) e [métodos](./04_13_metodos.md#métodos-definidos-pelo-usuário) cujos escopos consistem em qualquer tipo descrito na seção [Dados Solicitáveis](./05_14_outros_timeframes_e_dados.md#dados-solicitáveis) desta página.

Por exemplo, este script contém uma função definida pelo usuário `weightedBB()` que calcula Bandas de Bollinger com a base média ponderada por uma série `weight` especificada. A função retorna uma [tupla](./04_09_tipagem_do_sistema.md#tuples-tuplas) de valores personalizados das bandas. O script chama `weightedBB()` como o argumento `expression` em [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security) para recuperar uma [tupla](./05_14_outros_timeframes_e_dados.md#tuples-tuplas) de valores das bandas calculados no `timeframe` especificado e [plota](./05_15_plots.md) os resultados no gráfico:

![Funções definidas pelo usuário](./imgs/Other-timeframes-and-data-Request-security-Requestable-data-User-defined-functions-1.DTi5QOZX_ZMtoWB.webp)

```c
//@version=5
indicator("Requesting user-defined functions demo", "Weighted Bollinger Bands", true)

//@variable The timeframe of the request.
string timeframe = input.timeframe("480", "Timeframe")

//@function     Calculates Bollinger Bands with a custom weighted basis.
//@param source The series of values to process.
//@param length The number of bars in the calculation.
//@param mult   The standard deviation multiplier.
//@param weight The series of weights corresponding to each `source` value.
//@returns      A tuple containing the basis, upper band, and lower band respectively.
weightedBB(float source, int length, float mult = 2.0, float weight = 1.0) =>
    //@variable The basis of the bands.
    float ma = math.sum(source * weight, length) / math.sum(weight, length)
    //@variable The standard deviation from the `ma`.
    float dev = 0.0
    // Loop to accumulate squared error.
    for i = 0 to length - 1
        difference = source[i] - ma
        dev += difference * difference
    // Divide `dev` by the `length`, take the square root, and multiply by the `mult`.
    dev := math.sqrt(dev / length) * mult
    // Return the bands.
    [ma, ma + dev, ma - dev]

// Request weighted bands calculated on the chart symbol's prices over 20 bars from the
// last confirmed bar on the `timeframe`.
[basis, highBand, lowBand] = request.security(
     syminfo.tickerid, timeframe, weightedBB(close[1], 20, 2.0, (high - low)[1]), lookahead = barmerge.lookahead_on
 )

// Plot the values.
basisPlot = plot(basis, "Basis", color.orange, 2)
upperPlot = plot(highBand, "Upper", color.teal, 2)
lowerPlot = plot(lowBand, "Lower", color.maroon, 2)
fill(upperPlot, lowerPlot, color.new(color.gray, 90), "Background")
```

__Note que:__

- Os argumentos `source` e `weight` na chamada `weightedBB()` usada como `expression` em [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security) são deslocados, e é utilizado [barmerge.lookahead_on](https://br.tradingview.com/pine-script-reference/v5/#var_barmerge.lookahead_on) para garantir que os resultados solicitados reflitam os últimos valores confirmados do `timeframe` nas barras em tempo real. Consulte [esta seção](./05_14_outros_timeframes_e_dados.md#evitando-repintura) para saber mais.

## Pontos do Gráfico

[Pontos do gráfico](./04_09_tipagem_do_sistema.md#chart-points-pontos-do-gráfico) são tipos de referência que representam coordenadas no gráfico. As [linhas](./05_12_lines_e_boxes.md#lines-linhas), [caixas](./05_12_lines_e_boxes.md#boxes-caixas), [polilinhas](./05_12_lines_e_boxes.md#polylines-polilinhas) e [labels](./05_20_text_e_shapes.md#labels) usam objetos [chart.point](https://br.tradingview.com/pine-script-reference/v5/#type_chart.point) para definir suas localizações de exibição.

A função [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security) pode usar o ID de uma instância de [chart.point](https://br.tradingview.com/pine-script-reference/v5/#type_chart.point) em seu argumento `expression`, permitindo que scripts recuperem coordenadas do gráfico de outros contextos.

O exemplo abaixo solicita uma tupla de [pontos do gráfico](./04_09_tipagem_do_sistema.md#chart-points-pontos-do-gráfico) históricos de um timeframe maior e os usa para desenhar [caixas](./05_12_lines_e_boxes.md#boxes-caixas) no gráfico. O script declara as variáveis `topLeft` e `bottomRight` que referenciam os IDs de [chart.point](https://br.tradingview.com/pine-script-reference/v5/#type_chart.point) da última barra confirmada. Em seguida, usa [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security) para solicitar uma [tupla](https://br.tradingview.com/pine-script-docs/concepts/other-timeframes-and-data#tuples) contendo os IDs dos [pontos do gráfico](./04_09_tipagem_do_sistema.md#chart-points-pontos-do-gráfico) representando `topLeft` e `bottomRight` de um `higherTimeframe`.

Quando uma nova barra começa no `higherTimeframe`, o script desenha uma [nova caixa](https://br.tradingview.com/pine-script-reference/v5/#fun_box.new) usando as coordenadas de `time` e `price` dos pontos do gráfico `requestedTopLeft` e `requestedBottomRight`:

![Pontos do gráfico](./imgs/Other-timeframes-and-data-Request-security-Requestable-data-Chart-points-1.C5dKnJ3R_Z1YxlJR.webp)

```c
//@version=5
indicator("Requesting chart points demo", "HTF Boxes", true, max_boxes_count = 500)

//@variable The timeframe to request data from.
string higherTimeframe = input.timeframe("1D", "Timeframe")

// Raise a runtime error if the `higherTimeframe` is smaller than the chart's timeframe.
if timeframe.in_seconds(higherTimeframe) < timeframe.in_seconds(timeframe.period)
    runtime.error("The selected timeframe is too small. Choose a higher timeframe.")

//@variable A `chart.point` containing top-left coordinates from the last confirmed bar.
topLeft = chart.point.now(high)[1]
//@variable A `chart.point` containing bottom-right coordinates from the last confirmed bar.
bottomRight = chart.point.from_time(time_close, low)[1]

// Request the last confirmed `topLeft` and `bottomRight` chart points from the `higherTimeframe`.
[requestedTopLeft, requestedBottomRight] = request.security(
     syminfo.tickerid, higherTimeframe, [topLeft, bottomRight], lookahead = barmerge.lookahead_on
 )

// Draw a new box when a new `higherTimeframe` bar starts.
// The box uses the `time` fields from the `requestedTopLeft` and `requestedBottomRight` as x-coordinates.
if timeframe.change(higherTimeframe)
    box.new(
         requestedTopLeft, requestedBottomRight, color.purple, 3, 
         xloc = xloc.bar_time, bgcolor = color.new(color.purple, 90)
     )
```

__Note que:__

- Este exemplo é projetado especificamente para timeframes maiores, é incluso um [erro de execução](https://br.tradingview.com/pine-script-reference/v5/#fun_runtime.error) personalizado que o script gera quando o [timeframe.in_seconds()](https://br.tradingview.com/pine-script-reference/v5/#fun_timeframe.in_seconds) do `higherTimeframe` é menor que o [timeframe do gráfico](https://br.tradingview.com/pine-script-reference/v5/#var_timeframe.period).

## Coleções

As _coleções_ do Pine Script ([arrays](./04_14_arrays.md) e [mapas](./04_16_mapas.md)) são estruturas de dados que contêm um número arbitrário de elementos com tipos especificados. A função [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security) pode recuperar os IDs de [coleções](./04_09_tipagem_do_sistema.md#coleções) cujos elementos consistem em:

- [Tipos fundamentais](./04_09_tipagem_do_sistema.md#tipos).
- [Pontos do gráfico](./04_09_tipagem_do_sistema.md#chart-points-pontos-do-gráfico).
- [Tipos definidos pelo usuário](./04_09_tipagem_do_sistema.md#tipos-definidos-pelo-usuário) que atendem aos critérios listados na [seção abaixo](./05_14_outros_timeframes_e_dados.md#tipos-definidos-pelo-usuário).

Este exemplo calcula o _ratio_ da variação alta-baixa de uma barra confirmada em relação à variação entre os valores [mais altos](https://br.tradingview.com/pine-script-reference/v5/#fun_ta.highest) e [mais baixos](https://br.tradingview.com/pine-script-reference/v5/#fun_ta.lowest) em 10 barras de um `symbol` e `timeframe` especificados. Ele usa [mapas](./04_16_mapas.md) para armazenar os valores usados nos cálculos.

O script cria um mapa `data` com chaves "string" e valores "float" para armazenar valores de preço de [high](https://br.tradingview.com/pine-script-reference/v5/#var_high), [low](https://br.tradingview.com/pine-script-reference/v5/#var_low), [highest](https://br.tradingview.com/pine-script-reference/v5/#fun_ta.highest) e [lowest](https://br.tradingview.com/pine-script-reference/v5/#fun_ta.lowest) em cada barra, que é usado como `expression` em [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security) para calcular um mapa `otherData` representando os `data` do contexto especificado. Ele usa os valores associados às chaves "high", "low", "highest" e "lowest" do mapa `otherData` para calcular o [_ratio_ que ele plota](./05_15_plots.md) no painel do gráfico:

![Coleções](./imgs/Other-timeframes-and-data-Request-security-Requestable-data-Collections-1.C6G31C3k_2bffgq.webp)

```c
//@version=5
indicator("Requesting collections demo", "Bar range ratio")

//@variable The ticker ID to request data from.
string symbol = input.symbol("", "Symbol")
//@variable The timeframe of the request.
string timeframe = input.timeframe("30", "Timeframe")

//@variable A map with "string" keys and "float" values.
var map<string, float> data = map.new<string, float>()

// Put key-value pairs into the `data` map.
map.put(data, "High", high)
map.put(data, "Low", low)
map.put(data, "Highest", ta.highest(10))
map.put(data, "Lowest", ta.lowest(10))

//@variable A new `map` whose data is calculated from the last confirmed bar of the requested context.
map<string, float> otherData = request.security(symbol, timeframe, data[1], lookahead = barmerge.lookahead_on)

//@variable The ratio of the context's bar range to the max range over 10 bars. Returns `na` if no data is available.
float ratio = na
if not na(otherData)
    ratio := (otherData.get("High") - otherData.get("Low")) / (otherData.get("Highest") - otherData.get("Lowest"))

//@variable A gradient color for the plot of the `ratio`.
color ratioColor = color.from_gradient(ratio, 0, 1, color.purple, color.orange)

// Plot the `ratio`.
plot(ratio, "Range Ratio", ratioColor, 3, plot.style_area)
```

__Note que:__

- A chamada [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security) neste script pode retornar [na](https://br.tradingview.com/pine-script-reference/v5/#var_na) se não houver dados disponíveis do contexto especificado. Como não é possível chamar [métodos](./04_13_metodos.md) em uma variável [map](https://br.tradingview.com/pine-script-reference/v5/#type_map) quando seu valor é [na](https://br.tradingview.com/pine-script-reference/v5/#var_na), foi adicionada uma estrutura [if](https://br.tradingview.com/pine-script-reference/v5/#kw_if) para calcular um novo valor de `ratio` apenas quando `otherData` referencia um ID [map](https://br.tradingview.com/pine-script-reference/v5/#type_map) válido.

## Tipos Definidos pelo Usuário

[Tipos Definidos pelo Usuário (UDTs)](./04_09_tipagem_do_sistema.md#tipos-definidos-pelo-usuário) são _tipos compostos_ que contêm um número arbitrário de _campos_, que podem ser de qualquer tipo disponível, incluindo outros [tipos definidos pelo usuário](./04_09_tipagem_do_sistema.md#tipos-definidos-pelo-usuário).

A função [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security) pode recuperar os IDs de [objetos](./04_12_objetos.md) produzidos por [UDTs](./04_09_tipagem_do_sistema.md#tipos-definidos-pelo-usuário) de outros contextos se seus campos consistirem em:

- [Tipos fundamentais](./04_09_tipagem_do_sistema.md#tipos).
- [Pontos do gráfico](./04_09_tipagem_do_sistema.md#chart-points-pontos-do-gráfico).
- [Coleções](./04_09_tipagem_do_sistema.md#coleções) que atendem aos critérios listados na [seção acima](./05_14_outros_timeframes_e_dados.md#coleções).
- Outros [UDTs](./04_09_tipagem_do_sistema.md#tipos-definidos-pelo-usuário) cujos campos consistem em qualquer um desses tipos.

O exemplo a seguir solicita um ID de [objeto](./04_12_objetos.md) usando um `symbol` especificado e exibe seus valores de campo em um painel do gráfico.

O script contém um UDT `TickerInfo` com campos "string" para valores `syminfo.*`, um campo [array](https://br.tradingview.com/pine-script-reference/v5/#type_array) para armazenar dados de preço recentes "float", e um campo "int" para manter o valor de [bar_index](https://br.tradingview.com/pine-script-reference/v5/#var_bar_index) do ticker solicitado. Ele atribui um novo ID `TickerInfo` a uma variável `info` em cada barra e usa essa variável como `expression` em [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security) para recuperar o ID de um [objeto](./04_12_objetos.md) representando o `info` calculado do `symbol` especificado.

O script exibe os valores `description`, `tickerType`, `currency` e `barIndex` do objeto `requestedInfo` em um [label](https://br.tradingview.com/pine-script-reference/v5/#type_label) e usa [plotcandle()](https://br.tradingview.com/pine-script-reference/v5/#fun_plotcandle) para exibir os valores de seu array `prices`:

![Tipos definidos pelo usuário](./imgs/Other-timeframes-and-data-Request-security-Requestable-data-User-defined-types-1.D90DRv4r_1OUO19.webp)

```c
//@version=5
indicator("Requesting user-defined types demo", "Ticker info")

//@variable The symbol to request information from.
string symbol = input.symbol("NASDAQ:AAPL", "Symbol")

//@type               A custom type containing information about a ticker.
//@field description  The symbol's description.
//@field tickerType   The type of ticker.
//@field currency     The symbol's currency.
//@field prices       An array of the symbol's current prices.
//@field barIndex     The ticker's `bar_index`.
type TickerInfo
    string       description
    string       tickerType
    string       currency
    array<float> prices
    int          barIndex

//@variable A `TickerInfo` object containing current data.
info = TickerInfo.new(
     syminfo.description, syminfo.type, syminfo.currency, array.from(open, high, low, close), bar_index
 )
//@variable The `info` requested from the specified `symbol`.
TickerInfo requestedInfo = request.security(symbol, timeframe.period, info)
// Assign a new `TickerInfo` instance to `requestedInfo` if one wasn't retrieved.
if na(requestedInfo)
    requestedInfo := TickerInfo.new(prices = array.new<float>(4))

//@variable A label displaying information from the `requestedInfo` object.
var infoLabel = label.new(
     na, na, "", color = color.purple, style = label.style_label_left, textcolor = color.white, size = size.large
 )
//@variable The text to display inside the `infoLabel`.
string infoText = na(requestedInfo) ? "" : str.format(
     "{0}\nType: {1}\nCurrency: {2}\nBar Index: {3}",
     requestedInfo.description, requestedInfo.tickerType, requestedInfo.currency, requestedInfo.barIndex
 )

// Set the `point` and `text` of the `infoLabel`.
label.set_point(infoLabel, chart.point.now(array.last(requestedInfo.prices)))
label.set_text(infoLabel, infoText)
// Plot candles using the values from the `prices` array of the `requestedInfo`.
plotcandle(
     requestedInfo.prices.get(0), requestedInfo.prices.get(1), requestedInfo.prices.get(2), requestedInfo.prices.get(3),
     "Requested Prices"
 )
```

__Note que:__

- As variáveis `syminfo.*` usadas neste script retornam tipos qualificados como "string simples". Contudo, os [objetos](./04_12_objetos.md) no Pine são _sempre_ qualificados como "series". Consequentemente, todos os valores atribuídos aos campos do objeto `info` automaticamente adotam o [qualificativo](./04_09_tipagem_do_sistema.md#qualificadores) "series".
- A chamada [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security) pode retornar [na](https://br.tradingview.com/pine-script-reference/v5/#var_na) devido a diferenças entre os dados solicitados do `symbol` e o gráfico principal. Este script atribui um novo objeto `TickerInfo` ao `requestedInfo` nesse caso para evitar erros de execução.

## `request.security_lower_tf()`

A função [request.security_lower_tf()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security_lower_tf) é uma alternativa à [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security) projetada para solicitar informações de contextos de menor timeframe (LTF) de forma confiável.

Enquanto [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security) pode recuperar dados de uma _única_ barra intradiária em cada barra do gráfico, [request.security_lower_tf()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security_lower_tf) recupera dados de _todas_ as barras intradiárias disponíveis em cada barra do gráfico, que o script pode acessar e usar em cálculos adicionais. Cada chamada pode recuperar até 100.000 intrabarras de um timeframe menor. Consulte [esta seção](./06_05_limitacoes.md#chamadas-request) da página de [Limitações](./06_05_limitacoes.md) para mais informações.

> __Observação!__\
> Trabalhar com [request.security_lower_tf()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security_lower_tf) envolve o uso frequente de [arrays](./04_14_arrays.md), pois sempre retorna resultados como [arrays](https://br.tradingview.com/pine-script-reference/v5/#type_array). Portanto, recomenda-se se familiarizar com [arrays](./04_14_arrays.md) para aproveitar ao máximo essa função em seus scripts.

Abaixo está a assinatura da função, semelhante à [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security):

```c
request.security_lower_tf(symbol, timeframe, expression, ignore_invalid_symbol, currency, ignore_invalid_timeframe, calc_bars_count) → array<type>
```

Essa função __somente__ solicita dados de timeframes menores ou iguais ao timeframe gráfico. Se o `timeframe` da solicitação representar um timeframe maior que o [timeframe gráfico](https://br.tradingview.com/pine-script-reference/v5/#var_timeframe.period), a função gerará um erro de execução ou retornará valores [na](https://br.tradingview.com/pine-script-reference/v5/#var_na) dependendo do argumento `ignore_invalid_timeframe` na chamada. O valor padrão para este parâmetro é `false`, o que significa que ele gerará um erro e interromperá a execução do script ao tentar solicitar dados de um timeframe maior.

### Solicitando Dados Intrabar

Os dados intrabar podem fornecer informações adicionais que podem não ser óbvias ou acessíveis apenas analisando os dados amostrados no timeframe gráfico. A função [request.security_lower_tf()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security_lower_tf) pode recuperar muitos tipos de dados de um contexto intrabar.

Antes de prosseguir nesta seção, recomenda-se explorar a seção [Dados Solicitáveis](./05_14_outros_timeframes_e_dados.md#dados-solicitáveis) da seção [request.security()](./05_14_outros_timeframes_e_dados.md#requestsecurity) acima, que fornece informações básicas sobre os tipos de dados que podem ser solicitados. O parâmetro `expression` em [request.security_lower_tf()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security_lower_tf) aceita a maioria dos mesmos argumentos discutidos nessa seção, excluindo referências diretas a [coleções](./04_09_tipagem_do_sistema.md#coleções) e variáveis mutáveis declaradas no escopo principal do script. Embora aceite muitos dos mesmos tipos de argumentos, essa função retorna resultados como [arrays](https://br.tradingview.com/pine-script-reference/v5/#type_array), o que traz algumas diferenças na interpretação e manipulação, conforme explicado abaixo.

> __Observação!__\
> Assim como a [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security), a [request.security_lower_tf()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security_lower_tf) duplica os escopos e operações necessárias para calcular a `expression` de outro contexto. Os escopos de [request.security_lower_tf()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security_lower_tf) aumentam o consumo de memória em tempo de execução e contam para os limites de compilação do script. Consulte a seção [Contagem de Escopos](./06_05_limitacoes.md#contagem-de-escopos) da página de [Limitações](./06_05_limitacoes.md) para saber mais.

### Arrays de Dados Intrabar

Timeframes menores contêm mais pontos de dados que timeframes maiores, à medida que novos valores entram em uma _frequência maior_. Por exemplo, ao comparar um gráfico de 1 minuto com um gráfico de horas, o gráfico de 1 minuto terá até 60 vezes o número de barras por hora, dependendo dos dados disponíveis.

Para lidar com o fato de que múltiplas intrabarras existem dentro de uma barra do gráfico, a [request.security_lower_tf()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security_lower_tf) sempre retorna seus resultados como [arrays](./04_14_arrays.md). Os elementos nos [arrays](./04_14_arrays.md) retornados representam os valores da `expression` recuperados do timeframe menor, classificados em ordem ascendente com base no timestamp de cada intrabarra.

O [template de tipo](./04_09_tipagem_do_sistema.md#templates-de-tipo) atribuído aos [arrays](./04_14_arrays.md) retornados corresponde aos tipos de valores passados na chamada [request.security_lower_tf()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security_lower_tf). Por exemplo, usar um "int" como `expression` produzirá uma instância de `array<int>`, um "bool" como `expression` produzirá uma instância de `array<bool>`, etc.

O script a seguir usa informações intrabar para decompor as mudanças de preço de fechamento a fechamento do gráfico em partes positivas e negativas. Ele chama [request.security_lower_tf()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security_lower_tf) para buscar um [array](https://br.tradingview.com/pine-script-reference/v5/#type_array) de "float" de valores de [ta.change(close)](https://br.tradingview.com/pine-script-reference/v5/#fun_ta.change) do `lowerTimeframe` em cada barra do gráfico, em seguida, acessa todos os elementos do array usando um loop [for…in](https://br.tradingview.com/pine-script-reference/v5/#kw_for...in) para acumular as somas de `positiveChange` e `negativeChange`. O script adiciona os valores acumulados para calcular o `netChange`, depois [plota](./05_15_plots.md) os resultados no gráfico ao lado do `priceChange` para comparação:

![Arrays de dados intrabar](./imgs/Other-timeframes-and-data-Request-security-lower-tf-Intrabar-data-arrays-1.BFy5KmoZ_CQoK9.webp)

```c
//@version=5
indicator("Intrabar arrays demo", "Intrabar price changes")

//@variable The lower timeframe of the requested data.
string lowerTimeframe = input.timeframe("1", "Timeframe")

//@variable The close-to-close price change.
float priceChange = ta.change(close)

//@variable An array of `close` values from available intrabars on the `lowerTimeframe`.
array<float> intrabarChanges = request.security_lower_tf(syminfo.tickerid, lowerTimeframe, priceChange)

//@variable The total positive intrabar `close` movement on the chart bar.
float positiveChange = 0.0
//@variable The total negative intrabar `close` movement on the chart bar.
float negativeChange = 0.0

// Loop to calculate totals, starting from the chart bar's first available intrabar.
for change in intrabarChanges
    // Add the `change` to `positiveChange` if its sign is 1, and add to `negativeChange` if its sign is -1.
    switch math.sign(change)
        1  => positiveChange += change
        -1 => negativeChange += change

//@variable The sum of `positiveChange` and `negativeChange`. Equals the `priceChange` on bars with available intrabars.
float netChange = positiveChange + negativeChange

// Plot the `positiveChange`, `negativeChange`, and `netChange`.
plot(positiveChange, "Positive intrabar change", color.teal, style = plot.style_area)
plot(negativeChange, "Negative intrabar change", color.maroon, style = plot.style_area)
plot(netChange, "Net intrabar change", color.yellow, 5)
// Plot the `priceChange` to compare.
plot(priceChange, "Chart price change", color.orange, 2)
```

__Note que:__

- Os [plots](./05_15_plots.md) baseados em dados intrabar podem não aparecer em todas as barras de gráfico disponíveis, pois [request.security_lower_tf()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security_lower_tf) pode acessar até 100.000 intrabarras mais recentes disponíveis a partir do contexto solicitado. Quando esta função é executada em uma barra de gráfico que não possui dados intrabar acessíveis, ela retornará um _array vazio_.
- O número de intrabarras por barra de gráfico pode variar dependendo dos dados disponíveis no contexto e no gráfico em que o script é executado. Por exemplo, o feed de dados de 1 minuto de um provedor pode não incluir dados para cada minuto dentro do timeframe de 60 minutos devido à falta de atividade de negociação em alguns intervalos de 1 minuto. Para verificar o número de intrabarras recuperadas para uma barra de gráfico, pode-se usar [array.size()](https://br.tradingview.com/pine-script-reference/v5/#fun_array.size) no [array](https://br.tradingview.com/pine-script-reference/v5/#type_array) resultante.
- Se o valor de `lowerTimeframe` for maior que o timeframe do gráfico, o script gerará um _erro de execução_, pois não foi fornecido um argumento `ignore_invalid_timeframe` na chamada [request.security_lower_tf()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security_lower_tf).

### Tuplas de dados intrabar

Ao passar uma tupla ou uma chamada de função que retorna uma tupla como argumento `expression` em [request.security_lower_tf()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security_lower_tf), o resultado é uma tupla de [arrays](./04_14_arrays.md) com [templates de tipo](./04_09_tipagem_do_sistema.md#templates-de-tipo) correspondentes aos tipos dentro do argumento. Por exemplo, usar uma tupla `[float, string, color]` como `expression` resultará em dados `[array<float>, array<string>, array<color>]` retornados pela função. Usar uma tupla `expression` permite que um script busque vários [arrays](./04_14_arrays.md) de dados intrabar com uma única chamada de função [request.security_lower_tf()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security_lower_tf).

> __Observação!__\
> O tamanho combinado de todas as tuplas retornadas pelas chamadas `request.*()` em um script é limitado a 127 elementos. Consulte [esta seção](./06_05_limitacoes.md#limite-de-elementos-da-tupla) da página de [Limitações](./06_05_limitacoes.md) para mais informações.

O exemplo a seguir solicita dados OHLC de um timeframe menor e visualiza as intrabarras da barra atual no gráfico usando [linhas e caixas](./05_12_lines_e_boxes.md). O script chama [request.security_lower_tf()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security_lower_tf) com a tupla `[open, high, low, close]` como `expression` para recuperar uma tupla de [arrays](./04_14_arrays.md) representando informações OHLC de um `lowerTimeframe` calculado. Em seguida, usa um loop [for](https://br.tradingview.com/pine-script-reference/v5/#kw_for) para definir as coordenadas das linhas com os dados recuperados e índices de barras atuais para exibir os resultados ao lado da barra de gráfico atual, fornecendo uma "visão ampliada" do movimento de preço dentro do candle mais recente. Também desenha uma [caixa](./05_12_lines_e_boxes.md#boxes-caixas) ao redor das [linhas](./05_12_lines_e_boxes.md#lines) para indicar a região do gráfico ocupada pelos desenhos intrabar:

![Tuplas de dados intrabar](./imgs/Other-timeframes-and-data-Request-security-lower-tf-Tuples-of-intrabar-data-1.C8-f9Sez_Z96QYf.webp)

```c
//@version=5
indicator("Tuples of intrabar data demo", "Candle magnifier", max_lines_count = 500)

//@variable The maximum number of intrabars to display.
int maxIntrabars = input.int(20, "Max intrabars", 1, 250)
//@variable The width of the drawn candle bodies.
int candleWidth = input.int(20, "Candle width", 2)

//@variable The largest valid timeframe closest to `maxIntrabars` times smaller than the chart timeframe.
string lowerTimeframe = timeframe.from_seconds(math.ceil(timeframe.in_seconds() / maxIntrabars))

//@variable An array of lines to represent intrabar wicks.
var array<line> wicks  = array.new<line>()
//@variable An array of lines to represent intrabar bodies.
var array<line> bodies = array.new<line>()
//@variable A box that surrounds the displayed intrabars.
var box magnifierBox = box.new(na, na, na, na, bgcolor = na)

// Fill the `wicks` and `bodies` arrays with blank lines on the first bar.
if barstate.isfirst
    for i = 1 to maxIntrabars
        array.push(wicks, line.new(na, na, na, na, color = color.gray))
        array.push(bodies, line.new(na, na, na, na, width = candleWidth))

//@variable A tuple of "float" arrays containing `open`, `high`, `low`, and `close` prices from the `lowerTimeframe`.
[oData, hData, lData, cData] = request.security_lower_tf(syminfo.tickerid, lowerTimeframe, [open, high, low, close])
//@variable The number of intrabars retrieved from the `lowerTimeframe` on the chart bar.
int numIntrabars = array.size(oData)

if numIntrabars > 0
    // Define the start and end bar index values for intrabar display.
    int startIndex = bar_index + 2
    int endIndex = startIndex + numIntrabars
    // Loop to update lines.
    for i = 0 to maxIntrabars - 1
        line wickLine = array.get(wicks, i)
        line bodyLine = array.get(bodies, i)
        if i < numIntrabars
            //@variable The `bar_index` of the drawing.
            int candleIndex = startIndex + i
            // Update the properties of the `wickLine` and `bodyLine`.
            line.set_xy1(wickLine, startIndex + i, array.get(hData, i))
            line.set_xy2(wickLine, startIndex + i, array.get(lData, i))
            line.set_xy1(bodyLine, startIndex + i, array.get(oData, i))
            line.set_xy2(bodyLine, startIndex + i, array.get(cData, i))
            line.set_color(bodyLine, bodyLine.get_y2() > bodyLine.get_y1() ? color.teal : color.maroon)
            continue
        // Set the coordinates of the `wickLine` and `bodyLine` to `na` if no intrabar data is available at the index.
        line.set_xy1(wickLine, na, na)
        line.set_xy2(wickLine, na, na)
        line.set_xy1(bodyLine, na, na)
        line.set_xy2(bodyLine, na, na)
    // Set the coordinates of the `magnifierBox`.
    box.set_lefttop(magnifierBox, startIndex - 1, array.max(hData))
    box.set_rightbottom(magnifierBox, endIndex, array.min(lData))
```

__Note que:__

- O script desenha cada candle usando duas [linhas](./05_12_lines_e_boxes.md#lines-linhas): uma para representar as sombras e outra para representar o corpo. Como o script pode exibir até 500 linhas no gráfico, o valor de entrada `maxIntrabars` foi limitado a 250.
- O valor de `lowerTimeframe` é o resultado do cálculo de [math.ceil()](https://br.tradingview.com/pine-script-reference/v5/#fun_math.ceil) de [timeframe.in_seconds()](https://br.tradingview.com/pine-script-reference/v5/#fun_timeframe.in_seconds) dividido por `maxIntrabars` e convertido para uma [string de timeframe válida](./05_22_timeframes.md#especificações-de-string-do-timeframe) com [timeframe.from_seconds()](https://br.tradingview.com/pine-script-reference/v5/#fun_timeframe.from_seconds).
- O script define o topo do desenho da caixa usando o [array.max()](https://br.tradingview.com/pine-script-reference/v5/#fun_array.max) do array `hData` solicitado, e define a parte inferior da caixa usando o [array.min()](https://br.tradingview.com/pine-script-reference/v5/#fun_array.min) do array `lData` solicitado. Como visto no gráfico, esses valores correspondem ao [high](https://br.tradingview.com/pine-script-reference/v5/#var_high) e [low](https://br.tradingview.com/pine-script-reference/v5/#var_low) da barra do gráfico.


# Comportamento Histórico e Tempo Real


# Evitando Repintura

## Dados Higher-Timeframe (HTF) _Timeframe Superior_

## Dados Lower-Timeframe (LTF) _Timeframe Inferior_


# Contextos Personalizados

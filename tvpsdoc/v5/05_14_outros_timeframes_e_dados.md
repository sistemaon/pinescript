
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

> __Nota!__ Ao longo desta página, e em outras partes da documentação que discutem as funções `request.*()`, é comum o uso do termo _"context"_ para descrever o ID do ticker, timeframe e quaisquer modificações (ajustes de preço, configurações de sessão, tipos de gráficos não padronizados, etc.) que se aplicam a um gráfico ou aos dados recuperados por um script.

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

> __Nota!__ Também é possível permitir que scripts compatíveis avaliem seus escopos em outros contextos sem exigir funções `request.*()` usando o parâmetro `timeframe` da declaração [indicator()](https://br.tradingview.com/pine-script-reference/v5/#fun_indicator).


# Características Comuns

Muitas funções no namespace `request.*()` compartilham algumas propriedades e parâmetros comuns. Antes de explorar cada função em profundidade, é importante familiarizar-se com essas características.

## Uso

Todas as funções `request.*()` retornam resultados "series", o que significa que podem produzir valores diferentes em cada barra. No entanto, a maioria dos parâmetros das funções `request.*()` requer argumentos "const", "input" ou "simple".

Em essência, Pine Script deve determinar os valores da maioria dos argumentos passados para uma função `request.*()` na compilação do script ou na primeira barra do gráfico, dependendo do [tipo qualificado](./04_09_tipagem_do_sistema.md#qualificadores) que cada parâmetro aceita, e esses valores não podem mudar durante a execução do script. A única exceção é o parâmetro `expression` em [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security), [request.security_lower_tf()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security_lower_tf), e [request.seed()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.seed), que aceita argumentos "series".

Chamadas para funções `request.*()` são executadas em cada barra do gráfico, e scripts não podem desativá-las seletivamente durante sua execução. Scripts não podem chamar funções `request.*()` dentro dos escopos locais de [estruturas condicionais](./04_07_estruturas_condicionais.md), ou funções e métodos exportados por [bibliotecas](./05_11_libraries.md), mas podem usar essas chamadas de função dentro dos corpos de [funções definidas pelo usuário](./04_11_funcoes_definida_pelo_usuario.md) e [métodos](./04_13_metodos.md#métodos-definidos-pelo-usuário) não exportados.

Ao usar qualquer função `request.*()` dentro de um script, a performance de execução é uma consideração importante. Essas funções podem ter um impacto significativo na performance do script. Enquanto scripts podem conter um máximo de 40 chamadas para o namespace `request.*()`, é recomendável minimizar o número de chamadas nos scripts para manter o consumo de recursos o mais baixo possível. Para mais informações sobre as limitações dessas funções, veja [esta seção](./06_05_limitacoes.md#chamadas-request) da página do Manual do Usuário sobre as [limitações](./06_05_limitacoes.md) do Pine.

## `gaps`

Ao usar uma função `request.*()` para recuperar dados de outro contexto, os dados podem não ser recebidos em cada nova barra como seriam no gráfico atual. O parâmetro `gaps` de uma função `request.*()` permite controlar como a função responde a valores inexistentes na série solicitada.

> > __Nota!__ Ao usar a função [indicator()](https://br.tradingview.com/pine-script-reference/v5/#fun_indicator) para avaliar um script em outro contexto, o parâmetro `timeframe_gaps` especifica como ele lida com valores inexistentes. O parâmetro é similar ao parâmetro `gaps` para funções `request.*()`.

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
- Ao usar [barmerge.gaps_off](https://br.tradingview.com/pine-script-reference/v5/#var_barmerge.gaps_off), a função [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security) retorna o último fechamento confirmado ([close](https://br.tradingview.com/pine-script-reference/v5/#var_close)) do timeframe horário em todas as barras históricas. Ao rodar em barras em tempo real (as barras com o fundo [color.aqua](https://br.tradingview.com/pine-script-reference/v5/#var_color.aqua) neste exemplo), ela retorna o valor de fechamento atual do símbolo, independentemente da confirmação. Para mais informações, veja a seção [Comportamento histórico e em tempo real](./05_14_outros_timeframes_e_dados.md#comportamento-histórico-e-tempo-real) desta página.

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
- Nem todas as empresas emissoras publicam relatórios financeiros trimestrais. Se a empresa emissora do `symbol` não relatar trimestralmente, altere o valor "FQ" neste script para o período mínimo de relatório da empresa. Veja a seção [request.financial()](https://br.tradingview.com/pine-script-reference/v5/#fun_request{dot}financial) para mais informações.
- Foi utilizado [format.volume](https://br.tradingview.com/pine-script-reference/v5/#var_format.volume) nas chamadas [indicator()](https://br.tradingview.com/pine-script-reference/v5/#fun_indicator) e [str.tostring()](https://br.tradingview.com/pine-script-reference/v5/#fun_str.tostring), que especificam que o eixo y do painel do gráfico representa valores formatados como volume e a representação "string" do valor de `marketCap` aparece como texto formatado como volume.
- Este script cria uma [tabela](https://br.tradingview.com/pine-script-reference/v5/#type_table) e inicializa sua célula na [primeira barra do gráfico](https://br.tradingview.com/pine-script-reference/v5/#var_barstate.isfirst), depois [atualiza o texto da célula](https://br.tradingview.com/pine-script-reference/v5/#fun_table.cell_set_text) nas barras subsequentes. Para saber mais sobre como trabalhar com tabelas, veja a página [Tabelas](./05_19_tables.md) do Manual do Usuário.

## `currency`

O parâmetro `currency` de uma função `request.*()` permite especificar a moeda dos dados solicitados. Quando o valor deste parâmetro difere do [syminfo.currency](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo.currency) do contexto solicitado, a função converterá os valores solicitados para expressá-los na moeda especificada. Este parâmetro pode aceitar uma variável embutida do namespace `currency.*`, como [currency.JPY](https://br.tradingview.com/pine-script-reference/v5/#var_currency.JPY), ou uma "string" representando o [código de moeda ISO 4217](https://pt.wikipedia.org/wiki/ISO_4217) (por exemplo, "JPY").

A taxa de conversão entre o [syminfo.currency](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo.currency) dos dados solicitados e a `currency` especificada depende da correspondente taxa diária "*FX_IDC*" do dia anterior. Se nenhum instrumento disponível fornecer a taxa de conversão diretamente, a função usará o valor de um [símbolo de spread](https://br.tradingview.com/support/solutions/43000502298) para derivar a taxa.

> __Nota!__ Nem todas as chamadas de função `request.*()` retornam valores expressos como uma quantia em moeda. Portanto, a conversão de moeda não é sempre necessária. Por exemplo, algumas séries retornadas por [request.financial()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.financial) são expressas em unidades diferentes de moeda, como as métricas "PIOTROSKI_F_SCORE" e "NUMBER_OF_EMPLOYEES". Cabe aos programadores determinar quando a conversão de moeda é apropriada em suas solicitações de dados.

## `lookahead`

O parâmetro `lookahead` em [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security), [request.dividends()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.dividends), [request.splits()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.splits), e [request.earnings()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.earnings) especifica o comportamento do lookahead da chamada da função. Seu valor padrão é [barmerge.lookahead_off](https://br.tradingview.com/pine-script-reference/v5/#var_barmerge.lookahead_off).

Ao solicitar dados de um contexto do timeframe superior _higher-timeframe (HTF)_, o valor `lookahead` determina se a função pode solicitar valores de tempos _além_ daqueles das barras históricas em que é executada. Em outras palavras, o valor `lookahead` determina se os dados solicitados podem conter _viés de lookahead_ em barras históricas.

Ao solicitar dados de um contexto de timeframe inferior _lower-timeframe (LTF)_, o parâmetro `lookahead` determina se a função solicita valores da primeira ou última barra _intrabar_ (LTF) em cada barra do gráfico.

__Os programadores devem ter extrema cautela ao usar lookahead em seus scripts, especialmente ao solicitar dados de timeframes superiores.__ Ao usar [barmerge.lookahead_on](https://br.tradingview.com/pine-script-reference/v5/#var_barmerge.lookahead_on) como valor `lookahead`, é necessário garantir que isso não comprometa a integridade da lógica do script, vazando dados _futuros_ em barras históricas do gráfico.

Os seguintes cenários são casos em que habilitar lookahead é aceitável em uma chamada `request.*()`:

- A `expression` em [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security) referencia uma série com um "_deslocamento histórico_" ("_historical offset_") (por exemplo, `close[1]`), o que impede a função de solicitar valores futuros que não teria acesso em tempo real.
- O `timeframe` especificado na chamada é o mesmo do gráfico em que o script é executado, ou seja, [timeframe.period](https://br.tradingview.com/pine-script-reference/v5/#var_timeframe.period).
- A chamada da função solicita dados de um timeframe intrabar, ou seja, um timeframe menor que o [timeframe.period](https://br.tradingview.com/pine-script-reference/v5/#var_timeframe.period). Veja [esta seção](./05_14_outros_timeframes_e_dados.md#lower-timeframe-ltf-timeframe-inferior) para mais informações.

> __Nota!__ Usar [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security) para vazar dados futuros para o passado é __enganoso__ e __não permitido__ em publicações de scripts. Embora os resultados do seu script em barras históricas possam parecer ótimos devido à sua aquisição aparentemente "mágico" de presciência (que não será capaz de reproduzir em barras em tempo real), isso enganará você e os usuários do seu script. Se [publicar seu script](./06_04_publicando_scripts.md) para compartilhá-lo com outros, certifique-se de __não enganar os usuários__ acessando informações futuras em barras históricas.

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

- A série solicitada usando [barmerge.lookahead_off](https://br.tradingview.com/pine-script-reference/v5/#var_barmerge.lookahead_off) tem um novo valor histórico no _final_ de cada período HTF, e ambas as séries solicitadas usando [barmerge.lookahead_on](https://br.tradingview.com/pine-script-reference/v5/#var_barmerge.lookahead_on) têm novos dados históricos no _início_ de cada período.
- Em barras em tempo real, o plot da série sem lookahead ([blue](https://br.tradingview.com/pine-script-reference/v5/#var_color.blue)) e a série com lookahead e sem deslocamento histórico ([fuchsia](https://br.tradingview.com/pine-script-reference/v5/#var_color.fuchsia)) mostram o _mesmo valor_ (ou seja, o preço máximo não confirmado do período HTF), pois não existem dados além desses pontos para vazar para o passado. Ambos esses plots __*repintarão*__ seus resultados após reiniciar a execução do script, pois as barras em [tempo real](https://br.tradingview.com/pine-script-reference/v5/#var_barstate.isrealtime) se tornarão barras [históricas](https://br.tradingview.com/pine-script-reference/v5/#var_barstate.ishistory).
- A série que usa lookahead e um deslocamento histórico ([aqua](https://br.tradingview.com/pine-script-reference/v5/#var_color.aqua)) não repinta seus valores, pois sempre referencia o último valor _confirmado_ do timeframe superior. Veja a seção [Evitando Repintura](./05_14_outros_timeframes_e_dados.md#evitando-repintura) desta página para mais informações.

> __Nota!__ No Pine Script v1 e v2, a função `security()` não incluía um parâmetro `lookahead`, mas se comportava como nas versões posteriores do Pine com `lookahead = barmerge.lookahead_on`, o que significa que sistematicamente usava dados do contexto HTF futuro em barras históricas. Portanto, deve-se _ter cautela_ com scripts do Pine v1 ou v2 que usam chamadas `security()` HTF, a menos que as chamadas da função contenham deslocamentos históricos.


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

> __Nota!__ Como regra, o TradingView _não_ gera dados; ele depende de seus provedores de dados para as informações exibidas nos gráficos e acessadas por scripts.

Ao usar feeds de dados solicitados de outros contextos, também é crucial considerar as diferenças de _eixo de tempo_ entre o gráfico em que o script é executado e os feeds solicitados, pois as funções `request.*()` adaptam a série retornada ao eixo de tempo do gráfico. Por exemplo, solicitar dados "BTCUSD" no gráfico "SPY" com [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security) mostrará novos valores apenas quando o gráfico "SPY" tiver novos dados também. Como "SPY" não é um símbolo 24 horas, os dados "BTCUSD" retornados conterão lacunas que não estão presentes ao visualizar seu gráfico diretamente.


# Comportamento Histórico e Tempo Real


# Timeframes

# Evitando Repintura

## Higher-Timeframe (HTF) _Timeframe Superior_

## Lower-Timeframe (LTF) _Timeframe Inferior_


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

<!-- ## `gaps`

Ao usar uma função `request.*()` para recuperar dados de outro contexto, os dados podem não ser recebidos em cada nova barra como seriam no gráfico atual. O parâmetro `gaps` de uma função `request.*()` permite controlar como a função responde a valores inexistentes na série solicitada.

> __Nota!__ Ao usar a função [indicator()](https://br.tradingview.com/pine-script-reference/v5/#fun_indicator) para avaliar um script em outro contexto, o parâmetro `timeframe_gaps` especifica como ele lida com valores inexistentes. O parâmetro é similar ao parâmetro `gaps` para funções `request.*()`.

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
- Este script cria uma [tabela](https://br.tradingview.com/pine-script-reference/v5/#type_table) e inicializa sua célula na [primeira barra do gráfico](https://br.tradingview.com/pine-script-reference/v5/#var_barstate.isfirst), depois [atualiza o texto da célula](https://br.tradingview.com/pine-script-reference/v5/#fun_table.cell_set_text) nas barras subsequentes. Para saber mais sobre como trabalhar com tabelas, veja a página [Tabelas](./05_19_tables.md) do Manual do Usuário. -->













# Comportamento Histórico e Tempo Real

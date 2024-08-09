
# Informações do Gráfico

A maneira como os scripts podem obter informações sobre o gráfico e o símbolo em que estão sendo executados é por meio de um subconjunto de [variáveis incorporadas](./04_10_incorporados.md#variáveis-incorporadas) do Pine Script.

As variáveis abordadas aqui permitem que os scripts acessem informações relacionadas a:

- Os preços e o volume do gráfico.
- O símbolo do gráfico.
- O _timeframe_ do gráfico.
- A sessão (ou período de tempo) em que o símbolo é negociado.


# Preços e Volume

As variáveis embutidas para valores __OHLCV__ são:

- [open](https://br.tradingview.com/pine-script-reference/v5/#var_open): o preço de _abertura_ da barra.
- [high](https://br.tradingview.com/pine-script-reference/v5/#var_high): o preço _mais alto_ da barra, ou o preço _mais alto alcançado_ durante o tempo decorrido da barra em tempo real.
- [low](https://br.tradingview.com/pine-script-reference/v5/#var_low): o preço _mais baixo_ da barra, ou o preço _mais baixo alcançado_ durante o tempo decorrido da barra em tempo real.
- [close](https://br.tradingview.com/pine-script-reference/v5/#var_close): o preço de _fechamento_ da barra, ou o __preço atual__ na barra em tempo real.
- [volume](https://br.tradingview.com/pine-script-reference/v5/#var_volume): o _volume negociado_ durante a barra, ou o _volume negociado durante_ o tempo decorrido da barra em tempo real. A unidade de informação de volume varia com o instrumento. É em ações para ações, em lotes para forex, em contratos para futuros, em moeda base para cripto, etc.

Outros valores estão disponíveis através de:

- [hl2](https://br.tradingview.com/pine-script-reference/v5/#var_hl2): a média dos valores de [high](https://br.tradingview.com/pine-script-reference/v5/#var_high) e [low](https://br.tradingview.com/pine-script-reference/v5/#var_low) da barra.
- [hlc3](https://br.tradingview.com/pine-script-reference/v5/#var_hl2): a média dos valores de [high](https://br.tradingview.com/pine-script-reference/v5/#var_high), [low](https://br.tradingview.com/pine-script-reference/v5/#var_low) e [close](https://br.tradingview.com/pine-script-reference/v5/#var_close) da barra.
- [ohlc4](https://br.tradingview.com/pine-script-reference/v5/#var_hlc3): a média dos valores de [open](https://br.tradingview.com/pine-script-reference/v5/#var_open), [high](https://br.tradingview.com/pine-script-reference/v5/#var_high), [low](https://br.tradingview.com/pine-script-reference/v5/#var_low) e [close](https://br.tradingview.com/pine-script-reference/v5/#var_close) da barra.

Em barras históricas, os valores das variáveis acima não variam durante a barra porque apenas informações OHLCV estão disponíveis nelas. Quando executados em barras históricas, os scripts são executados no [close](https://br.tradingview.com/pine-script-reference/v5/#var_close) da barra, quando todas as informações da barra são conhecidas e não podem mudar durante a execução do script na barra.

Barras em tempo real são outra história. Quando indicadores (ou estratégias usando `calc_on_every_tick = true`) são executados em tempo real, os valores das variáveis acima (exceto [open](https://br.tradingview.com/pine-script-reference/v5/#var_open)) variarão entre iterações sucessivas do script na barra em tempo real, porque representam seu valor __atual__ em um determinado momento durante o progresso da barra em tempo real. Isso pode levar a uma forma de [repintura](./05_16_repintura.md). Veja a página sobre o [modelo de execução](./04_01_modelo_de_execucao.md) do Pine Script para mais detalhes.

O [operador de referência histórica](./04_05_operadores.md#operador-de-referência-histórica-) [[]](https://br.tradingview.com/pine-script-reference/v5/#op_[]) pode ser usado para se referir a valores passados das variáveis incorporadas, por exemplo, `close[1]` refere-se ao valor de [close](https://br.tradingview.com/pine-script-reference/v5/#var_close) na barra anterior, em relação à barra específica em que o script está sendo executado.


# Informações do Símbolo

Variáveis embutidas no _namespace_ `syminfo` fornecem aos scripts informações sobre o símbolo do gráfico em que o script está sendo executado. Essas informações mudam toda vez que o usuário do script altera o símbolo do gráfico. O script, então, é reexecutado em todas as barras do gráfico usando os novos valores das variáveis embutidas:

- [syminfo.basecurrency](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo{dot}basecurrency): a moeda base, por exemplo, "BTC" em "BTCUSD", ou "EUR" em "EURUSD".
- [syminfo.currency](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo{dot}currency): a moeda de cotação, por exemplo, "USD" em "BTCUSD", ou "CAD" em "USDCAD".
- [syminfo.description](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo{dot}description): a descrição longa do símbolo.
- [syminfo.mintick](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo{dot}mintick): o valor do tick do símbolo, ou o incremento mínimo que o preço pode mover. Não deve ser confundido com _pips_ ou _pontos_. Em "ES1!" ("S&P 500 E-Mini"), o tamanho do tick é 0.25 porque esse é o incremento mínimo em que o preço se move.
- [syminfo.pointvalue](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo{dot}pointvalue): o valor do ponto é o múltiplo do ativo subjacente que determina o valor de um contrato. Em "ES1!" ("S&P 500 E-Mini"), o valor do ponto é 50, então um contrato vale 50 vezes o preço do instrumento.
- [syminfo.prefix](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo{dot}prefix): O prefixo é o identificador da bolsa ou corretora: "NASDAQ" ou "BATS" para "AAPL", "CME_MINI_DL" para "ES1!".
- [syminfo.root](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo{dot}root): É o prefixo do ticker para tickers estruturados como os de futuros. É "ES" para "ES1!", "ZW" para "ZW1!".
- [syminfo.session](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo{dot}session): Reflete a configuração de sessão no gráfico para esse símbolo. Se o campo "_Chart settings/Symbol/Session_" ("_Configurações do Gráfico/Símbolo/Sessão_") estiver definido como "_Extended_" ("_Estendido_"), ele retornará "_extended_" apenas se o símbolo e o feed do usuário permitirem sessões estendidas. É raramente exibido e usado principalmente como argumento para o parâmetro de `session` em [ticker.new()](https://br.tradingview.com/pine-script-reference/v5/#fun_ticker{dot}new).
- [syminfo.ticker](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo{dot}ticker): É o nome do símbolo, sem a parte da bolsa ([syminfo.prefix](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo{dot}prefix)): "BTCUSD", "AAPL", "ES1!", "USDCAD".
- [syminfo.tickerid](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo{dot}tickerid): Esta string é raramente exibida. É usada principalmente como argumento para o parâmetro `symbol` de [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request{dot}security). Inclui informações de sessão, prefixo e ticker.
- [syminfo.timezone](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo{dot}timezone): O fuso horário em que o símbolo é negociado. A string é um nome do [banco de dados de fuso horário IANA](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones) (por exemplo, "America/New_York").
- [syminfo.type](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo{dot}type): O tipo de mercado ao qual o símbolo pertence. Os valores são "stock", "futures", "index", "forex", "crypto", "fund", "dr", "cfd", "bond", "warrant", "structured" e "right".

Este script exibirá os valores dessas variáveis embutidas no gráfico:

```c
//@version=5
indicator("`syminfo.*` built-ins", "", true)
printTable(txtLeft, txtRight) =>
    var table t = table.new(position.middle_right, 2, 1)
    table.cell(t, 0, 0, txtLeft, bgcolor = color.yellow, text_halign = text.align_right)
    table.cell(t, 1, 0, txtRight, bgcolor = color.yellow, text_halign = text.align_left)

nl = "\n"
left =
  "syminfo.basecurrency: "  + nl +
  "syminfo.currency: "      + nl +
  "syminfo.description: "   + nl +
  "syminfo.mintick: "       + nl +
  "syminfo.pointvalue: "    + nl +
  "syminfo.prefix: "        + nl +
  "syminfo.root: "          + nl +
  "syminfo.session: "       + nl +
  "syminfo.ticker: "        + nl +
  "syminfo.tickerid: "      + nl +
  "syminfo.timezone: "      + nl +
  "syminfo.type: "

right =
  syminfo.basecurrency              + nl +
  syminfo.currency                  + nl +
  syminfo.description               + nl +
  str.tostring(syminfo.mintick)     + nl +
  str.tostring(syminfo.pointvalue)  + nl +
  syminfo.prefix                    + nl +
  syminfo.root                      + nl +
  syminfo.session                   + nl +
  syminfo.ticker                    + nl +
  syminfo.tickerid                  + nl +
  syminfo.timezone                  + nl +
  syminfo.type

printTable(left, right)
```


# Timeframe do Gráfico

Um script pode obter informações sobre o tipo de _timeframe_ usado no gráfico usando estas variáveis embutidas, que todas retornam um "simple bool":

- [timeframe.isseconds](https://br.tradingview.com/pine-script-reference/v5/#var_timeframe{dot}isseconds)
- [timeframe.isminutes](https://br.tradingview.com/pine-script-reference/v5/#var_timeframe{dot}isminutes)
- [timeframe.isintraday](https://br.tradingview.com/pine-script-reference/v5/#var_timeframe{dot}isintraday)
- [timeframe.isdaily](https://br.tradingview.com/pine-script-reference/v5/#var_timeframe{dot}isdaily)
- [timeframe.isweekly](https://br.tradingview.com/pine-script-reference/v5/#var_timeframe{dot}isweekly)
- [timeframe.ismonthly](https://br.tradingview.com/pine-script-reference/v5/#var_timeframe{dot}ismonthly)
- [timeframe.isdwm](https://br.tradingview.com/pine-script-reference/v5/#var_timeframe{dot}isdwm)

Duas variáveis embutidas adicionais retornam informações mais específicas sobre o timeframe:

- [timeframe.multiplier](https://br.tradingview.com/pine-script-reference/v5/#var_timeframe{dot}multiplier) retorna um "simple int" contendo o multiplicador da unidade de timeframe. Um gráfico com timeframe de _uma hora_ retornará `60` porque os timeframes intradiários são expressos em minutos. Um timeframe de _30sec_ retornará `30` (segundos), um gráfico diário retornará `1` (dia), um gráfico trimestral retornará `3` (meses) e um gráfico anual retornará `12` (meses). O valor dessa variável não pode ser usado como argumento para parâmetros de `timeframe` em funções incorporadas, pois elas esperam uma string no formato de especificações de timeframe.
- [timeframe.period](https://br.tradingview.com/pine-script-reference/v5/#var_timeframe{dot}period) retorna uma string no formato de especificação de timeframe do Pine Script.

Veja a página sobre [Timeframes](./05_22_timeframes.md) para mais informações.


# Informações de Sessão

As informações de sessão estão disponíveis em diferentes formas:

A variável embutida [syminfo.session](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo{dot}session) retorna um valor que é [session.regular](https://br.tradingview.com/pine-script-reference/v5/#const_session{dot}regular) ou [session.extended](https://br.tradingview.com/pine-script-reference/v5/#const_session{dot}extended). Reflete a configuração de sessão no gráfico para esse símbolo. Se o campo "_Chart settings/Symbol/Session_" ("_Configurações do Gráfico/Símbolo/Sessão_") estiver definido como "_Extended_" ("_Estendido_"), ele retornará "extended" apenas se o símbolo e o feed do usuário permitirem sessões estendidas. É usada quando um tipo de sessão é esperado, por exemplo, como argumento para o parâmetro de sessão em [ticker.new()](https://br.tradingview.com/pine-script-reference/v5/#fun_ticker{dot}new).

[Variáveis embutidas de estado de sessão](./05_17_sessoes.md#estados-da-sessão) fornecem informações sobre a sessão de negociação à qual uma barra pertence.

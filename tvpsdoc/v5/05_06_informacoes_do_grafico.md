
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

O [operador de referência histórica](./04_05_operadores.md#operador-de-referência-histórica-) [[]](https://br.tradingview.com/pine-script-reference/v5/#op_[]) pode ser usado para se referir a valores passados das variáveis incorporadas, por exemplo, `close[1]` refere-se ao valor de [close](https://www.tradingview.com/pine-script-reference/v5/#var_close) na barra anterior, em relação à barra específica em que o script está sendo executado.

<!-- Informações do símbolo

Variáveis embutidas no namespace `syminfo` fornecem aos scripts informações sobre o símbolo do gráfico em que o script está sendo executado. Essas informações mudam toda vez que o usuário do script altera o símbolo do gráfico. O script, então, é reexecutado em todas as barras do gráfico usando os novos valores das variáveis embutidas:

- `syminfo.basecurrency`: a moeda base, por exemplo, “BTC” em “BTCUSD”, ou “EUR” em “EURUSD”.
- `syminfo.currency`: a moeda de cotação, por exemplo, “USD” em “BTCUSD”, ou “CAD” em “USDCAD”.
- `syminfo.description`: a descrição longa do símbolo.
- `syminfo.mintick`: o valor do tick do símbolo, ou o incremento mínimo que o preço pode mover. Não deve ser confundido com pips ou pontos. Em “ES1!” (“S&P 500 E-Mini”), o tamanho do tick é 0.25 porque esse é o incremento mínimo em que o preço se move.
- `syminfo.pointvalue`: o valor do ponto é o múltiplo do ativo subjacente que determina o valor de um contrato. Em “ES1!” (“S&P 500 E-Mini”), o valor do ponto é 50, então um contrato vale 50 vezes o preço do instrumento. -->
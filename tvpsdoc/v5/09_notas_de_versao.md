
# Notas de Versão

Esta página contém notas de versão descrevendo mudanças notáveis na experiência do Pine Script.

## 2024

### Agosto 2024

As funções [ticker.new()](https://br.tradingview.com/pine-script-reference/v5/#fun_ticker.new) e [ticker.modify()](https://br.tradingview.com/pine-script-reference/v5/#fun_ticker.modify) possuem dois novos parâmetros: `settlement_as_close` e `backadjustment`. Os usuários podem especificar se esses parâmetros estão ativados, desativados ou configurados para herdar as configurações padrão do símbolo. Essas configurações afetam apenas os dados de símbolos futuros com essas opções disponíveis em seus gráficos. Não têm efeito em outros símbolos.

- O parâmetro `backadjustment` especifica se os dados de contratos passados em símbolos futuros contínuos são ajustados retroativamente. Seus valores possíveis são: [backadjustment.on](https://br.tradingview.com/pine-script-reference/v5/#const_backadjustment.on), [backadjustment.off](https://br.tradingview.com/pine-script-reference/v5/#const_backadjustment.off), ou [backadjustment.inherit](https://br.tradingview.com/pine-script-reference/v5/#const_backadjustment.inherit).
- O parâmetro `settlement_as_close` especifica se o valor de [close](https://br.tradingview.com/pine-script-reference/v5/#var_close) de um símbolo futuro representa o preço real de fechamento ou o preço de liquidação em timeframes de "1D" e superiores. Seus valores possíveis são: [settlement_as_close.on](https://br.tradingview.com/pine-script-reference/v5/#const_settlement_as_close.on), [settlement_as_close.off](https://br.tradingview.com/pine-script-reference/v5/#const_settlement_as_close.off), ou [settlement_as_close.inherit](https://br.tradingview.com/pine-script-reference/v5/#const_settlement_as_close.inherit).

### Junho 2024

Foi adicionado um novo parâmetro às funções [box.new()](https://br.tradingview.com/pine-script-reference/v5/#fun_box.new), [label.new()](https://br.tradingview.com/pine-script-reference/v5/#fun_label.new), [line.new()](https://br.tradingview.com/pine-script-reference/v5/#fun_line.new), [polyline.new()](https://br.tradingview.com/pine-script-reference/v5/#fun_polyline.new) e [table.new()](https://br.tradingview.com/pine-script-reference/v5/#fun_table.new):

- `force_overlay` Se verdadeiro, o desenho será exibido no painel principal do gráfico, mesmo quando o script ocupar um painel separado. Opcional. O padrão é falso.

#### Enums do Pine Script

Enums, também conhecidos como _enumerations_, _enumerated types_ ou [tipos enum](./04_09_tipagem_do_sistema.md#tipos-enum), são tipos de dados exclusivos com todos os valores possíveis declarados pelo programador. Eles podem ajudar os programadores a manter um controle mais rigoroso sobre os valores permitidos por variáveis, expressões condicionais e [coleções](./04_09_tipagem_do_sistema.md#coleções), e permitem a criação conveniente de entradas suspensas com a nova função [input.enum()](./05_09_inputs.md#input-enum). Veja a página [Enums](./04_17_enums.md) no Manual do Usuário para saber mais sobre esses novos tipos e como usá-los.

### Maio 2024

Foi adicionado um parâmetro opcional `calc_bars_count` às funções [indicator()](https://br.tradingview.com/pine-script-reference/v5/#fun_indicator), [strategy()](https://br.tradingview.com/pine-script-reference/v5/#fun_strategy), [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security), [request.security_lower_tf()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security_lower_tf) e [request.seed()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.seed) que permite aos usuários limitar o número de barras históricas recentes que um script ou solicitação de dados pode executar. Quando uma declaração de [indicator()](https://br.tradingview.com/pine-script-reference/v5/#fun_indicator) ou [strategy()](https://br.tradingview.com/pine-script-reference/v5/#fun_strategy) de um script inclui um argumento `calc_bars_count`, sua aba "Configurações/Entradas" incluirá uma entrada "Barras Calculadas" na seção "Cálculo". O valor padrão em todas essas funções é 0, o que significa que o script ou solicitação executa em todos os dados disponíveis.

O namespace `strategy.*` possui várias novas variáveis internas:

- [strategy.avg_trade](https://br.tradingview.com/pine-script-reference/v5/#var_strategy.avg_trade) Retorna a quantidade média de dinheiro ganho ou perdido por negociação. Calculado como a soma de todos os lucros e perdas dividida pelo número de negociações fechadas.
- [strategy.avg_trade_percent](https://br.tradingview.com/pine-script-reference/v5/#var_strategy.avg_trade_percent) Retorna o ganho ou perda percentual médio por negociação. Calculado como a soma de todos os percentuais de lucro e perda dividida pelo número de negociações fechadas.
- [strategy.avg_winning_trade](https://br.tradingview.com/pine-script-reference/v5/#var_strategy.avg_winning_trade) Retorna a quantidade média de dinheiro ganho por negociação vencedora. Calculado como a soma dos lucros dividida pelo número de negociações vencedoras.
- [strategy.avg_winning_trade_percent](https://br.tradingview.com/pine-script-reference/v5/#var_strategy.avg_winning_trade_percent) Retorna o ganho percentual médio por negociação vencedora. Calculado como a soma dos percentuais de lucro dividida pelo número de negociações vencedoras.
- [strategy.avg_losing_trade](https://br.tradingview.com/pine-script-reference/v5/#var_strategy.avg_losing_trade) Retorna a quantidade média de dinheiro perdida por negociação perdedora. Calculado como a soma das perdas dividida pelo número de negociações perdedoras.
- [strategy.avg_losing_trade_percent](https://br.tradingview.com/pine-script-reference/v5/#var_strategy.avg_losing_trade_percent) Retorna a perda percentual média por negociação perdedora. Calculado como a soma dos percentuais de perda dividida pelo número de negociações perdedoras.

#### Pine Profiler

O novo [Pine Profiler](./06_03_perfilamento_e_otimizacao.md#pine-profiler) é uma poderosa utilidade que analisa as execuções de todo o código significativo em um script e exibe informações úteis de desempenho ao lado das linhas de código _dentro_ do Pine Editor. As informações do [Profiler](./06_03_perfilamento_e_otimizacao.md#pine-profiler) fornecem insights sobre o tempo de execução de um script, a distribuição do tempo de execução em regiões de código significativas e o número de vezes que cada região de código é executada. Com esses insights, os programadores podem identificar efetivamente os _gargalos_ de desempenho e garantir que se concentrem em [otimizar](./06_03_perfilamento_e_otimizacao.md#otimização) seu código onde realmente importa quando precisam melhorar os tempos de execução.

Consulte a nova página [Perfilamento e Otimização](./06_03_perfilamento_e_otimizacao.md) para saber mais sobre o Profiler, como ele funciona e como usá-lo para analisar o desempenho de um script e identificar oportunidades de otimização.

#### Melhorias no Pine Editor

Ao abrir o Pine Editor destacado a partir de uma aba com um gráfico, ele agora se vincula diretamente a essa aba, conforme indicado pelo status "Vinculado" "_Linked_" e pelo ícone verde no canto inferior direito. Enquanto estiver vinculado, os botões "Adicionar ao gráfico" "_Add to chart_", "Atualizar no gráfico" "_Update on chart_" e "Aplicar ao layout inteiro" "_Apply to entire layout_" afetam os gráficos na aba principal.

O Pine Editor destacado agora inclui o console do Pine.

### Abril 2024

Foi adicionado um novo parâmetro às funções [plot()](https://br.tradingview.com/pine-script-reference/v5/#fun_plot), [plotchar()](https://br.tradingview.com/pine-script-reference/v5/#fun_plotchar), [plotcandle()](https://br.tradingview.com/pine-script-reference/v5/#fun_plotcandle), [plotbar()](https://br.tradingview.com/pine-script-reference/v5/#fun_plotbar), [plotarrow()](https://br.tradingview.com/pine-script-reference/v5/#fun_plotarrow), [plotshape()](https://br.tradingview.com/pine-script-reference/v5/#fun_plotshape) e [bgcolor()](https://br.tradingview.com/pine-script-reference/v5/#fun_bgcolor):

- `force_overlay` Se verdadeiro, a saída será exibida no painel principal do gráfico, mesmo quando o script ocupar um painel separado.

### Março 2024

O namespace `syminfo.*` possui uma nova variável interna:

- [syminfo.expiration_date](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo.expiration_date) Em símbolos futuros não contínuos, retorna um timestamp UNIX representando o início do último dia do contrato atual.

As funções [time()](https://br.tradingview.com/pine-script-reference/v5/#fun_time) e [time_close()](https://br.tradingview.com/pine-script-reference/v5/#fun_time_close) têm um novo parâmetro:

- `bars_back` Se especificado, a função calculará o timestamp a partir da barra N barras atrás em relação à barra atual no seu timeframe. Também pode calcular o tempo esperado de uma barra futura até 500 barras à frente se o argumento for um valor negativo. Opcional. O padrão é 0.

### Fevereiro 2024

Foi adicionado duas novas funções para trabalhar com strings:

- [str.repeat()](https://br.tradingview.com/pine-script-reference/v5/#fun_str.repeat) Constrói uma nova string contendo a string fonte repetida um número especificado de vezes com um separador injetado entre cada instância repetida.
- [str.trim()](https://br.tradingview.com/pine-script-reference/v5/#fun_str.trim) Constrói uma nova string com todos os espaços em branco consecutivos e outros caracteres de controle removidos da esquerda e direita da string fonte.

A função [request.financial()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.financial) agora aceita "D" como argumento `period`, permitindo que scripts solicitem dados financeiros diários disponíveis.

Por exemplo:

```c
//@version=5
indicator("Daily financial data demo")

//@variable The daily Premium/Discount to Net Asset Value for "AMEX:SPY"
float f1 = request.financial("AMEX:SPY", "NAV", "D")
plot(f1)
```

O namespace `strategy.*` possui uma nova variável para monitorar o capital disponível na simulação de uma estratégia:

- [strategy.opentrades.capital_held](https://br.tradingview.com/pine-script-reference/v5/#var_strategy.opentrades.capital_held) Retorna a quantidade de capital atualmente mantida por negociações abertas.

### Janeiro 2024

O namespace `syminfo.*` possui novas variáveis internas:

Syminfo:

- [syminfo.employees](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo.employees) O número de funcionários que a empresa possui.
- [syminfo.shareholders](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo.shareholders) O número de acionistas que a empresa possui.
- [syminfo.shares_outstanding_float](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo.shares_outstanding_float) O número total de ações em circulação que uma empresa tem disponível, excluindo suas ações restritas.
- [syminfo.shares_outstanding_total](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo.shares_outstanding_total) O número total de ações em circulação que uma empresa tem disponível, incluindo ações restritas mantidas por insiders, grandes acionistas e funcionários.

Preço alvo:

- [syminfo.target_price_average](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo.target_price_average) A média dos últimos preços-alvo anuais para o símbolo previstos por analistas.
- [syminfo.target_price_date](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo.target_price_date) A data de início da última previsão de preço-alvo para o símbolo atual.
- [syminfo.target_price_estimates](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo.target_price_estimates) O número total mais recente de previsões de preço-alvo para o símbolo atual.
- [syminfo.target_price_high](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo.target_price_high) O último maior preço-alvo anual para o símbolo previsto por analistas.
- [syminfo.target_price_low](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo.target_price_low) O último menor preço-alvo anual para o símbolo previsto por analistas.
- [syminfo.target_price_median](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo.target_price_median) A mediana dos últimos preços-alvo anuais para o símbolo previstos por analistas.

Recomendações:

- [syminfo.recommendations_buy](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo.recommendations_buy) O número de analistas que deram ao símbolo atual uma classificação de "Compra".
- [syminfo.recommendations_buy_strong](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo.recommendations_buy_strong) O número de analistas que deram ao símbolo atual uma classificação de "Compra Forte" "_Strong Buy_".
- [syminfo.recommendations_date](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo.recommendations_date) A data de início do último conjunto de recomendações para o símbolo atual.
- [syminfo.recommendations_hold](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo.recommendations_hold) O número de analistas que deram ao símbolo atual uma classificação de "Manter".
- [syminfo.recommendations_total](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo.recommendations_total) O número total de recomendações para o símbolo atual.
- [syminfo.recommendations_sell](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo.recommendations_sell) O número de analistas que deram ao símbolo atual uma classificação de "Venda".
- [syminfo.recommendations_sell_strong](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo.recommendations_sell_strong) O número de analistas que deram ao símbolo atual uma classificação de "Venda Forte" "_Strong Sell_".

## 2023

### Dezembro 2023

Adicionados parâmetros `format` e `precision` a todas as funções `plot*()`, permitindo que indicadores e estratégias apliquem seletivamente configurações de formatação e precisão decimal aos resultados plotados no eixo y do painel do gráfico, na linha de status do script e na Janela de Dados. Os argumentos passados para esses parâmetros substituem os valores nas funções [indicator()](https://br.tradingview.com/pine-script-reference/v5/#fun_indicator) e [strategy()](https://br.tradingview.com/pine-script-reference/v5/#fun_strategy). Ambos opcionais. Os padrões para esses parâmetros são os mesmos dos valores especificados na declaração do script.

Por exemplo:

```c
//@version=5
indicator("My script", format = format.percent, precision = 4)

plot(close, format = format.price)           // Price format with 4-digit precision.
plot(100 * bar_index / close, precision = 2) // Percent format with 2-digit precision.
```

### Novembro 2023

Adicionadas as seguintes variáveis e funções ao namespace `strategy.*`:

- [strategy.grossloss_percent](https://br.tradingview.com/pine-script-reference/v5/#var_strategy.grossloss_percent) O valor total da perda bruta de todas as negociações perdedoras concluídas, expresso como uma porcentagem do capital inicial.
- [strategy.grossprofit_percent](https://br.tradingview.com/pine-script-reference/v5/#var_strategy.grossprofit_percent) O valor total do lucro bruto de todas as negociações vencedoras concluídas, expresso como uma porcentagem do capital inicial.
- [strategy.max_runup_percent](https://br.tradingview.com/pine-script-reference/v5/#var_strategy.max_runup_percent) O aumento máximo de um vale na curva de capital, expresso como uma porcentagem do valor do vale.
- [strategy.max_drawdown_percent](https://br.tradingview.com/pine-script-reference/v5/#var_strategy.max_drawdown_percent) A queda máxima de um pico na curva de capital, expressa como uma porcentagem do valor do pico.
- [strategy.netprofit_percent](https://br.tradingview.com/pine-script-reference/v5/#var_strategy.netprofit_percent) O valor total de todas as negociações concluídas, expresso como uma porcentagem do capital inicial.
- [strategy.openprofit_percent](https://br.tradingview.com/pine-script-reference/v5/#var_strategy.openprofit_percent) O lucro ou prejuízo não realizado atual para todas as posições abertas, expresso como uma porcentagem do patrimônio realizado.
- [strategy.closedtrades.max_drawdown_percent()](https://br.tradingview.com/pine-script-reference/v5/#fun_strategy.closedtrades.max_drawdown_percent) Retorna o rebaixamento máximo da negociação fechada, ou seja, a perda máxima possível durante a negociação, expressa como uma porcentagem.
- [strategy.closedtrades.max_runup_percent()](https://br.tradingview.com/pine-script-reference/v5/#fun_strategy.closedtrades.max_runup_percent) Retorna o aumento máximo da negociação fechada, ou seja, o lucro máximo possível durante a negociação, expresso como uma porcentagem.
- [strategy.closedtrades.profit_percent()](https://br.tradingview.com/pine-script-reference/v5/#fun_strategy.closedtrades.profit_percent) Retorna o valor do lucro/prejuízo da negociação fechada, expresso como uma porcentagem. As perdas são expressas como valores negativos.
- [strategy.opentrades.max_drawdown_percent()](https://br.tradingview.com/pine-script-reference/v5/#fun_strategy.opentrades.max_drawdown_percent) Retorna o rebaixamento máximo da negociação aberta, ou seja, a perda máxima possível durante a negociação, expressa como uma porcentagem.
- [strategy.opentrades.max_runup_percent()](https://br.tradingview.com/pine-script-reference/v5/#fun_strategy.opentrades.max_runup_percent) Retorna o aumento máximo da negociação aberta, ou seja, o lucro máximo possível durante a negociação, expresso como uma porcentagem.
- [strategy.opentrades.profit_percent()](https://br.tradingview.com/pine-script-reference/v5/#fun_strategy.opentrades.profit_percent) Retorna o lucro/prejuízo da negociação aberta, expresso como uma porcentagem. As perdas são expressas como valores negativos.

### Outubro 2023

#### Polilinhas no Pine Script

Polilinhas são desenhos que conectam sequencialmente as coordenadas de um [array](https://br.tradingview.com/pine-script-reference/v5/#type_array) de até 10.000 [pontos do gráfico](./04_09_tipagem_do_sistema.md#chart-points-pontos-do-gráfico) usando segmentos de linha retos ou _curvos_, permitindo que scripts desenhem formações personalizadas que são difíceis ou impossíveis de alcançar usando objetos [linha](https://br.tradingview.com/pine-script-reference/v5/#type_line) ou [box](https://br.tradingview.com/pine-script-reference/v5/#type_box). Para saber mais sobre esse novo tipo de desenho, consulte a seção [Polilinhas](./05_12_lines_e_boxes.md#polylines-polilinhas) da página do Manual do Usuário sobre [Linhas e Caixas](./05_12_lines_e_boxes.md).

### Setembro 2023

Novas funções foram adicionadas:

- [strategy.default_entry_qty()](https://br.tradingview.com/pine-script-reference/v5/#fun_strategy.default_entry_qty) Calcula a quantidade padrão, em unidades, de uma ordem de entrada de [strategy.entry()](https://br.tradingview.com/pine-script-reference/v5/#fun_strategy.entry) ou [strategy.order()](https://br.tradingview.com/pine-script-reference/v5/#fun_strategy.order) se fosse preenchida ao valor especificado `fill_price`.
- [chart.point.new()](https://br.tradingview.com/pine-script-reference/v5/#fun_chart.point.new) Cria um novo objeto [chart.point](https://br.tradingview.com/pine-script-reference/v5/#type_chart.point) com o `time`, `index` e `price` especificados.
- [request.seed()](https://br.tradingview.com/pine-script-reference/v5/#fun_request%7Bdot%7Dseed) Solicita dados de um repositório GitHub mantido pelo usuário e os retorna como uma série. Um tutorial detalhado sobre como adicionar novos dados pode ser encontrado [aqui](https://github.com/tradingview-pine-seeds/docs).
- [ticker.inherit()](https://br.tradingview.com/pine-script-reference/v5/#fun_ticker%7Bdot%7Dinherit) Constrói um ID de ticker para o `symbol` especificado com parâmetros adicionais herdados do ID de ticker passado na chamada da função, permitindo que o script solicite dados de um símbolo usando os mesmos modificadores que o `from_tickerid` possui, incluindo sessão estendida, ajuste de dividendos, conversão de moeda, tipos de gráfico não padrão, ajuste retroativo, settlement-as-close, etc.
- [timeframe.from_seconds()](https://br.tradingview.com/pine-script-reference/v5/#fun_timeframe.from_seconds) Converte um número especificado de `seconds` em uma string de _timeframe_ válida com base no [formato de especificação de _timeframe_](./05_22_timeframes.md#especificações-de-string-do-timeframe).

O namespace `dividends.*` agora inclui variáveis para recuperar informações sobre dividendos futuros:

- [dividends.future_amount](https://br.tradingview.com/pine-script-reference/v5/#var_dividends.future_amount) Retorna o valor do pagamento do próximo dividendo na moeda do instrumento atual, ou `na` se esses dados não estiverem disponíveis.
- [dividends.future_ex_date](https://br.tradingview.com/pine-script-reference/v5/#var_dividends.future_ex_date) Retorna a data Ex-dividendo (Ex-date) do próximo pagamento de dividendo do instrumento atual, ou `na` se esses dados não estiverem disponíveis.
- [dividends.future_pay_date](https://br.tradingview.com/pine-script-reference/v5/#var_dividends.future_pay_date) Retorna a data de pagamento (Pay date) do próximo pagamento de dividendo do instrumento atual, ou `na` se esses dados não estiverem disponíveis.

A função [request.security_lower_tf()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security_lower_tf) tem um novo parâmetro:

- `ignore_invalid_timeframe` Determina como a função se comporta quando o _timeframe_ do gráfico é menor que o valor `timeframe` na chamada da função. Se `false`, a função gerará um erro de tempo de execução e interromperá a execução do script. Se `true`, a função retornará `na` sem gerar um erro.

Agora, os usuários podem declarar explicitamente variáveis com os qualificadores de tipo `const`, `simple` e `series`, permitindo um controle mais preciso sobre os tipos de variáveis em seus scripts. Por exemplo:

```c
//@version=5
indicator("My script")

//@variable A constant `string` used as the `title` in the `plot()` function.
const string plotTitle = "My plot"
//@variable An `int` variable whose value is consistent after the first chart bar.
simple int a = 10
//@variable An `int` variable whose value can change on every bar.
series int b = bar_index

plot(b % a, title = plotTitle)
```

### Agosto 2023

Adicionados os seguintes [placeholders](https://br.tradingview.com/support/solutions/43000531021) de alerta:

- `{{syminfo.currency}}` Retorna o código da moeda do símbolo atual ("EUR", "USD", etc.).
- `{{syminfo.basecurrency}}` Retorna o código da moeda base do símbolo atual se o símbolo se referir a um par de moedas. Caso contrário, retorna `na`. Por exemplo, retorna "EUR" quando o símbolo é "EURUSD".

#### Mapas no Pine Script

Mapas são coleções que mantêm elementos na forma de _pares chave-valor_. Eles associam chaves únicas de um _tipo fundamental_ com valores de um tipo _embutido_ ou [definido pelo usuário](./04_09_tipagem_do_sistema.md#tipos-definidos-pelo-usuário). Diferente de [arrays](./04_14_arrays.md), essas coleções são _não ordenadas_ e não utilizam um índice de busca interno. Em vez disso, os scripts acessam os valores dos mapas referenciando as _chaves_ dos pares chave-valor inseridos neles. Para mais informações sobre essas novas coleções, consulte a [página de Mapas do Manual do Usuário](./04_16_mapas.md).

### Julho 2023

Corrigido um problema que ocasionalmente fazia com que estratégias calculassem incorretamente os tamanhos das ordens limitadas devido ao arredondamento inadequado do preço `limit`.

Adicionada uma nova variável embutida ao namespace `strategy.*`:

- [strategy.margin_liquidation_price](https://br.tradingview.com/pine-script-reference/v5/#var_strategy%7Bdot%7Dmargin_liquidation_price) Quando uma estratégia usa margem, retorna o valor do preço após o qual ocorrerá uma chamada de margem.

### Junho 2023

Novas variáveis embutidas `syminfo.*` foram adicionadas:

- [syminfo.sector](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo%7Bdot%7Dsector) Retorna o setor do símbolo.
- [syminfo.industry](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo%7Bdot%7Dindustry) Retorna a indústria do símbolo.
- [syminfo.country](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo%7Bdot%7Dcountry) Retorna o código de duas letras do país onde o símbolo é negociado.

### Maio 2023

Novo parâmetro adicionado às funções [strategy.entry()](https://br.tradingview.com/pine-script-reference/v5/#fun_strategy%7Bdot%7Dentry), [strategy.order()](https://br.tradingview.com/pine-script-reference/v5/#fun_strategy%7Bdot%7Dorder), [strategy.close()](https://br.tradingview.com/pine-script-reference/v5/#fun_strategy%7Bdot%7Dclose), [strategy.close_all()](https://br.tradingview.com/pine-script-reference/v5/#fun_strategy%7Bdot%7Dclose_all) e [strategy.exit()](https://br.tradingview.com/pine-script-reference/v5/#fun_strategy%7Bdot%7Dexit):

- `disable_alert` Desativa alertas de preenchimento de ordens para quaisquer ordens colocadas pela função.

O recurso "Indicador em indicador", que permite que um script passe a plotagem de outro indicador como um valor de fonte por meio da função [input.source()](https://br.tradingview.com/pine-script-reference/v5/#fun_input%7Bdot%7Dsource), agora suporta várias entradas externas. Scripts podem usar uma multitude de entradas externas originadas de até 10 indicadores diferentes.

Adicionadas as seguintes funções de array:

- [array.every()](https://br.tradingview.com/pine-script-reference/v5/#fun_array%7Bdot%7Devery) Retorna `true` se todos os elementos do array `id` forem `true`, caso contrário, retorna `false`.

- [array.some()](https://br.tradingview.com/pine-script-reference/v5/#fun_array%7Bdot%7Dsome) Retorna `true` se pelo menos um elemento do array `id` for `true`, caso contrário, retorna `false`. Essas funções também funcionam com arrays dos tipos [int](https://br.tradingview.com/pine-script-reference/v5/#type_int) e [float](https://br.tradingview.com/pine-script-reference/v5/#type_float), em que valores zero são considerados `false` e todos os outros `true`.

### Abril 2023

Corrigido um problema com paradas de arrasto em [strategy.exit()](https://br.tradingview.com/pine-script-reference/v5/#fun_strategy%7Bdot%7Dexit) sendo preenchidas em preços altos/baixos em vez de preços intrabar.

Corrigido o comportamento de [array.mode()](https://br.tradingview.com/pine-script-reference/v5/#fun_array%7Bdot%7Dmode), [matrix.mode()](https://br.tradingview.com/pine-script-reference/v5/#fun_matrix%7Bdot%7Dmode) e [ta.mode()](https://br.tradingview.com/pine-script-reference/v5/#fun_ta%7Bdot%7Dmode). Agora essas funções retornarão o menor valor quando os dados não tiverem o valor mais frequente.

### Março 2023

Agora é possível usar strings de _timeframe_ baseadas em segundos para o parâmetro `timeframe` em [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request%7Bdot%7Dsecurity) e [request.security_lower_tf()](https://br.tradingview.com/pine-script-reference/v5/#fun_request%7Bdot%7Dsecurity_lower_tf).

Uma nova função foi adicionada:

- [request.currency_rate()](https://br.tradingview.com/pine-script-reference/v5/#fun_request%7Bdot%7Dcurrency_rate) fornece uma taxa diária para converter um valor expresso na moeda `from` para outro na moeda `to`.

### Fevereiro 2023

#### Métodos no Pine Script

Métodos no Pine Script são funções especializadas associadas a instâncias específicas de tipos embutidos ou definidos pelo usuário. Eles oferecem uma sintaxe mais conveniente do que as funções padrão, pois os usuários podem acessar métodos da mesma forma que campos de objeto usando a prática sintaxe de notação de ponto. O Pine Script inclui métodos embutidos para os tipos [array](https://br.tradingview.com/pine-script-reference/v5/#type_array), [matrix](https://br.tradingview.com/pine-script-reference/v5/#type_matrix), [line](https://br.tradingview.com/pine-script-reference/v5/#type_line), [linefill](https://br.tradingview.com/pine-script-reference/v5/#type_linefill), [label](https://br.tradingview.com/pine-script-reference/v5/#type_label), [box](https://br.tradingview.com/pine-script-reference/v5/#type_box) e [table](https://br.tradingview.com/pine-script-reference/v5/#type_table) e facilita métodos definidos pelo usuário com a nova palavra-chave [method](https://br.tradingview.com/pine-script-reference/v5/#kw_method). Para mais detalhes sobre esse novo recurso, consulte a [página de métodos do Manual do Usuário](./04_13_metodos.md).

### Janeiro 2023

Novas funções de array foram adicionadas:

- [array.first()](https://br.tradingview.com/pine-script-reference/v5/#fun_array%7Bdot%7Dfirst) Retorna o primeiro elemento do array.
- [array.last()](https://br.tradingview.com/pine-script-reference/v5/#fun_array%7Bdot%7Dlast) Retorna o último elemento do array.


## 2022

- [2022](https://www.tradingview.com/pine-script-docs/release-notes/#2022)

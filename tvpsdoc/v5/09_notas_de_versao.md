<!-- 
# Notas de Versão

Esta página contém notas de versão descrevendo mudanças notáveis na experiência do Pine Script™.

## 2024

### Agosto 2024

As funções [ticker.new()](https://br.tradingview.com/pine-script-reference/v5/#fun_ticker.new) e [ticker.modify()](https://br.tradingview.com/pine-script-reference/v5/#fun_ticker.modify) possuem dois novos parâmetros: `settlement_as_close` e `backadjustment`. Os usuários podem especificar se esses parâmetros estão ativados, desativados ou configurados para herdar as configurações padrão do símbolo. Essas configurações afetam apenas os dados de símbolos futuros com essas opções disponíveis em seus gráficos. Não têm efeito em outros símbolos.

  * O parâmetro `backadjustment` especifica se os dados de contratos passados em símbolos futuros contínuos são ajustados retroativamente. Seus valores possíveis são: [backadjustment.on](https://br.tradingview.com/pine-script-reference/v5/#const_backadjustment.on), [backadjustment.off](https://br.tradingview.com/pine-script-reference/v5/#const_backadjustment.off), ou [backadjustment.inherit](https://br.tradingview.com/pine-script-reference/v5/#const_backadjustment.inherit).

  * O parâmetro `settlement_as_close` especifica se o valor de [close](https://br.tradingview.com/pine-script-reference/v5/#var_close) de um símbolo futuro representa o preço real de fechamento ou o preço de liquidação em timeframes de “1D” e superiores. Seus valores possíveis são: [settlement_as_close.on](https://br.tradingview.com/pine-script-reference/v5/#const_settlement_as_close.on), [settlement_as_close.off](https://br.tradingview.com/pine-script-reference/v5/#const_settlement_as_close.off), ou [settlement_as_close.inherit](https://br.tradingview.com/pine-script-reference/v5/#const_settlement_as_close.inherit).

### Junho 2024

Foi adicionado um novo parâmetro às funções [box.new()](https://br.tradingview.com/pine-script-reference/v5/#fun_box.new), [label.new()](https://br.tradingview.com/pine-script-reference/v5/#fun_label.new), [line.new()](https://br.tradingview.com/pine-script-reference/v5/#fun_line.new), [polyline.new()](https://br.tradingview.com/pine-script-reference/v5/#fun_polyline.new) e [table.new()](https://br.tradingview.com/pine-script-reference/v5/#fun_table.new):

  * `force_overlay` \- Se verdadeiro, o desenho será exibido no painel principal do gráfico, mesmo quando o script ocupar um painel separado. Opcional. O padrão é falso.

#### Enums do Pine Script™

Enums, também conhecidos como _enumerations_, _enumerated types_ ou [tipos enum](https://br.tradingview.com/pine-script-docs/pt/v5/language/type-system/#tipos-enum), são tipos de dados exclusivos com todos os valores possíveis declarados pelo programador. Eles podem ajudar os programadores a manter um controle mais rigoroso sobre os valores permitidos por variáveis, expressões condicionais e [coleções](https://br.tradingview.com/pine-script-docs/pt/v5/language/type-system/#colecoes), e permitem a criação conveniente de entradas suspensas com a nova função [input.enum()](https://br.tradingview.com/pine-script-docs/pt/v5/concepts/inputs/#entrada-enum). Veja a página [Enums](https://br.tradingview.com/pine-script-docs/pt/v5/language/enums/) no Manual do Usuário para saber mais sobre esses novos tipos e como usá-los.

### Maio 2024

Foi adicionado um parâmetro opcional `calc_bars_count` às funções [indicator()](https://br.tradingview.com/pine-script-reference/v5/#fun_indicator), [strategy()](https://br.tradingview.com/pine-script-reference/v5/#fun_strategy), [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security), [request.security_lower_tf()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security_lower_tf) e [request.seed()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.seed) que permite aos usuários limitar o número de barras históricas recentes que um script ou solicitação de dados pode executar. Quando uma declaração de [indicator()](https://br.tradingview.com/pine-script-reference/v5/#fun_indicator) ou [strategy()](https://br.tradingview.com/pine-script-reference/v5/#fun_strategy) de um script inclui um argumento `calc_bars_count`, sua aba “Configurações/Entradas” incluirá uma entrada “Barras Calculadas” na seção “Cálculo”. O valor padrão em todas essas funções é 0, o que significa que o script ou solicitação executa em todos os dados disponíveis.

O namespace `strategy.*` possui várias novas variáveis internas:

  * [strategy.avg_trade](https://br.tradingview.com/pine-script-reference/v5/#var_strategy.avg_trade) \- Retorna a quantidade média de dinheiro ganho ou perdido por negociação. Calculado como a soma de todos os lucros e perdas dividida pelo número de negociações fechadas.
  * [strategy.avg_trade_percent](https://br.tradingview.com/pine-script-reference/v5/#var_strategy.avg_trade_percent) \- Retorna o ganho ou perda percentual médio por negociação. Calculado como a soma de todos os percentuais de lucro e perda dividida pelo número de negociações fechadas.
  * [strategy.avg_winning_trade](https://br.tradingview.com/pine-script-reference/v5/#var_strategy.avg_winning_trade) \- Retorna a quantidade média de dinheiro ganho por negociação vencedora. Calculado como a soma dos lucros dividida pelo número de negociações vencedoras.
  * [strategy.avg_winning_trade_percent](https://br.tradingview.com/pine-script-reference/v5/#var_strategy.avg_winning_trade_percent) \- Retorna o ganho percentual médio por negociação vencedora. Calculado como a soma dos percentuais de lucro dividida pelo número de negociações vencedoras.
  * [strategy.avg_losing_trade](https://br.tradingview.com/pine-script-reference/v5/#var_strategy.avg_losing_trade) \- Retorna a quantidade média de dinheiro perdida por negociação perdedora. Calculado como a soma das perdas dividida pelo número de negociações perdedoras.
  * [strategy.avg_losing_trade_percent](https://br.tradingview.com/pine-script-reference/v5/#var_strategy.avg_losing_trade_percent) \- Retorna a perda percentual média por negociação perdedora. Calculado como a soma dos percentuais de perda dividida pelo número de negociações perdedoras.

#### Pine Profiler

O novo [Pine Profiler](https://br.tradingview.com/pine-script-docs/pt/v5/writing/profiling-and-optimization/#pine-profiler) é uma poderosa utilidade que analisa as execuções de todo o código significativo em um script e exibe informações úteis de desempenho ao lado das linhas de código _dentro_ do Pine Editor. As informações do [Profiler](https://br.tradingview.com/pine-script-docs/pt/v5/writing/profiling-and-optimization/#pine-profiler) fornecem insights sobre o tempo de execução de um script, a distribuição do tempo de execução em regiões de código significativas e o número de vezes que cada região de código é executada. Com esses insights, os programadores podem identificar efetivamente os _gargalos_ de desempenho e garantir que se concentrem em [otimizar](https://br.tradingview.com/pine-script-docs/pt/v5/writing/profiling-and-optimization/#optimization) seu código onde realmente importa quando precisam melhorar os tempos de execução.

Consulte a nova página [Profiling and optimization](https://br.tradingview.com/pine-script-docs/pt/v5/writing/profiling-and-optimization/) para saber mais sobre o Profiler, como ele funciona e como usá-lo para analisar o desempenho de um script e identificar oportunidades de otimização.

#### Melhorias no Pine Editor

Ao abrir o Pine Editor destacado a partir de uma aba com um gráfico, ele agora se vincula diretamente a essa aba, conforme indicado pelo status “Vinculado” e pelo ícone verde no canto inferior direito. Enquanto estiver vinculado, os botões “Adicionar ao gráfico”, “Atualizar no gráfico” e “Aplicar ao layout inteiro” afetam os gráficos na aba principal.

O Pine Editor destacado agora inclui o console do Pine.

### Abril 2024

Foi adicionado um novo parâmetro às funções [plot()](https://br.tradingview.com/pine-script-reference/v5/#fun_plot), [plotchar()](https://br.tradingview.com/pine-script-reference/v5/#fun_plotchar), [plotcandle()](https://br.tradingview.com/pine-script-reference/v5/#fun_plotcandle), [plotbar()](https://br.tradingview.com/pine-script-reference/v5/#fun_plotbar), [plotarrow()](https://br.tradingview.com/pine-script-reference/v5/#fun_plotarrow), [plotshape()](https://br.tradingview.com/pine-script-reference/v5/#fun_plotshape) e [bgcolor()](https://br.tradingview.com/pine-script

-reference/v5/#fun_bgcolor):

  * `force_overlay` \- Se verdadeiro, a saída será exibida no painel principal do gráfico, mesmo quando o script ocupar um painel separado.

### Março 2024

O namespace `syminfo.*` possui uma nova variável interna:

  * [syminfo.expiration_date](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo.expiration_date) \- Em símbolos futuros não contínuos, retorna um timestamp UNIX representando o início do último dia do contrato atual.

As funções [time()](https://br.tradingview.com/pine-script-reference/v5/#fun_time) e [time_close()](https://br.tradingview.com/pine-script-reference/v5/#fun_time_close) têm um novo parâmetro:

  * `bars_back` \- Se especificado, a função calculará o timestamp a partir da barra N barras atrás em relação à barra atual no seu timeframe. Também pode calcular o tempo esperado de uma barra futura até 500 barras à frente se o argumento for um valor negativo. Opcional. O padrão é 0.

### Fevereiro 2024

Foi adicionado duas novas funções para trabalhar com strings:

  * [str.repeat()](https://br.tradingview.com/pine-script-reference/v5/#fun_str.repeat) \- Constrói uma nova string contendo a string fonte repetida um número especificado de vezes com um separador injetado entre cada instância repetida.
  * [str.trim()](https://br.tradingview.com/pine-script-reference/v5/#fun_str.trim) \- Constrói uma nova string com todos os espaços em branco consecutivos e outros caracteres de controle removidos da esquerda e direita da string fonte.

A função [request.financial()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.financial) agora aceita “D” como argumento `period`, permitindo que scripts solicitem dados financeiros diários disponíveis.

Por exemplo:

```pinescript
//@version=5  
indicator("Daily financial data demo")  

//@variable The daily Premium/Discount to Net Asset Value for "AMEX:SPY"  
float f1 = request.financial("AMEX:SPY", "NAV", "D")  
plot(f1)  
```

O namespace `strategy.*` possui uma nova variável para monitorar o capital disponível na simulação de uma estratégia:

  * [strategy.opentrades.capital_held](https://br.tradingview.com/pine-script-reference/v5/#var_strategy.opentrades.capital_held) \- Retorna a quantidade de capital atualmente mantida por negociações abertas.

### Janeiro 2024

O namespace `syminfo.*` possui novas variáveis internas:

Syminfo:

  * [syminfo.employees](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo.employees) \- O número de funcionários que a empresa possui.
  * [syminfo.shareholders](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo.shareholders) \- O número de acionistas que a empresa possui.
  * [syminfo.shares_outstanding_float](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo.shares_outstanding_float) \- O número total de ações em circulação que uma empresa tem disponível, excluindo suas ações restritas.
  * [syminfo.shares_outstanding_total](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo.shares_outstanding_total) \- O número total de ações em circulação que uma empresa tem disponível, incluindo ações restritas mantidas por insiders, grandes acionistas e funcionários.

Preço alvo:

  * [syminfo.target_price_average](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo.target_price_average) \- A média dos últimos preços-alvo anuais para o símbolo previstos por analistas.
  * [syminfo.target_price_date](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo.target_price_date) \- A data de início da última previsão de preço-alvo para o símbolo atual.
  * [syminfo.target_price_estimates](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo.target_price_estimates) \- O número total mais recente de previsões de preço-alvo para o símbolo atual.
  * [syminfo.target_price_high](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo.target_price_high) \- O último maior preço-alvo anual para o símbolo previsto por analistas.
  * [syminfo.target_price_low](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo.target_price_low) \- O último menor preço-alvo anual para o símbolo previsto por analistas.
  * [syminfo.target_price_median](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo.target_price_median) \- A mediana dos últimos preços-alvo anuais para o símbolo previstos por analistas.

Recomendações:

  * [syminfo.recommendations_buy](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo.recommendations_buy) \- O número de analistas que deram ao símbolo atual uma classificação de “Compra”.
  * [syminfo.recommendations_buy_strong](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo.recommendations_buy_strong) \- O número de analistas que deram ao símbolo atual uma classificação de “Compra Forte”.
  * [syminfo.recommendations_date](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo.recommendations_date) \- A data de início do último conjunto de recomendações para o símbolo atual.
  * [syminfo.recommendations_hold](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo.recommendations_hold) \- O número de analistas que deram ao símbolo atual uma classificação de “Manter”.
  * [syminfo.recommendations_total](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo.recommendations_total) \- O número total de recomendações para o símbolo atual.
  * [syminfo.recommendations_sell](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo.recommendations_sell) \- O número de analistas que deram ao símbolo atual uma classificação de “Venda”.
  * [syminfo.recommendations_sell_strong](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo.recommendations_sell_strong) \- O número de analistas que deram ao símbolo atual uma classificação de “Venda Forte”. -->

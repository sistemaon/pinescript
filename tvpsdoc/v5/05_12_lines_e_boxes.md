
# Lines e Boxes (_Linhas e Caixas_)

O Pine Script facilita a criação de linhas, caixas e outras formações geométricas a partir do código usando os tipos [line](https://br.tradingview.com/pine-script-reference/v5/#type_line), [box](https://br.tradingview.com/pine-script-reference/v5/#type_box) e [polyline](https://br.tradingview.com/pine-script-reference/v5/#type_polyline). Esses tipos fornecem utilidade para desenhar programaticamente níveis de suporte e resistência, linhas de tendência, faixas de preço e outras formações personalizadas em um gráfico.

Ao contrário dos [plots](./05_15_plots.md), a flexibilidade desses tipos os torna particularmente adequados para visualizar dados calculados atuais em praticamente qualquer ponto disponível no gráfico, independentemente da barra do gráfico em que o script é executado.

[Lines](./05_12_lines_e_boxes.md#lines-linhas), [Boxes](./05_12_lines_e_boxes.md#boxes-caixas) e [Polylines](./05_12_lines_e_boxes.md#polylines-polilinhas) são objetos, assim como [labels](./05_20_text_e_shapes.md#labels), [tables](./05_19_tables.md) e outros _tipos especiais_. Scripts referenciam objetos desses tipos usando IDs, que agem como _ponteiros_. Assim como outros objetos, IDs de [line](https://br.tradingview.com/pine-script-reference/v5/#type_line), [box](https://br.tradingview.com/pine-script-reference/v5/#type_box) e [polyline](https://br.tradingview.com/pine-script-reference/v5/#type_polyline) são qualificados como valores "series", e todas as funções que gerenciam esses objetos aceitam argumentos "series".

> __Observação__\
> Usar os tipos discutidos nesta página frequentemente envolve [arrays](./04_14_arrays.md), especialmente ao trabalhar com [polylines](./05_12_lines_e_boxes.md#polylines-polilinhas), que _requerem_ um [array](https://br.tradingview.com/pine-script-reference/v5/#type_array) de instâncias de [chart.point](https://br.tradingview.com/pine-script-reference/v5/#type_chart.point). Portanto, recomenda-se familiarizar-se com [arrays](./04_14_arrays.md) para aproveitar ao máximo esses tipos de desenho em seus scripts.

_Linhas_ desenhadas por um script podem ser verticais, horizontais ou anguladas. _Caixas_ são sempre retangulares. _Polilinhas_ conectam sequencialmente vários segmentos de linha verticais, horizontais, angulados ou curvos. Embora todos esses tipos de desenho tenham características diferentes, eles têm algumas coisas em comum:

- [linhas](./05_12_lines_e_boxes.md#lines-linhas), [caixas](./05_12_lines_e_boxes.md#boxes-caixas) e [polilinhas](./05_12_lines_e_boxes.md#polylines-polilinhas) podem ter coordenadas em qualquer localização disponível no gráfico, incluindo aquelas em tempos futuros além da última barra do gráfico.
- Objetos desses tipos podem usar instâncias de [chart.point](https://br.tradingview.com/pine-script-reference/v5/#type_chart.point) para definir suas coordenadas.
- As _coordenadas-x_ de cada objeto podem ser valores de índice da barra ou valores de tempo (_time_), dependendo da propriedade `xloc` especificada.
- Cada objeto pode ter um dos vários estilos de linha predefinidos.
- Scripts podem chamar as funções que gerenciam esses objetos dentro dos escopos de [loops](./04_08_loops.md) e [estruturas condicionais](./04_07_estruturas_condicionais.md), permitindo controle iterativo e condicional de seus desenhos.
- Há limites no número de objetos que um script pode referenciar e exibir no gráfico. Uma única instância de script pode exibir até 500 _linhas_, 500 _caixas_ e 100 _polilinhas_. Usuários podem especificar o número máximo permitido para cada tipo por meio dos parâmetros `max_lines_count`, `max_boxes_count` e `max_polylines_count` da declaração [indicator()](https://br.tradingview.com/pine-script-reference/v5/#fun_indicator) ou [strategy()](https://br.tradingview.com/pine-script-reference/v5/#fun_strategy) do script. Se não especificado, o padrão é _50_. Assim como os tipos [label](https://br.tradingview.com/pine-script-reference/v5/#type_label) e [table](https://br.tradingview.com/pine-script-reference/v5/#type_table), _linhas_, _caixas_ e _polilinhas_ utilizam um mecanismo de "_garbage collection_" ("_coleta de lixo_") que exclui os objetos mais antigos no gráfico quando o número total de desenhos excede o limite do script.

> __Observação__\
> Nos gráficos do TradingView, um conjunto completo de _Ferramentas de Desenho_ permite aos usuários criar e modificar desenhos usando ações do mouse. Embora às vezes possam se assemelhar a objetos de desenho criados com código Pine Script, são entidades __não relacionadas__. Scripts Pine não podem interagir com as _ferramentas de desenho_ da interface do usuário do gráfico, e ações do mouse não afetam diretamente os objetos de desenho Pine.

# Lines (_Linhas_)

# Boxes (_Caixas_)

# Polylines (_Polilinhas_)

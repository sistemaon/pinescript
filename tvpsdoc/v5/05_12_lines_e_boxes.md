
# Lines e Boxes (_Linhas e Caixas_)

O Pine Script facilita a criação de linhas, caixas e outras formações geométricas a partir do código usando os tipos [line](https://br.tradingview.com/pine-script-reference/v5/#type_line), [box](https://br.tradingview.com/pine-script-reference/v5/#type_box) e [polyline](https://br.tradingview.com/pine-script-reference/v5/#type_polyline). Esses tipos fornecem utilidade para desenhar programaticamente níveis de suporte e resistência, linhas de tendência, faixas de preço e outras formações personalizadas em um gráfico.

Ao contrário dos [plots](./05_15_plots.md), a flexibilidade desses tipos os torna particularmente adequados para visualizar dados calculados atuais em praticamente qualquer ponto disponível no gráfico, independentemente da barra do gráfico em que o script é executado.

[Lines](./05_12_lines_e_boxes.md#lines-linhas), [Boxes](./05_12_lines_e_boxes.md#boxes-caixas) e [Polylines](./05_12_lines_e_boxes.md#polylines-polilinhas) são objetos, assim como [labels](./05_20_text_e_shapes.md#labels), [tables](./05_19_tables.md) e outros _tipos especiais_. Scripts referenciam objetos desses tipos usando IDs, que agem como _ponteiros_. Assim como outros objetos, IDs de [line](https://br.tradingview.com/pine-script-reference/v5/#type_line), [box](https://br.tradingview.com/pine-script-reference/v5/#type_box) e [polyline](https://br.tradingview.com/pine-script-reference/v5/#type_polyline) são qualificados como valores "series", e todas as funções que gerenciam esses objetos aceitam argumentos "series".

> __Observação__\
> Usar os tipos discutidos nesta página frequentemente envolve [arrays](./04_14_arrays.md), especialmente ao trabalhar com [polylines](./05_12_lines_e_boxes.md#polylines-polilinhas), que _requerem_ um [array](https://br.tradingview.com/pine-script-reference/v5/#type_array) de instâncias de [chart.point](https://br.tradingview.com/pine-script-reference/v5/#type_chart.point). Portanto, recomenda-se familiarizar-se com [arrays](./04_14_arrays.md) para aproveitar ao máximo esses tipos de desenho em seus scripts.

_Linhas_ desenhadas por um script podem ser verticais, horizontais ou anguladas. _Caixas_ são sempre retangulares. _Polilinhas_ conectam sequencialmente vários segmentos de linha verticais, horizontais, angulados ou curvos. Embora todos esses tipos de desenho tenham características diferentes, eles têm algumas coisas em comum:

- As [linhas](./05_12_lines_e_boxes.md#lines-linhas), [caixas](./05_12_lines_e_boxes.md#boxes-caixas) e [polilinhas](./05_12_lines_e_boxes.md#polylines-polilinhas) podem ter coordenadas em qualquer localização disponível no gráfico, incluindo aquelas em tempos futuros além da última barra do gráfico.
- Objetos desses tipos podem usar instâncias de [chart.point](https://br.tradingview.com/pine-script-reference/v5/#type_chart.point) para definir suas coordenadas.
- As _coordenadas-x_ de cada objeto podem ser valores de índice da barra ou valores de tempo (_time_), dependendo da propriedade `xloc` especificada.
- Cada objeto pode ter um dos vários estilos de linha predefinidos.
- Scripts podem chamar as funções que gerenciam esses objetos dentro dos escopos de [loops](./04_08_loops.md) e [estruturas condicionais](./04_07_estruturas_condicionais.md), permitindo controle iterativo e condicional de seus desenhos.
- Há limites no número de objetos que um script pode referenciar e exibir no gráfico. Uma única instância de script pode exibir até 500 _linhas_, 500 _caixas_ e 100 _polilinhas_. Usuários podem especificar o número máximo permitido para cada tipo por meio dos parâmetros `max_lines_count`, `max_boxes_count` e `max_polylines_count` da declaração [indicator()](https://br.tradingview.com/pine-script-reference/v5/#fun_indicator) ou [strategy()](https://br.tradingview.com/pine-script-reference/v5/#fun_strategy) do script. Se não especificado, o padrão é _50_. Assim como os tipos [label](https://br.tradingview.com/pine-script-reference/v5/#type_label) e [table](https://br.tradingview.com/pine-script-reference/v5/#type_table), _linhas_, _caixas_ e _polilinhas_ utilizam um mecanismo de "_garbage collection_" ("_coleta de lixo_") que exclui os objetos mais antigos no gráfico quando o número total de desenhos excede o limite do script.

> __Observação__\
> Nos gráficos do TradingView, um conjunto completo de _Ferramentas de Desenho_ permite aos usuários criar e modificar desenhos usando ações do mouse. Embora às vezes possam se assemelhar a objetos de desenho criados com código Pine Script, são entidades __não relacionadas__. Scripts Pine não podem interagir com as _ferramentas de desenho_ da interface do usuário do gráfico, e ações do mouse não afetam diretamente os objetos de desenho Pine.


# Lines (_Linhas_)

As funções incorporadas no namespace `line.*` controlam a criação e gerenciamento de objetos de [line](https://br.tradingview.com/pine-script-reference/v5/#type_line):

- A função [line.new()](https://br.tradingview.com/pine-script-reference/v5/#fun_line.new) cria uma nova _linha_.
- As funções `line.set_*()` modificam as propriedades da _linha_.
- As funções `line.get_*()` recuperam valores de uma instância de _linha_.
- A função [line.copy()](https://br.tradingview.com/pine-script-reference/v5/#fun_line.copy) clona uma instância de _linha_.
- A função [line.delete()](https://br.tradingview.com/pine-script-reference/v5/#fun_line.delete) exclui uma instância de _linha_ existente.
- A variável [line.all](https://br.tradingview.com/pine-script-reference/v5/#var_line.all) referencia um [array](https://br.tradingview.com/pine-script-reference/v5/#type_array) somente leitura contendo os IDs de todas as linhas exibidas pelo script. O [tamanho](https://br.tradingview.com/pine-script-reference/v5/#fun_array.size) do array depende do `max_lines_count` da declaração [indicator()](https://br.tradingview.com/pine-script-reference/v5/#fun_indicator) ou [strategy()](https://br.tradingview.com/pine-script-reference/v5/#fun_strategy) e do número de linhas que o script desenhou.

Scripts podem chamar as funções incorporadas `line.set_*()`, `line.get_*()`, [line.copy()](https://br.tradingview.com/pine-script-reference/v5/#fun_line.copy) e [line.delete()](https://br.tradingview.com/pine-script-reference/v5/#fun_line.delete) como funções ou [métodos](./04_13_metodos.md).

## Criando _Linhas_

A função [line.new()](https://br.tradingview.com/pine-script-reference/v5/#fun_line.new) cria uma nova instância de [line](https://br.tradingview.com/pine-script-reference/v5/#type_line) para exibir no gráfico. Ela possui as seguintes assinaturas:

```c
line.new(first_point, second_point, xloc, extend, color, style, width) → series line

line.new(x1, y1, x2, y2, xloc, extend, color, style, width) → series line
```

A primeira sobrecarga desta função contém os parâmetros `first_point` e `second_point`. O `first_point` é um [chart.point](https://br.tradingview.com/pine-script-reference/v5/#type_chart.point) representando o início da _linha_, e o `second_point` é um [chart.point](https://br.tradingview.com/pine-script-reference/v5/#type_chart.point) representando o fim da _linha_. A função copia as informações desses [pontos do gráfico](./04_09_tipagem_do_sistema.md#chart-points-pontos-do-gráfico) para determinar as coordenadas da _linha_. Se usa os campos de índice ou de tempo dos pontos `first_point` e `second_point` como _coordenadas-x_ depende do valor de `xloc` da função.

A segunda sobrecarga especifica valores `x1`, `y1`, `x2` e `y2` independentemente, onde `x1` e `x2` são valores [int](https://br.tradingview.com/pine-script-reference/v5/#type_int) representando as _coordenadas-x_ de início e fim da _linha_, e `y1` e `y2` são valores [float](https://br.tradingview.com/pine-script-reference/v5/#type_float) representando as _coordenadas-y_. Se a _linha_ considera os valores `x` como índices de barra ou carimbos de tempo depende do valor `xloc` na chamada da função.

Ambas as sobrecargas compartilham os mesmos parâmetros adicionais:

`xloc`

- Controla se as _coordenadas-x_ da nova _linha_ usam valores de índice de barra ou de tempo. Seu valor padrão é [xloc.bar_index](https://br.tradingview.com/pine-script-reference/v5/#var_xloc.bar_index).

- Ao chamar a primeira sobrecarga, usar um valor `xloc` de [xloc.bar_index](https://br.tradingview.com/pine-script-reference/v5/#var_xloc.bar_index) indica à função que use os campos de índice dos pontos`first_point` e `second_point`, e um valor de [xloc.bar_time](https://br.tradingview.com/pine-script-reference/v5/#var_xloc.bar_time) indica que use os campos de tempo dos pontos.

- Ao chamar a segunda sobrecarga, um valor `xloc` de [xloc.bar_index](https://br.tradingview.com/pine-script-reference/v5/#var_xloc.bar_index) faz com que a função trate os argumentos `x1` e `x2` como valores deíndice de barra. Ao usar [xloc.bar_time](https://br.tradingview.com/pine-script-reference/v5/#var_xloc.bar_time), a função tratará `x1` e `x2` como valores de tempo.

- Quando as _coordenadas-x_ especificadas representam valores de índice de barra, é importante notar que a coordenada x mínima permitida é `bar_index - 9999`. Para deslocamentos maiores, pode-se usar [xloc.bar_time](https://br.tradingview.com/pine-script-reference/v5/#var_xloc.bar_time).

`extend`

- Determina se a _linha_ desenhada se estenderá infinitamente além de suas coordenadas de início e fim definidas. Aceita um dos seguintes valores: [extend.left](https://br.tradingview.com/pine-script-reference/v5/#var_extend.left), [extend.right](https://br.tradingview.com/pine-script-reference/v5/#var_extend.right), [extend.both](https://br.tradingview.com/pine-script-reference/v5/#var_extend.both) ou [extend.none](https://br.tradingview.com/pine-script-reference/v5/#var_extend.none) (padrão).
    
`color`

- Especifica a cor da _linha_ desenhada. O padrão é [color.blue](https://br.tradingview.com/pine-script-reference/v5/#var_color.blue).
    
`style`
    
- Especifica o estilo da _linha_, que pode ser qualquer uma das opções listadas na seção [Estilos de Linha](./05_12_lines_e_boxes.md#line-styles-estilos-de-linha) desta página. O valor padrão é [line.style_solid](https://br.tradingview.com/pine-script-reference/v5/#var_line.style_solid).
    
`width`

- Controla a largura da _linha_, em pixels. O valor padrão é 1.

Este exemplo demonstra como se pode desenhar _linhas_ em sua forma mais simples. Este script desenha uma nova _linha_ vertical conectando os preços de [abertura](https://br.tradingview.com/pine-script-reference/v5/#var_open) e [fechamento](https://br.tradingview.com/pine-script-reference/v5/#var_close) no centro horizontal de cada barra do gráfico:

![Criando linhas 01](./imgs/Lines-and-boxes-Lines-Creating-lines-1.png)

```c
//@version=5
indicator("Creating lines demo", overlay = true)

//@variable The `chart.point` for the start of the line. Contains `index` and `time` information.
firstPoint = chart.point.now(open)
//@variable The `chart.point` for the end of the line. Contains `index` and `time` information.
secondPoint = chart.point.now(close)

// Draw a basic line with a `width` of 5 connecting the `firstPoint` to the `secondPoint`.
// This line uses the `index` field from each point for its x-coordinates.
line.new(firstPoint, secondPoint, width = 5)

// Color the background on the unconfirmed bar.
bgcolor(barstate.isconfirmed ? na : color.new(color.orange, 70), title = "Unconfirmed bar highlight")
```

__Note que:__

- Se `firstPoint` e `secondPoint` referenciam coordenadas idênticas, o script __não__ exibirá uma linha, pois não há distância entre elas para desenhar. No entanto, o ID da linha ainda existirá.
- O script exibirá aproximadamente as últimas 50 _linhas_ no gráfico, pois não possui um `max_lines_count` especificado na chamada da função [indicator()](https://br.tradingview.com/pine-script-reference/v5/#fun_indicator). Desenhos de _linhas_ persistem no gráfico até serem excluídos usando [line.delete()](https://br.tradingview.com/pine-script-reference/v5/#fun_line.delete) ou removidos pelo coletor de lixo.
- O script _redesenha_ a linha na barra aberta do gráfico (ou seja, a barra com destaque de fundo laranja) até que ela feche. Após a barra fechar, ela não atualizará mais o desenho.

Veja este exemplo mais complexo. Este script usa o preço [hl2](https://br.tradingview.com/pine-script-reference/v5/#var_hl2) da barra anterior e os preços [alto](https://br.tradingview.com/pine-script-reference/v5/#var_high) e [baixo](https://br.tradingview.com/pine-script-reference/v5/#var_low) da barra atual para desenhar um leque com um número especificado pelo usuário de _linhas_ projetando uma faixa de valores hipotéticos para a barra seguinte do gráfico. Ele chama [line.new()](https://br.tradingview.com/pine-script-reference/v5/#fun_line.new) dentro de um loop [for](./04_08_loops.md#for) para criar `linesPerBar` _linhas_ em cada barra:

![Criando linhas 02](./imgs/Lines-and-boxes-Lines-Creating-lines-2.png)

```c
//@version=5
indicator("Creating lines demo", "Simple projection fan", true, max_lines_count = 500)

//@variable The number of fan lines drawn on each chart bar.
int linesPerBar = input.int(20, "Line drawings per bar", 2, 100)

//@variable The distance between each y point on the current bar.
float step = (high - low) / (linesPerBar - 1)

//@variable The `chart.point` for the start of each line. Does not contain `time` information.
firstPoint = chart.point.from_index(bar_index - 1, hl2[1])
//@variable The `chart.point` for the end of each line. Does not contain `time` information.
secondPoint = chart.point.from_index(bar_index + 1, float(na))

//@variable The stepped y value on the current bar for `secondPoint.price` calculation, starting from the `low`.
float barValue = low
// Loop to draw the fan.
for i = 1 to linesPerBar
    // Update the `price` of the `secondPoint` using the difference between the `barValue` and `firstPoint.price`.
    secondPoint.price := 2.0 * barValue - firstPoint.price
    //@variable Is `color.aqua` when the line's slope is positive, `color.fuchsia` otherwise.
    color lineColor = secondPoint.price > firstPoint.price ? color.aqua : color.fuchsia
    // Draw a new `lineColor` line connecting the `firstPoint` and `secondPoint` coordinates.
    // This line uses the `index` field from each point for its x-coordinates.
    line.new(firstPoint, secondPoint, color = lineColor)
    // Add the `step` to the `barValue`.
    barValue += step

// Color the background on the unconfirmed bar.
bgcolor(barstate.isconfirmed ? na : color.new(color.orange, 70), title = "Unconfirmed bar highlight")
```

__Note que:__

- Foi incluído `max_lines_count = 500` na chamada da função [indicator()](https://br.tradingview.com/pine-script-reference/v5/#fun_indicator), o que significa que o script preserva até 500 _linhas_ no gráfico.
- Cada chamada [line.new()](https://br.tradingview.com/pine-script-reference/v5/#fun_line.new) copia a informação do [chart.point](https://br.tradingview.com/pine-script-reference/v5/#type_chart.point) referenciado pelas variáveis `firstPoint` e `secondPoint`. Como tal, o script pode alterar o campo de `price` do `secondPoint` em cada iteração do [loop](./04_08_loops.md) sem afetar as _coordenadas-y_ em outras _linhas_.





## Line Styles (_Estilos de Linha_)


# Boxes (_Caixas_)


# Polylines (_Polilinhas_)


# Fills (_Preenchimentos_)

Algumas das saídas visuais do Pine Script, incluindo [plots](./04_09_tipagem_do_sistema.md#plot-e-hline), [hlines](./04_09_tipagem_do_sistema.md#plot-e-hline), [lines](./05_12_lines_e_boxes.md#lines-linhas), [boxes](./05_12_lines_e_boxes.md#boxes-caixas) e [polylines](./05_12_lines_e_boxes.md#polylines), permitem preencher o espaço do gráfico que ocupam com cores. Três mecanismos diferentes facilitam o preenchimento do espaço entre essas saídas:

- A função [fill()](https://br.tradingview.com/pine-script-reference/v5/#fun_fill) preenche o espaço entre dois plots de chamadas [plot()](https://br.tradingview.com/pine-script-reference/v5/#fun_plot) ou duas linhas horizontais (hlines) de chamadas [hline()](https://br.tradingview.com/pine-script-reference/v5/#fun_hline) com uma cor especificada.
- Objetos do tipo [linefill](https://br.tradingview.com/pine-script-reference/v5/#type_linefill) preenchem o espaço entre instâncias de [line](https://br.tradingview.com/pine-script-reference/v5/#type_line) criadas com [line.new()](https://br.tradingview.com/pine-script-reference/v5/#fun_line.new).
- Outros tipos de desenho, nomeadamente [boxes](./05_12_lines_e_boxes.md#boxes-caixas) e [polylines](./05_12_lines_e_boxes.md#polylines), têm propriedades incorporadas que permitem preencher os espaços visuais que ocupam.

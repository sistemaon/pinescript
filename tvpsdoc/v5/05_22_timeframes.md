
# Timeframes

O _timeframe_ de um gráfico também é referido como seu _intervalo_ ou _resolução_. Ele é a unidade de tempo representada por uma barra no gráfico. Todos os tipos de gráficos padrão utilizam um _timeframe_: "Barras", "Velas", "Velas Vazias", "Linha", "Área" e "Linha de Base". Um tipo de gráfico não padrão que também usa _timeframes_ é o "Heikin Ashi".

> __Observação!__\
> Programadores interessados em acessar dados de múltiplos _timeframes_ precisarão se familiarizar com como os _timeframes_ são expressos no Pine Script e como utilizá-los.

__Strings de Timeframe__ são utilizadas em diferentes contextos:

- Devem ser usadas em [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request%7Bdot%7Dsecurity) ao solicitar dados de outro símbolo e/ou _timeframe_. Veja a página sobre [Outros timeframes e dados](./05_14_outros_timeframes_e_dados.md) para explorar o uso de [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request%7Bdot%7Dsecurity).
- Podem ser usadas como argumento para as funções [time()](https://br.tradingview.com/pine-script-reference/v5/#fun_time) e [time_close()](https://br.tradingview.com/pine-script-reference/v5/#fun_time_close), para retornar o tempo de uma barra de _timeframe_ superior. Isso pode ser usado para detectar mudanças em _timeframes_ superiores a partir do _timeframe_ do gráfico sem usar [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request%7Bdot%7Dsecurity). Veja a seção [Testando Mudanças em Timeframes Superiores](./05_21_time.md#testando-mudanças-em-timeframes-superiores) para ver como fazer isso.
- A função [input.timeframe()](https://br.tradingview.com/pine-script-reference/v5/#fun_input.timeframe) fornece uma maneira de permitir que os usuários definam um _timeframe_ através da aba de "_Entradas_" "_Inputs_" de um script (veja a seção [Entrada de Timeframe](./05_09_inputs.md#input-timeframe) para mais informações).
- A declaração [indicator()](https://br.tradingview.com/pine-script-reference/v5/#fun_indicator) possui um parâmetro opcional `timeframe` que pode ser usado para fornecer capacidades de múltiplos _timeframes_ a scripts simples sem usar [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request%7Bdot%7Dsecurity).
- Muitas variáveis incorporadas fornecem informações sobre o _timeframe_ usado pelo gráfico em que o script está sendo executado. Veja a seção [Timeframe do Gráfico](./05_06_informacoes_do_grafico.md#timeframe-do-gráfico) para mais informações sobre elas, incluindo [timeframe.period](https://br.tradingview.com/pine-script-reference/v5/#var_timeframe%7Bdot%7Dperiod), que retorna uma string no formato de especificação de _timeframe_ do Pine Script.

<!-- ## Especificações de String do Timeframe

Strings de _timeframe_ seguem estas regras:

- São compostas pelo multiplicador e a unidade de _timeframe_, por exemplo, "1S", "30" (30 minutos), "1D" (um dia), "3M" (três meses).
- A unidade é representada por uma única letra, sem letra usada para minutos: "S" para segundos, "D" para dias, "W" para semanas e "M" para meses.
- Quando nenhum multiplicador é usado, assume-se 1: "S" é equivalente a "1S", "D" a "1D", etc. Se apenas "1" é usado, é interpretado como "1min", já que nenhuma letra de identificação de unidade é usada para minutos.
- Não existe unidade para "hora"; "1H" __não__ é válido. O formato correto para uma hora é "60" (lembre-se que nenhuma letra de unidade é especificada para minutos).
- Os multiplicadores válidos variam para cada unidade de _timeframe_:
    - Para segundos, apenas os multiplicadores discretos 1, 5, 10, 15 e 30 são válidos.
    - Para minutos, de 1 a 1440.
    - Para dias, de 1 a 365.
    - Para semanas, de 1 a 52.
    - Para meses, de 1 a 12.

## Comparando Timeframes

Pode ser útil comparar diferentes strings de _timeframe_ para determinar, por exemplo, se o _timeframe_ usado no gráfico é menor do que os _timeframes_ superiores usados no script.

Converter strings de _timeframe_ para uma representação em minutos fracionários fornece uma maneira de compará-las usando uma unidade universal. Este script utiliza a função [timeframe.in_seconds()](https://br.tradingview.com/pine-script-reference/v5/#fun_timeframe%7Bdot%7Din_seconds) para converter um _timeframe_ em segundos flutuantes e, em seguida, converte o resultado em minutos:

```c
//@version=5
indicator("Timeframe in minutes example", "", true)
string tfInput = input.timeframe(defval = "", title = "Input TF")

float chartTFInMinutes = timeframe.in_seconds() / 60
float inputTFInMinutes = timeframe.in_seconds(tfInput) / 60

var table t = table.new(position.top_right, 1, 1)
string txt = "Chart TF: "    + str.tostring(chartTFInMinutes, "#.##### minutes") + 
"\nInput TF: " + str.tostring(inputTFInMinutes, "#.##### minutes")
if barstate.isfirst
    table.cell(t, 0, 0, txt, bgcolor = color.yellow)
else if barstate.islast
    table.cell_set_text(t, 0, 0, txt)

if chartTFInMinutes > inputTFInMinutes
    runtime.error("The chart's timeframe must not be higher than the input's timeframe.")
```

__Note que:__

- Usa-se a função incorporada [timeframe.in_seconds()](https://br.tradingview.com/pine-script-reference/v5/#fun_timeframe%7Bdot%7Din_seconds) para converter o _timeframe_ do gráfico e a função [input.timeframe()](https://br.tradingview.com/pine-script-reference/v5/#fun_input.timeframe) em segundos, depois divide-se por 60 para converter em minutos.
- São feitas duas chamadas para a função [timeframe.in_seconds()](https://br.tradingview.com/pine-script-reference/v5/#fun_timeframe%7Bdot%7Din_seconds) na inicialização das variáveis `chartTFInMinutes` e `inputTFInMinutes`. Na primeira instância, não se fornece um argumento para seu parâmetro `timeframe`, então a função retorna o _timeframe_ do gráfico em segundos. Na segunda chamada, fornece-se o _timeframe_ selecionado pelo usuário do script através da chamada para [input.timeframe()](https://br.tradingview.com/pine-script-reference/v5/#fun_input.timeframe).
- Em seguida, valida-se os _timeframes_ para garantir que o _timeframe_ de entrada seja igual ou superior ao _timeframe_ do gráfico. Caso contrário, gera-se um erro de execução.
- Finalmente, imprimem-se os dois valores de _timeframe_ convertidos em minutos. -->

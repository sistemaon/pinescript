
# Perfilamento e Otimização

Pine Script é uma linguagem compilada baseada em nuvem voltada para a execução eficiente de scripts repetidos. Quando um usuário adiciona um script Pine a um gráfico, ele é executado _numerosas_ vezes, uma vez para cada barra ou tick disponível nos feeds de dados que acessa, conforme explicado na página [Modelo de Execução](./04_01_modelo_de_execucao.md) deste manual.

O compilador Pine Script realiza automaticamente várias otimizações internas para acomodar scripts de vários tamanhos e ajudar na execução suave. No entanto, essas otimizações _não_ evitam gargalos de desempenho nas execuções dos scripts. Portanto, cabe aos programadores [perfilarem](./06_03_perfilamento_e_otimizacao.md#pine-profiler) o desempenho em tempo de execução de um script e identificarem maneiras de modificar blocos e linhas de código críticos quando precisam melhorar os tempos de execução.

Esta página aborda como perfilar e monitorar o tempo de execução de um script e suas execuções com o [Pine Profiler](./06_03_perfilamento_e_otimizacao.md#pine-profiler) e explica algumas maneiras de os programadores modificarem seu código para [otimizar](./06_03_perfilamento_e_otimizacao.md#otimização) o desempenho em tempo de execução.

## Pine Profiler

Antes de mergulhar na [otimização](./06_03_perfilamento_e_otimizacao.md#otimização), é prudente avaliar o tempo de execução de um script e identificar _gargalos_, ou seja, áreas no código que impactam substancialmente o desempenho geral. Com esses insights, os programadores podem garantir que se concentrem na otimização onde realmente importa, em vez de gastar tempo e esforço em códigos de baixo impacto.

Entre no _Pine Profiler_, uma ferramenta poderosa que analisa as execuções de todas as linhas e blocos de código significativos em um script e exibe informações úteis de desempenho ao lado das linhas dentro do Pine Editor. Ao inspecionar os resultados do Profiler, os programadores podem obter uma perspectiva mais clara sobre o tempo de execução geral do script, a distribuição do tempo de execução nas suas regiões de código significativas e as porções críticas que podem precisar de atenção e otimização extra.

### Profilando um Script

O Pine Profiler pode analisar o desempenho em tempo de execução de qualquer script _editável_ codificado em Pine Script v5. Para perfilar um script, adicione-o ao gráfico, abra o código-fonte no Pine Editor e selecione "Enable profiler mode" no menu suspenso ao lado da opção "Add to chart/Update on chart" no canto superior direito:

![Profilando um script 01](./imgs/Profiling-and-optimization-Pine-profiler-Profiling-a-script-1.Bm1SQAOz_ZcitjV.webp)

O script abaixo é utilizado como exemplo inicial de profiling, que calcula um `oscillator` personalizado com base nas distâncias médias do preço de [fechamento](https://br.tradingview.com/pine-script-reference/v5/#var_close) aos [percentis](https://br.tradingview.com/pine-script-reference/v5/#fun_ta.percentile_linear_interpolation) superior e inferior ao longo das barras `lengthInput`. Ele inclui alguns tipos diferentes de regiões de código _significativas_, que têm algumas diferenças na [interpretação](./06_03_perfilamento_e_otimizacao.md#interpretando-resultados-perfilados) durante a análise:

```c
//@version=5
indicator("Pine Profiler demo")

//@variable The number of bars in the calculations.
int lengthInput = input.int(100, "Length", 2)
//@variable The percentage for upper percentile calculation.
float upperPercentInput = input.float(75.0, "Upper percentile", 50.0, 100.0)
//@variable The percentage for lower percentile calculation.
float lowerPercentInput = input.float(25.0, "Lower percentile", 0.0, 50.0)

// Calculate percentiles using the linear interpolation method.
float upperPercentile = ta.percentile_linear_interpolation(close, lengthInput, upperPercentInput)
float lowerPercentile = ta.percentile_linear_interpolation(close, lengthInput, lowerPercentInput)

// Declare arrays for upper and lower deviations from the percentiles on the same line.
var upperDistances = array.new<float>(lengthInput), var lowerDistances = array.new<float>(lengthInput)

// Queue distance values through the `upperDistances` and `lowerDistances` arrays based on excessive price deviations.
if math.abs(close - 0.5 * (upperPercentile + lowerPercentile)) > 0.5 * (upperPercentile - lowerPercentile)
    array.push(upperDistances, math.max(close - upperPercentile, 0.0))
    array.shift(upperDistances)
    array.push(lowerDistances, math.max(lowerPercentile - close, 0.0))
    array.shift(lowerDistances)

//@variable The average distance from the `upperDistances` array.
float upperAvg = upperDistances.avg()
//@variable The average distance from the `lowerDistances` array.
float lowerAvg = lowerDistances.avg()
//@variable The ratio of the difference between the `upperAvg` and `lowerAvg` to their sum.
float oscillator = (upperAvg - lowerAvg) / (upperAvg + lowerAvg)
//@variable The color of the plot. A green-based gradient if `oscillator` is positive, a red-based gradient otherwise.
color oscColor = oscillator > 0 ?
     color.from_gradient(oscillator, 0.0, 1.0, color.gray, color.green) :
     color.from_gradient(oscillator, -1.0, 0.0, color.red, color.gray)

// Plot the `oscillator` with the `oscColor`.
plot(oscillator, "Oscillator", oscColor, style = plot.style_area)
```

Uma vez ativado, o Profiler coleta informações de todas as execuções das linhas e blocos de código significativos do script e exibe barras e percentagens aproximadas de tempo de execução à esquerda das linhas de código no Pine Editor:

![Profilando um script 02](./imgs/Profiling-and-optimization-Pine-profiler-Profiling-a-script-2.Ud_wxirg_1toi2O.webp)

__Note que:__

- O Profiler rastreia cada execução de uma região de código significativa, incluindo as execuções em _ticks em tempo real_. Suas informações são atualizadas ao longo do tempo, conforme ocorrem novas execuções.
- Os resultados do Profiler __não__ aparecem para declarações de script, declarações de tipo, outras linhas de código _insignificantes_, como declarações de variáveis sem impacto tangível, _código não utilizado_ do qual as saídas do script não dependem ou _código repetitivo_ que o compilador otimiza durante a tradução. Veja [esta seção](./06_03_perfilamento_e_otimizacao.md#código-insignificante-inusitado-e-redundante) para mais informações.

Quando um script contém pelo menos _quatro_ linhas de código significativas, o Profiler inclui ícones de "chama" ao lado das _três principais_ regiões de código com maior impacto de desempenho. Se uma ou mais das regiões de código com maior impacto estiverem _fora_ das linhas visíveis no Pine Editor, um ícone de "chama" e um número indicando quantas linhas críticas estão fora da vista aparecerão na parte superior ou inferior da margem esquerda. Clicar no ícone fará a janela do Editor rolar verticalmente para mostrar a linha crítica mais próxima:

![Profilando um script 03](./imgs/Profiling-and-optimization-Pine-profiler-Profiling-a-script-3.CRdP8jVv_Z2lC0Am.webp)

Ao passar o ponteiro do mouse sobre o espaço ao lado de uma linha, o código analisado é destacado e um tooltip com informações adicionais é exibido, incluindo o tempo gasto e o número de execuções. As informações mostradas ao lado de cada linha e no tooltip correspondente dependem da região de código perfilada. A [seção abaixo](./06_03_perfilamento_e_otimizacao.md#interpretando-resultados-perfilados) explica os diferentes tipos de código que o Profiler analisa e como interpretar seus resultados de desempenho.

![Profilando um script 04](./imgs/Profiling-and-optimization-Pine-profiler-Profiling-a-script-4.B8hBGa6N_2sbc0.webp)

> __Observação!__\
> Assim como nas ferramentas de profiling para outras linguagens, o Pine Profiler _envolve_ um script e seu código significativo com [cálculos extras](./06_03_perfilamento_e_otimizacao.md#uma-visão-geral-do-funcionamento-interno-do-profiler) necessários para coletar dados de desempenho. Como tal, o uso de recursos do script __aumenta__ durante o profiling, e os resultados do Profiler refletem o tempo de execução do script com esses cálculos incluídos. Portanto, deve-se interpretar os resultados como __estimativas__ em vez de medições de desempenho precisas. Além disso, o Profiler não pode coletar e exibir dados de desempenho individuais para os _cálculos internos_ que também afetam o tempo de execução, incluindo os cálculos necessários para rastrear o desempenho, o que significa que os valores de tempo mostrados para todas as regiões de código de um script __não somarão__ exatamente 100% do tempo de execução geral.

### Interpretando Resultados Perfilados

#### Resultados de Linha Única

Para uma linha de código contendo expressões de linha única, a barra do Profiler e a percentagem exibida representam a porção relativa do tempo total de execução do script gasto nessa linha. O tooltip correspondente exibe três campos:

- O campo "Line number" indica a linha de código analisada.
- O campo "Time" mostra a percentagem de tempo de execução para a linha de código, o tempo de execução gasto nessa linha e o tempo total de execução do script.
- O campo "Executions" mostra o número de vezes que essa linha específica foi executada durante a execução do script.

Aqui, o ponteiro foi passado sobre o espaço ao lado da linha 12 do código perfilado para visualizar seu tooltip:

![Resultados de Linha Única 01](./imgs/Profiling-and-optimization-Pine-profiler-Interpreting-profiled-results-Single-line-results-1.DxmafMJF_Z1xuVkr.webp)

```c
float upperPercentile = ta.percentile_linear_interpolation(close, lengthInput, upperPercentInput)
```

__Note que:__

- As informações de tempo para a linha representam o tempo gasto completando _todas_ as execuções, __não__ o tempo gasto em uma única execução.
- Para estimar o tempo _médio_ gasto por execução, divida o tempo da linha pelo número de execuções. Neste caso, o tooltip mostra que a linha 12 levou cerca de 14,1 milissegundos para ser executada 20.685 vezes, o que significa que o tempo médio por execução foi aproximadamente 14,1 ms / 20685 = 0,0006816534 milissegundos (0,6816534 microssegundos).

Quando uma linha de código consiste em mais de uma expressão separada por vírgulas, o número de execuções mostrado no tooltip representa a _soma_ das execuções totais de cada expressão, e o valor do tempo exibido representa o tempo total gasto na avaliação de todas as expressões da linha.

Por exemplo, esta linha global do exemplo inicial inclui duas [declarações de variáveis](./04_06_declaracoes_de_variavel.md) separadas por vírgulas. Cada uma usa a palavra-chave [var](https://br.tradingview.com/pine-script-reference/v5/#kw_var), o que significa que o script só as executa uma vez na primeira barra disponível. Como visto no tooltip do Profiler para a linha, foram contadas _duas_ execuções (uma para cada expressão) e o valor do tempo mostrado é o resultado _combinado_ de ambas as expressões na linha:

![Resultados de Linha Única 02](./imgs/Profiling-and-optimization-Pine-profiler-Interpreting-profiled-results-Single-line-results-2.CGsjIphG_1lBWSK.webp)

```c
var upperDistances = array.new<float>(lengthInput), var lowerDistances = array.new<float>(lengthInput)
```

__Note que:__

- Ao analisar scripts com mais de uma expressão na mesma linha, recomenda-se mover cada expressão para uma _linha separada_ para obter insights mais detalhados durante o profiling, especialmente se elas podem conter cálculos de _maior impacto_.

Ao usar [quebra de linha](./06_01_guia_de_estilo.md#quebra-de-linha) para fins de legibilidade ou estilísticos, o Profiler considera todas as partes de uma linha quebrada como parte da _primeira linha_ onde ela começa no Pine Editor.

Por exemplo, embora este código do script inicial ocupe mais de uma linha no Pine Editor, ele ainda é tratado como uma _única_ linha de código, e o tooltip do Profiler exibe resultados de linha única, com o campo "Line number" mostrando a _primeira_ linha no Editor que a linha quebrada ocupa:

![Resultados de Linha Única 03](./imgs/Profiling-and-optimization-Pine-profiler-Interpreting-profiled-results-Single-line-results-3.8u0gLHs0_yR0hQ.webp)

```c
color oscColor = oscillator > 0 ?
     color.from_gradient(oscillator, 0.0, 1.0, color.gray, color.green) :
     color.from_gradient(oscillator, -1.0, 0.0, color.red, color.gray)
```

#### Resultados de Blocos de Código

Para uma linha no início de um [loop](./04_08_loops.md) ou estrutura [condicional](./04_07_estruturas_condicionais.md), a barra do Profiler e a percentagem representam a porção relativa do tempo de execução do script gasto em __todo o bloco de código__, não apenas na linha única. O tooltip correspondente exibe quatro campos:

- O campo "Code block range" indica o intervalo de linhas incluídas na estrutura.
- O campo "Time" mostra a percentagem do tempo de execução do bloco de código, o tempo gasto em todas as execuções do bloco e o tempo total de execução do script.
- O campo "Line time" mostra a percentagem do tempo de execução da linha inicial do bloco, o tempo gasto nessa linha e o tempo total de execução do script. A interpretação difere para blocos [switch](https://br.tradingview.com/pine-script-reference/v5/#kw_switch) ou blocos [if](https://br.tradingview.com/pine-script-reference/v5/#kw_if) _com_ declarações [else if](https://br.tradingview.com/pine-script-reference/v5/#kw_if), pois os valores representam o tempo total gasto em __todas__ as declarações condicionais da estrutura. Veja abaixo para mais informações.
- O campo "Executions" mostra o número de vezes que o bloco de código foi executado durante a execução do script.

Aqui, o ponteiro foi passado sobre o espaço ao lado da linha 19 no script inicial, o início de uma estrutura [if](https://br.tradingview.com/pine-script-reference/v5/#kw_if) simples _sem_ declarações [else if](https://br.tradingview.com/pine-script-reference/v5/#kw_if). Como visto abaixo, o tooltip mostra informações de desempenho para todo o bloco de código e a linha atual:

![Resultados de Blocos de Código 01](./imgs/Profiling-and-optimization-Pine-profiler-Interpreting-profiled-results-Code-block-results-1.Cp7Cs5Lf_Z1aX7Bw.webp)

```c
if math.abs(close - 0.5 * (upperPercentile + lowerPercentile)) > 0.5 * (upperPercentile - lowerPercentile)
    array.push(upperDistances, math.max(close - upperPercentile, 0.0))
    array.shift(upperDistances)
    array.push(lowerDistances, math.max(lowerPercentile - close, 0.0))
    array.shift(lowerDistances)
```

__Note que:__

- O campo "Time" mostra que o tempo total gasto na avaliação da estrutura 20.685 vezes foi de 7,2 milissegundos.
- O campo "Line time" indica que o tempo de execução gasto na _primeira linha_ desta estrutura [if](https://br.tradingview.com/pine-script-reference/v5/#kw_if) foi de cerca de três milissegundos.

Os usuários também podem inspecionar os resultados das linhas e blocos aninhados dentro do intervalo do bloco de código para obter insights de desempenho mais granulares. Aqui, o ponteiro foi passado sobre o espaço ao lado da linha 20 dentro do bloco de código para visualizar seu [resultado de linha única](./06_03_perfilamento_e_otimizacao.md#resultados-de-linha-única):

![Resultados de Blocos de Código 02](./imgs/Profiling-and-optimization-Pine-profiler-Interpreting-profiled-results-Code-block-results-2.Cy1_m4AY_ZsCh1h.webp)

__Note que:__

- O número de execuções mostrado é _menor_ do que o resultado para todo o bloco de código, pois a condição que controla a execução desta linha não retorna `true` o tempo todo. O oposto se aplica ao código dentro de [loops](./04_08_loops.md), já que cada execução de uma declaração de loop pode acionar __várias__ execuções do bloco local do loop.

Ao perfilar uma estrutura [switch](https://br.tradingview.com/pine-script-reference/v5/#kw_switch) ou uma estrutura [if](https://br.tradingview.com/pine-script-reference/v5/#kw_if) que inclui declarações [else if](https://br.tradingview.com/pine-script-reference/v5/#kw_if), o campo "Line time" mostrará o tempo gasto na execução de __todas__ as expressões condicionais da estrutura, __não__ apenas a primeira linha do bloco. Os resultados para as linhas dentro do intervalo do bloco de código mostrarão o tempo de execução e as execuções para cada __bloco local__. Este formato é necessário para essas estruturas devido às limitações de cálculo e exibição do Profiler. Veja [esta seção](./06_03_perfilamento_e_otimizacao.md#uma-visão-geral-do-funcionamento-interno-do-profiler) para mais informações.

Por exemplo, o "Line time" para a estrutura [switch](https://br.tradingview.com/pine-script-reference/v5/#kw_switch) neste script representa o tempo gasto na avaliação de _todas as quatro_ declarações condicionais dentro de seu corpo, pois o Profiler _não pode_ rastreá-las separadamente. Os resultados para cada linha no intervalo do bloco de código representam as informações de desempenho para cada _bloco local_:

![Resultados de Blocos de Código 03](./imgs/Profiling-and-optimization-Pine-profiler-Interpreting-profiled-results-Code-block-results-3.D12wyQyn_ZXHSFp.webp)

```c
//@version=5
indicator("`switch` and `if...else if` results demo")

//@variable The upper band for oscillator calculation.
var float upperBand = close
//@variable The lower band for oscillator calculation.
var float lowerBand = close

// Update the `upperBand` and `lowerBand` based on the proximity of the `close` to the current band values.
// The "Line time" field on line 11 represents the time spent on all 4 conditional expressions in the structure.
switch
    close > upperBand                     => upperBand := close
    close < lowerBand                     => lowerBand := close
    upperBand - close > close - lowerBand => upperBand := 0.9 * upperBand + 0.1 * close
    close - lowerBand > upperBand - close => lowerBand := 0.9 * lowerBand + 0.1 * close

//@variable The ratio of the difference between `close` and `lowerBand` to the band range.
float oscillator = 100.0 * (close - lowerBand) / (upperBand - lowerBand)

// Plot the `oscillator` as columns with a dynamic color.
plot(
     oscillator, "Oscillator", oscillator > 50.0 ? color.teal : color.maroon,
     style = plot.style_columns, histbase = 50.0
 )
```

Quando a lógica condicional em tais estruturas envolve cálculos significativos, os programadores podem precisar de informações de desempenho mais granulares para cada condição calculada. Uma maneira eficaz de alcançar essa análise é usar blocos [if](https://br.tradingview.com/pine-script-reference/v5/#kw_if) _aninhados_ em vez das estruturas mais compactas [switch](https://br.tradingview.com/pine-script-reference/v5/#kw_switch) ou [if...else if](https://br.tradingview.com/pine-script-reference/v5/#kw_if). Por exemplo, em vez de:

```c
switch
    <expression1> => <localBlock1>
    <expression2> => <localBlock2>
    =>               <localBlock3>
```

Ou:

```c
if <expression1>
    <localBlock1>
else if <expression2>
    <localBlock2>
else
    <localBlock3>
```

Pode-se usar blocos [if](https://br.tradingview.com/pine-script-reference/v5/#kw_if) aninhados para uma análise mais aprofundada, mantendo o mesmo fluxo lógico:

```c
if <expression1>
    <localBlock1>
else
    if <expression2>
        <localBlock2>
    else
        <localBlock3>
```

Abaixo, o exemplo anterior [switch](https://br.tradingview.com/pine-script-reference/v5/#kw_switch) foi alterado para uma estrutura [if](https://br.tradingview.com/pine-script-reference/v5/#kw_if) aninhada equivalente. Agora, é possível visualizar o tempo de execução e as execuções para cada parte significativa do padrão condicional individualmente:

![Resultados de Blocos de Código 04](./imgs/Profiling-and-optimization-Pine-profiler-Interpreting-profiled-results-Code-block-results-4.OxkZ6XRw_Zs1azS.webp)

```c
//@version=5
indicator("`switch` and `if...else if` results demo")

//@variable The upper band for oscillator calculation.
var float upperBand = close
//@variable The lower band for oscillator calculation.
var float lowerBand = close

// Update the `upperBand` and `lowerBand` based on the proximity of the `close` to the current band values.
if close > upperBand
    upperBand := close
else
    if close < lowerBand
        lowerBand := close
    else
        if upperBand - close > close - lowerBand
            upperBand := 0.9 * upperBand + 0.1 * close
        else
            if close - lowerBand > upperBand - close
                lowerBand := 0.9 * lowerBand + 0.1 * close

//@variable The ratio of the difference between `close` and `lowerBand` to the band range.
float oscillator = 100.0 * (close - lowerBand) / (upperBand - lowerBand)

// Plot the `oscillator` as columns with a dynamic color.
plot(
     oscillator, "Oscillator", oscillator > 50.0 ? color.teal : color.maroon,
     style = plot.style_columns, histbase = 50.0
 )
```

__Note que:__

- Este mesmo processo também pode ser aplicado a [expressões ternárias](https://br.tradingview.com/pine-script-reference/v5/#op_?:). Quando os operandos de uma expressão ternária complexa contêm cálculos significativos, reorganizar a lógica em uma estrutura [if](https://br.tradingview.com/pine-script-reference/v5/#kw_if) aninhada permite resultados de Profiler mais detalhados, facilitando a identificação das partes críticas.

#### Chamadas de Funções Definidas pelo Usuário

[Funções definidas pelo usuário](./04_11_funcoes_definidas_pelo_usuario.md) e [métodos](./04_13_metodos.md#métodos-definidos-pelo-usuário) são funções escritas por usuários. Elas encapsulam sequências de código que um script pode executar várias vezes. Funções e métodos são frequentemente escritos para melhorar a modularidade, reutilização e manutenção do código.

As linhas de código indentadas dentro de uma função representam seu _escopo local_, ou seja, a sequência que é executada _cada vez_ que o script a chama. Diferente do código no escopo global do script, que é avaliado uma vez a cada execução, o código dentro de uma função pode ser ativado zero, uma ou _várias vezes_ em cada execução do script, dependendo das condições que acionam as chamadas, do número de chamadas que ocorrem e da lógica da função.

Essa distinção é crucial ao interpretar os resultados do Profiler. Quando um código perfilado contém chamadas de [funções definidas pelo usuário](./04_11_funcoes_definidas_pelo_usuario.md) ou [métodos](./04_13_metodos.md#métodos-definidos-pelo-usuário):

- Os resultados para cada _chamada de função_ refletem o tempo de execução alocado para ela e o número total de vezes que o script ativou essa chamada específica.
- As informações de tempo e execução para todo o código local _dentro_ do escopo de uma função refletem os resultados combinados de __todas__ as chamadas para a função.

Este exemplo contém uma função definida pelo usuário `similarity()` que estima a similaridade de duas séries, chamada pelo script apenas _uma vez_ no escopo global em cada execução. Nesse caso, os resultados do Profiler para o código dentro do corpo da função correspondem a essa chamada específica:

![Chamadas de funções definidas pelo usuário 01](./imgs/Profiling-and-optimization-Pine-profiler-Interpreting-profiled-results-User-defined-function-calls-1.DUf4uWCa_1zRHzX.webp)

```c
//@version=5
indicator("User-defined function calls demo")

//@function Estimates the similarity between two standardized series over `length` bars.
//          Each individual call to this function activates its local scope.
similarity(float sourceA, float sourceB, int length) =>
    // Standardize `sourceA` and `sourceB` for comparison.
    float normA = (sourceA - ta.sma(sourceA, length)) / ta.stdev(sourceA, length)
    float normB = (sourceB - ta.sma(sourceB, length)) / ta.stdev(sourceB, length)
    // Calculate and return the estimated similarity of `normA` and `normB`.
    float abSum = math.sum(normA * normB, length)
    float a2Sum = math.sum(normA * normA, length)
    float b2Sum = math.sum(normB * normB, length)
    abSum / math.sqrt(a2Sum * b2Sum)

// Plot the similarity between the `close` and an offset `close` series.
plot(similarity(close, close[1], 100), "Similarity 1", color.red)
```

É adicionado o número de vezes que o script chama a função a cada execução. Aqui, o script foi alterado para chamar a [função definida pelo usuário](./04_11_funcoes_definidas_pelo_usuario.md) _cinco vezes_:

```c
//@version=5
indicator("User-defined function calls demo")

//@function Estimates the similarity between two standardized series over `length` bars.
//          Each individual call to this function activates its local scope.
similarity(float sourceA, float sourceB, int length) =>
    // Standardize `sourceA` and `sourceB` for comparison.
    float normA = (sourceA - ta.sma(sourceA, length)) / ta.stdev(sourceA, length)
    float normB = (sourceB - ta.sma(sourceB, length)) / ta.stdev(sourceB, length)
    // Calculate and return the estimated similarity of `normA` and `normB`.
    float abSum = math.sum(normA * normB, length)
    float a2Sum = math.sum(normA * normA, length)
    float b2Sum = math.sum(normB * normB, length)
    abSum / math.sqrt(a2Sum * b2Sum)

// Plot the similarity between the `close` and several offset `close` series.
plot(similarity(close, close[1], 100), "Similarity 1", color.red)
plot(similarity(close, close[2], 100), "Similarity 2", color.orange)
plot(similarity(close, close[4], 100), "Similarity 3", color.green)
plot(similarity(close, close[8], 100), "Similarity 4", color.blue)
plot(similarity(close, close[16], 100), "Similarity 5", color.purple)
```

Nesse caso, os resultados do código local não correspondem mais a uma _única_ avaliação por execução do script. Em vez disso, eles representam o tempo de execução _combinado_ e as execuções do código local de __todas as cinco__ chamadas. Como visto abaixo, os resultados após executar esta versão do script nos mesmos dados mostram 137.905 execuções do código local, _cinco vezes_ o número de quando o script continha apenas uma chamada da função `similarity()`:

![Chamadas de funções definidas pelo usuário 02](./imgs/Profiling-and-optimization-Pine-profiler-Interpreting-profiled-results-User-defined-function-calls-2.BJPTeot1_Z1mNfz1.webp)

> __Observação!__\
> Quando os escopos locais de [funções definidas pelo usuário](./04_11_funcoes_definidas_pelo_usuario.md) ou [métodos](./04_13_metodos.md#métodos-definidos-pelo-usuário) de um script contêm chamadas para funções `request.*()`, a _forma traduzida_ do script extrai essas chamadas __fora__ dos escopos das funções e as avalia __separadamente__. Consequentemente, os resultados do Profiler para linhas com chamadas para essas [funções definidas pelo usuário](./04_11_funcoes_definidas_pelo_usuario.md) __não__ incluirão o tempo gasto nas chamadas `request.*()`. Veja a [seção abaixo](./06_03_perfilamento_e_otimizacao.md#solicitando-outros-contextos) para mais informações.

#### Solicitando Outros Contextos

Scripts Pine podem solicitar dados de outros _contextos_, ou seja, símbolos diferentes, _timeframes_ ou modificações de dados diferentes dos utilizados pelos dados do gráfico, chamando a família de funções `request.*()` ou especificando um `timeframe` alternativo na declaração [indicator()](https://br.tradingview.com/pine-script-reference/v5/#fun_indicator).

Quando um script solicita dados de outro contexto, ele avalia todos os escopos e cálculos necessários dentro desse contexto, conforme explicado na página [Outros _Timeframes_ e Dados](./05_14_outros_timeframes_e_dados.md). Esse comportamento pode afetar o tempo de execução das regiões de código do script e o número de vezes que são executadas.

As informações do Profiler para qualquer [linha](./06_03_perfilamento_e_otimizacao.md#resultados-de-linha-única) ou [bloco](./06_03_perfilamento_e_otimizacao.md#resultados-de-blocos-de-código) de código representam os resultados da execução do código em __todos os contextos necessários__, que podem ou não incluir os dados do gráfico. O Pine Script determina quais contextos executar o código com base nos cálculos necessários pelas solicitações de dados e saídas do script.

Este script inicial utiliza apenas os dados do gráfico para seus cálculos. Ele declara uma variável `pricesArray` com a palavra-chave [varip](https://br.tradingview.com/pine-script-reference/v5/#kw_varip), o que significa que o [array](https://br.tradingview.com/pine-script-reference/v5/#type_array) atribuído a ela persiste ao longo do histórico dos dados e de todos os ticks em tempo real disponíveis. Em cada execução, o script chama [array.push()](https://br.tradingview.com/pine-script-reference/v5/#fun_array.push) para adicionar um novo valor de [close](https://br.tradingview.com/pine-script-reference/v5/#var_close) ao [array](https://br.tradingview.com/pine-script-reference/v5/#type_array) e [plota](./05_15_plots.md) o [tamanho do array](https://br.tradingview.com/pine-script-reference/v5/#fun_array.size).

Após perfilar o script em todas as barras de um gráfico intradiário, vê-se que o número de elementos no `pricesArray` corresponde ao número de execuções que o Profiler mostra para a chamada [array.push()](https://br.tradingview.com/pine-script-reference/v5/#fun_array.push) na linha 8:

![Solicitando outros contextos 01](./imgs/Profiling-and-optimization-Pine-profiler-Interpreting-profiled-results-When-requesting-other-contexts-1.4ixtln-7_gTemj.webp)

```c
//@version=5
indicator("When requesting other contexts demo")

//@variable An array containing the `close` value from every available price update.
varip array<float> pricesArray = array.new<float>()

// Push a new `close` value into the `pricesArray` on each update.
array.push(pricesArray, close)

// Plot the size of the `pricesArray`.
plot(array.size(pricesArray), "Total number of chart price updates")
```

Agora, ao avaliar o tamanho do `pricesArray` de _outro contexto_ em vez de usar os dados do gráfico. Abaixo, uma chamada [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security) foi adicionada com [array.size(pricesArray)](https://br.tradingview.com/pine-script-reference/v5/#fun_array.size) como argumento `expression` para recuperar o valor calculado no timeframe "1D" e plotar esse resultado.

Nesse caso, o número de execuções que o Profiler mostra na linha 8 ainda corresponde ao número de elementos no `pricesArray`. No entanto, não foi executado o mesmo número de vezes, pois o script não precisava dos _dados do gráfico_ nos cálculos. Foi necessário apenas inicializar o [array](https://br.tradingview.com/pine-script-reference/v5/#type_array) e avaliar [array.push()](https://br.tradingview.com/pine-script-reference/v5/#fun_array.push) em todos os dados _diários_ solicitados, que têm um número diferente de atualizações de preços em comparação com o gráfico intradiário atual:

![Solicitando outros contextos 02](./imgs/Profiling-and-optimization-Pine-profiler-Interpreting-profiled-results-When-requesting-other-contexts-2.COS2z1lh_s4Tg3.webp)

```c
//@version=5
indicator("When requesting other contexts demo")

//@variable An array containing the `close` value from every available price update.
varip array<float> pricesArray = array.new<float>()

// Push a new `close` value into the `pricesArray` on each update.
array.push(pricesArray, close)

// Plot the size of the `pricesArray` requested from the daily timeframe.
plot(request.security(syminfo.tickerid, "1D", array.size(pricesArray)), "Total number of daily price updates")
```

__Note que:__

- Os dados EOD solicitados neste exemplo tinham menos pontos de dados do que o gráfico intradiário, portanto, a chamada [array.push()](https://br.tradingview.com/pine-script-reference/v5/#fun_array.push) exigiu menos execuções nesse caso. No entanto, os feeds EOD __não__ têm limitações de histórico, o que significa que também é possível que os dados HTF solicitados abranjam __mais__ barras do que o gráfico do usuário, dependendo do timeframe, do provedor de dados e do [plano](https://br.tradingview.com/pricing) do usuário.

Se este script fosse plotar o valor de [array.size()](https://br.tradingview.com/pine-script-reference/v5/#fun_array.size) diretamente além do valor diário solicitado, seria necessário criar _dois_ [arrays](./04_14_arrays.md) (um para cada contexto) e executar [array.push()](https://br.tradingview.com/pine-script-reference/v5/#fun_array.push) tanto nos dados do gráfico _quanto_ nos dados do timeframe diário. Assim, a declaração na linha 5 será executada __duas vezes__ e os resultados na linha 8 refletirão o tempo e as execuções acumuladas da avaliação da chamada [array.push()](https://br.tradingview.com/pine-script-reference/v5/#fun_array.push) em __ambos os conjuntos de dados separados__:

![Solicitando outros contextos 03](./imgs/Profiling-and-optimization-Pine-profiler-Interpreting-profiled-results-When-requesting-other-contexts-3.CPXHEchh_1SVGGO.webp)

```c
//@version=5
indicator("When requesting other contexts demo")

//@variable An array containing the `close` value from every available price update.
varip array<float> pricesArray = array.new<float>()

// Push a new `close` value into the `pricesArray` on each update.
array.push(pricesArray, close)

// Plot the size of the `pricesArray` from the daily timeframe and the chart's context.
// Including both in the outputs requires executing line 5 and line 8 across BOTH datasets.  
plot(request.security(syminfo.tickerid, "1D", array.size(pricesArray)), "Total number of daily price updates")
plot(array.size(pricesArray), "Total number of chart price updates")
```

É importante notar que quando um script chama uma [função definida pelo usuário](./04_11_funcoes_definidas_pelo_usuario.md) ou um [método](./04_13_metodos.md#métodos-definidos-pelo-usuário) que contém chamadas `request.*()` em seu escopo local, a _forma traduzida_ do script extrai as chamadas `request.*()` __fora__ do escopo e encapsula as expressões das quais dependem dentro de __funções separadas__. Quando o script é executado, ele avalia as chamadas `request.*()` necessárias primeiro e, em seguida, __passa__ os dados solicitados para uma _forma modificada_ da [função definida pelo usuário](./04_11_funcoes_definidas_pelo_usuario.md).

Como o script traduzido executa as solicitações de dados de uma [função definida pelo usuário](./04_11_funcoes_definidas_pelo_usuario.md) separadamente __antes__ de avaliar cálculos não solicitados em seu escopo local, os resultados do Profiler para linhas contendo chamadas para a função __não__ incluirão o tempo gasto nos cálculos necessários para as solicitações de dados.

Como exemplo, o script a seguir contém uma função definida pelo usuário `getCompositeAvg()` com uma chamada [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security) que solicita a [math.avg()](https://br.tradingview.com/pine-script-reference/v5/#fun_math.avg) de 10 chamadas [ta.wma()](https://br.tradingview.com/pine-script-reference/v5/#fun_ta.wma) com diferentes argumentos `length` de um `symbol` especificado. O script usa a função para solicitar o resultado médio usando um ticker ID [Heikin Ashi](https://br.tradingview.com/pine-script-reference/v5/#fun_ticker.heikinashi):

```c
//@version=5
indicator("User-defined functions with `request.*()` calls demo", overlay = true)

int multInput = input.int(10, "Length multiplier", 1)

string tickerID = ticker.heikinashi(syminfo.tickerid)

getCompositeAvg(string symbol, int lengthMult) =>
    request.security(
         symbol, timeframe.period, math.avg(
             ta.wma(close, lengthMult), ta.wma(close, 2 * lengthMult), ta.wma(close, 3 * lengthMult), 
             ta.wma(close, 4 * lengthMult), ta.wma(close, 5 * lengthMult), ta.wma(close, 6 * lengthMult),
             ta.wma(close, 7 * lengthMult), ta.wma(close, 8 * lengthMult), ta.wma(close, 9 * lengthMult), 
             ta.wma(close, 10 * lengthMult)
         )
     )

plot(getCompositeAvg(tickerID, multInput), "Composite average", linewidth = 3)
```

Após perfilar o script, os usuários podem se surpreender ao ver que os resultados de tempo de execução mostrados dentro do corpo da função excedem __consideravelmente__ os resultados mostrados para a _única_ chamada `getCompositeAvg()`:

![Solicitando outros contextos 04](./imgs/Profiling-and-optimization-Pine-profiler-Interpreting-profiled-results-When-requesting-other-contexts-4.CkN5cmYP_xCyjm.webp)

Os resultados aparecem dessa forma porque o script traduzido inclui modificações internas que _movem_ a chamada [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security) e sua expressão __fora__ do escopo da função, e o Profiler não tem como representar os resultados desses cálculos além de exibi-los ao lado da linha [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security) nesse cenário. O código abaixo ilustra aproximadamente como o script traduzido fica:

```c
//@version=5
indicator("User-defined functions with `request.*()` calls demo", overlay = true)

int multInput = input.int(10, "Length multiplier")

string tickerID = ticker.heikinashi(syminfo.tickerid)

secExpr(int lengthMult)=>
    math.avg(
         ta.wma(close, lengthMult), ta.wma(close, 2 * lengthMult), ta.wma(close, 3 * lengthMult),
         ta.wma(close, 4 * lengthMult), ta.wma(close, 5 * lengthMult), ta.wma(close, 6 * lengthMult),
         ta.wma(close, 7 * lengthMult), ta.wma(close, 8 * lengthMult), ta.wma(close, 9 * lengthMult),
         ta.wma(close, 10 * lengthMult)
     )

float sec = request.security(tickerID, timeframe.period, secExpr(multInput))

getCompositeAvg(float s) => s

plot(getCompositeAvg(sec), "Composite average", linewidth = 3)
```

__Note que:__

- O código `secExpr()` representa a __função separada__ usada por [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security) para calcular a expressão necessária no contexto solicitado.
- A chamada [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security) ocorre no __escopo externo__, fora da função `getCompositeAvg()`.
- A tradução reduziu substancialmente o código local de `getCompositeAvg()`. Agora, ele apenas retorna um valor passado para ele, já que todos os cálculos necessários da função ocorrem __fora__ de seu escopo. Devido a essa redução, os resultados de desempenho da chamada da função __não__ refletirão nenhum do tempo gasto nos cálculos necessários para a solicitação de dados.

#### Código Insignificante, Inusitado e Redundante

Ao inspecionar os resultados de um script perfilado, é crucial entender que _nem todo_ o código de um script impacta necessariamente o desempenho de tempo de execução. Alguns códigos não têm impacto direto no desempenho, como a declaração do script e declarações de [tipo](https://br.tradingview.com/pine-script-reference/v5/#kw_type). Outras regiões de código com expressões insignificantes, como a maioria das chamadas `input.*()`, referências de variáveis ou [declarações de variáveis](./04_06_declaracoes_de_variavel.md) sem cálculos significativos, têm pouco ou _nenhum efeito_ no tempo de execução do script. Portanto, o Profiler __não__ exibirá resultados de desempenho para esses tipos de código.

Além disso, scripts Pine não executam regiões de código que suas _saídas_ ([plots](./05_15_plots.md), [desenhos](./04_09_tipagem_do_sistema.md#tipos-de-desenho), [logs](./06_02_debugging.md#pine-logs), etc.) não dependem, pois o compilador as __remove__ automaticamente durante a tradução. Como regiões de código não utilizadas têm _zero_ impacto no desempenho de um script, o Profiler __não__ exibirá nenhum resultado para elas.

O exemplo a seguir contém uma variável `barsInRange` e um loop [for](https://br.tradingview.com/pine-script-reference/v5/#kw_for) que adiciona 1 ao valor da variável para cada preço de [fechamento](https://br.tradingview.com/pine-script-reference/v5/#var_close) histórico entre o [máximo](https://br.tradingview.com/pine-script-reference/v5/#var_high) e o [mínimo](https://br.tradingview.com/pine-script-reference/v5/#var_low) atuais ao longo de `lengthInput` barras. No entanto, o script __não utiliza__ esses cálculos em suas saídas, pois ele apenas [plota](./05_15_plots.md) o preço de [fechamento](https://br.tradingview.com/pine-script-reference/v5/#var_close). Consequentemente, a forma compilada do script __descarta__ esse código não utilizado e considera apenas a chamada [plot(close)](https://br.tradingview.com/pine-script-reference/v5/#fun_plot).

O Profiler não exibe __nenhum__ resultado para este script, pois ele não executa nenhum cálculo __significativo__:

![Código insignificante, inusitado e redundante 01](./imgs/Profiling-and-optimization-Pine-profiler-Interpreting-profiled-results-Insignificant-unused-and-redundant-code-1.CVzX40Kz_HGwJR.webp)

```c
//@version=5
indicator("Unused code demo")

//@variable The number of historical bars in the calculation.
int lengthInput = input.int(100, "Length", 1)

//@variable The number of closes over `lengthInput` bars between the current bar's `high` and `low`.
int barsInRange = 0

for i = 1 to lengthInput
    //@variable The `close` price from `i` bars ago.
    float pastClose = close[i]
    // Add 1 to `barsInRange` if the `pastClose` is between the current bar's `high` and `low`.
    if pastClose > low and pastClose < high
        barsInRange += 1

// Plot the `close` price. This is the only output. 
// Since the outputs do not require any of the above calculations, the compiled script will not execute them.
plot(close)
```

__Note que:__

- Embora este script não utilize o [input.int()](https://br.tradingview.com/pine-script-reference/v5/#fun_input.int) da linha 5 e descarte todos os cálculos associados, a entrada "Length" ainda aparecerá nas configurações do script, pois o compilador __não__ remove completamente [inputs](./05_09_inputs.md) não utilizados.

Ao alterar o script para plotar o valor de `barsInRange`, as variáveis declaradas e o loop [for](https://br.tradingview.com/pine-script-reference/v5/#kw_for) não são mais não utilizados, pois a saída depende deles, e o Profiler agora exibirá informações de desempenho para esse código:

![Código insignificante, inusitado e redundante 02](./imgs/Profiling-and-optimization-Pine-profiler-Interpreting-profiled-results-Insignificant-unused-and-redundant-code-2.TBOBJdXS_Z1yftUf.webp)

```c
//@version=5
indicator("Unused code demo")

//@variable The number of historical bars in the calculation.
int lengthInput = input.int(100, "Length", 1)

//@variable The number of closes over `lengthInput` bars between the current bar's `high` and `low`.
int barsInRange = 0

for i = 1 to lengthInput
    //@variable The `close` price from `i` bars ago.
    float pastClose = close[i]
    // Add 1 to `barsInRange` if the `pastClose` is between the current bar's `high` and `low`.
    if pastClose > low and pastClose < high
        barsInRange += 1

// Plot the `barsInRange` value. The above calculations will execute since the output requires them.
plot(barsInRange, "Bars in range")
```

__Note que:__

- O Profiler não mostra informações de desempenho para a declaração `lengthInput` na linha 5 ou a declaração `barsInRange` na linha 8, pois as expressões nessas linhas não impactam o desempenho do script.

Quando possível, o compilador também simplifica certas instâncias de _código redundante_ em um script, como algumas formas de expressões idênticas com os mesmos valores de [tipo fundamental](./04_09_tipagem_do_sistema.md#tipos). Essa otimização permite que o script compilado execute tais cálculos apenas _uma vez_, na primeira ocorrência, e _reutilize_ o resultado calculado para cada instância repetida da qual as saídas dependem.

Se um script contiver código repetitivo e o compilador o simplificar, o Profiler mostrará resultados apenas para a __primeira ocorrência__ do código, pois essa é a única vez que o script requer o cálculo.

Por exemplo, este script contém uma linha de código que plota o valor de [ta.sma(close, 100)](https://br.tradingview.com/pine-script-reference/v5/#fun_ta.sma) e 12 linhas de código que plotam o valor de [ta.sma(close, 500)](https://br.tradingview.com/pine-script-reference/v5/#fun_ta.sma):

```c
//@version=5
indicator("Redundant calculations demo", overlay = true)

// Plot the 100-bar SMA of `close` values one time.
plot(ta.sma(close, 100), "100-bar SMA", color.teal, 3)

// Plot the 500-bar SMA of `close` values 12 times. After compiler optimizations, only the first `ta.sma(close, 500)`
// call on line 9 requires calculation in this case.
plot(ta.sma(close, 500), "500-bar SMA", #001aff, 12)
plot(ta.sma(close, 500), "500-bar SMA", #4d0bff, 11)
plot(ta.sma(close, 500), "500-bar SMA", #7306f7, 10)
plot(ta.sma(close, 500), "500-bar SMA", #920be9, 9)
plot(ta.sma(close, 500), "500-bar SMA", #ae11d5, 8)
plot(ta.sma(close, 500), "500-bar SMA", #c618be, 7)
plot(ta.sma(close, 500), "500-bar SMA", #db20a4, 6)
plot(ta.sma(close, 500), "500-bar SMA", #eb2c8a, 5)
plot(ta.sma(close, 500), "500-bar SMA", #f73d6f, 4)
plot(ta.sma(close, 500), "500-bar SMA", #fe5053, 3)
plot(ta.sma(close, 500), "500-bar SMA", #ff6534, 2)
plot(ta.sma(close, 500), "500-bar SMA", #ff7a00, 1)
```

Como as últimas 12 linhas contêm chamadas idênticas a [ta.sma()](https://br.tradingview.com/pine-script-reference/v5/#fun_ta.sma), o compilador pode simplificar automaticamente o script para que precise avaliar [ta.sma(close, 500)](https://br.tradingview.com/pine-script-reference/v5/#fun_ta.sma) apenas _uma vez_ por execução, em vez de repetir o cálculo mais 11 vezes.

Como visto abaixo, o Profiler só mostra resultados para as linhas 5 e 9. Estas são as únicas partes do código que requerem cálculos significativos, pois as chamadas [ta.sma()](https://br.tradingview.com/pine-script-reference/v5/#fun_ta.sma) nas linhas 10-20 são redundantes neste caso:

![Código insignificante, inusitado e redundante 03](./imgs/Profiling-and-optimization-Pine-profiler-Interpreting-profiled-results-Insignificant-unused-and-redundant-code-3.B3yrx82E_2jeci8.webp)

Outro tipo de otimização de código repetitivo ocorre quando um script contém duas ou mais [funções definidas pelo usuário](./04_11_funcoes_definidas_pelo_usuario.md) ou [métodos](./04_13_metodos.md#métodos-definidos-pelo-usuário) com formas compiladas idênticas. Nesse caso, o compilador simplifica o script __removendo__ as funções redundantes, e o script tratará todas as chamadas para as funções redundantes como chamadas para a versão __primeira__ definida. Portanto, o Profiler mostrará resultados de desempenho do código local apenas para a _primeira_ função, pois os "clones" descartados nunca serão executados.

Por exemplo, o script abaixo contém duas [funções definidas pelo usuário](./04_11_funcoes_definidas_pelo_usuario.md), `metallicRatio()` e `calcMetallic()`, que calculam uma [média metálica](https://pt.wikipedia.org/wiki/M%C3%A9dia_met%C3%A1lica) de uma ordem dada elevada a um expoente especificado:

```c
//@version=5
indicator("Redundant functions demo")

//@variable Controls the base ratio for the `calcMetallic()` call.
int order1Input = input.int(1, "Order 1", 1)
//@variable Controls the base ratio for the `metallicRatio()` call.
int order2Input = input.int(2, "Order 2", 1)

//@function       Calculates the value of a metallic ratio with a given `order`, raised to a specified `exponent`.
//@param order    Determines the base ratio used. 1 = Golden Ratio, 2 = Silver Ratio, 3 = Bronze Ratio, and so on.
//@param exponent The exponent applied to the ratio.
metallicRatio(int order, float exponent) =>
    math.pow((order + math.sqrt(4.0 + order * order)) * 0.5, exponent)

//@function       A function with the same signature and body as `metallicRatio()`.
//                The script discards this function and treats `calcMetallic()` as an alias for `metallicRatio()`.
calcMetallic(int ord, float exp) =>
    math.pow((ord + math.sqrt(4.0 + ord * ord)) * 0.5, exp)

// Plot the results from a `calcMetallic()` and `metallicRatio()` call.
plot(calcMetallic(order1Input, bar_index % 5), "Ratio 1", color.orange, 3)
plot(metallicRatio(order2Input, bar_index % 5), "Ratio 2", color.maroon)
```

Apesar das diferenças nos nomes das funções e parâmetros, as duas funções são idênticas, o que o compilador detecta ao traduzir o script. Nesse caso, ele __descarta__ a função redundante `calcMetallic()`, e o script compilado trata a chamada `calcMetallic()` como uma chamada `metallicRatio()`.

Como visto aqui, o Profiler mostra informações de desempenho para as chamadas `calcMetallic()` e `metallicRatio()` nas linhas 21 e 22, mas __não__ mostra nenhum resultado para o código local da função `calcMetallic()` na linha 18. Em vez disso, as informações do Profiler na linha 13 dentro da função `metallicRatio()` refletem os resultados do código local de __ambas__ [chamadas de função](./06_03_perfilamento_e_otimizacao.md#chamadas-de-funções-definidas-pelo-usuário):

![Código insignificante, inusitado e redundante 04](./imgs/Profiling-and-optimization-Pine-profiler-Interpreting-profiled-results-Insignificant-unused-and-redundant-code-4.DrYo57fh_Z1tHCEy.webp)

### Uma Visão Geral do Funcionamento Interno do Profiler

O Pine Profiler envolve todas as regiões de código necessárias com _funções internas especializadas_ para rastrear e coletar informações necessárias durante as execuções do script. Em seguida, ele passa as informações para cálculos adicionais que organizam e exibem os resultados de desempenho dentro do Pine Editor. Esta seção oferece aos usuários uma visão de como o Profiler aplica funções internas para envolver o código Pine e coletar dados de desempenho.

Existem duas funções principais internas __(não Pine)__ que o Profiler usa para envolver o código significativo e facilitar a análise em tempo de execução. A primeira função recupera a hora atual do sistema em pontos específicos da execução do script, e a segunda mapeia o tempo acumulado e os dados de execução para regiões de código específicas. Estas funções são representadas nesta explicação como `System.timeNow()` e `registerPerf()`, respectivamente.

Quando o Profiler detecta código que requer análise, ele adiciona `System.timeNow()` acima do código para obter o tempo inicial antes da execução. Em seguida, ele adiciona `registerPerf()` abaixo do código para mapear e acumular o tempo decorrido e o número de execuções. O tempo decorrido adicionado em cada chamada de `registerPerf()` é o valor de `System.timeNow()` _após_ a execução menos o valor _antes_ da execução.

O _pseudocódigo_ a seguir descreve esse processo para uma [linha única](./06_03_perfilamento_e_otimizacao.md#resultados-de-linha-única) de código, onde `_startX` representa o tempo de início para a `lineX`:

```c
long _startX = System.timeNow()
<code_line_to_analyze>
registerPerf(System.timeNow() - _startX, lineX)
```

O processo é semelhante para [blocos de código](./06_03_perfilamento_e_otimizacao.md#resultados-de-blocos-de-código). A diferença é que a chamada de `registerPerf()` mapeia os dados para um _intervalo de linhas_ em vez de uma linha única. Aqui, `lineX` representa a _primeira_ linha no bloco de código, e `lineY` representa a _última_ linha do bloco:

```c
long _startX = System.timeNow()
<code_block_to_analyze>
registerPerf(System.timeNow() - _startX, lineX, lineY)
```

__Note que:__

- Nos trechos acima, `long`, `System.timeNow()` e `registerPerf()` representam _código interno_, __não__ código Pine Script.

Agora, veja como o Profiler envolve um script completo e todas as suas regiões de código significativas. Este script calcula três séries pseudorandômicas e exibe seu resultado [médio](https://br.tradingview.com/pine-script-reference/v5/#fun_math.avg). O script utiliza um [objeto](./04_12_objetos.md) de um [tipo definido pelo usuário](./04_09_tipagem_do_sistema.md#tipos-definidos-pelo-usuário) para armazenar um estado pseudorandômico, um [método](./04_13_metodos.md#métodos-definidos-pelo-usuário) para calcular novos valores e atualizar o estado, e uma estrutura [if…else if](https://br.tradingview.com/pine-script-reference/v5/#kw_if) para atualizar cada série com base nos valores gerados:

```c
//@version=5
indicator("Profiler's inner workings demo")

int seedInput = input.int(12345, "Seed")

type LCG
    float state

method generate(LCG this, int generations = 1) =>
    float result = 0.0
    for i = 1 to generations
        this.state := 16807 * this.state % 2147483647
        result += this.state / 2147483647
    result / generations

var lcg = LCG.new(seedInput)

var float val0 = 1.0
var float val1 = 1.0
var float val2 = 1.0

if lcg.generate(10) < 0.5
    val0 *= 1.0 + (2.0 * lcg.generate(50) - 1.0) * 0.1
else if lcg.generate(10) < 0.5
    val1 *= 1.0 + (2.0 * lcg.generate(50) - 1.0) * 0.1
else if lcg.generate(10) < 0.5
    val2 *= 1.0 + (2.0 * lcg.generate(50) - 1.0) * 0.1

plot(math.avg(val0, val1, val2), "Average pseudorandom result", color.purple)
```

O Profiler envolverá o script inteiro e todas as regiões de código necessárias, excluindo qualquer [código insignificante, inusitado ou redundante](./06_03_perfilamento_e_otimizacao.md#código-insignificante-inusitado-e-redundante), com as mencionadas __funções internas__ para coletar dados de desempenho. O _pseudocódigo_ abaixo demonstra como esse processo se aplica ao script acima:

```c
long _startMain = System.timeNow() // Start time for the script's overall execution.

// <Additional internal code executes here>

//@version=5
indicator("Profiler's inner workings demo") // Declaration statements do not require profiling.

int seedInput = input.int(12345, "Seed") // Variable declaration without significant calculation.

type LCG        // Type declarations do not require profiling.
    float state

method generate(LCG this, int generations = 1) => // Function signature does not affect runtime.
    float result = 0.0 // Variable declaration without significant calculation.

    long _start11 = System.timeNow() // Start time for the loop block that begins on line 11.
    for i = 1 to generations // Loop header calculations are not independently wrapped.

        long _start12 = System.timeNow() // Start time for line 12.
        this.state := 16807 * this.state % 2147483647
        registerPerf(System.timeNow() - _start12, line12) // Register performance info for line 12.

        long _start13 = System.timeNow() // Start time for line 13.
        result += this.state / 2147483647
        registerPerf(System.timeNow() - _start13, line13) // Register performance info for line 13.

    registerPerf(System.timeNow() - _start11, line11, line13) // Register performance info for the block (line 11 - 13).    

    long _start14 = System.timeNow() // Start time for line 14.
    result / generations
    registerPerf(System.timeNow() - _start14, line14) // Register performance info for line 14.

long _start16 = System.timeNow() // Start time for line 16.
var lcg = LCG.new(seedInput)
registerPerf(System.timeNow() - _start16, line16) // Register performance info for line 16.

var float val0 = 1.0 // Variable declarations without significant calculations.
var float val1 = 1.0
var float val2 = 1.0

long _start22 = System.timeNow() // Start time for the `if` block that begins on line 22.
if lcg.generate(10) < 0.5 // `if` statement is not independently wrapped.

    long _start23 = System.timeNow() // Start time for line 23.
    val0 *= 1.0 + (2.0 * lcg.generate(50) - 1.0) * 0.1
    registerPerf(System.timeNow() - _start23, line23) // Register performance info for line 23.

else if lcg.generate(10) < 0.5 // `else if` statement is not independently wrapped.

    long _start25 = System.timeNow() // Start time for line 25.
    val1 *= 1.0 + (2.0 * lcg.generate(50) - 1.0) * 0.1
    registerPerf(System.timeNow() - _start25, line25) // Register performance info for line 25.

else if lcg.generate(10) < 0.5 // `else if` statement is not independently wrapped.

    long _start27 = System.timeNow() // Start time for line 27.
    val2 *= 1.0 + (2.0 * lcg.generate(50) - 1.0) * 0.1
    registerPerf(System.timeNow() - _start27, line27) // Register performance info for line 27.

registerPerf(System.timeNow() - _start22, line22, line28) // Register performance info for the block (line 22 - 28).

long _start29 = System.timeNow() // Start time for line 29.
plot(math.avg(val0, val1, val2), "Average pseudorandom result", color.purple)
registerPerf(System.timeNow() - _start29, line29) // Register performance info for line 29.

// <Additional internal code executes here>

registerPerf(System.timeNow() - _startMain, total) // Register the script's overall performance info.
```

__Note que:__

- Este exemplo é um __pseudocódigo__ que fornece um esboço básico dos __cálculos internos__ que o Profiler aplica para coletar dados de desempenho. Salvar este exemplo no Pine Editor resultará em um erro de compilação, pois `long`, `System.timeNow()` e `registerPerf()` __não__ representam código Pine Script.
- Esses cálculos internos que o Profiler envolve em um script requerem __recursos computacionais adicionais__, o que é a razão pela qual o tempo de execução de um script __aumenta__ durante o perfilamento. Programadores devem sempre interpretar os resultados como __estimativas__ já que refletem o desempenho de um script com os cálculos extras incluídos.

Após executar o script envolvido para coletar dados de desempenho, cálculos _adicionais_ internos organizam os resultados e exibem informações relevantes dentro do Pine Editor:

![Uma visão geral do funcionamento interno do profiler](./imgs/Profiling-and-optimization-Pine-profiler-A-look-into-the-profilers-inner-workings-1.wt5GoYky_Z1cXiWC.webp)

O cálculo _"Line time"_ para [blocos de código](./06_03_perfilamento_e_otimizacao.md#resultados-de-blocos-de-código) também ocorre nesta etapa, já que o Profiler não pode envolver individualmente cabeçalhos de [loop](./04_08_loops.md) ou as declarações condicionais em estruturas [if](https://br.tradingview.com/pine-script-reference/v5/#kw_if) ou [switch](https://br.tradingview.com/pine-script-reference/v5/#kw_switch). O valor deste campo representa a _diferença_ entre o tempo total de um bloco e a soma dos tempos de seu código local, o que explica por que o valor "Line time" para um bloco [switch](https://br.tradingview.com/pine-script-reference/v5/#kw_switch) ou um bloco [if](https://br.tradingview.com/pine-script-reference/v5/#kw_if) com expressões [else if](https://br.tradingview.com/pine-script-reference/v5/#kw_if) representa o tempo gasto em __todas__ as declarações condicionais da estrutura, não apenas na _linha inicial_ de código do bloco. Se um programador precisar de informações mais detalhadas para cada expressão condicional em um bloco, ele pode reorganizar a lógica em uma estrutura [if](https://br.tradingview.com/pine-script-reference/v5/#kw_if) _aninhada_, como explicado [aqui](./06_03_perfilamento_e_otimizacao.md#resultados-de-blocos-de-código).

> __Observação!__\
> O Profiler __não pode__ coletar dados de desempenho individuais para cálculos _internos_ necessários e exibir seus resultados no Pine Editor. Consequentemente, os valores de tempo exibidos pelo Profiler para todas as regiões de código em um script __não__ somarão 100% do tempo total de execução.

### Profilando Entre Configurações

Quando a [complexidade temporal](https://pt.wikipedia.org/wiki/Complexidade_de_tempo) de um código não é constante ou seu padrão de execução varia com suas entradas, argumentos de função ou dados disponíveis, muitas vezes é aconselhável fazer o perfilamento do código em _diferentes configurações_ e feeds de dados para obter uma perspectiva mais abrangente sobre seu desempenho geral.

Por exemplo, este script simples usa um loop [for](https://br.tradingview.com/pine-script-reference/v5/#kw_for) para calcular a soma das distâncias quadradas entre o preço de [fechamento](https://br.tradingview.com/pine-script-reference/v5/#var_close) atual e os preços de `lengthInput` barras anteriores, em seguida, plota a [raiz quadrada](https://br.tradingview.com/pine-script-reference/v5/#fun_math.sqrt) dessa soma em cada barra. Nesse caso, o valor de `lengthInput` impacta diretamente o tempo de execução do cálculo, pois determina o número de vezes que o loop executa seu código local:

```c
//@version=5
indicator("Profiling across configurations demo")

//@variable The number of previous bars in the calculation. Directly affects the number of loop iterations.
int lengthInput = input.int(25, "Length", 1)

//@variable The sum of squared distances from the current `close` to `lengthInput` past `close` values.
float total = 0.0

// Look back across `lengthInput` bars and accumulate squared distances.
for i = 1 to lengthInput
    float distance = close - close[i]
    total += distance * distance

// Plot the square root of the `total`.
plot(math.sqrt(total))
```

Perfilando este script com diferentes valores de `lengthInput`. Primeiro, com o valor padrão de 25. Os resultados do Profiler para esta execução específica mostram que o script completou 20.685 execuções em cerca de 96,7 milissegundos:

![Profilando em diferentes configurações 01](./imgs/Profiling-and-optimization-Pine-profiler-Profiling-across-configurations-1.DH6uvleV_15wBst.webp)

Agora, aumentando o valor da entrada para 50 nas configurações do script. Os resultados desta execução mostram que o tempo total de execução do script foi de 194,3 milissegundos, quase _o dobro_ do tempo da execução anterior:

![Profilando em diferentes configurações 02](./imgs/Profiling-and-optimization-Pine-profiler-Profiling-across-configurations-2.ggfGlT9L_ZW6q7G.webp)

Na próxima execução, alterado o valor da entrada para 200. Desta vez, os resultados do Profiler mostram que o script terminou todas as execuções em aproximadamente 0,8 segundos, cerca de _quatro vezes_ o tempo da execução anterior:

![Profilando em diferentes configurações 03](./imgs/Profiling-and-optimization-Pine-profiler-Profiling-across-configurations-3.CZLkQfeW_Z1lXDjU.webp)

Com essas observações, é possível ver que o tempo de execução do script parece escalar de forma _linear_ com o valor de `lengthInput`, excluindo outros fatores que podem afetar o desempenho, como era de se esperar, já que a maior parte dos cálculos do script ocorre dentro do loop e o valor da entrada controla quantas vezes o loop deve ser executado.

> __Observação!__\
> Muitas vezes, é aconselhável fazer o perfilamento de cada configuração _mais de uma vez_ para reduzir o impacto de valores atípicos ao avaliar como o desempenho de um script varia com suas entradas ou dados. Veja a [seção abaixo](./06_03_perfilamento_e_otimizacao.md#perfilamento-repetitivo) para mais informações.

### Perfilamento Repetitivo

Os recursos de tempo de execução disponíveis para um script _variam_ ao longo do tempo. Consequentemente, o tempo necessário para avaliar uma região de código, mesmo uma com [complexidade](https://pt.wikipedia.org/wiki/Complexidade_de_tempo) constante, _flutua_ entre as execuções, e os resultados de desempenho cumulativo mostrados pelo Profiler __variarão__ com cada execução independente do script.

Os usuários podem aprimorar sua análise _reiniciando_ um script várias vezes e perfilando cada execução independente. A média dos resultados de cada execução perfilada e a avaliação da dispersão dos resultados de tempo de execução podem ajudar os usuários a estabelecer benchmarks de desempenho mais robustos e reduzir o impacto de _outliers_ (tempos de execução anormalmente longos ou curtos) em suas conclusões.

Incorporar uma _entrada fictícia_ (ou seja, uma entrada que não faz nada) no código de um script é uma técnica simples que permite aos usuários _reiniciá-lo_ durante o perfilamento. A entrada não afetará diretamente nenhum cálculo ou saída. No entanto, ao alterar seu valor nas configurações do script, o script reinicia e o Profiler reanalisa o código executado.

Por exemplo, este script [fila](./04_14_arrays.md#utilizando-um-array-como-uma-fila) valores pseudorandômicos com uma semente constante através de um [array](https://br.tradingview.com/pine-script-reference/v5/#type_array) com tamanho fixo e calcula e plota o valor [médio](https://br.tradingview.com/pine-script-reference/v5/#fun_array.avg) do array em cada barra. Para fins de perfilamento, o script inclui uma variável `dummyInput` com um valor [input.int()](https://br.tradingview.com/pine-script-reference/v5/#fun_input.int) atribuído a ela. A entrada não faz nada no código além de permitir _reiniciar_ o script cada vez que seu valor é alterado:

```c
//@version=5
indicator("Repetitive profiling demo")

//@variable An input not connected to script calculations. Changing its value in the "Inputs" tab restarts the script.
int dummyInput = input.int(0, "Dummy input")

//@variable An array of pseudorandom values.
var array<float> randValues = array.new<float>(2500, 0.0)

// Push a new `math.random()` value with a fixed `seed` into the `randValues` array and remove the oldest value.
array.push(randValues, math.random(seed = 12345))
array.shift(randValues)

// Plot the average of all elements in the `randValues` array.
plot(array.avg(randValues), "Pseudorandom average")
```

Após a primeira execução do script, o Profiler mostra que levou 308,6 milissegundos para executar em todos os dados do gráfico:

![Perfilamento repetitivo 01](./imgs/Profiling-and-optimization-Pine-profiler-Repetitive-profiling-1.CfionGK2_pmOYe.webp)

Agora, alterando o valor da entrada fictícia nas configurações do script para reiniciá-lo sem alterar os cálculos. Desta vez, completou as mesmas execuções de código em 424,6 milissegundos, 116 milissegundos a mais do que a execução anterior:

![Perfilamento repetitivo 02](./imgs/Profiling-and-optimization-Pine-profiler-Repetitive-profiling-2.LouNiZpq_o9WWS.webp)

Reiniciar o script novamente produz um novo resultado. Na terceira execução, o script terminou todas as execuções de código em 227,4 milissegundos, o menor tempo até agora:

![Perfilamento repetitivo 03](./imgs/Profiling-and-optimization-Pine-profiler-Repetitive-profiling-3.UGJOj4nF_Z1p4czi.webp)

Após repetir este processo várias vezes e documentar os resultados de cada execução, pode-se calcular manualmente a _média_ para estimar o tempo total esperado de execução do script:

`AverageTime = (time1 + time2 + ... + timeN) / N`

> __Observação!__\
> Seja perfilando um script em uma única execução ou várias, é crucial entender que __os resultados variarão__. Embora a média dos resultados em várias execuções perfiladas possa ajudar a obter estimativas de desempenho mais estáveis, tais estimativas __não__ são imunes a variações.

## Otimização

__Otimização de código__, não confundida com otimização de indicador ou estratégia, envolve modificar o código-fonte de um script para melhorar o tempo de execução, a eficiência dos recursos e a escalabilidade. Os programadores podem usar várias abordagens para otimizar um script quando precisam de um desempenho de tempo de execução aprimorado, dependendo do que os cálculos do script envolvem.

Fundamentalmente, a maioria das técnicas usadas para otimizar o código Pine envolve __reduzir__ o número de vezes que cálculos críticos ocorrem ou __substituir__ cálculos significativos por fórmulas ou funções internas simplificadas. Ambos esses paradigmas frequentemente se sobrepõem.

As seções seguintes explicam vários conceitos simples que os programadores podem aplicar para otimizar seu código Pine Script.

> __Observação!__\
> Antes de procurar maneiras de otimizar um script, [perfile-o](./06_03_perfilamento_e_otimizacao.md#profilando-um-script) para avaliar seu desempenho e identificar as __regiões críticas do código__ que mais se beneficiarão da otimização.

### Usando Funções Incorporadas

Pine Script oferece uma variedade de funções e variáveis _internas_ que ajudam a simplificar a criação de scripts. Muitas dessas funções internas possuem otimizações internas para maximizar a eficiência e minimizar o tempo de execução. Assim, uma das maneiras mais simples de otimizar o código Pine é utilizar essas funções internas eficientes nos cálculos do script sempre que possível.

Este exemplo mostra como substituir cálculos definidos pelo usuário por uma chamada interna concisa pode melhorar substancialmente o desempenho. Suponha que um programador queira calcular o valor mais alto de uma série em um número especificado de barras. Alguém não familiarizado com todas as funções internas do Pine pode abordar a tarefa usando um código como o seguinte, que usa um [loop](./04_08_loops.md) em cada barra para comparar `length` valores históricos de uma série `source`:

```c
//@variable A user-defined function to calculate the highest `source` value over `length` bars.
pineHighest(float source, int length) =>
    float result = na
    if bar_index + 1 >= length
        result := source
        if length > 1
            for i = 1 to length - 1
                result := math.max(result, source[i])
    result
```

Alternativamente, pode-se elaborar uma função Pine mais otimizada reduzindo o número de vezes que o loop é executado, já que iterar sobre o histórico do `source` para obter o resultado é necessário apenas quando ocorrem condições específicas:

```c
//@variable A faster user-defined function to calculate the highest `source` value over `length` bars.
//          This version only requires a loop when the highest value is removed from the window, the `length` 
//          changes, or when the number of bars first becomes sufficient to calculate the result. 
fasterPineHighest(float source, int length) =>
    var float result = na
    if source[length] == result or length != length[1] or bar_index + 1 == length
        result := source
        if length > 1
            for i = 1 to length - 1
                result := math.max(result, source[i])
    else
        result := math.max(result, source)
    result
```

A função interna [ta.highest()](https://br.tradingview.com/pine-script-reference/v5/#fun_ta.highest) superará __ambas__ essas implementações, pois seus cálculos internos são altamente otimizados para execução eficiente. Abaixo, um script que plota os resultados das chamadas `pineHighest()`, `fasterPineHighest()` e [ta.highest()](https://br.tradingview.com/pine-script-reference/v5/#fun_ta.highest) para comparar seu desempenho usando o [Profiler](./06_03_perfilamento_e_otimizacao.md#pine-profiler):

```c
//@version=5 
indicator("Using built-ins demo")

//@variable A user-defined function to calculate the highest `source` value over `length` bars.
pineHighest(float source, int length) =>
    float result = na
    if bar_index + 1 >= length
        result := source
        if length > 1
            for i = 1 to length - 1
                result := math.max(result, source[i])
    result

//@variable A faster user-defined function to calculate the highest `source` value over `length` bars.
//          This version only requires a loop when the highest value is removed from the window, the `length` 
//          changes, or when the number of bars first becomes sufficient to calculate the result. 
fasterPineHighest(float source, int length) =>
    var float result = na
    if source[length] == result or length != length[1] or bar_index + 1 == length
        result := source
        if length > 1
            for i = 1 to length - 1
                result := math.max(result, source[i])
    else
        result := math.max(result, source)
    result

plot(pineHighest(close, 20))
plot(fasterPineHighest(close, 20))
plot(ta.highest(close, 20))
```

Os [resultados perfilados](./06_03_perfilamento_e_otimizacao.md#interpretando-resultados-perfilados) em 20.735 execuções de script mostram que a chamada para `pineHighest()` levou mais tempo para executar, com um tempo de execução de 57,9 milissegundos, cerca de 69,3% do tempo total do script. A chamada `fasterPineHighest()` foi mais eficiente, levando apenas cerca de 16,9 milissegundos, aproximadamente 20,2% do tempo total, para calcular os mesmos valores.

A função mais eficiente _de longe_ foi a [ta.highest()](https://br.tradingview.com/pine-script-reference/v5/#fun_ta.highest), que levou apenas 3,2 milissegundos (~3,8% do tempo total de execução) para executar em todos os dados do gráfico e calcular os mesmos valores nesta execução:

![Usando funções incorporadas 01](./imgs/Profiling-and-optimization-Optimization-Using-built-ins-1.CXnfIZo4_ZWjpAS.webp)

Embora esses resultados demonstrem efetivamente que a função interna supera as [funções definidas pelo usuário](./04_11_funcoes_definidas_pelo_usuario.md) com um pequeno argumento `length` de 20, é crucial considerar que os cálculos requeridos pelas funções _variarão_ com o valor do argumento. Portanto, pode-se perfilar o código usando [diferentes argumentos](./06_03_perfilamento_e_otimizacao.md#profilando-entre-configurações) para avaliar como o tempo de execução se escala.

Aqui, o argumento `length` em cada chamada de função foi alterado de 20 para 200 e o [script foi perfilado](./06_03_perfilamento_e_otimizacao.md#profilando-um-script) novamente para observar as mudanças no desempenho. O tempo gasto na função `pineHighest()` nesta execução aumentou para cerca de 0,6 segundos (~86% do tempo total de execução), e o tempo gasto na função `fasterPineHighest()` aumentou para cerca de 75 milissegundos. A função [ta.highest()](https://br.tradingview.com/pine-script-reference/v5/#fun_ta.highest), por outro lado, _não_ teve uma mudança substancial no tempo de execução. Levou cerca de 5,8 milissegundos desta vez, apenas alguns milissegundos a mais do que na execução anterior.

Em outras palavras, enquanto as [funções definidas pelo usuário](./04_11_funcoes_definidas_pelo_usuario.md) experimentaram um crescimento significativo no tempo de execução com um argumento `length` maior nesta execução, a mudança no tempo de execução da função interna [ta.highest()](https://br.tradingview.com/pine-script-reference/v5/#fun_ta.highest) foi relativamente marginal neste caso, enfatizando ainda mais seus benefícios de desempenho:

![Usando funções incorporadas 02](./imgs/Profiling-and-optimization-Optimization-Using-built-ins-2.wlIsvoLn_Z1yQ1xR.webp)

__Note que:__

- Em muitos cenários, o tempo de execução de um script pode se beneficiar do uso de funções internas quando aplicável. No entanto, a vantagem de desempenho relativa obtida com o uso de funções internas depende do _código de alto impacto_ de um script e das funções internas específicas utilizadas. Em qualquer caso, deve-se sempre [perfilar seus scripts](./06_03_perfilamento_e_otimizacao.md#profilando-um-script), de preferência [várias vezes](./06_03_perfilamento_e_otimizacao.md#perfilamento-repetitivo), ao explorar soluções otimizadas.
- Os cálculos realizados pelas funções neste exemplo também dependem da sequência dos dados do gráfico. Portanto, programadores podem obter mais insights sobre o desempenho geral ao perfilar o script em [diferentes conjuntos de dados](./06_03_perfilamento_e_otimizacao.md#profilando-entre-configurações).

### Reduzindo Repetição

O compilador do Pine Script pode simplificar automaticamente alguns tipos de [código repetitivo](./06_03_perfilamento_e_otimizacao.md#código-insignificante-inusitado-e-redundante) sem a intervenção do programador. No entanto, esse processo automático tem suas limitações. Se um script contiver cálculos repetitivos que o compilador _não_ pode reduzir, os programadores podem reduzir a repetição _manualmente_ para melhorar o desempenho do script.

Por exemplo, este script contém um [método](./04_13_metodos.md#métodos-definidos-pelo-usuário) `valuesAbove()` que conta o número de elementos em um [array](https://br.tradingview.com/pine-script-reference/v5/#type_array) acima do elemento em um índice especificado. O script plota o número de valores acima do elemento no último índice de um array `data` com uma `plotColor` calculada. Calcula a `plotColor` dentro de uma estrutura [switch](https://br.tradingview.com/pine-script-reference/v5/#kw_switch) que chama `valuesAbove()` em todas as 10 de suas expressões condicionais:

```c
//@version=5
indicator("Reducing repetition demo")

//@function Counts the number of elements in `this` array above the element at a specified `index`.
method valuesAbove(array<float> this, int index) =>
    int result = 0
    float reference = this.get(index)
    for [i, value] in this
        if i == index
            continue
        if value > reference
            result += 1
    result

//@variable An array containing the most recent 100 `close` prices.
var array<float> data = array.new<float>(100)
data.push(close)
data.shift()

//@variable Returns `color.purple` with a varying transparency based on the `valuesAbove()`.
color plotColor = switch
    data.valuesAbove(99) <= 10  => color.new(color.purple, 90)
    data.valuesAbove(99) <= 20  => color.new(color.purple, 80)
    data.valuesAbove(99) <= 30  => color.new(color.purple, 70)
    data.valuesAbove(99) <= 40  => color.new(color.purple, 60)
    data.valuesAbove(99) <= 50  => color.new(color.purple, 50)
    data.valuesAbove(99) <= 60  => color.new(color.purple, 40)
    data.valuesAbove(99) <= 70  => color.new(color.purple, 30)
    data.valuesAbove(99) <= 80  => color.new(color.purple, 20)
    data.valuesAbove(99) <= 90  => color.new(color.purple, 10)
    data.valuesAbove(99) <= 100 => color.new(color.purple, 0)

// Plot the number values in the `data` array above the value at its last index. 
plot(data.valuesAbove(99), color = plotColor, style = plot.style_area)
```

Os [resultados perfilados](./06_03_perfilamento_e_otimizacao.md#interpretando-resultados-perfilados) para este script mostram que ele gastou cerca de 2,5 segundos executando 21.201 vezes. As regiões de código com maior impacto no tempo de execução do script são o loop [for](https://br.tradingview.com/pine-script-reference/v5/#kw_for) dentro do escopo local de `valuesAbove()` começando na linha 8 e o bloco [switch](https://br.tradingview.com/pine-script-reference/v5/#kw_switch) que começa na linha 21:

![Reduzindo repetição 01](./imgs/Profiling-and-optimization-Optimization-Reducing-repetition-1.DzXiqnj9_Z1IB5Dn.webp)

Observe que o número de execuções mostrado para o código local dentro de `valuesAbove()` é substancialmente _maior_ do que o número mostrado para o código no escopo global do script, pois o script chama o método até 11 vezes por execução, e os resultados para o [código local da função](./06_03_perfilamento_e_otimizacao.md#chamadas-de-funções-definidas-pelo-usuário) refletem o tempo e as execuções _combinados_ de cada chamada separada:

![Reduzindo repetição 02](./imgs/Profiling-and-optimization-Optimization-Reducing-repetition-2.QnMs1Dg3_ZE1OU8.webp)

Embora cada chamada para `valuesAbove()` use os _mesmos_ argumentos e retorne o _mesmo_ resultado, o compilador não pode reduzir automaticamente esse código durante a tradução. Necessário fazer manualmente. Pode-se otimizar este script atribuindo o valor de `data.valuesAbove(99)` a uma _variável_ e _reutilizando_ o valor em todas as outras áreas que exigem o resultado.

Na versão abaixo, o script adiciona uma variável `count` para referenciar o valor de `data.valuesAbove(99)`. O script usa essa variável no cálculo de `plotColor` e na chamada de [plot()](https://br.tradingview.com/pine-script-reference/v5/#fun_plot):

```c
//@version=5
indicator("Reducing repetition demo")

//@function Counts the number of elements in `this` array above the element at a specified `index`.
method valuesAbove(array<float> this, int index) =>
    int result = 0
    float reference = this.get(index)
    for [i, value] in this
        if i == index
            continue
        if value > reference
            result += 1
    result

//@variable An array containing the most recent 100 `close` prices.
var array<float> data = array.new<float>(100)
data.push(close)
data.shift()

//@variable The number values in the `data` array above the value at its last index.
int count = data.valuesAbove(99)

//@variable Returns `color.purple` with a varying transparency based on the `valuesAbove()`.
color plotColor = switch
    count <= 10  => color.new(color.purple, 90)
    count <= 20  => color.new(color.purple, 80)
    count <= 30  => color.new(color.purple, 70)
    count <= 40  => color.new(color.purple, 60)
    count <= 50  => color.new(color.purple, 50)
    count <= 60  => color.new(color.purple, 40)
    count <= 70  => color.new(color.purple, 30)
    count <= 80  => color.new(color.purple, 20)
    count <= 90  => color.new(color.purple, 10)
    count <= 100 => color.new(color.purple, 0)

// Plot the `count`.
plot(count, color = plotColor, style = plot.style_area)
```

Com essa modificação, os [resultados perfilados](./06_03_perfilamento_e_otimizacao.md#interpretando-resultados-perfilados) mostram uma melhoria significativa no desempenho, pois o script agora só precisa avaliar a chamada `valuesAbove()` __uma vez__ por execução, em vez de até 11 vezes separadas:

![Reduzindo repetição 03](./imgs/Profiling-and-optimization-Optimization-Reducing-repetition-3.Nge1aqtk_2w60SG.webp)

__Note que:__

- Como este script só chama `valuesAbove()` uma vez, o [código local do método](./04_13_metodos.md#métodos-definidos-pelo-usuário) agora refletirá os resultados dessa chamada específica. Consulte [esta seção](./06_03_perfilamento_e_otimizacao.md#chamadas-de-funções-definidas-pelo-usuário) para aprender mais sobre como interpretar resultados de chamadas de funções e métodos perfilados.

### Minimizando Chamadas `request.*()`

As funções internas no namespace `request.*()` permitem que os scripts recuperem dados de [outros contextos](./05_14_outros_timeframes_e_dados.md). Embora essas funções sejam úteis em muitas aplicações, é importante considerar que cada chamada a essas funções pode ter um impacto significativo no uso de recursos de um script.

Um único script pode conter até 40 chamadas para a família de funções `request.*()`. No entanto, os usuários devem se esforçar para manter suas chamadas `request.*()` bem _abaixo_ desse limite para minimizar o impacto de desempenho de suas solicitações de dados.

Quando um script solicita os valores de várias expressões do _mesmo_ contexto com múltiplas chamadas de [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security) ou [request.security_lower_tf()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security_lower_tf), uma maneira eficaz de otimizar essas solicitações é _condensá-las_ em uma única chamada `request.*()` que usa uma [tupla](./05_14_outros_timeframes_e_dados.md#tuples-tuplas) como seu argumento `expression`. Essa otimização não apenas melhora o tempo de execução das solicitações; ela também ajuda a reduzir o _uso de memória_ e o tamanho compilado do script.

Como exemplo simples, o script a seguir solicita nove valores de [ta.percentrank()](https://br.tradingview.com/pine-script-reference/v5/#fun_ta.percentrank) com diferentes comprimentos de um símbolo especificado usando nove chamadas separadas de [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security). Em seguida, ele [plota](./05_15_plots.md) todos os nove valores solicitados no gráfico para utilizá-los nas saídas:

```c
//@version=5
indicator("Minimizing `request.*()` calls demo")

//@variable The symbol to request data from.
string symbolInput = input.symbol("BINANCE:BTCUSDT", "Symbol")

// Request 9 `ta.percentrank()` values from the `symbolInput` context using 9 `request.security()` calls.
float reqRank1 = request.security(symbolInput, timeframe.period, ta.percentrank(close, 10))
float reqRank2 = request.security(symbolInput, timeframe.period, ta.percentrank(close, 20))
float reqRank3 = request.security(symbolInput, timeframe.period, ta.percentrank(close, 30))
float reqRank4 = request.security(symbolInput, timeframe.period, ta.percentrank(close, 40))
float reqRank5 = request.security(symbolInput, timeframe.period, ta.percentrank(close, 50))
float reqRank6 = request.security(symbolInput, timeframe.period, ta.percentrank(close, 60))
float reqRank7 = request.security(symbolInput, timeframe.period, ta.percentrank(close, 70))
float reqRank8 = request.security(symbolInput, timeframe.period, ta.percentrank(close, 80))
float reqRank9 = request.security(symbolInput, timeframe.period, ta.percentrank(close, 90))

// Plot the `reqRank*` values.
plot(reqRank1)
plot(reqRank2)
plot(reqRank3)
plot(reqRank4)
plot(reqRank5)
plot(reqRank6)
plot(reqRank7)
plot(reqRank8)
plot(reqRank9)
```

Os resultados de [perfilamento do script](./06_03_perfilamento_e_otimizacao.md#profilando-um-script) mostram que o script levou 340,8 milissegundos para completar suas solicitações e plotar os valores nesta execução:

![Minimizando chamadas request.*() 01](./imgs/Profiling-and-optimization-Optimization-Minimizing-request-calls-1.Canv49So_1Cn30c.webp)

Como todas as chamadas de [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security) solicitam dados do __mesmo contexto__, podendo otimizar o uso de recursos do código mesclando todas elas em uma única chamada de [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security) que usa uma [tupla](./05_14_outros_timeframes_e_dados.md#tuples-tuplas) como seu argumento `expression`:

```c
//@version=5
indicator("Minimizing `request.*()` calls demo")

//@variable The symbol to request data from.
string symbolInput = input.symbol("BINANCE:BTCUSDT", "Symbol")

// Request 9 `ta.percentrank()` values from the `symbolInput` context using a single `request.security()` call.
[reqRank1, reqRank2, reqRank3, reqRank4, reqRank5, reqRank6, reqRank7, reqRank8, reqRank9] = 
 request.security(
     symbolInput, timeframe.period, [
             ta.percentrank(close, 10), ta.percentrank(close, 20), ta.percentrank(close, 30), 
             ta.percentrank(close, 40), ta.percentrank(close, 50), ta.percentrank(close, 60), 
             ta.percentrank(close, 70), ta.percentrank(close, 80), ta.percentrank(close, 90)
         ]
 )

// Plot the `reqRank*` values.
plot(reqRank1)
plot(reqRank2)
plot(reqRank3)
plot(reqRank4)
plot(reqRank5)
plot(reqRank6)
plot(reqRank7)
plot(reqRank8)
plot(reqRank9)
```

Visto abaixo, os [resultados perfilados](./06_03_perfilamento_e_otimizacao.md#interpretando-resultados-perfilados) desta versão do script mostram que ele levou 228,3 milissegundos desta vez, uma melhoria considerável em relação à execução anterior:

![Minimizando chamadas request.*() 02](./imgs/Profiling-and-optimization-Optimization-Minimizing-request-calls-2.BZ75zb8R_Z21D1UV.webp)

__Note que:__

- Os recursos computacionais disponíveis para um script __flutuam__ ao longo do tempo. Portanto, geralmente é uma boa ideia perfilar um script [várias vezes](./06_03_perfilamento_e_otimizacao.md#perfilamento-repetitivo) para ajudar a solidificar as conclusões de desempenho.
- Outra maneira de solicitar vários valores do mesmo contexto com uma única chamada `request.*()` é passar um [objeto](./04_12_objetos.md) de um [tipo definido pelo usuário (UDT)](./04_09_tipagem_do_sistema.md#tipos-definidos-pelo-usuário) como argumento `expression`. Veja [esta seção](./05_14_outros_timeframes_e_dados.md#tipos-definidos-pelo-usuário) da página [Outros tempos e dados](./05_14_outros_timeframes_e_dados.md) para aprender mais sobre como solicitar [UDTs](./04_09_tipagem_do_sistema.md#tipos-definidos-pelo-usuário).
- Programadores também podem reduzir o tempo total de execução de uma chamada de [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security), [request.security_lower_tf()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.security_lower_tf) ou [request.seed()](https://br.tradingview.com/pine-script-reference/v5/#fun_request.seed) passando um argumento para o parâmetro `calc_bars_count` da função, que _restringe_ o número de pontos de dados _históricos_ que ela pode acessar de um contexto e executar cálculos necessários. Em geral, se as chamadas para essas funções `request.*()` recuperarem _mais_ dados históricos do que um script _necessita_, limitar as solicitações com `calc_bars_count` pode ajudar a melhorar o desempenho do script.

### Evitando Redesenhos

Os [tipos de desenho](./04_09_tipagem_do_sistema.md#tipos-de-desenho) do Pine Script permitem que scripts desenhem visuais personalizados em um gráfico que não podem ser alcançados através de outras saídas, como [plots](./05_15_plots.md). Embora esses tipos forneçam maior flexibilidade visual, eles também têm um custo _maior_ de tempo de execução e memória, especialmente quando um script _recria_ desnecessariamente desenhos em vez de atualizar diretamente suas propriedades para alterar sua aparência.

A maioria dos [tipos de desenho](./04_09_tipagem_do_sistema.md#tipos-de-desenho), exceto [polilinhas](./05_12_lines_e_boxes.md#polylines-polilinhas), possui _funções setter_ internas em seus namespaces que permitem que scripts modifiquem um desenho _sem_ deletar e recriar. Utilizar esses setters é tipicamente menos custoso computacionalmente do que criar um novo objeto de desenho quando apenas _propriedades específicas_ exigem modificação.

Por exemplo, o script abaixo compara deletar e redesenhar [caixas](./05_12_lines_e_boxes.md#boxes-caixas) usando funções `box.set*()`. Na primeira barra, ele declara os arrays `redrawnBoxes` e `updatedBoxes` e executa um [loop](./04_08_loops.md) para adicionar 25 elementos de [box](https://br.tradingview.com/pine-script-reference/v5/#type_box) a eles.

O script usa um loop [for](https://br.tradingview.com/pine-script-reference/v5/#kw_for) separado para iterar pelos [arrays](./04_14_arrays.md) e atualizar os desenhos em cada execução. Ele _recria_ as [caixas](./05_12_lines_e_boxes.md#boxes-caixas) no array `redrawnBoxes` usando [box.delete()](https://br.tradingview.com/pine-script-reference/v5/#fun_box.delete) e [box.new()](https://br.tradingview.com/pine-script-reference/v5/#fun_box.new), enquanto _modifica diretamente_ as propriedades das [caixas](./05_12_lines_e_boxes.md#boxes-caixas) no array `updatedBoxes` usando [box.set_lefttop()](https://br.tradingview.com/pine-script-reference/v5/#fun_box.set_lefttop) e [box.set_rightbottom()](https://br.tradingview.com/pine-script-reference/v5/#fun_box.set_rightbottom). Ambas as abordagens alcançam o mesmo resultado visual. No entanto, a última é mais eficiente:

```c
//@version=5
indicator("Avoiding redrawing demo")

//@variable An array of `box` IDs deleted with `box.delete()` and redrawn with `box.new()` on each execution.
var array<box> redrawnBoxes = array.new<box>()
//@variable An array of `box` IDs with properties that update across executions update via `box.set*()` functions.
var array<box> updatedBoxes = array.new<box>()

// Populate both arrays with 25 elements on the first bar. 
if barstate.isfirst
    for i = 1 to 25
        array.push(redrawnBoxes, box(na))
        array.push(updatedBoxes, box.new(na, na, na, na))

for i = 0 to 24
    // Calculate coordinates.
    int x = bar_index - i
    float y = close[i + 1] - close
    // Get the `box` ID from each array at the `i` index.
    box redrawnBox = redrawnBoxes.get(i)
    box updatedBox = updatedBoxes.get(i)
    // Delete the `redrawnBox`, create a new `box` ID, and replace that element in the `redrawnboxes` array.
    box.delete(redrawnBox)
    redrawnBox := box.new(x - 1, y, x, 0.0)
    array.set(redrawnBoxes, i, redrawnBox)
    // Update the properties of the `updatedBox` rather than redrawing it. 
    box.set_lefttop(updatedBox, x - 1, y)
    box.set_rightbottom(updatedBox, x, 0.0)
```

Os resultados de [perfilamento deste script](./06_03_perfilamento_e_otimizacao.md#profilando-um-script) mostram que a linha 24, que contém a chamada de [box.new()](https://br.tradingview.com/pine-script-reference/v5/#fun_box.new), é a linha mais _pesada_ no [bloco de código](./06_03_perfilamento_e_otimizacao.md#resultados-de-blocos-de-código) que executa em cada barra, com um tempo de execução quase _dobro_ do tempo combinado gasto nas chamadas [box.set_lefttop()](https://br.tradingview.com/pine-script-reference/v5/#fun_box.set_lefttop) e [box.set_rightbottom()](https://br.tradingview.com/pine-script-reference/v5/#fun_box.set_rightbottom) nas linhas 27 e 28:

![Evitando redesenhos](./imgs/Profiling-and-optimization-Optimization-Avoiding-redrawing-1.CVCJc2lm_ZJcTqr.webp)

__Note que:__

- O número de execuções mostrado para o _código local_ do loop é 25 vezes o número mostrado para o código no _escopo global_ do script, já que cada execução da instrução do loop aciona 25 execuções do bloco local.
- Este script atualiza seus desenhos em _todas as barras_ no histórico do gráfico para fins de __teste__. No entanto, ele __não__ precisa realmente executar todas essas atualizações históricas, pois os usuários só verão o __resultado final__ da _última barra histórica_ e as mudanças nas _barras em tempo real_. Veja a [próxima seção](./06_03_perfilamento_e_otimizacao.md#reduzindo-atualizações-de-desenho) para aprender mais.

### Reduzindo Atualizações de Desenho

Quando um script produz [objetos de desenho](./04_09_tipagem_do_sistema.md#tipos-de-desenho) que mudam em _barras históricas_, os usuários só verão seus __resultados finais__ nessas barras, pois o script completa suas execuções históricas quando é carregado pela primeira vez no gráfico. A única vez que se verá esses desenhos _evoluírem_ em execuções é durante as _barras em tempo real_, à medida que novos dados são recebidos.

Como as saídas dinâmicas de [desenhos](./04_09_tipagem_do_sistema.md#tipos-de-desenho) em barras históricas são __nunca visíveis__ para um usuário, muitas vezes é possível melhorar o desempenho de um script _eliminando_ as atualizações históricas que não afetam os resultados finais.

Por exemplo, este script cria uma [tabela](https://br.tradingview.com/pine-script-reference/v5/#type_table) com duas colunas e 21 linhas para visualizar o histórico de um [RSI](https://br.tradingview.com/pine-script-reference/v5/#fun_ta.rsi) em um formato tabular paginado. O script inicializa as células da `infoTable` na [primeira barra](https://br.tradingview.com/pine-script-reference/v5/#var_barstate.isfirst), e referencia o histórico do `rsi` calculado para atualizar o `text` e `bgcolor` das células na segunda coluna dentro de um loop [for](https://br.tradingview.com/pine-script-reference/v5/#kw_for) em cada barra:

```c
//@version=5
indicator("Reducing drawing updates demo")

//@variable The first offset shown in the paginated table.
int offsetInput = input.int(0, "Page", 0, 249) * 20

//@variable A table that shows the history of RSI values.
var table infoTable = table.new(position.top_right, 2, 21, border_color = chart.fg_color, border_width = 1)
// Initialize the table's cells on the first bar.
if barstate.isfirst
    table.cell(infoTable, 0, 0, "Offset", text_color = chart.fg_color)
    table.cell(infoTable, 1, 0, "RSI", text_color = chart.fg_color)
    for i = 0 to 19
        table.cell(infoTable, 0, i + 1, str.tostring(offsetInput + i))
        table.cell(infoTable, 1, i + 1)

float rsi = ta.rsi(close, 14)

// Update the history shown in the `infoTable` on each bar. 
for i = 0 to 19
    float historicalRSI = rsi[offsetInput + i]
    table.cell_set_text(infoTable, 1, i + 1, str.tostring(historicalRSI))
    table.cell_set_bgcolor(
         infoTable, 1, i + 1, color.from_gradient(historicalRSI, 30, 70, color.red, color.green)
     )

plot(rsi, "RSI")
```

Após [perfilar](./06_03_perfilamento_e_otimizacao.md#profilando-um-script) o script, o código com maior impacto no desempenho é o loop [for](https://br.tradingview.com/pine-script-reference/v5/#kw_for) que começa na linha 20, ou seja, o [bloco de código](./06_03_perfilamento_e_otimizacao.md#resultados-de-blocos-de-código) que atualiza as células da tabela:

![Reduzindo atualizações de desenho 01](./imgs/Profiling-and-optimization-Optimization-Reducing-drawing-updates-1.DGjGYc9o_Z2h4bJz.webp)

Essa região crítica do código é executada excessivamente ao longo do histórico do gráfico, pois os usuários só verão o resultado final da [tabela](./05_19_tables.md) no histórico. O único momento em que os usuários verão a [tabela](https://br.tradingview.com/pine-script-reference/v5/#type_table) ser atualizada é na última barra histórica e em todas as barras em tempo real subsequentes. Portanto, otimizar o uso de recursos deste script restringindo a execução desse código apenas à [última barra disponível](https://br.tradingview.com/pine-script-reference/v5/#var_barstate.islast) é eficaz.

Nesta versão do script, o loop [for](./04_08_loops.md) que atualiza as células da [tabela](https://br.tradingview.com/pine-script-reference/v5/#type_table) está dentro de uma estrutura [if](https://br.tradingview.com/pine-script-reference/v5/#kw_if) que usa [barstate.islast](https://br.tradingview.com/pine-script-reference/v5/#var_barstate.islast) como condição, restringindo efetivamente as execuções do bloco de código apenas à última barra histórica e todas as barras em tempo real. Agora, o script carrega mais eficientemente, já que todos os cálculos da tabela requerem apenas uma execução histórica:

![Reduzindo atualizações de desenho 02](./imgs/Profiling-and-optimization-Optimization-Reducing-drawing-updates-2.DVDSH-lG_Z1AqBk6.webp)

```c
//@version=5
indicator("Reducing drawing updates demo")

//@variable The first offset shown in the paginated table.
int offsetInput = input.int(0, "Page", 0, 249) * 20

//@variable A table that shows the history of RSI values.
var table infoTable = table.new(position.top_right, 2, 21, border_color = chart.fg_color, border_width = 1)
// Initialize the table's cells on the first bar.
if barstate.isfirst
    table.cell(infoTable, 0, 0, "Offset", text_color = chart.fg_color)
    table.cell(infoTable, 1, 0, "RSI", text_color = chart.fg_color)
    for i = 0 to 19
        table.cell(infoTable, 0, i + 1, str.tostring(offsetInput + i))
        table.cell(infoTable, 1, i + 1)

float rsi = ta.rsi(close, 14)

// Update the history shown in the `infoTable` on the last available bar.
if barstate.islast
    for i = 0 to 19
        float historicalRSI = rsi[offsetInput + i]
        table.cell_set_text(infoTable, 1, i + 1, str.tostring(historicalRSI))
        table.cell_set_bgcolor(
             infoTable, 1, i + 1, color.from_gradient(historicalRSI, 30, 70, color.red, color.green)
         )

plot(rsi, "RSI")
```

__Note que:__

- O script ainda atualizará as células quando novas atualizações em tempo real chegarem, pois os usuários podem observar essas mudanças no gráfico, ao contrário das mudanças que o script executava ao longo das barras históricas.

### Armazenando Valores Calculados

Quando um script realiza um cálculo crítico que muda infrequentemente ao longo das execuções, pode-se reduzir seu tempo de execução salvando o resultado em uma variável declarada com as palavras-chave [var](https://br.tradingview.com/pine-script-reference/v5/#kw_var) ou [varip](https://br.tradingview.com/pine-script-reference/v5/#kw_varip) e somente atualizando o valor se o cálculo mudar. Se o script calcular múltiplos valores excessivamente, pode-se armazená-los em [coleções](./04_09_tipagem_do_sistema.md#coleções), [_matrices_](./04_15_matrices.md), [maps](./04_16_mapas.md) ou [objetos](./04_12_objetos.md) de [tipos definidos pelo usuário](./04_09_tipagem_do_sistema.md#tipos-definidos-pelo-usuário).

Este exemplo calcula uma média móvel ponderada com _weight_ personalizados baseados em uma [_window function_](https://en.wikipedia.org/wiki/Window_function) generalizada. O `numerator` é a soma dos valores de [close](https://br.tradingview.com/pine-script-reference/v5/#var_close) ponderados, e o `denominator` é a soma dos _weight_ calculados. O script usa um loop [for](https://br.tradingview.com/pine-script-reference/v5/#kw_for) que itera `lengthInput` vezes para calcular essas somas, em seguida plota sua razão, ou seja, a média resultante:

```c
//@version=5
indicator("Storing calculated values demo", overlay = true)

//@variable The number of bars in the weighted average calculation.
int lengthInput = input.int(50, "Length", 1, 5000)
//@variable Window coefficient. 
float coefInput = input.float(0.5, "Window coefficient", 0.0, 1.0, 0.01)

//@variable The sum of weighted `close` prices.
float numerator = 0.0
//@variable The sum of weights.
float denominator = 0.0

//@variable The angular step in the cosine calculation.
float step = 2.0 * math.pi / lengthInput
// Accumulate weighted sums.
for i = 0 to lengthInput - 1
    float weight = coefInput - (1 - coefInput) * math.cos(step * i)
    numerator += close[i] * weight
    denominator += weight

// Plot the weighted average result.
plot(numerator / denominator, "Weighted average", color.purple, 3)
```

Após [perfilar](./06_03_perfilamento_e_otimizacao.md#profilando-um-script) o desempenho do script nos dados do gráfico, levou cerca de 241,3 milissegundos para calcular a média padrão de 50 barras em 20.155 atualizações de gráfico, e o código crítico com maior impacto no desempenho do script é o [bloco](./06_03_perfilamento_e_otimizacao.md#resultados-de-blocos-de-código) de loop que começa na linha 17:

![Armazenando valores calculados 01](./imgs/Profiling-and-optimization-Optimization-Storing-calculated-values-1.Db6QOvTY_1SK8Tz.webp)

Como o número de iterações do loop depende do valor de `lengthInput`, testar como seu tempo de execução se escala com [outra configuração](./06_03_perfilamento_e_otimizacao.md#profilando-entre-configurações) que exige mais iterações é relevante. Aqui, o valor foi configurado para 2500. Desta vez, o script levou cerca de 12 segundos para completar todas as execuções:

![Armazenando valores calculados 02](./imgs/Profiling-and-optimization-Optimization-Storing-calculated-values-2.DsxEbDvA_DT4zd.webp)

Agora que o código de alto impacto do script foi identificado e um benchmark para melhorar estabelecido, inspecionar o bloco de código crítico para identificar oportunidades de otimização é fundamental. Após examinar os cálculos, é observado o seguinte:

- O único valor que faz com que o cálculo de `weight` na linha 18 varie entre as iterações do loop é o índice do loop. Todos os outros valores em seu cálculo permanecem consistentes. Consequentemente, o `weight` calculado em cada iteração do loop não varia entre as barras do gráfico. Portanto, em vez de calcular os _weight_ em cada atualização, calcular uma vez, na primeira barra, e armazená-los em uma [coleção](./04_09_tipagem_do_sistema.md#coleções) para acesso futuro nas execuções subsequentes do script é adequado.
- Como os _weight_ nunca mudam, o `denominator` resultante nunca muda. Portanto, adicionar a palavra-chave [var](https://br.tradingview.com/pine-script-reference/v5/#kw_var) à [declaração da variável](./04_06_declaracoes_de_variavel.md) e calcular seu valor uma vez reduz o número de operações de [atribuição de adição](https://br.tradingview.com/pine-script-reference/v5/#op_+=) executadas.
- Ao contrário do `denominator`, não é possível armazenar o valor do `numerator` para simplificar seu cálculo, pois ele muda consistentemente ao longo do tempo.

No script modificado abaixo, foi adicionada uma variável `weights` para referenciar um [array](https://br.tradingview.com/pine-script-reference/v5/#type_array) que armazena cada `weight` calculado. Essa variável e o `denominator` incluem a palavra-chave [var](https://br.tradingview.com/pine-script-reference/v5/#kw_var) em suas declarações, significando que os valores atribuídos a elas _permanecerão_ ao longo de todas as execuções do script até serem explicitamente reatribuídos. O script calcula seus valores usando um loop [for](https://br.tradingview.com/pine-script-reference/v5/#kw_for) que só é executado na [primeira barra do gráfico](https://br.tradingview.com/pine-script-reference/v5/#var_barstate.isfirst). Em todas as outras barras, ele calcula o `numerator` usando um loop [for…in](https://br.tradingview.com/pine-script-reference/v5/#kw_for...in) que referencia os _valores salvos_ do array `weights`:

```c
//@version=5
indicator("Storing calculated values demo", overlay = true)

//@variable The number of bars in the weighted average calculation.
int lengthInput = input.int(50, "Length", 1, 5000)
//@variable Window coefficient. 
float coefInput = input.float(0.5, "Window coefficient", 0.0, 1.0, 0.01)

//@variable An array that stores the `weight` values calculated on the first chart bar. 
var array<float> weights = array.new<float>()

//@variable The sum of weighted `close` prices.
float numerator = 0.0
//@variable The sum of weights. The script now only calculates this value on the first bar. 
var float denominator = 0.0

//@variable The angular step in the cosine calculation.
float step = 2.0 * math.pi / lengthInput

// Populate the `weights` array and calculate the `denominator` only on the first bar.
if barstate.isfirst
    for i = 0 to lengthInput - 1
        float weight = coefInput - (1 - coefInput) * math.cos(step * i)
        array.push(weights, weight)
        denominator += weight
// Calculate the `numerator` on each bar using the stored `weights`. 
for [i, w] in weights
    numerator += close[i] * w

// Plot the weighted average result.
plot(numerator / denominator, "Weighted average", color.purple, 3)
```

Com essa estrutura otimizada, os [resultados perfilados](./06_03_perfilamento_e_otimizacao.md#interpretando-resultados-perfilados) mostram que o script modificado com um valor alto de `lengthInput` de 2500 levou cerca de 5,9 segundos para calcular nos mesmos dados, cerca de _metade_ do tempo da versão anterior:

![Armazenando valores calculados 03](./imgs/Profiling-and-optimization-Optimization-Storing-calculated-values-3.k6QQwb-Z_ZqmRWt.webp)

__Note que:__

- Embora o desempenho deste script tenha sido significativamente melhorado ao salvar seus valores invariantes de execução em variáveis, ele ainda envolve um custo computacional mais alto com valores grandes de `lengthInput` devido aos cálculos de loop restantes que são executados em cada barra.
- Outra maneira mais _avançada_ de melhorar ainda mais o desempenho deste script é armazenar os _weight_ em uma [_matrix_](https://br.tradingview.com/pine-script-reference/v5/#type_matrix) de uma única linha na primeira barra, usar um [array](https://br.tradingview.com/pine-script-reference/v5/#type_array) como [fila](./04_14_arrays.md#utilizando-um-array-como-uma-fila) para armazenar os valores recentes de close e substituir o loop for…in por uma chamada para [matrix.mult()](https://br.tradingview.com/pine-script-reference/v5/#fun_matrix.mult). Veja a página de [_Matrices_](./04_15_matrices.md) para aprender mais sobre como trabalhar com funções `matrix.*()`.

### Eliminando Loops

[Loops](./04_08_loops.md) permitem que scripts Pine realizem cálculos _iterativos_ em cada execução. Cada vez que um loop é ativado, seu código local pode ser executado _várias vezes_, muitas vezes levando a um _aumento substancial_ no uso de recursos.

Loops em Pine são necessários para _alguns_ cálculos, como manipular elementos dentro de [coleções](./04_09_tipagem_do_sistema.md#coleções) ou olhar para trás através do histórico de um conjunto de dados para calcular valores _somente_ obtidos na barra atual. No entanto, em muitos outros casos, os programadores usam loops quando __não precisam__, levando a um desempenho de tempo de execução subótimo. Nesses casos, pode-se eliminar loops desnecessários de várias maneiras, dependendo do que seus cálculos envolvem:

- Identificar expressões simplificadas, __sem loops__, que alcancem o mesmo resultado sem iteração.
- Substituir um loop por funções otimizadas [internas](./06_03_perfilamento_e_otimizacao.md#usando-funções-incorporadas) sempre que possível
- Distribuir as iterações de um loop _através das barras_ quando viável, em vez de avaliá-las todas de uma vez.

Este exemplo simples contém uma função `avgDifference()` que calcula a diferença média entre o valor `source` da barra atual e todos os valores das `length` barras anteriores. O script chama essa função para calcular a diferença média entre o preço de [fechamento](https://br.tradingview.com/pine-script-reference/v5/#var_close) atual e os preços anteriores de `lengthInput`, depois [plota](./05_15_plots.md) o resultado no gráfico:

```c
//@version=5
indicator("Eliminating loops demo")

//@variable The number of bars in the calculation.
int lengthInput = input.int(20, "Length", 1)

//@function Calculates the average difference between the current `source` and `length` previous `source` values.
avgDifference(float source, int length) =>
    float diffSum = 0.0
    for i = 1 to length
        diffSum += source - source[i]
    diffSum / length

plot(avgDifference(close, lengthInput))
```

Após inspecionar os [resultados perfilados](./06_03_perfilamento_e_otimizacao.md#interpretando-resultados-perfilados) do script com as configurações padrão, vemos que ele levou cerca de 64 milissegundos para executar 20.157 vezes:

![Eliminando loops 01](./imgs/Profiling-and-optimization-Optimization-Eliminating-loops-1.CagHTlaL_Z1XidNw.webp)

Como o `lengthInput` é usado como argumento `length` na chamada `avgDifference()` e esse argumento controla quantas vezes o loop dentro da função deve iterar, o tempo de execução do script __aumentará__ com o valor de `lengthInput`. Aqui, o valor de entrada foi configurado para 2000 nas configurações do script. Desta vez, o script completou suas execuções em cerca de 3,8 segundos:

![Eliminando loops 02](./imgs/Profiling-and-optimization-Optimization-Eliminating-loops-2.GHOgXfCo_Z2hwC8b.webp)

Como visto nesses resultados, a função `avgDifference()` pode ser custosa de chamar, dependendo do valor especificado de `lengthInput`, devido ao loop [for](https://br.tradingview.com/pine-script-reference/v5/#kw_for) que executa em cada barra. No entanto, [loops](./04_08_loops.md) __não__ são necessários para alcançar o resultado. Para entender por quê, vamos examinar mais de perto os cálculos do loop. Podemos representá-los com a seguinte expressão:

```c
(source - source[1]) + (source - source[2]) + ... + (source - source[length])
```

Note que ele adiciona o valor _atual_ de `source` `length` vezes. Essas adições iterativas não são necessárias. Podemos simplificar essa parte da expressão para `source * length`, o que a reduz para o seguinte:

```c
source * length - source[1] - source[2] - ... - source[length]
```

Ou de forma equivalente:

```c
source * length - (source[1] + source[2] + ... + source[length])
```

Após simplificar e reorganizar essa representação dos cálculos do loop, podemos ver que é possível calcular o resultado de uma maneira mais simples e __eliminar__ o loop subtraindo a [soma cumulativa](https://br.tradingview.com/pine-script-reference/v5/#fun_math.sum) dos valores `source` da barra anterior de `source * length`, ou seja:

```c
source * length - math.sum(source, length)[1]
```

A função `fastAvgDifference()` abaixo é uma alternativa __sem loop__ à função original `avgDifference()`, que usa a expressão acima para calcular a soma das diferenças `source`, depois divide a expressão pelo `length` para retornar a diferença média:

```c
//@function A faster way to calculate the `avgDifference()` result. 
//          Eliminates the `for` loop using the relationship: 
//          `(x - x[1]) + (x - x[2]) + ... + (x - x[n]) = x * n - math.sum(x, n)[1]`.
fastAvgDifference(float source, int length) =>
    (source * length - math.sum(source, length)[1]) / length
```

Agora que uma solução potencial otimizada foi identificada, é possível comparar o desempenho de `fastAvgDifference()` com a função original `avgDifference()`. O script abaixo é uma versão modificada da anterior que plota os resultados das chamadas de ambas as funções com `lengthInput` como argumento `length`:

```c
//@version=5
indicator("Eliminating loops demo")

//@variable The number of bars in the calculation.
int lengthInput = input.int(20, "Length", 1)

//@function Calculates the average difference between the current `source` and `length` previous `source` values.
avgDifference(float source, int length) =>
    float diffSum = 0.0
    for i = 1 to length
        diffSum += source - source[i]
    diffSum / length

//@function A faster way to calculate the `avgDifference()` result. 
//          Eliminates the `for` loop using the relationship: 
//          `(x - x[1]) + (x - x[2]) + ... + (x - x[n]) = x * n - math.sum(x, n)[1]`.
fastAvgDifference(float source, int length) =>
    (source * length - math.sum(source, length)[1]) / length

plot(avgDifference(close, lengthInput))
plot(fastAvgDifference(close, lengthInput))
```

Os [resultados perfilados](./06_03_perfilamento_e_otimizacao.md#interpretando-resultados-perfilados) do script com o `lengthInput` padrão de 20 mostram uma diferença substancial no tempo de execução gasto nas duas chamadas de função. A chamada para a função original levou cerca de 47,3 milissegundos para executar 20.157 vezes nesta execução, enquanto a função otimizada levou apenas 4,5 milissegundos:

![Eliminando loops 03](./imgs/Profiling-and-optimization-Optimization-Eliminating-loops-3.DiuPpBDh_1dyGfx.webp)

Agora, comparar o desempenho com o valor _maior_ de `lengthInput` de 2000 é relevante. Como antes, o tempo de execução gasto na função `avgDifference()` aumentou significativamente. No entanto, o tempo gasto na execução da função `fastAvgDifference()` permaneceu muito próximo ao resultado da [configuração](./06_03_perfilamento_e_otimizacao.md#profilando-entre-configurações) anterior. Em outras palavras, enquanto o tempo de execução da função original escala diretamente com seu argumento `length`, a função otimizada demonstra desempenho relativamente _consistente_ por não exigir um loop:

![Eliminando loops 04](./imgs/Profiling-and-optimization-Optimization-Eliminating-loops-4.V8AwhZcD_23XFmP.webp)

> __Observação!__\
> Nem todos os cálculos iterativos terão necessariamente alternativas sem loop. No caso de um script só conseguir alcançar seus resultados por meio de iteração, os programadores podem identificar maneiras possíveis de otimizar loops para melhorar o desempenho. Veja a [próxima seção](./06_03_perfilamento_e_otimizacao.md#otimizando-loops) para mais informações.

### Otimizando Loops

Embora o [modelo de execução](./04_01_modelo_de_execucao.md) do Pine e as funções internas disponíveis frequentemente _eliminem_ a necessidade de [loops](./04_08_loops.md) em muitos casos, ainda há instâncias em que um script __necessitará__ de [loops](./04_08_loops.md) para alguns tipos de tarefas, incluindo:

- Manipular [coleções](./04_09_tipagem_do_sistema.md#coleções) ou executar cálculos sobre os elementos de uma coleção quando as funções internas disponíveis __não são__ suficientes.
- Realizar cálculos em barras históricas que __não podem__ ser alcançados com expressões simplificadas sem loop ou funções internas otimizadas.
- Calcular valores que são __somente__ obtíveis através de iteração.

Quando um script usa [loops](./04_08_loops.md) que um programador não pode [eliminar](./06_03_perfilamento_e_otimizacao.md#eliminando-loops), há [várias técnicas](https://en.wikipedia.org/wiki/Loop_optimization) que podem ser usadas para reduzir seu impacto no desempenho. Esta seção explica duas das técnicas mais comuns e úteis que podem ajudar a melhorar a eficiência de um loop necessário.

> __Observação!__\
> Antes de identificar maneiras de _otimizar_ um loop, recomenda-se procurar maneiras de [eliminá-lo](./06_03_perfilamento_e_otimizacao.md#eliminando-loops) primeiro. Se __não existir solução__ que torne o loop desnecessário, então prossiga tentando reduzir seu overhead.

### Reduzindo Cálculos em Loops

O código executado dentro do escopo local de um [loop](./04_08_loops.md) pode ter um impacto __multiplicativo__ no seu tempo de execução geral, já que cada vez que uma instrução de loop é executada, ela normalmente aciona _várias_ iterações do código local. Portanto, os programadores devem se esforçar para manter os cálculos de um loop o mais simples possível, eliminando estruturas, chamadas de funções e operações desnecessárias para minimizar o impacto no desempenho, especialmente quando o script precisa avaliar seus loops _várias vezes_ ao longo de todas as suas execuções.

Por exemplo, este script contém uma função `filteredMA()` que calcula uma média móvel de até `length` valores únicos de `source`, dependendo dos elementos `true` em um `mask` [array](https://br.tradingview.com/pine-script-reference/v5/#type_array) especificado. A função coloca os valores únicos de `source` em um `data` [array](https://br.tradingview.com/pine-script-reference/v5/#type_array), usa um loop [for…in](https://br.tradingview.com/pine-script-reference/v5/#kw_for...in) para iterar sobre o `data` e calcular as somas do `numerator` e `denominator`, e então retorna a razão dessas somas. Dentro do loop, ela só adiciona valores às somas quando o elemento `data` não é [na](https://br.tradingview.com/pine-script-reference/v5/#var_na) e o elemento `mask` no `index` é `true`. O script utiliza esta [função definida pelo usuário](./04_11_funcoes_definidas_pelo_usuario.md) para calcular a média de até 100 preços de [fechamento](https://br.tradingview.com/pine-script-reference/v5/#var_close) únicos filtrados por `randMask` e plota o resultado no gráfico:

```c
//@version=5
indicator("Reducing loop calculations demo", overlay = true)

//@function Calculates a moving average of up to `length` unique `source` values filtered by a `mask` array.
filteredMA(float source, int length, array<bool> mask) =>
    // Raise a runtime error if the size of the `mask` doesn't equal the `length`.
    if mask.size() != length
        runtime.error("The size of the `mask` array used in the `filteredMA()` call must match the `length`.")
    //@variable An array containing `length` unique `source` values.
    var array<float> data = array.new<float>(length)
    // Queue unique `source` values into the `data` array.
    if not data.includes(source)
        data.push(source)
        data.shift()
    // The numerator and denominator of the average.
    float numerator   = 0.0
    float denominator = 0.0
    // Loop to calculate sums.
    for item in data
        if na(item)
            continue
        int index = array.indexof(data, item)
        if mask.get(index)
            numerator   += item
            denominator += 1.0
    // Return the average, or the last non-`na` average value if the current value is `na`.
    fixnan(numerator / denominator)

//@variable An array of 100 pseudorandom "bool" values.
var array<bool> randMask = array.new<bool>(100, true)
// Push the first element from `randMask` to the end and queue a new pseudorandom value.
randMask.push(randMask.shift())
randMask.push(math.random(seed = 12345) < 0.5)
randMask.shift()

// Plot the `filteredMA()` of up to 100 unique `close` values filtered by the `randMask`.
plot(filteredMA(close, 100, randMask))
```

Após [perfilar o script](./06_03_perfilamento_e_otimizacao.md#profilando-um-script), observou-se que ele levou cerca de dois segundos para executar 21.778 vezes. O código com maior impacto no desempenho é a expressão na linha 37, que chama a função `filteredMA()`. Dentro do escopo da função `filteredMA()`, o loop [for…in](https://br.tradingview.com/pine-script-reference/v5/#kw_for...in) tem o maior impacto, com o cálculo de `index` no escopo do loop (linha 22) contribuindo mais para o tempo de execução do loop:

![Reduzindo cálculos em loops 01](./imgs/Profiling-and-optimization-Optimization-Optimizing-loops-Reducing-loop-calculations-1.DATOq5cw_2jUPjt.webp)

O código acima demonstra o uso subótimo de um loop [for…in](https://br.tradingview.com/pine-script-reference/v5/#kw_for...in), pois __não__ é necessário chamar [array.indexof()](https://br.tradingview.com/pine-script-reference/v5/#fun_array.indexof) para recuperar o `index` neste caso. A função [array.indexof()](https://br.tradingview.com/pine-script-reference/v5/#fun_array.indexof) pode ser _custosa_ de chamar dentro de um loop, pois precisa procurar nos conteúdos do [array](./04_14_arrays.md) e localizar o índice do elemento correspondente _cada vez_ que o script a chama.

Para eliminar essa chamada custosa do loop [for…in](https://br.tradingview.com/pine-script-reference/v5/#kw_for...in), pode-se usar a _segunda forma_ da estrutura, que produz uma _tupla_ contendo o __index__ e o valor do elemento em cada iteração:

```c
for [index, item] in data
```

Nesta versão do script, a chamada [array.indexof()](https://br.tradingview.com/pine-script-reference/v5/#fun_array.indexof) na linha 22 foi removida, pois __não__ é necessária para alcançar o resultado pretendido, e o loop [for…in](https://br.tradingview.com/pine-script-reference/v5/#kw_for...in) foi alterado para usar a forma alternativa:

```c
//@version=5
indicator("Reducing loop calculations demo", overlay = true)

//@function Calculates a moving average of up to `length` unique `source` values filtered by a `mask` array.
filteredMA(float source, int length, array<bool> mask) =>
    // Raise a runtime error if the size of the `mask` doesn't equal the `length`.
    if mask.size() != length
        runtime.error("The size of the `mask` array used in the `filteredMA()` call must match the `length`.")
    //@variable An array containing `length` unique `source` values.
    var array<float> data = array.new<float>(length)
    // Queue unique `source` values into the `data` array.
    if not data.includes(source)
        data.push(source)
        data.shift()
    // The numerator and denominator of the average.
    float numerator   = 0.0
    float denominator = 0.0
    // Loop to calculate sums.
    for [index, item] in data
        if na(item)
            continue
        if mask.get(index)
            numerator   += item
            denominator += 1.0
    // Return the average, or the last non-`na` average value if the current value is `na`.
    fixnan(numerator / denominator)

//@variable An array of 100 pseudorandom "bool" values.
var array<bool> randMask = array.new<bool>(100, true)
// Push the first element from `randMask` to the end and queue a new pseudorandom value.
randMask.push(randMask.shift())
randMask.push(math.random(seed = 12345) < 0.5)
randMask.shift()

// Plot the `filteredMA()` of up to 100 unique `close` values filtered by the `randMask`. 
plot(filteredMA(close, 100, randMask))
```

Com essa simples mudança, o loop está muito mais eficiente, pois não precisa mais procurar redundantemente no [array](https://br.tradingview.com/pine-script-reference/v5/#type_array) em cada iteração para acompanhar o índice. Os [resultados perfilados](./06_03_perfilamento_e_otimizacao.md#interpretando-resultados-perfilados) desta execução do script mostram que ele levou apenas 0,6 segundos para completar suas execuções, uma melhoria significativa em relação ao resultado da versão anterior:

![Reduzindo cálculos em loops 02](./imgs/Profiling-and-optimization-Optimization-Optimizing-loops-Reducing-loop-calculations-2.ChKixPxa_1dGXff.webp)

### Moção do Código Invariante ao Loop

Código _invariante ao loop_ é qualquer região de código dentro do escopo de um [loop](./04_08_loops.md) que produz um resultado __imutável__ em cada iteração. Quando os [loops](./04_08_loops.md) de um script contêm código invariante, isso pode impactar substancialmente o desempenho devido a cálculos __desnecessários__ e excessivos.

Programadores podem otimizar um loop com código invariante movendo os cálculos imutáveis __fora__ do escopo do loop, para que o script precise avaliá-los apenas uma vez por execução, em vez de repetidamente.

O exemplo a seguir contém uma função `featureScale()` que cria uma versão reescalada de um [array](https://br.tradingview.com/pine-script-reference/v5/#type_array). Dentro do loop [for…in](https://br.tradingview.com/pine-script-reference/v5/#kw_for...in) da função, cada elemento é escalado calculando sua distância do [array.min()](https://br.tradingview.com/pine-script-reference/v5/#fun_array.min) e dividindo o valor pelo [array.range()](https://br.tradingview.com/pine-script-reference/v5/#fun_array.range). O script usa essa função para criar uma versão `rescaled` de um array de preços e [plota](./05_15_plots.md) a diferença entre os valores de [rescaled.first()](https://br.tradingview.com/pine-script-reference/v5/#fun_array.first) e [rescaled.avg()](https://br.tradingview.com/pine-script-reference/v5/#fun_array.avg) no gráfico:

```c
//@version=5
indicator("Loop-invariant code motion demo")

//@function Returns a feature scaled version of `this` array.
featureScale(array<float> this) =>
    array<float> result = array.new<float>()
    for item in this
        result.push((item - array.min(this)) / array.range(this))
    result

//@variable An array containing the most recent 100 `close` prices.
var array<float> prices = array.new<float>(100, close)
// Queue the `close` through the `prices` array.
prices.unshift(close)
prices.pop()

//@variable A feature scaled version of the `prices` array.
array<float> rescaled = featureScale(prices)

// Plot the difference between the first element and the average value in the `rescaled` array.
plot(rescaled.first() - rescaled.avg())
```

Como visto abaixo, os [resultados perfilados](./06_03_perfilamento_e_otimizacao.md#interpretando-resultados-perfilados) para este script após 20.187 execuções mostram que ele completou sua execução em cerca de 3,3 segundos. O código com maior impacto no desempenho é a linha que contém a chamada da função `featureScale()`, e o código crítico da função é o bloco do loop [for…in](https://br.tradingview.com/pine-script-reference/v5/#kw_for...in) que começa na linha 7:

![Moção do código invariante ao loop 01](./imgs/Profiling-and-optimization-Optimization-Optimizing-loops-Loop-invariant-code-motion-1.BAn098-h_10uiO5.webp)

Ao examinar os cálculos do loop, é possível ver que as chamadas [array.min()](https://br.tradingview.com/pine-script-reference/v5/#fun_array.min) e [array.range()](https://br.tradingview.com/pine-script-reference/v5/#fun_array.range) na linha 8 são __invariantes ao loop__, pois sempre produzem o __mesmo resultado__ em cada iteração. Pode-se tornar o loop muito mais eficiente atribuindo os resultados dessas chamadas a variáveis __fora__ do escopo do loop e referenciando-as conforme necessário.

A função `featureScale()` no script abaixo atribui os valores de [array.min()](https://br.tradingview.com/pine-script-reference/v5/#fun_array.min) e [array.range()](https://br.tradingview.com/pine-script-reference/v5/#fun_array.range) às variáveis `minValue` e `rangeValue` _antes_ de executar o loop [for…in](https://br.tradingview.com/pine-script-reference/v5/#kw_for...in). Dentro do escopo local do loop, ela _referencia_ as variáveis em vez de chamar repetidamente essas funções `array.*()`:

```c
//@version=5
indicator("Loop-invariant code motion demo")

//@function Returns a feature scaled version of `this` array.
featureScale(array<float> this) =>
    array<float> result = array.new<float>()
    float minValue      = array.min(this)
    float rangeValue    = array.range(this)
    for item in this
        result.push((item - minValue) / rangeValue)
    result

//@variable An array containing the most recent 100 `close` prices.
var array<float> prices = array.new<float>(100, close)
// Queue the `close` through the `prices` array.
prices.unshift(close)
prices.pop()

//@variable A feature scaled version of the `prices` array.
array<float> rescaled = featureScale(prices)

// Plot the difference between the first element and the average value in the `rescaled` array.
plot(rescaled.first() - rescaled.avg())
```

Como visto nos [resultados perfilados](./06_03_perfilamento_e_otimizacao.md#interpretando-resultados-perfilados) do script, mover os cálculos _invariantes ao loop_ para fora do loop leva a uma melhoria substancial no desempenho. Desta vez, o script completou suas execuções em apenas 289,3 milissegundos:

![Moção do código invariante ao loop 02](./imgs/Profiling-and-optimization-Optimization-Optimizing-loops-Loop-invariant-code-motion-2.9LhBcnjw_Zn7pRo.webp)

### Minimizando Cálculos de Buffer Histórico

Scripts Pine criam _buffers históricos_ para todas as variáveis e chamadas de função das quais dependem suas saídas. Cada buffer contém informações sobre o intervalo de valores históricos que o script pode acessar com o operador de referência histórica [[]](https://br.tradingview.com/pine-script-reference/v5/#op_%5B%5D).

Um script determina _automaticamente_ o tamanho necessário do buffer para todas as suas variáveis e chamadas de função analisando as referências históricas executadas durante as __primeiras 244 barras__ de um conjunto de dados. Quando um script só referencia o histórico de um valor calculado _após_ essas barras iniciais, ele __reinicia__ suas execuções repetidamente em barras anteriores com buffers históricos sucessivamente maiores até determinar o tamanho apropriado ou gerar um erro de tempo de execução. Essas execuções repetitivas podem aumentar significativamente o tempo de execução de um script em alguns casos.

Quando um script _executa excessivamente_ em um conjunto de dados para calcular buffers históricos, uma maneira eficaz de melhorar seu desempenho é definir _explicitamente_ tamanhos de buffer adequados usando a função [max_bars_back()](https://br.tradingview.com/pine-script-reference/v5/#fun_max_bars_back). Com tamanhos de buffer apropriados declarados explicitamente, o script não precisa reexecutar dados passados para determinar os tamanhos.

Por exemplo, o script abaixo usa uma [polilinha](./05_12_lines_e_boxes.md#polylines-polilinhas) para desenhar um histograma básico representando a distribuição dos valores calculados de `source` ao longo de 500 barras. Na [última barra disponível](https://br.tradingview.com/pine-script-reference/v5/#var_barstate.islast), o script usa um loop [for](https://br.tradingview.com/pine-script-reference/v5/#kw_for) para analisar os valores históricos da série `source` calculada e determinar os [pontos do gráfico](./04_09_tipagem_do_sistema.md#chart-points-pontos-do-gráfico) usados pelo desenho da [polilinha](https://br.tradingview.com/pine-script-reference/v5/#type_polyline). Ele também [plota](./05_15_plots.md) o valor de `bar_index + 1` para verificar o número de barras nas quais foi executado:

```c
//@version=5
indicator("Minimizing historical buffer calculations demo", overlay = true)

//@variable A polyline with points that form a histogram of `source` values.
var polyline display = na
//@variable The difference Q3 of `high` prices and Q1 of `low` prices over 500 bars.
float innerRange = ta.percentile_nearest_rank(high, 500, 75) - ta.percentile_nearest_rank(low, 500, 25)
// Calculate the highest and lowest prices, and the total price range, over 500 bars.
float highest    = ta.highest(500)
float lowest     = ta.lowest(500)
float totalRange = highest - lowest

//@variable The source series for histogram calculation. Its value is the midpoint between the `open` and `close`.
float source = math.avg(open, close)

if barstate.islast
    polyline.delete(display)
    // Calculate the number of histogram bins and their size.
    int   bins    = int(math.round(5 * totalRange / innerRange))
    float binSize = totalRange / bins
    //@variable An array of chart points for the polyline.
    array<chart.point> points = array.new<chart.point>(bins, chart.point.new(na, na, na))
    // Loop to build the histogram.
    for i = 0 to 499
        //@variable The histogram bin number. Uses past values of the `source` for its calculation.
        //          The script must execute across all previous bars AGAIN to determine the historical buffer for 
        //          `source`, as initial references to the calculated series occur AFTER the first 244 bars. 
        int index = int((source[i] - lowest) / binSize)
        if na(index)
            continue
        chart.point currentPoint = points.get(index)
        if na(currentPoint.index)
            points.set(index, chart.point.from_index(bar_index + 1, (index + 0.5) * binSize + lowest))
            continue
        currentPoint.index += 1
    // Add final points to the `points` array and draw the new `display` polyline.
    points.unshift(chart.point.now(lowest))
    points.push(chart.point.now(highest))
    display := polyline.new(points, closed = true)

plot(bar_index + 1, "Number of bars", display = display.data_window)
```

Como o script _somente_ referencia valores passados de `source` na _última barra_, ele __não__ construirá um buffer histórico adequado para a série dentro das primeiras 244 barras em um conjunto de dados maior. Consequentemente, ele __reexecutará__ em todas as barras históricas para identificar o tamanho adequado do buffer.

Como visto nos [resultados perfilados](./06_03_perfilamento_e_otimizacao.md#interpretando-resultados-perfilados) após executar o script em 20.320 barras, o número de execuções de código _global_ foi 162.560, o que é __oito vezes__ o número de barras do gráfico. Em outras palavras, o script teve que _repetir_ as execuções históricas __mais sete vezes__ para determinar o buffer apropriado para a série `source` neste caso:

![Minimizando cálculos de buffer histórico 01](./imgs/Profiling-and-optimization-Optimization-Minimizing-historical-buffer-calculations-1.Cyx3FoQJ_Z1xcPM1.webp)

Este script só referenciará os valores mais recentes de 500 `source` na última barra histórica e em todas as barras em tempo real. Portanto, pode-se ajudar a estabelecer o buffer correto _sem_ reexecução definindo um comprimento de referência de 500 barras com [max_bars_back()](https://br.tradingview.com/pine-script-reference/v5/#fun_max_bars_back).

No script abaixo, foi adicionado [max_bars_back(source, 500)](https://br.tradingview.com/pine-script-reference/v5/#fun_max_bars_back) após a declaração da variável para especificar explicitamente que o script acessará até 500 valores históricos de `source` durante suas execuções:

```c
//@version=5
indicator("Minimizing historical buffer calculations demo", overlay = true)

//@variable A polyline with points that form a histogram of `source` values.
var polyline display = na
//@variable The difference Q3 of `high` prices and Q1 of `low` prices over 500 bars.
float innerRange = ta.percentile_nearest_rank(high, 500, 75) - ta.percentile_nearest_rank(low, 500, 25)
// Calculate the highest and lowest prices, and the total price range, over 500 bars.
float highest    = ta.highest(500)
float lowest     = ta.lowest(500)
float totalRange = highest - lowest

//@variable The source series for histogram calculation. Its value is the midpoint between the `open` and `close`.
float source = math.avg(open, close)
// Explicitly define a 500-bar historical buffer for the `source` to prevent recalculation.
max_bars_back(source, 500)

if barstate.islast
    polyline.delete(display)
    // Calculate the number of histogram bins and their size.
    int   bins    = int(math.round(5 * totalRange / innerRange))
    float binSize = totalRange / bins
    //@variable An array of chart points for the polyline.
    array<chart.point> points = array.new<chart.point>(bins, chart.point.new(na, na, na))
    // Loop to build the histogram.
    for i = 0 to 499
        //@variable The histogram bin number. Uses past values of the `source` for its calculation.
        //          Since the `source` now has an appropriate predefined buffer, the script no longer needs 
        //          to recalculate across previous bars to determine the referencing length. 
        int index = int((source[i] - lowest) / binSize)
        if na(index)
            continue
        chart.point currentPoint = points.get(index)
        if na(currentPoint.index)
            points.set(index, chart.point.from_index(bar_index + 1, (index + 0.5) * binSize + lowest))
            continue
        currentPoint.index += 1
    // Add final points to the `points` array and draw the new `display` polyline.
    points.unshift(chart.point.now(lowest))
    points.push(chart.point.now(highest))
    display := polyline.new(points, closed = true)

plot(bar_index + 1, "Number of bars", display = display.data_window)
```

Com essa mudança, o script não precisa mais reexecutar em todos os dados históricos para determinar o tamanho do buffer. Como visto nos [resultados perfilados](./06_03_perfilamento_e_otimizacao.md#interpretando-resultados-perfilados) abaixo, o número de execuções do código global agora está alinhado com o número de barras do gráfico, e o script levou significativamente menos tempo para completar todas as suas execuções históricas:

![Minimizando cálculos de buffer histórico 02](./imgs/Profiling-and-optimization-Optimization-Minimizing-historical-buffer-calculations-2.DPrfVLfJ_Z1XIu4W.webp)

__Note que:__

- Este script só requer até as 501 barras históricas mais recentes para calcular sua saída de desenho. Neste caso, outra maneira de otimizar o uso de recursos é incluir `calc_bars_count = 501` na função [indicator()](https://br.tradingview.com/pine-script-reference/v5/#fun_indicator), o que reduz execuções desnecessárias do script ao restringir os dados históricos que o script pode calcular para 501 barras.

> __Observação!__\
> Ao definir explicitamente um tamanho de buffer para uma referência histórica problemática com [max_bars_back()](https://br.tradingview.com/pine-script-reference/v5/#fun_max_bars_back), é imperativo garantir que o script __não__ usará mais dados do que o especificado posteriormente em suas execuções. Caso contrário, o script ainda tentará recalcular o buffer se o tamanho especificado pelo usuário for insuficiente.
> Outra consideração ao definir tamanhos de buffer explicitamente é que, quanto maior o buffer, maior o _custo de memória_. Portanto, os programadores devem tentar manter o comprimento do buffer explícito limitado a __apenas__ o número máximo de valores históricos que o script referenciará e __não mais__. Por exemplo, definir um buffer de 5000 barras quando o script só requer 500 valores históricos resultará em um desperdício desnecessário de memória.

## Dicas

### Contornando a Sobrecarga do Profiler

Como o [Pine Profiler](./06_03_perfilamento_e_otimizacao.md#pine-profiler) deve realizar _cálculos extras_ para coletar dados de desempenho, conforme explicado em [esta seção](./06_03_perfilamento_e_otimizacao.md#uma-visão-geral-do-funcionamento-interno-do-profiler), o tempo necessário para executar um script __aumenta__ durante o perfilamento.

A maioria dos scripts funcionará conforme esperado com a sobrecarga do Profiler incluída. No entanto, quando o tempo de execução de um script complexo se aproxima do [limite do plano](https://br.tradingview.com/support/solutions/43000579793), usar o [Profiler](./06_03_perfilamento_e_otimizacao.md#pine-profiler) nele pode fazer com que seu tempo de execução __exceda__ o limite. Esse caso indica que o script provavelmente precisa de [otimização](./06_03_perfilamento_e_otimizacao.md#otimização), mas pode ser desafiador saber por onde começar sem poder [perfilar o código](./06_03_perfilamento_e_otimizacao.md#profilando-um-script). A solução mais eficaz nesse cenário é reduzir o número de barras nas quais o script deve ser executado. Os usuários podem conseguir essa redução de qualquer uma das seguintes maneiras:

- Selecionar um conjunto de dados que tenha menos pontos de dados em seu histórico, por exemplo, um intervalo de tempo mais alto ou um símbolo com dados limitados.
- Usar lógica condicional para limitar as execuções de código a um intervalo de tempo ou barras específicas.
- Incluir um argumento `calc_bars_count` na declaração do script para especificar quantas barras históricas recentes ele pode usar.

Reduzir o número de pontos de dados funciona na maioria dos casos porque diminui diretamente o número de vezes que o script deve ser executado, geralmente resultando em menos tempo de execução acumulado.

Como demonstração, este script contém uma função `gcd()` que usa um algoritmo _simples_ para calcular o [max divisor comum](https://pt.wikipedia.org/wiki/M%C3%A1ximo_divisor_comum) de dois inteiros. A função inicializa seu `result` usando o menor valor absoluto dos dois números. Em seguida, ela reduz o valor de `result` em um dentro de um loop [while](https://br.tradingview.com/pine-script-reference/v5/#kw_while) até que possa dividir ambos os números sem restos. Essa estrutura implica que o loop iterará até _N_ vezes, onde _N_ é o menor dos dois argumentos.

Neste exemplo, o script plota o valor de `gcd(10000, 10000 + bar_index)`. O menor dos dois argumentos é sempre 10.000 neste caso, o que significa que o loop [while](https://br.tradingview.com/pine-script-reference/v5/#kw_while) dentro da função exigirá até 10.000 iterações por execução do script, dependendo do valor de [bar_index](https://br.tradingview.com/pine-script-reference/v5/#var_bar_index):

```c
//@version=5
indicator("Script takes too long while profiling demo")

//@function Calculates the greatest common divisor of `a` and `b` using a naive algorithm.
gcd(int a, int b) =>
    //@variable The greatest common divisor.
    int result = math.max(math.min(math.abs(a), math.abs(b)), 1)
    // Reduce the `result` by 1 until it divides `a` and `b` without remainders. 
    while result > 0
        if a % result == 0 and b % result == 0
            break
        result -= 1
    // Return the `result`.
    result

plot(gcd(10000, 10000 + bar_index), "GCD")
```

Quando o script é adicionado ao gráfico, ele leva algum tempo para executar nos dados do gráfico, mas não gera um erro. No entanto, __após__ habilitar o [Profiler](./06_03_perfilamento_e_otimizacao.md#pine-profiler), o script gera um erro de tempo de execução indicando que excedeu o [limite de tempo de execução](https://br.tradingview.com/support/solutions/43000579793) do plano Premium (40 segundos):

![Contornando a sobrecarga do profiler 01](./imgs/Profiling-and-optimization-Tips-Working-around-profiler-overhead-The-script-takes-too-long-to-execute-1.ChWuvP-N_1heDWT.webp)

O gráfico atual tem mais de 20.000 barras históricas, o que pode ser demais para o script lidar dentro do tempo alocado enquanto o [Profiler](./06_03_perfilamento_e_otimizacao.md#pine-profiler) está ativo. Pode-se tentar limitar o número de execuções históricas para contornar o problema neste caso.

Abaixo, foi incluído `calc_bars_count = 10000` na função [indicator()](https://br.tradingview.com/pine-script-reference/v5/#fun_indicator), que limita o histórico disponível do script às 10.000 barras históricas mais recentes. Após restringir as execuções históricas do script, ele não excede mais o limite do plano Premium durante o perfilamento, permitindo agora inspecionar os resultados de desempenho:

![Contornando a sobrecarga do profiler 02](./imgs/Profiling-and-optimization-Tips-Working-around-profiler-overhead-The-script-takes-too-long-to-execute-2.DN-scZLp_Z1HYEfF.webp)

```c
//@version=5
indicator("Script takes too long while profiling demo", calc_bars_count = 10000)

//@function Calculates the greatest common divisor of `a` and `b` using a naive algorithm.
gcd(int a, int b) =>
    //@variable The greatest common divisor.
    int result = math.max(math.min(math.abs(a), math.abs(b)), 1)
    // Reduce the `result` by 1 until it divides `a` and `b` without remainders.
    while result > 0
        if a % result == 0 and b % result == 0
            break
        result -= 1
    // Return the `result`.
    result

plot(gcd(10000, 10000 + bar_index), "GCD")
```

> __Observação!__\
> Este processo pode exigir tentativa e erro, pois determinar o número de execuções que um script computacionalmente pesado pode lidar antes de exceder o tempo limite não é necessariamente direto. Se um script demorar muito para ser executado após habilitar o [Profiler](./06_03_perfilamento_e_otimizacao.md#pine-profiler), experimente diferentes maneiras de limitar suas execuções até conseguir perfilar com sucesso.


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

Agora, veja como o Profiler envolve um script completo e todas as suas regiões de código significativas. Começaremos com este script, que calcula três séries pseudorandômicas e exibe seu resultado [médio](https://br.tradingview.com/pine-script-reference/v5/#fun_math.avg). O script utiliza um [objeto](./04_12_objetos.md) de um [tipo definido pelo usuário](./04_09_tipagem_do_sistema.md#tipos-definidos-pelo-usuário) para armazenar um estado pseudorandômico, um [método](./04_13_metodos.md#métodos-definidos-pelo-usuário) para calcular novos valores e atualizar o estado, e uma estrutura [if…else if](https://br.tradingview.com/pine-script-reference/v5/#kw_if) para atualizar cada série com base nos valores gerados:

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

### Profilando em Diferentes Configurações

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

## Otimização


# Style Guide (_Guia de Estilo_)

Este guia de estilo fornece recomendações sobre como nomear variáveis e organizar seus scripts Pine de uma maneira padrão que funcione bem. Scripts que seguem as melhores práticas serão mais fáceis de ler, entender e manter.

Scripts que utilizam essas diretrizes podem ser encontrados nas contas [TradingView](https://br.tradingview.com/u/TradingView/#published-scripts) e [PineCoders](https://br.tradingview.com/u/PineCoders/#published-scripts) na plataforma.

## Convenções de Nomeação

Recomenda-se o uso de:

- `camelCase` para todos os identificadores, ou seja, nomes de variáveis ou funções: `ma`, `maFast`, `maLengthInput`, `maColor`, `roundedOHLC()`, `pivotHi()`.
- `SNAKE_CASE` em letras maiúsculas para constantes: `BULL_COLOR`, `BEAR_COLOR`, `MAX_LOOKBACK`.
- O uso de sufixos qualificadores quando fornecem pistas valiosas sobre o tipo ou a procedência de uma variável: `maShowInput`, `bearColor`, `bearColorInput`, `volumesArray`, `maPlotID`, `resultsTable`, `levelsColorArray`.

## Organização do Script

O compilador Pine Script é bastante tolerante quanto à posição de declarações específicas ou à [anotação do compilador](./04_03_estrutura_do_script.md#anotações-do-compilador) no script. Embora outras disposições sejam sintaticamente corretas, recomenda-se organizar os scripts da seguinte maneira:

```c
<license>
<version>
<declaration_statement>
<import_statements>
<constant_declarations>
<inputs>
<function_declarations>
<calculations>
<strategy_calls>
<visuals>
<alerts>
```

### `<license>`

Se publicar seus scripts de código aberto publicamente no TradingView (scripts também podem ser publicados de forma privada), seu código de código aberto é, por padrão, protegido pela licença Mozilla. Pode-se escolher qualquer outra licença preferida.

A reutilização de código desses scripts é regida pelas [Regras da Casa sobre Publicação de Scripts](https://br.tradingview.com/support/solutions/43000590599), que têm precedência sobre a licença do autor.

Os comentários padrão de licença que aparecem no início dos scripts são:

```c
// This source code is subject to the terms of the Mozilla Public License 2.0 at https://mozilla.org/MPL/2.0/
// © username
```

### `<version>`

Esta é a [anotação do compilador](./04_03_estrutura_do_script.md#anotações-do-compilador) que define a versão do Pine Script que o script usará. Se nenhuma estiver presente, v1 será usada. Para v5, utilize:

```c
//@version=5  
```

### `<declaration_statement>`

Esta é a declaração obrigatória que define o tipo de script. Deve ser uma chamada para [indicator()](https://br.tradingview.com/pine-script-reference/v5/#fun_indicator), [strategy()](https://br.tradingview.com/pine-script-reference/v5/#fun_strategy) ou [library()](https://br.tradingview.com/pine-script-reference/v5/#fun_library).

### `<import_statements>`

Se o script utilizar uma ou mais [bibliotecas Pine Script](./05_11_libraries.md), as instruções de [importação](https://br.tradingview.com/pine-script-reference/v5/#kw_import) pertencem a esta seção.

### `<constant_declarations>`

Scripts podem declarar variáveis qualificadas como “const”, ou seja, que referenciam um valor constante.

Referem-se a variáveis como “constantes” quando atendem a estes critérios:

- Sua declaração usa a palavra-chave opcional `const` (veja a seção sobre [qualificadores de tipo](./04_09_tipagem_do_sistema.md#qualificadores) no Manual do Usuário para mais informações).
- São inicializadas usando um literal (por exemplo, `100` ou `"AAPL"`) ou um valor incorporado qualificado como “const” (por exemplo, `color.green`).
- Seu valor não muda durante a execução do script.

Utiliza-se `SNAKE_CASE` para nomear essas variáveis e agrupar suas declarações próximo ao início do script. Por exemplo:

```c
// ————— Constants
int     MS_IN_MIN   = 60 * 1000
int     MS_IN_HOUR  = MS_IN_MIN  * 60
int     MS_IN_DAY   = MS_IN_HOUR * 24

color   GRAY        = #808080ff
color   LIME        = #00FF00ff
color   MAROON      = #800000ff
color   ORANGE      = #FF8000ff
color   PINK        = #FF0080ff
color   TEAL        = #008080ff
color   BG_DIV      = color.new(ORANGE, 90)
color   BG_RESETS   = color.new(GRAY, 90)

string  RST1        = "No reset; cumulate since the beginning of the chart"
string  RST2        = "On a stepped higher timeframe (HTF)"
string  RST3        = "On a fixed HTF"
string  RST4        = "At a fixed time"
string  RST5        = "At the beginning of the regular session"
string  RST6        = "At the first visible chart bar"
string  RST7        = "Fixed rolling period"

string  LTF1        = "Least precise, covering many chart bars"
string  LTF2        = "Less precise, covering some chart bars"
string  LTF3        = "More precise, covering less chart bars"
string  LTF4        = "Most precise, 1min intrabars"

string  TT_TOTVOL     = "The 'Bodies' value is the transparency of the total volume candle bodies. Zero is opaque, 100 is transparent."
string  TT_RST_HTF    = "This value is used when '" + RST3 +"' is selected."
string  TT_RST_TIME   = "These values are used when '" + RST4 +"' is selected.
  A reset will occur when the time is greater or equal to the bar's open time, and less than its close time.\nHour: 0-23\nMinute: 0-59"
string  TT_RST_PERIOD = "This value is used when '" + RST7 +"' is selected."
```

Neste exemplo:

- As constantes `RST*` e `LTF*` serão usadas como elementos de tupla no argumento `options` de chamadas `input.*()`.
- As constantes `TT_*` serão usadas como argumentos `tooltip` em chamadas `input.*()`. Note como se utiliza uma continuação de linha para literais de string longos.
- Não se utiliza [var](https://br.tradingview.com/pine-script-reference/v5/#kw_var) para inicializar constantes. O tempo de execução do Pine Script é otimizado para lidar com declarações em cada barra, mas usar [var](https://br.tradingview.com/pine-script-reference/v5/#kw_var) para inicializar uma variável apenas na primeira vez que é declarada implica uma pequena penalidade no desempenho do script devido à manutenção que as variáveis [var](https://br.tradingview.com/pine-script-reference/v5/#kw_var) requerem em barras subsequentes.

__Note que:__

- Literais usados em mais de um lugar em um script devem sempre ser declarados como constantes. Usar a constante em vez do literal torna o script mais legível se receber um nome significativo, e a prática facilita a manutenção do código. Mesmo que a quantidade de milissegundos em um dia seja improvável de mudar no futuro, `MS_IN_DAY` é mais significativo do que `1000 * 60 * 60 * 24`.
- Constantes usadas apenas no bloco local de uma função ou declaração [if](https://br.tradingview.com/pine-script-reference/v5/#kw_if), [while](https://br.tradingview.com/pine-script-reference/v5/#kw_while), etc., podem ser declaradas nesse bloco local.

### `<inputs>`

É __muito__ mais fácil ler scripts quando todas as entradas estão na mesma seção de código. Colocar essa seção no início do script também reflete como são processadas em tempo de execução, ou seja, antes do restante do script ser executado.

Sufixar nomes de variáveis de entrada com `input` as torna mais facilmente identificáveis quando são usadas posteriormente no script: `maLengthInput`, `bearColorInput`, `showAvgInput`, etc.

```c
// ————— Inputs
string  resetInput              = input.string(RST2,        "CVD Resets",                       inline = "00", options = [RST1, RST2, RST3, RST4, RST5, RST6, RST7])
string  fixedTfInput            = input.timeframe("D",      "  Fixed HTF:  ",                   tooltip = TT_RST_HTF)
int     hourInput               = input.int(9,              "  Fixed time hour:  ",             inline = "01", minval = 0, maxval = 23)
int     minuteInput             = input.int(30,             "minute",                           inline = "01", minval = 0, maxval = 59, tooltip = TT_RST_TIME)
int     fixedPeriodInput        = input.int(20,             "  Fixed period:  ",                inline = "02", minval = 1, tooltip = TT_RST_PERIOD)
string  ltfModeInput            = input.string(LTF3,        "Intrabar precision",               inline = "03", options = [LTF1, LTF2, LTF3, LTF4]) 
```

### `<function_declarations>`

Todas as funções definidas pelo usuário devem ser definidas no escopo global do script; definições de funções aninhadas não são permitidas no Pine Script.

O design ideal da função deve minimizar o uso de variáveis globais no escopo da função, pois prejudicam a portabilidade da função. Quando não for possível evitar, essas funções devem seguir as declarações de variáveis globais no código, o que implica que nem sempre podem ser colocadas na seção `<function_declarations>`. Tais dependências de variáveis globais devem ser documentadas idealmente nos comentários da função.

Também ajudará os leitores se documentar o objetivo da função, parâmetros e resultado. A mesma sintaxe usada em [bibliotecas](./05_11_libraries.md) pode ser usada para documentar suas funções. Isso pode facilitar a portabilidade de suas funções para uma biblioteca, caso decida fazer isso:

```c
//@version=5
indicator("<function_declarations>", "", true)

string SIZE_LARGE  = "Large"
string SIZE_NORMAL = "Normal"
string SIZE_SMALL  = "Small"

string sizeInput = input.string(SIZE_NORMAL, "Size", options = [SIZE_LARGE, SIZE_NORMAL, SIZE_SMALL])

// @function        Used to produce an argument for the `size` parameter in built-in functions.
// @param userSize  (simple string) User-selected size.
// @returns         One of the `size.*` built-in constants.
// Dependencies     SIZE_LARGE, SIZE_NORMAL, SIZE_SMALL
getSize(simple string userSize) =>
    result = 
      switch userSize
        SIZE_LARGE  => size.large
        SIZE_NORMAL => size.normal
        SIZE_SMALL  => size.small
        => size.auto

if ta.rising(close, 3)
    label.new(bar_index, na, yloc = yloc.abovebar, style = label.style_arrowup, size = getSize(sizeInput))
```

### `<calculations>`

Esta é a seção onde devem ser colocados os cálculos principais e a lógica do script. O código pode ser mais fácil de ler quando as declarações de variáveis são colocadas próximas ao segmento de código que as usa. Alguns programadores preferem colocar todas as suas declarações de variáveis não constantes no início desta seção, o que nem sempre é possível para todas as variáveis, pois algumas podem exigir que alguns cálculos tenham sido executados antes de sua declaração.

### `<strategy_calls>`

As estratégias são mais fáceis de ler quando as chamadas de estratégia são agrupadas na mesma seção do script.

### `<visuals>`

Esta seção deve incluir idealmente todas as instruções que produzem os visuais do script, sejam eles gráficos, desenhos, cores de fundo, plotagem de velas, etc. Veja a seção sobre [cores](./05_07_cores.md) do manual do usuário do Pine Script para mais informações sobre como a profundidade relativa dos visuais é determinada.

### `<alerts>`

O código de alertas geralmente exigirá que os cálculos do script tenham sido executados antes dele, então faz sentido colocá-lo no final do script.

## Espaçamento

Um espaço deve ser usado em ambos os lados de todos os operadores, exceto operadores unários (`-1`). Também é recomendado um espaço após todas as vírgulas e ao usar argumentos nomeados de funções, como em `plot(series = close)`:

```c
int a = close > open ? 1 : -1
var int newLen = 2
newLen := min(20, newlen + 1)
float a = -b
float c = d > e ? d - e : d
int index = bar_index % 2 == 0 ? 1 : 2
plot(close, color = color.red)
```

## Quebra de Linha

A quebra de linha pode tornar linhas longas mais fáceis de ler. Quebras de linha são definidas usando um nível de indentação que não é múltiplo de quatro, pois quatro espaços ou uma tabulação são usados para definir blocos locais. Aqui, usando dois espaços:

```c
plot(
   series = close,
   title = "Close",
   color = color.blue,
   show_last = 10
 )
```

## Alinhamento Vertical

O alinhamento vertical usando tabulações ou espaços pode ser útil em seções de código contendo muitas linhas semelhantes, como declarações de constantes ou entradas. Elas podem facilitar edições em massa usando o recurso de múltiplos cursos do Pine Editor (`ctrl` \+ `alt` \+ `🠅`):

```c
// Colors used as defaults in inputs.
color COLOR_AQUA  = #0080FFff
color COLOR_BLACK = #000000ff
color COLOR_BLUE  = #013BCAff
color COLOR_CORAL = #FF8080ff
color COLOR_GOLD  = #CCCC00ff
```

## Tipagem Explícita

Incluir o tipo de variáveis ao declará-las não é obrigatório. No entanto, isso ajuda a tornar os scripts mais fáceis de ler, navegar e entender. Pode ajudar a esclarecer os tipos esperados em cada ponto da execução do script e distinguir a declaração de uma variável (usando `=`) de suas reatribuições (usando `:=`). Usar tipagem explícita também pode tornar os scripts mais fáceis de [depurar](./06_02_debugging.md).

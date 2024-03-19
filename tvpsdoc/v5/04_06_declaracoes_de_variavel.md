
# Declarações de Variável

Variáveis são [identificadores](./04_04_identificadores.md) que armazenam valores. Devem ser _declaradas_ no código antes de serem utilizadas.

A sintaxe para declaração de variáveis é:

```c
[<declaration_mode>] [<type>] <identifier> = <expression> | <structure>
```

_OU_

```c
<tuple_declaration> = <function_call> | <structure>
```

Onde:

- O símbolo `|` significa "`or`" (_ou_), e partes contidas entre colchetes (`[]`) podem aparecer zero ou uma vez, significa que a parte entre colchetes pode estar presente no texto (aparecer uma vez) ou pode não estar presente (aparecer zero vezes), ou seja, é opcional.
- '<declaration_mode>' é o [modo de declaração](./04_06_declaracoes_de_variavel.md#modos-de-declaração) da variável. Pode ser [var](https://br.tradingview.com/pine-script-reference/v5/#op_var) ou [varip](https://br.tradingview.com/pine-script-reference/v5/#op_varip), ou nothing (_nada_).
- '<type>' é opcional, como na maioria das declarações de variáveis no Pine Script (veja [tipos](./000_type_system.md#tipos)).
- '<identifier>' é o [nome](./04_04_identificadores.md) da variável.
- '<expression>' pode ser um literal, uma variável, uma expressão ou uma chamada de função.
- '<structure>' pode ser uma estrutura [if](https://br.tradingview.com/pine-script-reference/v5/#op_if), [for](https://br.tradingview.com/pine-script-reference/v5/#op_for), [while](https://br.tradingview.com/pine-script-reference/v5/#op_while) ou [switch](https://br.tradingview.com/pine-script-reference/v5/#op_switch).
- '<tuple_declaration>' é uma lista separada por vírgulas de nomes de variáveis entre colchetes (`[]`), por exemplo, `[ma, upperBand, lowerBand]`.

Estas são declarações válidas de variáveis. A última requer quatro linhas:

```c
BULL_COLOR = color.lime
i = 1
len = input(20, "Length")
float f = 10.5
closeRoundedToTick = math.round_to_mintick(close)
st = ta.supertrend(4, 14)
var barRange = float(na)
var firstBarOpen = open
varip float lastClose = na
[macdLine, signalLine, histLine] = ta.macd(close, 12, 26, 9)
plotColor = if close > open
    color.green
else
    color.red
```

> __Observação__\
> As declarações acima todas contêm o operador de atribuição `=` porque são __declarações de variáveis__. Quando encontrar linhas semelhantes usando o operador de reatribuição `:=`, o código está __reatribuindo__ um valor a uma variável que __já foi declarada__. Essas são __reatribuições de variáveis__. Certifique-se de entender a distinção, pois isso é uma armadilha comum para iniciantes no Pine Script. Consulte a próxima seção sobre [Reatribuição de Variáveis](./04_06_declaracoes_de_variavel.md#reatribuição-de-variável) para mais detalhes.

A sintaxe formal de uma declaração de variável é:

```c
<variable_declaration>
    [<declaration_mode>] [<type>] <identifier> = <expression> | <structure>
    |
    <tuple_declaration> = <function_call> | <structure>

<declaration_mode>
    var | varip

<type>
    int | float | bool | color | string | line | linefill | label | box | table | array<type> | matrix<type> | UDF
```


# Inicialização com `na`

Na maioria dos casos, uma declaração explícita de tipo é redundante porque o tipo é automaticamente inferido a partir do valor à direita do `=` durante a compilação, então a decisão de usá-los geralmente é uma questão de preferência. 

Por exemplo:

```c
baseLine0 = na          // compile time error!
float baseLine1 = na    // OK
baseLine2 = float(na)   // OK
```

Na primeira linha do exemplo, o compilador não pode determinar o tipo da variável `baseLine0` porque [na](https://br.tradingview.com/pine-script-reference/v5/#var_na) é um valor genérico sem um tipo específico. A declaração da variável `baseLine1` está correta porque seu tipo [float](https://br.tradingview.com/pine-script-reference/v5/#op_float) é declarado explicitamente. A declaração da variável `baseLine2` também está correta porque seu tipo pode ser derivado da expressão `float(na)`, que é uma conversão explícita do valor [na](https://br.tradingview.com/pine-script-reference/v5/#var_na) para o tipo [float](https://br.tradingview.com/pine-script-reference/v5/#op_float). As declarações de `baseLine1` e `baseLine2` são equivalentes.


# Declarações de Tupla

Chamadas de função ou estruturas podem retornar múltiplos valores. Ao chamá-las e armazená-las os valores que retornam, é necessário usar uma _declaração de tupla_, que é um conjunto separado por vírgulas de um ou mais valores entre colchetes. Isso nos permite declarar múltiplas variáveis simultaneamente.

Como exemplo, a função embutida [ta.bb()](https://br.tradingview.com/pine-script-reference/v5/#fun_ta{dot}bb) para _Bandas de Bollinger_ retorna três valores:

```c
[bbMiddle, bbUpper, bbLower] = ta.bb(close, 5, 4)
```


# Reatribuição de Variável

Uma reatribuição de variável é realizada usando o operador de reatribuição [:=](./04_05_operadores.md#operador-de-reatribuição). Isso só pode ser feito depois que uma variável foi declarada inicialmente e recebeu um valor inicial. Reatribuir um novo valor a uma variável é frequentemente necessário em cálculos, e é sempre necessário quando uma variável do escopo global precisa receber um novo valor de dentro do bloco local de uma estrutura.

Por exemplo:

```c
//@version=5
indicator("", "", true)
sensitivityInput = input.int(2, "Sensitivity", minval = 1, tooltip = "Higher values make color changes less sensitive.")
ma = ta.sma(close, 20)
maUp = ta.rising(ma, sensitivityInput)
maDn = ta.falling(ma, sensitivityInput)

// On first bar only, initialize color to gray
var maColor = color.gray
if maUp
    // MA has risen for two bars in a row; make it lime.
    maColor := color.lime
else if maDn
    // MA has fallen for two bars in a row; make it fuchsia.
    maColor := color.fuchsia

plot(ma, "MA", maColor, 2)
```

Perceba que:

- A variável `maColor` foi inicializada apenas na primeira barra, para que preserve seu valor através das barras subsequentes.
- A cada barra, a instrução [if](https://br.tradingview.com/pine-script-reference/v5/#op_if) verifica se a média móvel tem aumentado ou diminuído pelo número especificado de barras pelo usuário (o padrão é 2). Quando isso acontece, o valor de `maColor` precisa ser reatribuído a um novo valor de dentro dos blocos locais [if](https://br.tradingview.com/pine-script-reference/v5/#op_if). Para fazer isso, utilize-se o operador de reatribuição [:=](./04_05_operadores.md#operador-de-reatribuição).
- Caso não utiliza-se o operador de reatribuição [:=](./04_05_operadores.md#operador-de-reatribuição), o efeito seria inicializar uma nova variável local `maColor` que teria o mesmo nome que a do escopo global, mas na verdade seria uma entidade independente muito confusa que persistiria apenas pelo comprimento do bloco local e então desapareceria sem deixar rastros.

Todas as variáveis definidas pelo usuário no Pine Script são _mutáveis_, o que significa que seu valor pode ser alterado usando o operador de reatribuição [:=](./04_05_operadores.md#operador-de-reatribuição). Atribuir um novo valor a uma variável pode alterar seu _qualificador de tipo_ (consulte [Tipagem do Sistema](./000_type_system.md) do Pine Script para maiores informações). Uma variável pode receber um novo valor quantas vezes forem necessárias durante a execução do script em uma barra, então um script pode conter qualquer número de reatribuições de uma variável. O [modo de declaração](./04_06_declaracoes_de_variavel.md#modos-de-declaração) de uma variável determina como os novos valores atribuídos a ela serão salvos.


# Modos de Declaração

Para compreender o impacto que os modos de declaração têm sobre o comportamento das variáveis, é necessário ter conhecimento prévio do [modelo de execução](./04_01_modelo_de_execucao.md) do Pine Script.

Ao declarar uma variável, se um modo de declaração for especificado, esse deve vir primeiro.

Três modos podem ser usados:

- "Em cada barra", quando nenhum é especificado.
- [var](https://br.tradingview.com/pine-script-reference/v5/#op_var).
- [varip](https://br.tradingview.com/pine-script-reference/v5/#op_varip).

# Var

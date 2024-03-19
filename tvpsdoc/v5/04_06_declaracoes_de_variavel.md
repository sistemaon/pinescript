
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

> __Observação__
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


# Reatribuição de Variável



# Modos de Declaração


# Var

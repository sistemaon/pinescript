
# Arrays

> __Observação__\
> Esta seção contém material avançado. Recomenda-se que programadores iniciantes em Pine Script se familiarizem com outras funcionalidades do Pine Script mais acessíveis antes de explorarem este conteúdo.

Arrays do Pine Script são coleções unidimensionais capazes de armazenar múltiplas referências de valores. Podem ser considerados uma forma mais eficiente de lidar com situações nas quais seria necessário declarar explicitamente um conjunto de variáveis similares (por exemplo, `price00`, `price01`, `price02`, ...).

Todos os elementos dentro de um array devem ser do mesmo tipo, que pode ser um tipo [integrado](./04_09_tipagem_do_sistema.md#qualificadores) ou [definido pelo usuário](./04_09_tipagem_do_sistema.md#tipos-definidos-pelo-usuário), sempre qualificado como "series". Scripts referenciam arrays usando um ID de array semelhante aos IDs de linhas, labels e outros tipos especiais. O Pine Script não utiliza um operador de indexação para referenciar elementos individuais de array. Em vez disso, funções como [array.get()](https://br.tradingview.com/pine-script-reference/v5/#fun_array{dot}get) e [array.set()](https://br.tradingview.com/pine-script-reference/v5/#fun_array{dot}set) são utilizadas para ler e escrever os valores dos elementos do array. Valores de array podem ser utilizados em expressões e funções que aceitam valores do tipo "series".

Scripts referenciam os elementos de um array usando um _index_, que começa em _0_ e vai até o _número de elementos no array menos um_. Arrays no Pine Script podem ter um tamanho dinâmico que varia ao longo das barras, já que é possível alterar o número de elementos em um array a cada iteração de um script. Scripts podem conter múltiplas instâncias de arrays. O tamanho dos arrays é limitado a 100.000 elementos.

> __Observação__\
> Utiliza-se o termo "início de um array" para designar o index 0, e "final de um array" para designar o elemento do array com o maior valor de index. Também se ampliará o significado de array para incluir IDs de array, em prol da brevidade.


# Declaração de Arrays

O Pine Script utiliza a seguinte sintaxe para declarar arrays:

```c
[var/varip ][array<type>/<type[]> ]<identifier> = <expression>
```

Onde `<type>` é um [modelo de tipo](./04_09_tipagem_do_sistema.md#templates-de-tipo) para o array que declara o tipo de valores que ele conterá, e a `<expression>` retorna um array do tipo especificado ou `na`.

Ao declarar uma variável como um array, pode-se utilizar a palavra-chave [array](https://br.tradingview.com/pine-script-reference/v5/#type_array) seguida de um [template de tipo](./04_09_tipagem_do_sistema.md#templates-de-tipo). Alternativamente, pode-se usar o nome do `type` seguido do modificador `[]` (que não deve ser confundido com o _operador de referência histórica_ [[]](https://br.tradingview.com/pine-script-reference/v5/#op_[])).

Como o Pine sempre utiliza funções específicas de tipo para criar arrays, a parte `array<type>/type[]` da declaração é redundante, exceto ao declarar uma variável de array atribuída a `na`. Mesmo quando não é necessário, declarar explicitamente o tipo do array ajuda a esclarecer a intenção para os leitores.

Esta linha de código declara uma variável de array chamada `prices` que aponta para `na`. Neste caso, é necessário especificar o tipo para declarar que a variável pode referenciar arrays contendo valores do tipo "float":

```c
array<float> prices = na
```

Também é possível criar o exemplo acima desta forma:

```c
float[] prices = na
```

Ao declarar um array e a `<expression>` não for `na`, utiliza uma das seguintes funções: [array.new<type>(size, initial_value)](https://br.tradingview.com/pine-script-reference/v5/#fun_array.new%3Ctype%3E), [array.from()](https://br.tradingview.com/pine-script-reference/v5/#fun_array{dot}from), ou [array.copy()](https://br.tradingview.com/pine-script-reference/v5/#fun_array{dot}copy). Para as funções [`array.new<type>(size, initial_value)`](https://br.tradingview.com/pine-script-reference/v5/#fun_array.new%3Ctype%3E), os argumentos dos parâmetros `size` e `initial_value` podem ser do tipo "series" para permitir o dimensionamento dinâmico e a inicialização dos elementos do array.

O exemplo a seguir cria um array contendo zero elementos do tipo "float", e desta vez, o ID do array retornado pela chamada da função [`array.new<float>()`](https://br.tradingview.com/pine-script-reference/v5/#fun_array.new%3Ctype%3E) é atribuído a `prices`:

```c
prices = array.new<float>(0)
```

> __Observação__\
> O _namespace_ `array.*` também contém funções específicas de tipo para a criação de arrays, incluindo [array.new_int()](https://br.tradingview.com/pine-script-reference/v5/#fun_array{dot}new_int), [array.new_float()](https://br.tradingview.com/pine-script-reference/v5/#fun_array{dot}new_float), [array.new_bool()](https://br.tradingview.com/pine-script-reference/v5/#fun_array{dot}new_bool), [array.new_color()](https://br.tradingview.com/pine-script-reference/v5/#fun_array{dot}new_color), [array.new_string()](https://br.tradingview.com/pine-script-reference/v5/#fun_array{dot}new_string), [array.new_line()](https://br.tradingview.com/pine-script-reference/v5/#fun_array{dot}new_line), [array.new_linefill()](https://br.tradingview.com/pine-script-reference/v5/#fun_array{dot}new_linefill), [array.new_label()](https://br.tradingview.com/pine-script-reference/v5/#fun_array{dot}new_label), [array.new_box()](https://br.tradingview.com/pine-script-reference/v5/#fun_array{dot}new_box) e [array.new_table()](https://br.tradingview.com/pine-script-reference/v5/#fun_array{dot}new_table). A função [array.new<type>()](https://br.tradingview.com/pine-script-reference/v5/#fun_array.new%3Ctype%3E) pode criar um array de qualquer tipo, incluindo [tipos definidos pelo usuário](./04_09_tipagem_do_sistema.md#tipos-definidos-pelo-usuário).

O parâmetro `initial_value` das funções `array.new*` permite aos usuários definir todos os elementos do array para um valor especificado. Se nenhum argumento for fornecido para `initial_value`, o array será preenchido com valores `na`.

Esta linha declara um ID de array chamado `prices` que aponta para um array contendo dois elementos, cada um atribuído ao valor de `close` da barra:

```c
prices = array.new<float>(2, close)
```

Para criar um array e inicializar seus elementos com valores diferentes, utiliza-se [array.from()](https://br.tradingview.com/pine-script-reference/v5/#fun_array{dot}from). Esta função infere o tamanho do array e o tipo de elementos que conterá a partir dos argumentos na chamada da função. Assim como as funções `array.new*`, aceita argumentos do tipo "series". Todos os valores fornecidos à função devem ser do mesmo tipo.

Por exemplo, todas estas três linhas de código criarão arrays "bool" idênticos com os mesmos dois elementos:

```c
statesArray = array.from(close > open, high != close)
bool[] statesArray = array.from(close > open, high != close)
array<bool> statesArray = array.from(close > open, high != close)
```

## Utilizando as Palavras-Chave `var` e `varip`

É possível utilizar as palavras-chave [var](https://br.tradingview.com/pine-script-reference/v5/#kw_var) e [varip](https://br.tradingview.com/pine-script-reference/v5/#kw_varip) para instruir um script a declarar uma variável de array apenas uma vez na primeira iteração do script na primeira barra do gráfico. Variáveis de array declaradas usando essas palavras-chave apontam para as mesmas instâncias de array até serem explicitamente reatribuídas, permitindo que um array e suas referências de elementos persistam ao longo das barras.

Ao declarar uma variável de array usando essas palavras-chave e adicionar um novo valor ao final do array referenciado em cada barra, o array crescerá em um a cada barra e terá o tamanho de `bar_index + 1` ([bar_index](https://br.tradingview.com/pine-script-reference/v5/#var_bar_index) começa em _zero_) quando o script for executado na última barra, conforme demonstra o código abaixo:

```c
//@version=5
indicator("Using `var`")
//@variable An array that expands its size by 1 on each bar.
var a = array.new<float>(0)
array.push(a, close)

if barstate.islast
    //@variable A string containing the size of `a` and the current `bar_index` value.
    string labelText = "Array size: " + str.tostring(a.size()) + "\nbar_index: " + str.tostring(bar_index)
    // Display the `labelText`.
    label.new(bar_index, 0, labelText, size = size.large)
```

O mesmo código sem a palavra-chave [var](https://br.tradingview.com/pine-script-reference/v5/#kw_var) resultaria na redeclaração do array em cada barra. Neste caso, após a execução da chamada [array.push()](https://br.tradingview.com/pine-script-reference/v5/#fun_array{dot}push), a chamada [a.size()](https://br.tradingview.com/pine-script-reference/v5/#fun_array{dot}size) retornaria o valor de 1.

> __Observação__\
> Variáveis de array declaradas usando [varip](https://br.tradingview.com/pine-script-reference/v5/#kw_varip) comportam-se como as que utilizam [var](https://br.tradingview.com/pine-script-reference/v5/#kw_var) em _dados históricos_, mas atualizam seus valores para barras em tempo real (ou seja, as barras desde a última compilação do script) a cada novo tick de preço. Arrays atribuídos a variáveis [varip](https://br.tradingview.com/pine-script-reference/v5/#kw_varip) podem conter apenas tipos [int](https://br.tradingview.com/pine-script-reference/v5/#type_int), [float](https://br.tradingview.com/pine-script-reference/v5/#type_float), [bool](https://br.tradingview.com/pine-script-reference/v5/#type_bool), [color](https://br.tradingview.com/pine-script-reference/v5/#type_color) ou [string](https://br.tradingview.com/pine-script-reference/v5/#type_string) ou [tipos definidos pelo usuário](./04_09_tipagem_do_sistema.md#tipos-definidos-pelo-usuário) que contenham exclusivamente em seus campos esses tipos ou coleções ([arrays](./04_14_arrays.md), [matrices](./000_matrices.md) ou [maps](./000_maps.md)) desses tipos.


# Leitura e Escrita de Elementos de Array

Scripts podem escrever valores em elementos individuais existentes de um array usando [array.set(id, index, value)](https://br.tradingview.com/pine-script-reference/v5/#fun_array{dot}set) e ler usando [array.get(id, index)](https://br.tradingview.com/pine-script-reference/v5/#fun_array{dot}get). Ao usar essas funções, é crucial que o `index` na chamada da função seja sempre menor ou igual ao tamanho do array (porque os _índices_ de array começam em _zero_). Para obter o tamanho de um array, utiliza-se a função [array.size(id)](https://br.tradingview.com/pine-script-reference/v5/#fun_array{dot}size).

O exemplo a seguir utiliza o método [array.set()](https://br.tradingview.com/pine-script-reference/v5/#fun_array{dot}set) para preencher um array `fillColors` com instâncias de uma cor base usando diferentes níveis de transparência. Em seguida, usa [array.get()](https://br.tradingview.com/pine-script-reference/v5/#fun_array{dot}get) para buscar uma das cores do array com base na localização da barra com o preço mais alto dentro das últimas barras `lookbackInput`:

![Leitura e escrita de elementos de array](./imgs/Arrays-ReadingAndWriting-DistanceFromHigh.png)

```c
//@version=5
indicator("Distance from high", "", true)
lookbackInput = input.int(100)
FILL_COLOR = color.green
// Declare array and set its values on the first bar only.
var fillColors = array.new<color>(5)
if barstate.isfirst
    // Initialize the array elements with progressively lighter shades of the fill color.
    fillColors.set(0, color.new(FILL_COLOR, 70))
    fillColors.set(1, color.new(FILL_COLOR, 75))
    fillColors.set(2, color.new(FILL_COLOR, 80))
    fillColors.set(3, color.new(FILL_COLOR, 85))
    fillColors.set(4, color.new(FILL_COLOR, 90))

// Find the offset to highest high. Change its sign because the function returns a negative value.
lastHiBar = - ta.highestbars(high, lookbackInput)
// Convert the offset to an array index, capping it to 4 to avoid a runtime error.
// The index used by `array.get()` will be the equivalent of `floor(fillNo)`.
fillNo = math.min(lastHiBar / (lookbackInput / 5), 4)
// Set background to a progressively lighter fill with increasing distance from location of highest high.
bgcolor(array.get(fillColors, fillNo))
// Plot key values to the Data Window for debugging.
plotchar(lastHiBar, "lastHiBar", "", location.top, size = size.tiny)
plotchar(fillNo, "fillNo", "", location.top, size = size.tiny)
```

Outra técnica para inicializar os elementos em um array é criar um _array vazio_ (um array sem elementos) e, em seguida, utilizar [array.push()](https://br.tradingview.com/pine-script-reference/v5/#fun_array{dot}push) para adicionar __novos__ elementos ao _final_ do array, aumentando o tamanho do array em um a cada chamada.

O código a seguir tem a mesma funcionalidade que a seção de inicialização do script anterior:

```c
// Declare array and set its values on the first bar only.
var fillColors = array.new<color>(0)
if barstate.isfirst
    // Initialize the array elements with progressively lighter shades of the fill color.
    array.push(fillColors, color.new(FILL_COLOR, 70))
    array.push(fillColors, color.new(FILL_COLOR, 75))
    array.push(fillColors, color.new(FILL_COLOR, 80))
    array.push(fillColors, color.new(FILL_COLOR, 85))
    array.push(fillColors, color.new(FILL_COLOR, 90))
```

O código abaixo é equivalente ao anterior, mas utiliza [array.unshift()](https://br.tradingview.com/pine-script-reference/v5/#fun_array{dot}unshift) para inserir novos elementos no _início_ do array `fillColors`:

```c
// Declare array and set its values on the first bar only.
var fillColors = array.new<color>(0)
if barstate.isfirst
    // Initialize the array elements with progressively lighter shades of the fill color.
    array.unshift(fillColors, color.new(FILL_COLOR, 90))
    array.unshift(fillColors, color.new(FILL_COLOR, 85))
    array.unshift(fillColors, color.new(FILL_COLOR, 80))
    array.unshift(fillColors, color.new(FILL_COLOR, 75))
    array.unshift(fillColors, color.new(FILL_COLOR, 70))
```

Também é possível utilizar [array.from()](https://br.tradingview.com/pine-script-reference/v5/#fun_array{dot}from) para criar o mesmo array `fillColors` com uma única chamada de função:

```c
//@version=5
indicator("Using `var`")
FILL_COLOR = color.green
var array<color> fillColors = array.from(
     color.new(FILL_COLOR, 70),
     color.new(FILL_COLOR, 75),
     color.new(FILL_COLOR, 80),
     color.new(FILL_COLOR, 85),
     color.new(FILL_COLOR, 90)
 )
// Cycle background through the array's colors.
bgcolor(array.get(fillColors, bar_index % (fillColors.size())))
```

A função [array.fill(id, value, index_from, index_to)](https://br.tradingview.com/pine-script-reference/v5/#fun_array{dot}fill) direciona todos os elementos do array, ou os elementos dentro do intervalo de `index_from` a `index_to`, para um `value` especificado. Sem os dois últimos parâmetros opcionais, a função preenche todo o array, então:

```c
a = array.new<float>(10, close)
```

E:

```c
a = array.new<float>(10)
a.fill(close)
```

São equivalentes, porém:

```c
a = array.new<float>(10)
a.fill(close, 1, 3)
```

Preenche apenas o segundo e o terceiro elementos (nos _index_ 1 e 2) do array com `close`. Note como o último parâmetro de [array.fill()](https://br.tradingview.com/pine-script-reference/v5/#fun_array{dot}fill), `index_to`, deve ser um a mais que o último _index_ que a função preencherá. Os elementos restantes conterão valores `na`, pois a chamada da função [array.new()](https://br.tradingview.com/pine-script-reference/v5/#fun_array.new%3Ctype%3E) não contém um argumento `initial_value`.


<!-- # Percorrendo Elementos de um Array

Ao percorrer os índices dos elementos de um array e o tamanho do array é desconhecido, pode-se usar a função array.size() para obter o valor máximo do índice. Por exemplo: -->

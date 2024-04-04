
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

<!-- ## Utilizando as Palavras-Chave `var` e `varip`

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
> Variáveis de array declaradas usando [varip](https://br.tradingview.com/pine-script-reference/v5/#kw_varip) comportam-se como as que utilizam [var](https://br.tradingview.com/pine-script-reference/v5/#kw_var) em _dados históricos_, mas atualizam seus valores para barras em tempo real (ou seja, as barras desde a última compilação do script) a cada novo tick de preço. Arrays atribuídos a variáveis [varip](https://br.tradingview.com/pine-script-reference/v5/#kw_varip) podem conter apenas tipos [int](https://br.tradingview.com/pine-script-reference/v5/#type_int), [float](https://br.tradingview.com/pine-script-reference/v5/#type_float), [bool](https://br.tradingview.com/pine-script-reference/v5/#type_bool), [color](https://br.tradingview.com/pine-script-reference/v5/#type_color) ou [string](https://br.tradingview.com/pine-script-reference/v5/#type_string) ou [tipos definidos pelo usuário](./04_09_tipagem_do_sistema.md#tipos-definidos-pelo-usuário) que contenham exclusivamente em seus campos esses tipos ou coleções ([arrays](./04_14_arrays.md), [matrices](./000_matrices.md) ou [maps](./000_maps.md)) desses tipos. -->




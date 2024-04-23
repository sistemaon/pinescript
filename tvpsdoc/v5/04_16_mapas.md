
# Maps (_Mapas_)

> __Observação__\
> Esta seção contém material avançado. Recomenda-se que programadores iniciantes em Pine Script se familiarizem com outras funcionalidades do Pine Script mais acessíveis antes de explorarem este conteúdo.

Os mapas do Pine Script são coleções que armazenam elementos em _key-value pairs_ (_pares de chave-valor_). Eles permitem que os scripts armazenem e associem múltiplos valores referenciados associados a identificadores únicos (chaves).

Ao contrário de [arrays](./04_14_arrays.md) e [_matrices_](./04_15_matrices.md), os mapas são considerados coleções _não ordenadas_. Scripts acessam rapidamente os valores de um mapa referenciando as chaves dos pares chave-valor inseridos neles, em vez de percorrer um _index_ interno.

As chaves de um mapa podem ser de qualquer _tipo fundamental_, e seus valores podem ser de qualquer tipo incorporado ou [definido pelo usuário](./04_09_tipagem_do_sistema.md#tipos-definidos-pelo-usuário). Mapas não podem usar diretamente outras _coleções_ ([mapas](./04_16_mapas.md), [arrays](./04_14_arrays.md) ou [_matrices_](./04_15_matrices.md)) como valores, mas podem conter instâncias de [UDT](./04_09_tipagem_do_sistema.md#tipos-definidos-pelo-usuário) que contêm essas estruturas de dados em seus campos. Veja [esta seção](./04_16_mapas.md#mapas-de-outras-coleções) para mais informações.

Assim como outras coleções, os mapas podem conter até 100.000 elementos no total. Uma vez que cada par chave-valor em um mapa consiste em dois elementos (uma chave _única_ e seu _valor_ associado), o número máximo de pares chave-valor que um mapa pode conter é 50.000.


# Declarando Mapa

Pine Script utiliza a seguinte sintaxe para declarar mapas:

```c
[var/varip ][map<keyType, valueType> ]<identifier> = <expression>
```

Onde `<keyType, valueType>` é o [modelo de tipo](./04_09_tipagem_do_sistema.md#templates-de-tipo) do mapa que declara os tipos de chaves e valores que ele conterá, e a `<expression>` retorna uma instância de mapa ou `na`.

Ao declarar uma variável de mapa atribuída a `na`, os usuários devem incluir a palavra-chave [map](https://br.tradingview.com/pine-script-reference/v5/#type_map) seguida de um [modelo de tipo](./04_09_tipagem_do_sistema.md#templates-de-tipo) para informar ao compilador que a variável pode aceitar mapas com chaves do tipo `keyType` e valores do tipo `valueType`.

Por exemplo, esta linha de código declara uma nova variável `myMap` que pode aceitar instâncias de mapa contendo pares de chaves do tipo [string](https://br.tradingview.com/pine-script-reference/v5/#type_string) e valores do tipo [float](https://br.tradingview.com/pine-script-reference/v5/#type_float):

```c
map<string, float> myMap = na
```

## Utilizando as Palavras-Chave `var` e `varip`

Usuários podem incluir as palavras-chave [var](https://br.tradingview.com/pine-script-reference/v5/#kw_var) ou [varip](https://br.tradingview.com/pine-script-reference/v5/#kw_varip) para instruir seus scripts a declarar variáveis de mapa apenas na primeira barra do gráfico. Variáveis que usam essas palavras-chave apontam para as mesmas instâncias de mapa em cada iteração do script até serem explicitamente reatribuídas.

Por exemplo, este script declara uma variável `colorMap` atribuída a um mapa que contém pares de chaves do tipo [string](https://br.tradingview.com/pine-script-reference/v5/#type_string) e valores do tipo [color](https://br.tradingview.com/pine-script-reference/v5/#type_color) (_cor_) na primeira barra do gráfico. O script exibe um `oscilador` no gráfico e usa os valores [put](https://br.tradingview.com/pine-script-reference/v5/#fun_map.put) (_inseridos_) no `colorMap` na _primeira_ barra para colorir os plots em _todas_ as barras:

![Declarando mapa utilizando as palavras-chave var e varip](./imgs/Maps-Declaring-a-map-Using-var-and-varip-keywords-1.png)

```c
//@version=5
indicator("var map demo")

//@variable A map associating color values with string keys.
var colorMap = map.new<string, color>()

// Put `<string, color>` pairs into `colorMap` on the first bar.
if bar_index == 0
    colorMap.put("Bull", color.green)
    colorMap.put("Bear", color.red)
    colorMap.put("Neutral", color.gray)

//@variable The 14-bar RSI of `close`.
float oscillator = ta.rsi(close, 14)

//@variable The color of the `oscillator`.
color oscColor = switch
    oscillator > 50 => colorMap.get("Bull")
    oscillator < 50 => colorMap.get("Bear")
    =>                 colorMap.get("Neutral")

// Plot the `oscillator` using the `oscColor` from our `colorMap`.
plot(oscillator, "Histogram", oscColor, 2, plot.style_histogram, histbase = 50)
plot(oscillator, "Line", oscColor, 3)
```

> __Observação__\
> Variáveis de mapa declaradas usando [varip](https://br.tradingview.com/pine-script-reference/v5/#kw_varip) comportam-se como aquelas usando [var](https://br.tradingview.com/pine-script-reference/v5/#kw_var) em dados históricos, mas atualizam seus pares chave-valor para barras em tempo real (ou seja, as barras desde a última compilação do script) a cada novo tick de preço. Mapas atribuídos a variáveis [varip](https://br.tradingview.com/pine-script-reference/v5/#kw_varip) só podem conter valores dos tipos [int](https://br.tradingview.com/pine-script-reference/v5/#type_int), [float](https://br.tradingview.com/pine-script-reference/v5/#type_float), [bool](https://br.tradingview.com/pine-script-reference/v5/#type_bool), [color](https://br.tradingview.com/pine-script-reference/v5/#type_color) ou [string](https://br.tradingview.com/pine-script-reference/v5/#type_string) ou [tipos definidos pelo usuário](./04_09_tipagem_do_sistema.md#tipos-definidos-pelo-usuário) que contenham exclusivamente dentro de seus campos esses tipos ou coleções ([arrays](./04_14_arrays.md), [matrizes](./04_15_matrices.md) ou [mapas](./04_16_mapas.md)) desses tipos.


# Lendo e Escrevendo

## Inserindo e Obtendo Pares Chave-Valor

A função [map.put()](https://br.tradingview.com/pine-script-reference/v5/#fun_map.put) é uma que os usuários de mapas utilizarão frequentemente, pois é o método principal para inserir um novo par chave-valor em um mapa. Ela associa o argumento `key` (_chave_) ao argumento `value` (_valor_) na chamada e adiciona o par ao id do mapa.

Se o argumento `key` (_chave_) na chamada de [map.put()](https://br.tradingview.com/pine-script-reference/v5/#fun_map.put) já existir nas [chaves](https://br.tradingview.com/pine-script-reference/v5/#fun_map.keys) do mapa, o novo par passado para a função __substituirá__ o existente.

Para recuperar o valor de um `id` de mapa associado a uma `key` (_chave_) específica, use [map.get()](https://br.tradingview.com/pine-script-reference/v5/#fun_map.get). Esta função retorna o valor se o mapa `id` [contiver](https://br.tradingview.com/pine-script-reference/v5/#fun_map.contains) a `key` (_chave_). Caso contrário, retorna [na](https://br.tradingview.com/pine-script-reference/v5/#var_na).

O exemplo a seguir calcula a diferença entre os valores de [bar_index](https://br.tradingview.com/pine-script-reference/v5/#var_bar_index) de quando o [fechamento](https://br.tradingview.com/pine-script-reference/v5/#var_close) estava [subindo](https://br.tradingview.com/pine-script-reference/v5/#fun_ta.rising) e [caindo](https://br.tradingview.com/pine-script-reference/v5/#fun_ta.falling) ao longo de um determinado `length`, com a ajuda dos métodos [map.put()](https://br.tradingview.com/pine-script-reference/v5/#fun_map.put) e [map.get()](https://br.tradingview.com/pine-script-reference/v5/#fun_map.get). O script insere um par `("Rising", bar_index)` no mapa de `data` (_dados_) quando o preço está subindo e insere um par `("Falling", bar_index)` no mapa quando o preço está caindo. Em seguida, insere no mapa um par contendo a "Difference" ("_Diferença_") entre os valores "Rising" ("_Subindo_") e "Falling" ("_Caindo_") e plota seu valor no gráfico:

![Lendo e escrevendo inserindo e obtendo pares chave-valor 01](./imgs/Maps-Reading-and-writing-Putting-and-getting-key-value-pairs-1.png)

```c
//@version=5
indicator("Putting and getting demo")

//@variable The length of the `ta.rising()` and `ta.falling()` calculation.
int length = input.int(2, "Length")

//@variable A map associating `string` keys with `int` values.
var data = map.new<string, int>()

// Put a new ("Rising", `bar_index`) pair into the `data` map when `close` is rising.
if ta.rising(close, length)
    data.put("Rising", bar_index)
// Put a new ("Falling", `bar_index`) pair into the `data` map when `close` is falling.
if ta.falling(close, length)
    data.put("Falling", bar_index)

// Put the "Difference" between current "Rising" and "Falling" values into the `data` map.
data.put("Difference", data.get("Rising") - data.get("Falling"))

//@variable The difference between the last "Rising" and "Falling" `bar_index`.
int index = data.get("Difference")

//@variable Returns `color.green` when `index` is positive, `color.red` when negative, and `color.gray` otherwise.
color indexColor = index > 0 ? color.green : index < 0 ? color.red : color.gray

plot(index, color = indexColor, style = plot.style_columns)
```

__Note que:__

- Este script substitui os valores associados às chaves "Rising" ("_Subindo_"), "Falling" ("_Caindo_") e "Difference" ("_Diferença_") em chamadas sucessivas de [data.put()](https://br.tradingview.com/pine-script-reference/v5/#fun_map.put), já que cada uma dessas chaves é única e só pode aparecer uma vez no mapa de `data` (_dados_).
- Substituir os pares em um mapa não altera a _ordem_ interna de _inserção_ de suas chaves. Isso é discutido mais detalhadamente na [próxima seção](./04_16_mapas.md#inspecionando-chaves-e-valores).

Semelhante ao trabalho com outras coleções, ao inserir um valor de um tipo especial ([line](https://br.tradingview.com/pine-script-reference/v5/#type_line), [linefill](https://br.tradingview.com/pine-script-reference/v5/#type_linefill), [box](https://br.tradingview.com/pine-script-reference/v5/#type_box), [polyline](https://br.tradingview.com/pine-script-reference/v5/#type_polyline), [label](https://br.tradingview.com/pine-script-reference/v5/#type_label), [table](https://br.tradingview.com/pine-script-reference/v5/#type_table) ou [chart.point](https://br.tradingview.com/pine-script-reference/v5/#type_chart.point)) ou [tipos definidos pelo usuário](./04_09_tipagem_do_sistema.md#tipos-definidos-pelo-usuário) em um mapa, é importante notar que o `value` (_valor_) do par inserido aponta para o mesmo objeto sem copiá-lo. Modificar o valor referenciado por um par chave-valor também afetará o objeto original.

Por exemplo, este script contém um tipo personalizado `ChartData` com os campos `o`, `h`, `l` e `c`. Na primeira barra do gráfico, o script declara uma variável `myMap` e adiciona o par `("A", myData)`, onde `myData` é uma instância de `ChartData` com valores iniciais de campo `na`. Ele adiciona o par `("B", myData)` ao `myMap` e atualiza o objeto deste par em cada barra por meio do método `update()` definido pelo usuário.

A cada alteração no objeto com a chave "B" afeta o objeto referenciado pela chave "A", como mostrado pelo gráfico de velas dos campos do objeto "A":

![Lendo e escrevendo inserindo e obtendo pares chave-valor 02](./imgs/Maps-Reading-and-writing-Putting-and-getting-key-value-pairs-2.png)

```c
//@version=5
indicator("Putting and getting objects demo")

//@type A custom type to hold OHLC data.
type ChartData
    float o
    float h
    float l
    float c

//@function Updates the fields of a `ChartData` object.
method update(ChartData this) =>
    this.o := open
    this.h := high
    this.l := low
    this.c := close

//@variable A new `ChartData` instance declared on the first bar.
var myData = ChartData.new()
//@variable A map associating `string` keys with `ChartData` instances.
var myMap = map.new<string, ChartData>()

// Put a new pair with the "A" key into `myMap` only on the first bar.
if bar_index == 0
    myMap.put("A", myData)

// Put a pair with the "B" key into `myMap` on every bar.
myMap.put("B", myData)

//@variable The `ChartData` value associated with the "A" key in `myMap`.
ChartData oldest = myMap.get("A")
//@variable The `ChartData` value associated with the "B" key in `myMap`.
ChartData newest = myMap.get("B")

// Update `newest`. Also affects `oldest` and `myData` since they all reference the same `ChartData` object.
newest.update()

// Plot the fields of `oldest` as candles.
plotcandle(oldest.o, oldest.h, oldest.l, oldest.c)
```

__Note que:__

- Este script se comportaria de maneira diferente se passasse uma cópia de `myData` em cada chamada de [myMap.put()](https://br.tradingview.com/pine-script-reference/v5/#fun_map.put). Para mais informações, consulte [esta seção](./04_12_objetos.md#copiando-objetos) da página do Manual do Usuário sobre [objetos](./04_12_objetos.md).

## Inspecionando Chaves e Valores

### `map.keys()` e `map.values()`


# Mapas de Outras Coleções


# Métodos

> __Observação__\
> Esta seção contém material avançado. Se és um programador iniciante em Pine Script, recomendamos que se familiarize com outros recursos do Pine Script, mais acessíveis, antes de se _aventurar_ nessa parte.

Os métodos do Pine Script são funções especializadas associadas a instâncias específicas de [tipos](./04_09_tipagem_do_sistema.md) incorporados ou definidos pelo usuário. São essencialmente os mesmos que funções regulares na maioria dos aspectos, mas oferecem uma sintaxe mais curta e conveniente. Os usuários podem acessar os métodos utilizando a _dot notation_ (_notação de ponto_) diretamente nas variáveis, assim como acessam os campos de um [objeto](./04_12_objetos.md) do Pine Script.


# Métodos Incorporados

O Pine Script inclui métodos integrados para todos os _tipos especiais_, incluindo [array](https://br.tradingview.com/pine-script-reference/v5/#type_array), [matrix](https://br.tradingview.com/pine-script-reference/v5/#type_matrix), [map](https://br.tradingview.com/pine-script-reference/v5/#type_map), [line](https://br.tradingview.com/pine-script-reference/v5/#type_line), [linefill](https://br.tradingview.com/pine-script-reference/v5/#type_linefill), [box](https://br.tradingview.com/pine-script-reference/v5/#type_box), [polyline](https://br.tradingview.com/pine-script-reference/v5/#type_polyline), [label](https://br.tradingview.com/pine-script-reference/v5/#type_label), e [table](https://br.tradingview.com/pine-script-reference/v5/#type_table) (_array_, _matriz_, _mapa_, _linha_, _preenchimento de linha_, _caixa_, _polilinha_, _rótulo_ e _tabela_). Esses métodos oferecem aos usuários uma maneira mais concisa de chamar rotinas especializadas para esses tipos dentro dos scripts.

Ao usar esses tipos especiais, as expressões:

```c
<namespace>.<functionName>([paramName =] <objectName>, …)
```

E:

```c
<objectName>.<functionName>(…)
```

São equivalentes. Por exemplo, em vez de usar:

```c
array.get(id, index)
```

Para obter o valor de um `id` de array no `index` especificado, pode-se simplesmente usar:

```c
id.get(index)
```

Para alcançar o mesmo efeito. Esta notação elimina a necessidade dos usuários referenciarem o _namespace_ da função, já que [get()](https://br.tradingview.com/pine-script-reference/v5/#fun_array{dot}get) é um método de `id` neste contexto.

Abaixo está um exemplo prático para demonstrar o uso de métodos integrados em lugar de funções.

O script a seguir calcula as _Bandas de Bollinger_ sobre um número especificado de preços amostrados uma vez a cada `n` barras. Chamando [array.push()](https://br.tradingview.com/pine-script-reference/v5/#fun_array{dot}push) e [array.shift()](https://br.tradingview.com/pine-script-reference/v5/#fun_array{dot}shift) para enfileirar valores de `sourceInput` através do `sourceArray`, em seguida [array.avg()](https://br.tradingview.com/pine-script-reference/v5/#fun_array{dot}avg) e [array.stdev()](https://br.tradingview.com/pine-script-reference/v5/#fun_array{dot}stdev) para calcular o `sampleMean` e `sampleDev`. O script então utiliza esses valores para calcular o `highBand` e `lowBand`, que é plotado no gráfico junto com o `sampleMean`:

![Métodos incorporados bandas de bollinger](./imgs/Methods_custom_bb.png)

```c
//@version=5
indicator("Custom Sample BB", overlay = true)

float sourceInput  = input.source(close, "Source")
int   samplesInput = input.int(20, "Samples")
int   n            = input.int(10, "Bars")
float multiplier   = input.float(2.0, "StdDev")

var array<float> sourceArray = array.new<float>(samplesInput)
var float        sampleMean  = na
var float        sampleDev   = na

// Identify if `n` bars have passed.
if bar_index % n == 0
    // Update the queue.
    array.push(sourceArray, sourceInput)
    array.shift(sourceArray)
    // Update the mean and standard deviaiton values.
    sampleMean := array.avg(sourceArray)
    sampleDev  := array.stdev(sourceArray) * multiplier

// Calculate bands.
float highBand = sampleMean + sampleDev
float lowBand  = sampleMean - sampleDev

plot(sampleMean, "Basis", color.orange)
plot(highBand, "Upper", color.lime)
plot(lowBand, "Lower", color.red)
```

Reescrevendo este código para utilizar métodos em vez de funções integradas. Nesta versão, foi substituído todas as funções integradas [array.*](https://br.tradingview.com/pine-script-reference/v5/#type_array) no script por métodos equivalentes:

```c
//@version=5
indicator("Custom Sample BB", overlay = true)

float sourceInput  = input.source(close, "Source")
int   samplesInput = input.int(20, "Samples")
int   n            = input.int(10, "Bars")
float multiplier   = input.float(2.0, "StdDev")

var array<float> sourceArray = array.new<float>(samplesInput)
var float        sampleMean  = na
var float        sampleDev   = na

// Identify if `n` bars have passed.
if bar_index % n == 0
    // Update the queue.
    sourceArray.push(sourceInput)
    sourceArray.shift()
    // Update the mean and standard deviaiton values.
    sampleMean := sourceArray.avg()
    sampleDev  := sourceArray.stdev() * multiplier

// Calculate band values.
float highBand = sampleMean + sampleDev
float lowBand  = sampleMean - sampleDev

plot(sampleMean, "Basis", color.orange)
plot(highBand, "Upper", color.lime)
plot(lowBand, "Lower", color.red)
```

Observe que:

- Chama-se os métodos do array usando `sourceArray.*` em vez de referenciar o _namespace_ do [array](https://br.tradingview.com/pine-script-reference/v5/#type_array).
- Não é incluído `sourceArray` como um parâmetro quando chama-se os métodos, uma vez que já fazem referência ao objeto.
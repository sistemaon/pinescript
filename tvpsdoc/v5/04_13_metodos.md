
# Métodos

> __Observação!__\
> Esta seção contém material avançado. Recomenda-se que programadores iniciantes em Pine Script se familiarizem com outras funcionalidades do Pine Script mais acessíveis antes de explorarem este conteúdo.

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

__Note que:__

- Chama-se os métodos do array usando `sourceArray.*` em vez de referenciar o _namespace_ do [array](https://br.tradingview.com/pine-script-reference/v5/#type_array).
- Não é incluído `sourceArray` como um parâmetro quando chama-se os métodos, uma vez que já fazem referência ao objeto.


# Métodos Definidos pelo Usuário

O Pine Script permite que os usuários definam métodos personalizados para uso com objetos de qualquer tipo embutido ou definido pelo usuário.
Definir um método é essencialmente o mesmo que definir uma função, mas com duas principais diferenças:

- A palavra-chave [method](https://br.tradingview.com/pine-script-reference/v5/#kw_method) deve ser incluída antes do nome da função.
- O tipo do primeiro parâmetro na assinatura deve ser explicitamente declarado, pois representa o tipo de objeto ao qual o método será associado.

```c
[export] method <functionName>(<paramType> <paramName> [= <defaultValue>], …) =>
    <functionBlock>
```

Aplicando métodos definidos pelo usuário ao exemplo anterior de _Bandas de Bollinger_ para encapsular operações do escopo global, na qual simplificará o código e promoverá a reutilização.

Veja esta parte do exemplo:

```c
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
```

Definindo um método simples para enfileirar valores através de um array em uma única chamada.
Este método `maintainQueue()` invoca os métodos [push()](https://br.tradingview.com/pine-script-reference/v5/#fun_array{dot}push) e [shift()](https://br.tradingview.com/pine-script-reference/v5/#fun_array{dot}shift) no `srcArray` quando `takeSample` é `true` e retorna o objeto:

```c
// @function         Maintains a queue of the size of `srcArray`.
//                   It appends a `value` to the array and removes its oldest element at position zero.
// @param srcArray   (array<float>) The array where the queue is maintained.
// @param value      (float) The new value to be added to the queue.
//                   The queue's oldest value is also removed, so its size is constant.
// @param takeSample (bool) A new `value` is only pushed into the queue if this is true.
// @returns          (array<float>) `srcArray` object.
method maintainQueue(array<float> srcArray, float value, bool takeSample = true) =>
    if takeSample
        srcArray.push(value)
        srcArray.shift()
    srcArray
```

__Note que:__

- Assim como com funções definidas pelo usuário, usa-se a [anotação de compilador](./04_03_estrutura_do_script.md#anotações-do-compilador) `@function` para documentar descrições de métodos.

Agora pode substituir `sourceArray.push()` e `sourceArray.shift()` por `sourceArray.maintainQueue()` no seguinte exemplo:

```c
// Identify if `n` bars have passed.
if bar_index % n == 0
    // Update the queue.
    sourceArray.maintainQueue(sourceInput)
    // Update the mean and standard deviaiton values.
    sampleMean  := sourceArray.avg()
    sampleDev   := sourceArray.stdev() * multiplier

// Calculate band values.
float highBand  = sampleMean + sampleDev
float lowBand   = sampleMean - sampleDev
```

A partir daqui, é possível simplificar ainda mais o código definindo um método que lida com todos os cálculos das _Bandas de Bollinger_ dentro do próprio escopo.

O método `calcBB()` aciona os métodos [avg()](https://br.tradingview.com/pine-script-reference/v5/#fun_array{dot}avg) e [stdev()](https://br.tradingview.com/pine-script-reference/v5/#fun_array{dot}stdev) no `srcArray` para atualizar os valores de `mean` e `dev` quando `calculate` for `true`. O método utiliza esses valores para retornar uma tupla contendo os valores de _basis_, _upper band_, e _lower band_ (_base_, _banda superior_ e _banda inferior_), respectivamente:

```c
// @function         Computes Bollinger Band values from an array of data.
// @param srcArray   (array<float>) The array where the queue is maintained.
// @param multiplier (float) Standard deviaiton multiplier.
// @param calcuate   (bool) The method will only calculate new values when this is true.
// @returns          A tuple containing the basis, upper band, and lower band respectively.
method calcBB(array<float> srcArray, float mult, bool calculate = true) =>
    var float mean = na
    var float dev  = na
    if calculate
        // Compute the mean and standard deviation of the array.
        mean := srcArray.avg()
        dev  := srcArray.stdev() * mult
    [mean, mean + dev, mean - dev]
```

Com este método, agora é possível remover os cálculos das _Bandas de Bollinger_ do escopo global e melhorar a legibilidade do código:

```c
// Identify if `n` bars have passed.
bool newSample = bar_index % n == 0

// Update the queue and compute new BB values on each new sample.
[sampleMean, highBand, lowBand] = sourceArray.maintainQueue(sourceInput, newSample).calcBB(multiplier, newSample)
```

__Note que:__

- Em vez de usar um bloco `if` no escopo global, foi definido uma variável `newSample` que é `true` apenas uma vez a cada `n` barras. Os métodos `maintainQueue()` e `calcBB()` utilizam esse valor para seus respectivos parâmetros `takeSample` e `calculate`.
- Uma vez que o método `maintainQueue()` retorna o objeto ao qual se refere, é possível invocar `calcBB()` na mesma linha de código, já que ambos os métodos se aplicam a instâncias de `array<float>`.

Eis como o script completo se apresenta após a aplicação dos métodos definidos pelo usuário:

```c
//@version=5
indicator("Custom Sample BB", overlay = true)

float sourceInput  = input.source(close, "Source")
int   samplesInput = input.int(20, "Samples")
int   n            = input.int(10, "Bars")
float multiplier   = input.float(2.0, "StdDev")

var array<float> sourceArray = array.new<float>(samplesInput)

// @function         Maintains a queue of the size of `srcArray`.
//                   It appends a `value` to the array and removes its oldest element at position zero.
// @param srcArray   (array<float>) The array where the queue is maintained.
// @param value      (float) The new value to be added to the queue.
//                   The queue's oldest value is also removed, so its size is constant.
// @param takeSample (bool) A new `value` is only pushed into the queue if this is true.
// @returns          (array<float>) `srcArray` object.
method maintainQueue(array<float> srcArray, float value, bool takeSample = true) =>
    if takeSample
        srcArray.push(value)
        srcArray.shift()
    srcArray

// @function         Computes Bollinger Band values from an array of data.
// @param srcArray   (array<float>) The array where the queue is maintained.
// @param multiplier (float) Standard deviaiton multiplier.
// @param calcuate   (bool) The method will only calculate new values when this is true.
// @returns          A tuple containing the basis, upper band, and lower band respectively.
method calcBB(array<float> srcArray, float mult, bool calculate = true) =>
    var float mean = na
    var float dev  = na
    if calculate
        // Compute the mean and standard deviation of the array.
        mean := srcArray.avg()
        dev  := srcArray.stdev() * mult
    [mean, mean + dev, mean - dev]

// Identify if `n` bars have passed.
bool newSample = bar_index % n == 0

// Update the queue and compute new BB values on each new sample.
[sampleMean, highBand, lowBand] = sourceArray.maintainQueue(sourceInput, newSample).calcBB(multiplier, newSample)

plot(sampleMean, "Basis", color.orange)
plot(highBand, "Upper", color.lime)
plot(lowBand, "Lower", color.red)
```


# Sobrecarga de Métodos

Métodos definidos pelo usuário podem sobrescrever e sobrecarregar métodos integrados e definidos pelo usuário já existentes com o mesmo identificador. Essa capacidade permite aos usuários definir múltiplas rotinas associadas a diferentes assinaturas de parâmetros sob o mesmo nome de método.

Como um exemplo simples, suponha a definição de um método para identificar o tipo de uma variável. Uma vez que é necessário especificar explicitamente o tipo de objeto associado a um método definido pelo usuário, será preciso definir sobrecargas para cada tipo que se deseja reconhecer.

Abaixo, é definido um método `getType()` que retorna uma representação em string do tipo de uma variável, com sobrecargas para os cinco tipos primitivos:

```c
// @function   Identifies an object's type.
// @param this Object to inspect.
// @returns    (string) A string representation of the type.
method getType(int this) =>
    na(this) ? "int(na)" : "int"

method getType(float this) =>
    na(this) ? "float(na)" : "float"

method getType(bool this) =>
    na(this) ? "bool(na)" : "bool"

method getType(color this) =>
    na(this) ? "color(na)" : "color"

method getType(string this) =>
    na(this) ? "string(na)" : "string"
```

Agora é possível utilizar essas sobrecargas para inspecionar algumas variáveis. Este script usa [str.format()](https://br.tradingview.com/pine-script-reference/v5/#fun_str{dot}format) para formatar os resultados obtidos ao chamar o método `getType()` em cinco variáveis diferentes em uma única string de `results` (_resultados_), e então exibe a string no label `lbl` usando o método integrado [set_text()](https://br.tradingview.com/pine-script-reference/v5/#fun_label{dot}set_text):

![Sobrecarga de métodos](./imgs/Methods_overloads_type_inspection.png)

```c
//@version=5
indicator("Type Inspection")

// @function   Identifies an object's type.
// @param this Object to inspect.
// @returns    (string) A string representation of the type.
method getType(int this) =>
    na(this) ? "int(na)" : "int"

method getType(float this) =>
    na(this) ? "float(na)" : "float"

method getType(bool this) =>
    na(this) ? "bool(na)" : "bool"

method getType(color this) =>
    na(this) ? "color(na)" : "color"

method getType(string this) =>
    na(this) ? "string(na)" : "string"

a = 1
b = 1.0
c = true
d = color.white
e = "1"

// Inspect variables and format results.
results = str.format(
 "a: {0}\nb: {1}\nc: {2}\nd: {3}\ne: {4}",
 a.getType(), b.getType(), c.getType(), d.getType(), e.getType()
 )

var label lbl = label.new(0, 0)
lbl.set_x(bar_index)
lbl.set_text(results)
```

__Note que:__

- O tipo subjacente de cada variável determina qual sobrecarga do `getType()` o compilador utilizará.
- O método irá juntar "(na)" à string de saída quando uma variável estiver como `na` para demarcar que está vazia.


# Exemplo Avançado

Aplicando o conhecimento adquirido, é possível construir um script que estima a distribuição cumulativa dos elementos em um array, isto é, a fração dos elementos no array que são menores ou iguais a um determinado valor.

Existem várias maneiras de abordar esse objetivo. Neste exemplo, inicia-se definindo um método para substituir elementos de um array, o que facilitará a contagem das ocorrências de elementos dentro de um intervalo de valores.

O script a seguir, está apresentada uma sobrecarga do método integrado [fill()](https://br.tradingview.com/pine-script-reference/v5/#fun_array{dot}fill) para instâncias de `array<float>`. Essa sobrecarga substitui elementos em `srcArray` dentro do intervalo entre `lowerBound` e `upperBound` com um `innerValue` e substitui todos os elementos fora do intervalo com `outerValue`:

```c
// @function          Replaces elements in a `srcArray` between `lowerBound` and `upperBound` with an `innerValue`,
//                    and replaces elements outside the range with an `outerValue`.
// @param srcArray    (array<float>) Array to modify.
// @param innerValue  (float) Value to replace elements within the range with.
// @param outerValue  (float) Value to replace elements outside the range with.
// @param lowerBound  (float) Lowest value to replace with `innerValue`.
// @param upperBound  (float) Highest value to replace with `innerValue`.
// @returns           (array<float>) `srcArray` object.
method fill(array<float> srcArray, float innerValue, float outerValue, float lowerBound, float upperBound) =>
    for [i, element] in srcArray
        if (element >= lowerBound or na(lowerBound)) and (element <= upperBound or na(upperBound))
            srcArray.set(i, innerValue)
        else
            srcArray.set(i, outerValue)
    srcArray
```

Com este método, é possível filtrar um array por intervalos de valores para produzir um array de ocorrências.

Por exemplo, a expressão:

```c
srcArray.copy().fill(1.0, 0.0, min, val)
```

Copia o objeto `srcArray`, substitui todos os elementos entre `min` e `val` por _1.0_ e, em seguida, substitui todos os elementos acima de `val` por _0.0_. A partir daqui, é fácil estimar o resultado da função de distribuição cumulativa no `val`, já que é simplesmente a média do array resultante:

```c
srcArray.copy().fill(1.0, 0.0, min, val).avg()
```

__Note que:__

- O compilador só usará essa sobrecarga de `fill()` em vez da incorporada quando o usuário fornecer argumentos para `innerValue`, `outerValue`, `lowerBound` e `upperBound` na chamada.
- Se `lowerBound` ou `upperBound` for `na`, o valor é ignorado ao filtrar o intervalo de preenchimento.
- É possível chamar `copy(),` `fill()` e `avg()` sucessivamente na mesma linha de código porque os dois primeiros métodos retornam uma instância de `array<float>`.

Agora é possível utilizar isso para definir um método que calculará os valores de distribuição empírica. O método `eCDF()` a seguir estima um número de etapas ascendentes igualmente `steps` (_espaçadas_) a partir da função de distribuição cumulativa de `srcArray` e insere os resultados em `cdfArray`:

```c
// @function       Estimates the empirical CDF of a `srcArray`.
// @param srcArray (array<float>) Array to calculate on.
// @param steps    (int) Number of steps in the estimation.
// @returns        (array<float>) Array of estimated CDF ratios.
method eCDF(array<float> srcArray, int steps) =>
    float min = srcArray.min()
    float rng = srcArray.range() / steps
    array<float> cdfArray = array.new<float>()
    // Add averages of `srcArray` filtered by value region to the `cdfArray`.
    float val = min
    for i = 1 to steps
        val += rng
        cdfArray.push(srcArray.copy().fill(1.0, 0.0, min, val).avg())
    cdfArray
```

Por fim, para garantir que o método `eCDF()` funcione adequadamente para arrays contendo valores pequenos e grandes, é definido um método para normalizar os arrays.

O método `featureScale()` utiliza os métodos [min()](https://br.tradingview.com/pine-script-reference/v5/#fun_array{dot}min) e [range()](https://br.tradingview.com/pine-script-reference/v5/#fun_array{dot}range) do array para produzir uma cópia redimensionada de `srcArray`. Utiliza-se isso para normalizar os arrays antes de invocar o método `eCDF()`:

```c
// @function        Rescales the elements within a `srcArray` to the interval [0, 1].
// @param srcArray  (array<float>) Array to normalize.
// @returns         (array<float>) Normalized copy of the `srcArray`.
method featureScale(array<float> srcArray) =>
    float min = srcArray.min()
    float rng = srcArray.range()
    array<float> scaledArray = array.new<float>()
    // Push normalized `element` values into the `scaledArray`.
    for element in srcArray
        scaledArray.push((element - min) / rng)
    scaledArray
```

__Note que:__

- Este método não possui tratamento específico para situações de divisão por zero. Se `rng` for 0, o valor do elemento do array será considerado como _não disponível_ `na`.

No exemplo completo a seguir, o `sourceArray` de `length` (_tamanho_) especificado é preenchido com valores de `sourceInput` utilizando o método `maintainQueue()` mencionado anteriormente. Em seguida, os elementos do array são normalizados por meio do método `featureScale()`, e o método `eCDF()` é invocado para buscar um array de estimativas para `n` _um número definido_ de etapas distribuídas uniformemente pela distribuição. Por fim, uma função `makeLabel()`, definida pelo usuário, é empregada para apresentar as estimativas e preços em num label posicionado no lado direito do gráfico:

![Exemplo avançado](./imgs/Methods_empirical_distribution.png)

```c
//@version=5
indicator("Empirical Distribution", overlay = true)

float sourceInput = input.source(close, "Source")
int length        = input.int(20, "Length")
int n             = input.int(20, "Steps")

// @function         Maintains a queue of the size of `srcArray`.
//                   It appends a `value` to the array and removes its oldest element at position zero.
// @param srcArray   (array<float>) The array where the queue is maintained.
// @param value      (float) The new value to be added to the queue.
//                   The queue's oldest value is also removed, so its size is constant.
// @param takeSample (bool) A new `value` is only pushed into the queue if this is true.
// @returns          (array<float>) `srcArray` object.
method maintainQueue(array<float> srcArray, float value, bool takeSample = true) =>
    if takeSample
        srcArray.push(value)
        srcArray.shift()
    srcArray

// @function          Replaces elements in a `srcArray` between `lowerBound` and `upperBound` with an `innerValue`,
//                    and replaces elements outside the range with an `outerValue`.
// @param srcArray    (array<float>) Array to modify.
// @param innerValue  (float) Value to replace elements within the range with.
// @param outerValue  (float) Value to replace elements outside the range with.
// @param lowerBound  (float) Lowest value to replace with `innerValue`.
// @param upperBound  (float) Highest value to replace with `innerValue`.
// @returns           (array<float>) `srcArray` object.
method fill(array<float> srcArray, float innerValue, float outerValue, float lowerBound, float upperBound) =>
    for [i, element] in srcArray
        if (element >= lowerBound or na(lowerBound)) and (element <= upperBound or na(upperBound))
            srcArray.set(i, innerValue)
        else
            srcArray.set(i, outerValue)
    srcArray

// @function       Estimates the empirical CDF of a `srcArray`.
// @param srcArray (array<float>) Array to calculate on.
// @param steps    (int) Number of steps in the estimation.
// @returns        (array<float>) Array of estimated CDF ratios.
method eCDF(array<float> srcArray, int steps) =>
    float min = srcArray.min()
    float rng = srcArray.range() / steps
    array<float> cdfArray = array.new<float>()
    // Add averages of `srcArray` filtered by value region to the `cdfArray`.
    float val = min
    for i = 1 to steps
        val += rng
        cdfArray.push(srcArray.copy().fill(1.0, 0.0, min, val).avg())
    cdfArray

// @function        Rescales the elements within a `srcArray` to the interval [0, 1].
// @param srcArray  (array<float>) Array to normalize.
// @returns         (array<float>) Normalized copy of the `srcArray`.
method featureScale(array<float> srcArray) =>
    float min = srcArray.min()
    float rng = srcArray.range()
    array<float> scaledArray = array.new<float>()
    // Push normalized `element` values into the `scaledArray`.
    for element in srcArray
        scaledArray.push((element - min) / rng)
    scaledArray

// @function        Draws a label containing eCDF estimates in the format "{price}: {percent}%"
// @param srcArray  (array<float>) Array of source values.
// @param cdfArray  (array<float>) Array of CDF estimates.
// @returns         (void)
makeLabel(array<float> srcArray, array<float> cdfArray) =>
    float max      = srcArray.max()
    float rng      = srcArray.range() / cdfArray.size()
    string results = ""
    var label lbl  = label.new(0, 0, "", style = label.style_label_left, text_font_family = font.family_monospace)
    // Add percentage strings to `results` starting from the `max`.
    cdfArray.reverse()
    for [i, element] in cdfArray
        results += str.format("{0}: {1}%\n", max - i * rng, element * 100)
    // Update `lbl` attributes.
    lbl.set_xy(bar_index + 1, srcArray.avg())
    lbl.set_text(results)

var array<float> sourceArray = array.new<float>(length)

// Add background color for the last `length` bars.
bgcolor(bar_index > last_bar_index - length ? color.new(color.orange, 80) : na)

// Queue `sourceArray`, feature scale, then estimate the distribution over `n` steps.
array<float> distArray = sourceArray.maintainQueue(sourceInput).featureScale().eCDF(n)
// Draw label.
makeLabel(sourceArray, distArray)
```


# Libraries (_Bibliotecas_)

As bibliotecas Pine Script são publicações que contêm funções que podem ser reutilizadas em indicadores, estratégias ou em outras bibliotecas. Elas são úteis para definir funções frequentemente usadas, de modo que seu "_open source_" ("_código fonte_") não precise ser incluído em todos os scripts onde são necessárias.

Uma biblioteca deve ser publicada (privadamente ou publicamente) antes de poder ser usada em outro script. Todas as bibliotecas são publicadas como código aberto. Scripts públicos só podem usar bibliotecas públicas e devem ser de código aberto. Scripts privados ou scripts pessoais salvos no Editor Pine Script podem usar bibliotecas públicas ou privadas. Uma biblioteca pode usar outras bibliotecas, ou até mesmo versões anteriores de si mesma.

Programadores de bibliotecas devem estar familiarizados com a nomenclatura de tipagem do Pine Script, escopos e funções definidas pelo usuário. Se for necessário revisar os tipos qualificados, veja a página do Manual do Usuário sobre o [Sistema de Tipos](./04_09_tipagem_do_sistema.md). Para mais informações sobre funções definidas pelo usuário e escopos, veja a página [Funções Definidas pelo Usuário](./04_11_funcoes_definida_pelo_usuario.md).

Você pode navegar pelos scripts de biblioteca publicados publicamente por membros na seção [Scripts da Comunidade](https://br.tradingview.com/scripts/?script_type=libraries) do TradingView.


# Criando uma Biblioteca

Uma biblioteca é um tipo especial de script que começa com a declaração [library()](https://br.tradingview.com/pine-script-reference/v5/#fun_library), em vez de [indicator()](https://br.tradingview.com/pine-script-reference/v5/#fun_indicator) ou [strategy()](https://br.tradingview.com/pine-script-reference/v5/#fun_strategy). Uma biblioteca contém definições de funções exportáveis, que constituem a única parte visível da biblioteca quando é usada por outro script. Bibliotecas também podem usar outro código Pine Script em seu escopo global, como um indicador normal. Esse código geralmente serve para demonstrar como usar as funções da biblioteca.

Um script de biblioteca tem a seguinte estrutura, onde uma ou mais funções exportáveis devem ser definidas:

```c
//@version=5

// @description <library_description>
library(title, overlay)

<script_code>

// @function <function_description>
// @param <parameter> <parameter_description>
// @returns <return_value_description>
export <function_name>([simple/series] <parameter_type> <parameter_name> [= <default_value>] [, ...]) =>
    <function_code>

<script_code>
```

__Note que:__

- As [anotações do compilador](./04_03_estrutura_do_script.md#anotações-do-compilador) `// @description`, `// @function`, `// @param` e `// @returns` são opcionais, mas é altamente recomendável usá-las. Elas têm um duplo propósito: documentar o código da biblioteca e preencher a descrição padrão da biblioteca que os autores podem usar ao publicar a biblioteca.
- A palavra-chave [`export`](https://br.tradingview.com/pine-script-reference/v5/#kw_export) é obrigatória.
- `<parameter_type>` é obrigatório, ao contrário das definições de parâmetros de funções definidas pelo usuário em indicadores ou estratégias, que são sem tipo.
- `<script_code>` pode ser qualquer código que normalmente seria usado em um indicador, incluindo entradas ou plots.

Este é um exemplo de biblioteca:

```c
//@version=5

// @description Provides functions calculating the all-time high/low of values.
library("AllTimeHighLow", true)

// @function Calculates the all-time high of a series.
// @param val Series to use (`high` is used if no argument is supplied).
// @returns The all-time high for the series.
export hi(float val = high) =>
    var float ath = val
    ath := math.max(ath, val)

// @function Calculates the all-time low of a series.
// @param val Series to use (`low` is used if no argument is supplied).
// @returns The all-time low for the series.
export lo(float val = low) =>
    var float atl = val
    atl := math.min(atl, val)

plot(hi())
plot(lo())
```

## Funções de Biblioteca

As definições de funções em bibliotecas são um pouco diferentes das funções definidas pelo usuário em indicadores e estratégias. Existem restrições quanto ao que pode ser incluído no corpo das funções de biblioteca.

Nas assinaturas de funções de biblioteca (sua primeira linha):

- A palavra-chave [export](https://br.tradingview.com/pine-script-reference/v5/#kw_export) é obrigatória.
- O tipo de argumento esperado para cada parâmetro deve ser explicitamente mencionado.
- Uma palavra-chave [simple](https://br.tradingview.com/pine-script-reference/v5/#type_simple) ou [series](https://br.tradingview.com/pine-script-reference/v5/#type_series) pode restringir os tipos qualificados permitidos para os argumentos (a próxima seção explica seu uso).

Estas são as restrições impostas às funções de biblioteca:

- Elas não podem usar variáveis do escopo global da biblioteca, a menos que sejam qualificadas como "const". Isso significa que não se pode usar variáveis globais inicializadas a partir de entradas de script, por exemplo, ou arrays declarados globalmente.
- Chamadas `request.*()` não são permitidas.
- Chamadas `input.*()` não são permitidas.
- Chamadas `plot*()`, `fill()` e `bgcolor()` não são permitidas.

As funções de biblioteca sempre retornam um resultado qualificado como "simple" ou "series". Elas não podem ser usadas onde são necessários valores qualificados como "const" ou "input", como é o caso de algumas funções embutidas. Por exemplo, uma função de biblioteca não pode ser usada para calcular um argumento para o parâmetro `show_last` em uma chamada [plot()](https://br.tradingview.com/pine-script-reference/v5/#fun_plot) porque ele requer um valor "input int".

## Controle de Tipo Qualificado

Os tipos qualificados de argumentos fornecidos em chamadas para funções de biblioteca são autodetectados com base em como cada argumento é usado dentro da função. Se o argumento pode ser usado como "series", então é qualificado como tal. Se não puder, uma tentativa é feita com o qualificador de tipo "simple". Isso explica por que este código:

```c
export myEma(int x) =>
    ta.ema(close, x)
```

Funcionará quando chamado usando `myCustomLibrary.myEma(20)`, mesmo que o parâmetro `length` de [ta.ema()](https://br.tradingview.com/pine-script-reference/v5/#fun_ta{dot}ema) exija um argumento "simple int". Quando o compilador Pine Script detecta que um comprimento "series" não pode ser usado com [ta.ema()](https://br.tradingview.com/pine-script-reference/v5/#fun_ta{dot}ema), ele tenta o qualificador "simple", que neste caso é permitido.

Embora funções de biblioteca não possam retornar valores "const" ou "input", elas podem ser escritas para produzir resultados "simple". Isso as torna úteis em mais contextos do que funções que retornam resultados "series", já que algumas funções embutidas não permitem argumentos "series". Por exemplo, [request.security()](https://br.tradingview.com/pine-script-reference/v5/#fun_request{dot}security) requer uma "simple string" para seu parâmetro `symbol`. Se uma função de biblioteca fosse escrita para montar o argumento para `symbol` da seguinte maneira, o resultado da função não funcionaria porque é do tipo qualificado "series string":

```c
export makeTickerid(string prefix, string ticker) =>
    prefix + ":" + ticker
```

No entanto, restringindo os qualificadores de parâmetro para "simple", é possível forçar a função a produzir um resultado "simple". Isso pode ser alcançado prefixando o tipo dos parâmetros com a palavra-chave [simple](https://br.tradingview.com/pine-script-reference/v5/#type_simple):

```c
export makeTickerid(simple string prefix, simple string ticker) =>
    prefix + ":" + ticker
```

Observe que para a função retornar um valor "simple", nenhum valor "series" pode ser usado em seu cálculo; caso contrário, o resultado será um valor "series".

Também é possível usar a palavra-chave [series](https://br.tradingview.com/pine-script-reference/v5/#type_simple) para prefixar o tipo de um parâmetro de função de biblioteca. No entanto, como os argumentos são qualificados como "series" por padrão, usar o modificador [series](https://br.tradingview.com/pine-script-reference/v5/#type_simple) é redundante.

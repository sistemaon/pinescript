
# Funções Definidas pelo Usuário

Funções definidas pelo usuário são funções que se cria, ao contrário das funções incorporadas no Pine Script. São úteis para definir cálculos que precisa fazer repetidamente ou que deseja isolar da seção principal de cálculos do script. Pense em funções definidas pelo usuário como uma forma de estender as capacidades do Pine Script quando nenhuma função integrada fizer o que precisa ser feito.

Pode-se escrever as funções em uma das duas maneiras:

- Em uma única linha, quando forem simples.
- Em várias linhas.

As funções podem ser localizadas em dois lugares:

- Se uma função é usada apenas em um script, pode incluí-la no script onde é usada. Consulte o [Guia de Estilo](./000_style_guide.md) para recomendações sobre onde colocar funções no script.
- Pode criar uma [biblioteca](./05_11_libraries.md) Pine Script para incluir suas funções, o que as torna reutilizáveis em outros scripts sem precisar copiar o código. Existem requisitos distintos para funções de biblioteca. Eles são explicados em [Bibliotecas](./05_11_libraries.md).

Sejam de uma linha ou várias, as funções definidas pelo usuário possui as seguintes características:

- Não podem ser aninhadas. Todas as funções são definidas no escopo global do script.
- Não suportam recursão. __Não é permitido__ que uma função se chame a si mesma dentro do seu próprio código.
- O tipo do valor retornado por uma função é determinado automaticamente e depende do tipo dos argumentos usados em cada chamada de função específica.
- O valor retornado por uma função é o último valor no corpo da função.
- Cada instância da chamada de função em um script mantém seu próprio histórico independente.


# Funções de uma Única Linha

Funções simples muitas vezes podem ser criadas em uma _única linha_.

Esta é a definição formal de funções de uma linha:

```c
<function_declaration>
    <identifier>(<parameter_list>) => <return_value>

<parameter_list>
    {<parameter_definition>{, <parameter_definition>}}

<parameter_definition>
    [<identifier> = <default_value>]

<return_value>
    <statement> | <expression> | <tuple>
```

Por exemplo:

```c
f(x, y) => x + y
```

Após a declaração da função `f()`, é possível chamá-la utilizando diferentes tipos de argumentos:

```c
a = f(open, close)
b = f(2, 2)
c = f(open, 2)
```

No exemplo acima, o tipo da variável `a` é "_series_" porque ambos os argumentos são _series_. O tipo da variável `b` é "_integer_" (_inteiro_) porque os argumentos são ambos "_literal integers_" (_inteiros literais_). O tipo da variável `c` é _series_ porque a adição de uma _series_ e um _literal integers_ produz um resultado _series_.


# Funções de Múltiplas Linhas

Pine Script também suporta funções de múltiplas linhas com a seguinte sintaxe:

```c
<identifier>(<parameter_list>) =>
    <local_block>

<identifier>(<list of parameters>) =>
    <variable declaration>
    ...
    <variable declaration or expression>
```

Onde:

```c
<parameter_list>
    {<parameter_definition>{, <parameter_definition>}}

<parameter_definition>
    [<identifier> = <default_value>]
```

O corpo de uma função de múltiplas linhas consiste em várias instruções. Cada instrução é colocada em uma linha separada e deve ser precedida por 1 indentação (4 espaços ou 1 tab). A indentação antes da instrução indica que ela faz parte do corpo da função e não faz parte do escopo global do script. Após o código da função, a primeira instrução sem indentação indica que o corpo da função terminou.

Tanto uma expressão quanto uma variável declarada devem ser a última instrução do corpo da função. O resultado dessa expressão (ou variável) será o resultado da chamada da função.

Por exemplo:

```c
geom_average(x, y) =>
    a = x*x
    b = y*y
    math.sqrt(a + b)
```

A função `geom_average` tem dois argumentos e cria duas variáveis no corpo: `a` e `b`. A última instrução chama a função `math.sqrt` (uma extração da raiz quadrada). A chamada de `geom_average` retornará o valor da última expressão: `(math.sqrt(a + b))`.


# Escopos no Script

Variáveis declaradas fora do corpo de uma função ou de outros blocos locais pertencem ao escopo _global_. Funções declaradas pelo usuário e funções embutidas, assim como variáveis incorporadas, também pertencem ao escopo global.

Cada função tem seu próprio escopo _local_. Todas as variáveis declaradas dentro da função, assim como os argumentos da função, pertencem ao escopo daquela função, o que significa que é impossível referenciá-los de fora; por exemplo, do escopo global ou do escopo local de outra função.

Por outro lado, como é possível referenciar qualquer variável ou função declarada no escopo global a partir do escopo de uma função (exceto chamadas recursivas de auto-referência), pode-se dizer que o escopo local está inserido no escopo global.

No Pine Script, funções aninhadas não são permitidas, ou seja, não se pode declarar uma função dentro de outra. Todas as funções do usuário são declaradas no escopo global. Os escopos locais não podem se intersectar entre si.


# Funções que Retornam Múltiplos Resultados

Na maioria dos casos, uma função retorna apenas um resultado, mas é possível retornar uma lista de resultados (um resultado semelhante a uma _tupla_).

Por exemplo:

```c
fun(x, y) =>
    a = x+y
    b = x-y
    [a, b]
```

É necessária uma sintaxe especial para chamar tais funções:

```c
[res0, res1] = fun(open, close)
plot(res0)
plot(res1)
```


# Limitações

Funções definidas pelo usuário podem usar qualquer um dos recursos incorporados do Pine Script, exceto: [barcolor()](https://br.tradingview.com/pine-script-reference/v5/#fun_barcolor), [fill()](https://br.tradingview.com/pine-script-reference/v5/#fun_fill), [hline()](https://br.tradingview.com/pine-script-reference/v5/#fun_hline), [indicator()](https://br.tradingview.com/pine-script-reference/v5/#fun_indicator), [library()](https://br.tradingview.com/pine-script-reference/v5/#fun_library), [plot()](https://br.tradingview.com/pine-script-reference/v5/#fun_plot), [plotbar()](https://br.tradingview.com/pine-script-reference/v5/#fun_plotbar), [plotcandle()](https://br.tradingview.com/pine-script-reference/v5/#fun_plotcandle), [plotchar()](https://br.tradingview.com/pine-script-reference/v5/#fun_plotchar), [plotshape()](https://br.tradingview.com/pine-script-reference/v5/#fun_plotshape) e [strategy()](https://br.tradingview.com/pine-script-reference/v5/#fun_strategy).

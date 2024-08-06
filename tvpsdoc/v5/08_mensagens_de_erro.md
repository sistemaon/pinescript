
# Mensagens de Erro

## Declaração if é muito Longa

Esse erro ocorre quando o código indentado dentro de uma [declaração if](https://br.tradingview.com/pine-script-reference/v5/#kw_if) é muito grande para o compilador. Devido ao funcionamento do compilador, não será exibida uma mensagem informando exatamente quantas linhas de código excedem o limite. A única solução é dividir a [declaração if](https://br.tradingview.com/pine-script-reference/v5/#kw_if) em partes menores (funções ou menores [declarações if](https://br.tradingview.com/pine-script-reference/v5/#kw_if)). O exemplo abaixo mostra uma [declaração if](https://br.tradingview.com/pine-script-reference/v5/#kw_if) razoavelmente longa; teoricamente, isso resultaria em `line 4: if statement is too long`:

```c
//@version=5
indicator("My script")

var e = 0
if barstate.islast
    a = 1
    b = 2
    c = 3
    d = 4
    e := a + b + c + d

plot(e)
```

Para corrigir esse código, pode-se mover essas linhas para sua própria função:

```c
//@version=5
indicator("My script")

var e = 0
doSomeWork() =>
    a = 1
    b = 2
    c = 3
    d = 4

    result = a + b + c + d

if barstate.islast
    e := doSomeWork()

plot(e)
```

## Script Solicitando muitos Ativos

O número máximo de ativos em um script é limitado a 40. Se uma variável for declarada como uma chamada de função `request.security` e depois for usada como entrada para outras variáveis e cálculos, isso não resultará em várias chamadas de `request.security`. No entanto, se uma função que chama `request.security` for declarada, cada chamada a essa função contará como uma chamada de `request.security`.

Não é fácil determinar quantos ativos serão chamados apenas olhando o código-fonte. O exemplo a seguir tem exatamente 3 chamadas a `request.security` após a compilação:

```c
//@version=5
indicator("Securities count")
a = request.security(syminfo.tickerid, '42', close)  // (1) first unique security call
b = request.security(syminfo.tickerid, '42', close)  // same call as above, will not produce new security call after optimizations

plot(a)
plot(a + 2)
plot(b)

sym(p) =>  // no security call on this line
    request.security(syminfo.tickerid, p, close)
plot(sym('D'))  // (2) one indirect call to security
plot(sym('W'))  // (3) another indirect call to security

request.security(syminfo.tickerid, timeframe.period, open)  // result of this line is never used, and will be optimized out
```

## Script não pôde ser Traduzido de: null

```c
study($)
```

Normalmente, esse erro ocorre em scripts Pine da versão 1, e significa que o código está incorreto. Pine Script da versão 2 (e superior) é melhor em explicar erros desse tipo. Portanto, pode-se tentar mudar para a versão 2 adicionando um [atributo especial](./04_03_estrutura_do_script.md#versão) na primeira linha. Receberá `line 2: no viable alternative at character '$'`:

```c
// @version=2
study($)
```

## linha 2: nenhuma alternativa viável no caractere '$'

Esta mensagem de erro dá uma dica sobre o que está errado. `$` está no lugar da string com o título do script. Por exemplo:

```c
//@version=2
study("title")
```

## Entrada incompatível <…> esperando <???>

Igual a `no viable alternative`, mas é conhecido o que deveria estar naquele lugar. Exemplo:

```c
//@version=5
indicator("My Script")
    plot(1)
```

`line 3: mismatched input 'plot' expecting 'end of line without line continuation'`

Para corrigir isso, deve-se começar a linha com `plot` em uma nova linha sem indentação:

```c
//@version=5
indicator("My Script")
plot(1)
```

## Loop é muito Longo (> 500 ms)

O tempo de computação de loop em cada barra histórica e tick em tempo real é limitado para proteger os servidores de loops infinitos ou muito longos. Esse limite também faz com que indicadores que demoram muito para computar falhem rapidamente. Por exemplo, se houver 5000 barras, e o indicador levar 500 milissegundos para computar em cada uma das barras, resultaria em mais de 16 minutos de carregamento:

```c
//@version=5
indicator("Loop is too long", max_bars_back = 101)
s = 0
for i = 1 to 1e3  // to make it longer
    for j = 0 to 100
        if timestamp(2017, 02, 23, 00, 00) <= time[j] and time[j] < timestamp(2017, 02, 23, 23, 59)
            s := s + 1
plot(s)
```

Pode ser possível otimizar o algoritmo para superar esse erro. Nesse caso, o algoritmo pode ser otimizado assim:

```c
//@version=5
indicator("Loop is too long", max_bars_back = 101)
bar_back_at(t) =>
    i = 0
    step = 51
    for j = 1 to 100
        if i < 0
            i := 0
            break
        if step == 0
            break
        if time[i] >= t
            i := i + step
            i
        else
            i := i - step
            i
        step := step / 2
        step
    i

s = 0
for i = 1 to 1e3  // to make it longer
    s := s - bar_back_at(timestamp(2017, 02, 23, 23, 59)) +
         bar_back_at(timestamp(2017, 02, 23, 00, 00))
    s
plot(s)
```

## Script tem muitas Variáveis Locais

Esse erro aparece se o script for muito grande para ser compilado. Uma declaração `var=expression` cria uma variável local para `var`. Além disso, é importante notar que variáveis auxiliares podem ser criadas implicitamente durante o processo de compilação de um script. O limite se aplica a variáveis criadas tanto explicitamente quanto implicitamente. A limitação de 1000 variáveis é aplicada a cada função individualmente. Na verdade, o código colocado em um escopo _global_ de um script também é implicitamente envolvido na função principal e o limite de 1000 variáveis se torna aplicável a ele. Existem algumas refatorações que podem ser feitas para evitar esse problema:

```c
var1 = expr1
var2 = expr2
var3 = var1 + var2 
```

Pode ser convertido em:

```c
var3 = expr1 + expr2
```

<!-- ## Pine Script não Consegue Determinar o Comprimento de Referência de uma Série. Tente Usar max_bars_back na Função Indicator ou Strategy

O erro aparece em casos onde o Pine Script detecta incorretamente o comprimento máximo necessário das séries usadas em um script. Isso acontece quando o fluxo de execução de um script não permite que o Pine Script inspecione o uso de séries em ramificações de declarações condicionais (`if`, `iff` ou `?`), e o Pine Script não consegue detectar automaticamente quão longe no passado a série é referenciada. Aqui está um exemplo de um script que causa esse problema:

```c
//@version=5
indicator("Requires max_bars_back")
test = 0.0
if bar_index > 1000
    test := ta.roc(close, 20)
plot(test)
```

Para ajudar o Pine Script com a detecção, deve-se adicionar o parâmetro `max_bars_back` à função `indicator` ou `strategy` do script:

```c
//@version=5
indicator("Requires max_bars_back", max_bars_back = 20)
test = 0.0
if bar_index > 1000
    test := ta.roc(close, 20)
plot(test) 
```

Pode-se também resolver o problema retirando a expressão problemática da ramificação condicional, caso em que o parâmetro `max_bars_back` não é necessário:

```c
//@version=5
indicator("My Script")
test = 0.0
roc20 = ta.roc(close, 20)
if bar_index > 1000
    test := roc20
plot(test)
```

Em casos onde o problema é causado por uma __variável__ em vez de uma __função__ interna (`vwma` no exemplo), pode-se usar a função `max_bars_back` para definir explicitamente o comprimento de referência apenas para essa variável. Isso tem a vantagem de requerer menos recursos de runtime, mas implica que a variável problemática seja identificada, por exemplo, a variável `s` no exemplo a seguir:

```c
//@version=5
indicator("My Script")
f(off) =>
    t = 0.0
    s = close
    if bar_index > 242
        t := s[off]
    t
plot(f(301))
```

Essa situação pode ser resolvida usando a função `max_bars_back` para definir o comprimento de referência apenas da variável `s`, em vez de para todas as variáveis do script:

```c
//@version=5
indicator("My Script")
f(off) =>
    t = 0.0
    s = close
    max_bars_back(s, 301)
    if bar_index > 242
        t := s[off]
    t
plot(f(301))
```

Ao usar desenhos que se referem a barras anteriores através de `bar_index[n]` e `xloc = xloc.bar_index`, a série temporal recebida dessa barra será usada para posicionar os desenhos no eixo do tempo. Portanto, se for impossível determinar o tamanho correto do buffer, esse erro pode ocorrer. Para evitar isso, é necessário usar `max_bars_back(time, n)`. Esse comportamento é descrito com mais detalhes na seção sobre [desenhos](./05_12_lines_e_boxes.md#buffer-histórico-e-max_bars_back). -->

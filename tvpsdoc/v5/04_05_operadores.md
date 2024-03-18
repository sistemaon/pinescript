
# Operadores

Alguns operadores são utilizados para construir _expressões_ que retornam um resultado:

- Operadores aritméticos.
- Operadores de comparação.
- Operadores lógicos.
- O operador ternário [?:](https://br.tradingview.com/pine-script-reference/v5/#op_{question}{colon}).
- O operador de referência de histórico [[]](https://br.tradingview.com/pine-script-reference/v5/#op_[]).

Outros operadores são usados para atribuir valores a variáveis:

- `=` é usado para atribuir um valor a uma variável, __mas apenas quando a variável é declarada__ (na primeira vez que a usa).
- `:=` é usado para atribuir um valor a uma variável __previamente declarada__. Os seguintes operadores também podem ser usados dessa maneira: `+=`, `-=`, `*=`, `/=`, `%=`.

Explicado em [Tipagem do Sistema](./000_type_system.md), _qualificadores_ e _tipos_ desempenham um papel crítico na determinação do tipo de resultados que as expressões produzem. Por sua vez, tem um impacto sobre como e com quais funções é permitido usar esses resultados. As expressões sempre retornam um valor com o qualificador mais forte usado na expressão, por exemplo, se multiplicar um "input int" com um "series int", a expressão produzirá um resultado "series int", o qual não poderá utilizar como argumento para o _comprimento_ (`length`) em [ta.ema()](https://br.tradingview.com/pine-script-reference/v5/#fun_ta{dot}ema).

O script seguinte produzirá um erro de compilação:

```c
//@version=5
indicator("")
lenInput = input.int(14, "Length")
factor = year > 2020 ? 3 : 1
adjustedLength = lenInput * factor
ma = ta.ema(close, adjustedLength)  // Compilation error!
plot(ma)
```

O compilador avisará: Não é possível chamar 'ta.ema' com o argumento 'length'='adjustedLength' (_Cannot call ‘ta.ema’ with argument ‘length’=’adjustedLength’_). _Foi utilizado um argumento do tipo 'series int', mas é esperado um 'simple int';_. Isso ocorre porque `lenInput` é um "input int", mas `factor` é um "series int" (só pode ser determinado olhando para o valor de [year](https://br.tradingview.com/pine-script-reference/v5/#var_year) em cada barra). Portanto, a variável `adjustedLength` recebe um valor do tipo "series int". O problema é que a entrada no _Manual de Referência_ para [ta.ema()](https://br.tradingview.com/pine-script-reference/v5/#fun_ta{dot}ema) nos diz que seu parâmetro `length` requer um valor "simple", que é um qualificador mais _fraco/menos restritivo_ do que "series", então um valor do tipo "series int" não é permitido.

A solução para este dilema requer uma das duas opções:

- Usar outra função de média móvel que suporta um comprimento "series int", como [ta.sma()](https://br.tradingview.com/pine-script-reference/v5/#fun_ta{dot}sma).
- Não utilizar o cálculo que produza um valor "series int" para o comprimento.


# Operadores Aritméticos

Existem cinco operadores aritméticos no Pine Script:

|     |                                  |
| --- | -------------------------------: |
| `+` | Adição e concatenação de strings |
| `-` | Subtração                        |
| `*` | Multiplicação                    |
| `/` | Divisão                          |
| `%` | Módulo (sobra/resto da divisão)  |
|     |                                  |

Os operadores aritméticos acima são todos binários (significa que precisam de _dois operandos_ - ou valores - para funcionar, exemplo `1 + 2`). O `+` e o `-` também servem como operadores unários (significa que funcionam com um único operando, como `-1` ou `+1`).

Se ambos os operandos são números, mas pelo menos um deles é do tipo [float](https://br.tradingview.com/pine-script-reference/v5/#op_float), o resultado também será um [float](https://br.tradingview.com/pine-script-reference/v5/#op_float). Se ambos os operandos são do tipo [int](https://br.tradingview.com/pine-script-reference/v5/#op_int), o resultado também será um [int](https://br.tradingview.com/pine-script-reference/v5/#op_int). Se pelo menos um operando for [na](https://br.tradingview.com/pine-script-reference/v5/#var_na), o resultado também será [na](https://br.tradingview.com/pine-script-reference/v5/#var_na).

O operador `+` também serve como operador de concatenação para strings. `"EUR" + "USD"` resulta na [string](https://br.tradingview.com/pine-script-reference/v5/#op_string) `"EURUSD"`.

O operador `%` calcula o módulo arredondando para baixo o quociente para o menor valor possível. Aqui está um exemplo simples que ajuda a ilustrar como o módulo é calculado nos bastidores:

```c
//@version=5
indicator("Modulo function")
modulo(series int a, series int b) =>
    a - b * math.floor(nz(a/b))
plot(modulo(-1, 100))
```


# Operadores de Comparação

Existem seis operadores de comparação no Pine Script:

|      |                  |
| ---- | ---------------: |
| `<`  | Menor que        |
| `<=` | Menor ou igual a |
| `!=` | Não igual        |
| `==` | Igual            |
| `>`  | Maior que        |
| `>=` | Maior ou igual a |
|      |                  |

As operações de comparação são binárias. Se ambos os operandos tiverem um valor numérico, o resultado será do tipo [bool](https://br.tradingview.com/pine-script-reference/v5/#type_bool), isto é, `true`, `false` ou [na](https://br.tradingview.com/pine-script-reference/v5/#var_na).

Exemplos:

```c
1 > 2  // false
1 != 1 // false
close >= open  // Depends on values of `close` and `open`
```


# Operadores Lógicos

Existem três operadores lógicos no Pine Script:

|       |                  |
| ----- | ---------------: |
| `not` | Negação          |
| `and` | Conjunção lógica |
| `or`  | Disjunção lógica |
|       |                  |

O operador `not` é unário. Quando aplicado a um operando `true`, o resultado será `false`, e vice-versa.

Tabela verdade do operador `and`:

|       |       |         |
| a     | b     | a and b |
| ----- | ----: | ------: |
| true  | true  | true    |
| true  | false | false   |
| false | true  | false   |
| false | false | false   |
|       |       |         |

Tabela verdade do operador `or`:

|       |       |         |
| a     | b     | a or b  |
| ----- | ----: | ------: |
| true  | true  | true    |
| true  | false | true    |
| false | true  | true    |
| false | false | false   |
|       |       |         |

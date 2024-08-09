
# Identificadores

Identificadores são nomes usados para as variáveis e funções definidas pelo usuário:

- Devem começar com uma letra maiúscula (`A-Z`) ou minúscula (`a-z`), ou um sublinhado (`_`).
- Os próximos caracteres podem ser letras, sublinhados ou dígitos (`0-9`).
- São sensíveis a maiúsculas e minúsculas (_case-sensitive_).

Alguns exemplos:

```c
myVar
_myVar
my123Var
functionName
MAX_LEN
max_len
maxLen
3barsDown  // NOT VALID!
```

O [Guia de Estilo](./06_01_guia_de_estilo.md) do Pine Script recomenda o uso de __SNAKE_CASE__ em maiúsculas para _constantes_ e __camelCase__ para _outros identificadores_:

```c
GREEN_COLOR = #4CAF50
MAX_LOOKBACK = 100
int fastLength = 7
// Returns 1 if the argument is `true`, 0 if it is `false` or `na`.
zeroOne(boolValue) => boolValue ? 1 : 0
```

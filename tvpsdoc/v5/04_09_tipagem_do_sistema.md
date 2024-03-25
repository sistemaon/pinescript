
# Tipagem do Sistema

O sistema de tipos do Pine Script determina a compatibilidade dos valores de um script com várias funções e operações. Embora seja possível criar scripts simples sabendo nada sobre o sistema de tipagem, um entendimento razoável dele é necessário para alcançar qualquer grau de proficiência da linguagem, e um conhecimento aprofundado de suas sutilezas permitirá que aproveite todo o seu potencial.

O Pine Script utiliza [tipos](./04_09_tipagem_do_sistema.md#tipos) para classificar todos os valores, e utiliza [qualificadores](./04_09_tipagem_do_sistema.md#qualificadores) para determinar se os valores são constantes, estabelecidos na primeira iteração do script, ou dinâmicos ao longo da execução do script. Este sistema se aplica a todos os valores do Pine, incluindo os de literais, variáveis, expressões, retornos de função e argumentos de função.

A tipagem do Sistema está intimamente entrelaçado com o [modelo de execução](./04_01_modelo_de_execucao.md) do Pine e os conceitos de [séries temporais](./04_02_series_temporais.md). Compreender os três é essencial para aproveitar ao máximo o poder do Pine Script.

> ## Note
> Para simplificar, muitas vezes usamos "tipo" para nos referirmos a um "tipo qualificado".


# Qualificadores

Os _qualificadores_ do Pine Script identificam quando um valor é acessível na execução do script:

- Os valores qualificados como [const](./04_09_tipagem_do_sistema.md#const) são estabelecidos em tempo de compilação (ou seja, ao salvar o script no Editor Pine ou adicioná-lo ao gráfico).
- Os valores qualificados como [input](./04_09_tipagem_do_sistema.md#input) estão disponíveis no momento da entrada (ou seja, ao alterar valores na aba "Configurações/Entradas" ("_Settings/Inputs_") do script).
- Os valores qualificados como [simple](./04_09_tipagem_do_sistema.md#simple) são estabelecidos na barra zero (o primeiro candle da execução do script).
- Os valores qualificados como [series](./04_09_tipagem_do_sistema.md#series) podem mudar ao longo da execução do script.

O Pine Script baseia a predominância dos qualificadores de tipo na seguinte hierarquia: __const < input < simple < series__, onde "const" é o qualificador _mais fraco_ (menos restritivo) e "series" é o _mais forte_ (mais restritivo). A hierarquia de qualificadores se traduz na seguinte regra: sempre que uma variável, função ou operação é compatível com um qualificador de tipo específico, valores com qualificadores mais _fracos_ também são permitidos.

Os scripts sempre qualificam os valores retornados de suas expressões com base no qualificador dominante em seus cálculos. Por exemplo, avaliar uma expressão que envolve valores "const" e "series" retornará um valor qualificado como "series". Além disso, os scripts não podem alterar o qualificador de um valor para um que esteja abaixo na hierarquia. Se um valor adquire um qualificador mais forte (por exemplo, um valor inicialmente inferido como "simple" se torna "series" mais tarde na execução do script), esse estado é irreversível.

Note que apenas os valores qualificados como "series" podem mudar ao longo da execução de um script, na qual inclui aos valores provenientes de várias funções integradas, como [close](https://br.tradingview.com/pine-script-reference/v5/#var_close) e [volume](https://br.tradingview.com/pine-script-reference/v5/#var_volume), e também resultados de quaisquer operações que envolvam valores "series". Valores qualificados como "const", "input" ou "simple" são consistentes ao longo da execução de um script.

## `const`

Os valores qualificados como "const" são estabelecidos durante o _tempo de compilação_, antes que o script inicie sua execução. A compilação ocorre inicialmente ao salvar um script no Editor Pine, sem a necessidade de executá-lo em um gráfico. Valores com o qualificador "const" nunca mudam entre as iterações do script, nem mesmo na barra inicial de sua execução.

Os scripts podem qualificar valores como "const" usando um valor _literal_ ou calculando valores a partir de expressões que usam apenas valores literais ou outras variáveis qualificadas como "const".

Exemplos de valores literais:

- _literal int_: `1`, `-1`, `42` .
- _literal float_: `1.`, `1.0`, `3.14`, `6.02E-23`, `3e8` .
- _literal bool_: `true`, `false` .
- _literal color_: `#FF55C6`, `#FF55C6ff` .
- _literal string_: `"A text literal"`, `"Embedded single quotes 'text'"`, `'Embedded double quotes "text"'` .

Os usuários podem definir explicitamente variáveis e parâmetros que aceitam apenas valores "const" incluindo a palavra-chave `const` na declaração.

O [guia de estilo](./000_style_guide.md) recomenda usar __SNAKE_CASE__ em maiúsculas para nomear variáveis "const" para melhor legibilidade. Embora não seja um requisito, também é possível usar a palavra-chave [var](https://br.tradingview.com/pine-script-reference/v5/#kw_var) ao declarar variáveis "const" para que o script as inicialize apenas no _primeiro candle_ do conjunto de dados. Consulte [esta seção](./04_06_declaracoes_de_variavel.md#var-var) do _Manual do Usuário_ para obter mais informações.

Abaixo está um exemplo que utiliza valores "const" dentro das funções [indicator()](https://br.tradingview.com/pine-script-reference/v5/#fun_indicator) e [plot()](https://br.tradingview.com/pine-script-reference/v5/#fun_plot), que ambas requerem um valor do tipo qualificado "const string" como seu argumento `title`:

```c
//@version=5

// The following global variables are all of the "const string" qualified type:

//@variable The title of the indicator.
INDICATOR_TITLE = "const demo"
//@variable The title of the first plot.
var PLOT1_TITLE = "High"
//@variable The title of the second plot.
const string PLOT2_TITLE = "Low"
//@variable The title of the third plot.
PLOT3_TITLE = "Midpoint between " + PLOT1_TITLE + " and " + PLOT2_TITLE

indicator(INDICATOR_TITLE, overlay = true)

plot(high, PLOT1_TITLE)
plot(low, PLOT2_TITLE)
plot(hl2, PLOT3_TITLE)
```

O exemplo a seguir gera um erro de compilação, pois utiliza [syminfo.ticker](https://br.tradingview.com/pine-script-reference/v5/#var_syminfo.ticker), que retorna um valor "simple" porque depende de informações do gráfico que só são acessíveis após o início da execução do script:

```c
//@version=5

//@variable The title in the `indicator()` call.
var NAME = "My indicator for " + syminfo.ticker

indicator(NAME, "", true) // Causes an error because `NAME` is qualified as a "simple string".
plot(close)
```

## `input`








## `simple`

## `series`



# Tipos


# Entrada



# Inputs (_Entradas_)

Os _Inputs_ ("_Entradas_") permitem que scripts recebam valores que os usuários podem alterar. Usá-las para valores-chave torna os scripts mais adaptáveis às preferências do usuário.

O script a seguir plota uma [média móvel simples (SMA)](https://br.tradingview.com/support/solutions/43000502589) de 20 períodos usando `ta.sma(close, 20)`. Embora seja simples de escrever, não é muito flexível, pois essa MA específica é tudo o que ele sempre plotará:

```c
//@version=5
indicator("MA", "", true)
plot(ta.sma(close, 20))
```

Se, em vez disso, o script for escrito desta maneira, ele se torna muito mais flexível, pois os usuários poderão selecionar a "_source_" ("_fonte_") e a "_length_" ("_comprimento_") que desejam usar para o cálculo da MA:

```c
//@version=5
indicator("MA", "", true)
sourceInput = input(close, "Source")
lengthInput = input(20, "Length")
plot(ta.sma(sourceInput, lengthInput))
```

Entradas só podem ser acessadas quando um script está sendo executado no gráfico. Os usuários do script acessam as entradas através da caixa de diálogo "_Settings_" ("_Configurações_") do script, que pode ser alcançada de uma das seguintes maneiras:

- Com um clique duplo no nome de um indicador no gráfico.
- Clicando com o botão direito do mouse no nome do script e escolhendo o item "_Settings_" ("_Configurações_") no menu suspenso.
- Escolhendo o item "_Settings_" ("_Configurações_") no ícone do menu "_More_" ("_Mais_") (três pontos) que aparece ao passar o cursor sobre o nome do indicador no gráfico.
- Com um clique duplo no nome do indicador na "_Data Window_" ("_Janela de Dados_") (quarto ícone abaixo à direita do gráfico).

A caixa de diálogo "_Settings_" ("_Configurações_") sempre contém as abas "_Style_" ("_Estilo_") e "_Visibility_" ("_Visibilidade_"), que permitem aos usuários especificar suas preferências sobre os visuais do script e os timeframes do gráfico onde ele deve ser visível.

Quando um script contém chamadas para funções `input.*()`, uma aba "_Inputs_" ("_Entradas_") aparece na caixa de diálogo "_Settings_" ("_Configurações_").

![Inputs](./imgs/Inputs-Introduction-1.png)

No fluxo de execução de um script, as entradas são processadas quando o script já está em um gráfico e um usuário altera valores na aba "_Inputs_" ("_Entradas_"). As mudanças desencadeiam uma reexecução do script em todas as barras do gráfico, então quando um usuário altera um valor de entrada, seu script recalcula usando esse novo valor.


# Funções de Input

As seguintes funções de _input_ (_entrada_) estão disponíveis:

- [input()](https://br.tradingview.com/pine-script-reference/v5/#fun_input)
- [input.int()](https://br.tradingview.com/pine-script-reference/v5/#fun_input{dot}int)
- [input.float()](https://br.tradingview.com/pine-script-reference/v5/#fun_input{dot}float)
- [input.bool()](https://br.tradingview.com/pine-script-reference/v5/#fun_input{dot}bool)
- [input.color()](https://br.tradingview.com/pine-script-reference/v5/#fun_input{dot}color)
- [input.string()](https://br.tradingview.com/pine-script-reference/v5/#fun_input{dot}string)
- [input.timeframe()](https://br.tradingview.com/pine-script-reference/v5/#fun_input{dot}timeframe)
- [input.symbol()](https://br.tradingview.com/pine-script-reference/v5/#fun_input{dot}symbol)
- [input.price()](https://br.tradingview.com/pine-script-reference/v5/#fun_input{dot}price)
- [input.source()](https://br.tradingview.com/pine-script-reference/v5/#fun_input{dot}source)
- [input.session()](https://br.tradingview.com/pine-script-reference/v5/#fun_input{dot}session)
- [input.time()](https://br.tradingview.com/pine-script-reference/v5/#fun_input{dot}time)

Um _widget_ de _entrada_ específico é criado na aba "_Inputs_" ("_Entradas_") para aceitar cada tipo de entrada. A menos que especificado de outra forma na chamada `input.*()`, cada entrada aparece em uma nova linha da aba "_Inputs_" ("_Entradas_"), na ordem em que as chamadas `input.*()` aparecem no script.

É recomendado pelo [Guia de Estilo](./000_style_guide.md) colocar as chamadas `input.*()` no início do script.

As definições das funções de entrada normalmente contêm muitos parâmetros, permitindo controlar o valor padrão das entradas, seus limites e sua organização na aba "_Inputs_" ("_Entradas_").

Uma chamada `input.*()` sendo apenas outra chamada de função no Pine Script, seu resultado pode ser combinado com operadores [aritméticos](./04_05_operadores.md#operadores-aritméticos), [comparação](./04_05_operadores.md#operadores-de-comparação), [lógicos](./04_05_operadores.md#operadores-lógicos) ou [ternários](./04_05_operadores.md#operador-ternário-) para formar uma expressão a ser atribuída à variável. Aqui, o resultado da chamada [input.string()](https://br.tradingview.com/pine-script-reference/v5/#fun_input{dot}string) é comparado à string "`On`". O resultado da expressão é então armazenado na variável `plotDisplayInput`. Como essa variável contém um valor `true` ou `false`, ela é do tipo "input bool":

```c
//@version=5
indicator("Input in an expression`", "", true)
bool plotDisplayInput = input.string("On", "Plot Display", options = ["On", "Off"]) == "On"
plot(plotDisplayInput ? close : na)
```

Todos os valores retornados pelas funções `input.*()`, exceto os de "source", são valores qualificados como "input". Veja a seção sobre [qualificadores de tipo](./04_09_tipagem_do_sistema.md#qualificadores) para mais informações.










# Input da Fonte

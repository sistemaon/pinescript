
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

# Input da Fonte

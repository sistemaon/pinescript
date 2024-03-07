
# O Editor Pine

O Editor Pine é a área de trabalho dos seus scripts.
Embora possa usar qualquer editor de texto que desejar para escrever seus scripts em Pine, usar nosso Editor possui muitas vantagens:

* Destaca seu código seguindo a sintaxe do Pine Script.
* Exibe lembretes de sintaxe para funções integradas e de bibliotecas ao passar o mouse sobre elas.
* Fornece rápido acesso ao Manual de Referência do Pine Script v5 através de um popup ao pressionar `ctrl` + `click` / `cmd` + `click` nas palavras-chave do Pine Script.
* Disponibiliza um recurso de auto-completar que pode ser ativado com `ctrl` + `espaço` / `cmd` + `espaço`.
* Execução imediata ao salvar uma nova versão de um script já carregado no gráfico, tornando o ciclo de escrever/compilar/executar ágil.
* Embora não seja tão repleto de recursos quanto os principais editores disponíveis, o Editor Pine oferece funcionalidades-chave, como busca e substituição, múltiplos cursores (edição simultânea) e versionamento.

Para abrir o Editor, clique na aba "Editor Pine" (_Pine Editor_) na parte inferior do gráfico no TradingView. E isso abrirá o painel do Editor.


# Primeira Versão

Criando o primeiro script funcional em Pine, uma implementação do indicador [_MACD_](https://www.tradingview.com/support/solutions/43000502344-macd-moving-average-convergence-divergence) em Pine Script:

```c
//@version=5
indicator("MACD #1")
fast = 12
slow = 26
fastMA = ta.ema(close, fast)
slowMA = ta.ema(close, slow)
macd = fastMA - slowMA
signal = ta.ema(macd, 9)
plot(macd, color = color.blue)
plot(signal, color = color.orange)
```

* Através do menu suspenso em "Abrir" (_Open_) localizado no canto superior direito do Editor selecione "Novo indicador" (_New indicator_).
* Em seguida, copie o script de exemplo acima.
* Selecione todo o código já no editor e substitua-o pelo script de exemplo copiado.
* Clique em "Salvar" (_Save_) e escolha um nome para o seu script. O script agora está salvo na nuvem do TradingView, mas sob o nome da sua conta. Ninguém além de você pode usá-lo.
* Ao clicar em "Adicionar ao gráfico" (_Add to chart_) na barra de menu do Editor, o indicador MACD aparecerá em um Painel (_Pane_) separado abaixo do seu gráfico.

Seu primeiro script Pine está sendo executado no gráfico, que deve se assemelhar a isto:

![Primeira versão](./imgs/FirstIndicator-Version1.png)

Vamos analisar o código do nosso script, linha por linha:

__Linha 1: `//@version=5`__
- Esta é uma [anotação do compilador](000_compiler_annotation.md) informando ao compilador que o script usará a versão 5 do Pine Script.

__Linha 2: `indicator("MACD #1")`__
- Define o nome do script que aparecerá no gráfico como "_MACD_".

__Linha 3: `fast = 12`__
- Define a variável do tipo _integer_ com nome de `fast` que será o comprimento da _EMA rápida_.

__Linha 4: `slow = 26`__
- Define a variável do tipo _integer_ com nome de `slow` que será o comprimento da _EMA lenta_.

__Linha 5: `fastMA = ta.ema(close, fast)`__
- Define a variável `fastMA`, contendo o resultado do cálculo da _EMA (Média Móvel Exponencial)_ com um comprimento igual a _fast (12)_, na série de fechamento, ou seja, o preço de fechamento das barras (_candlestick_).
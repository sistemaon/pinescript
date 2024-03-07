
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

Criando o primeiro script funcional em Pine, uma implementação do indicador [(_MACD_)](https://www.tradingview.com/support/solutions/43000502344-macd-moving-average-convergence-divergence) em Pine Script:

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
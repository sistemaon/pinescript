
# Estrutura do Script

Um script Pine segue a seguinte estrutura geral:

```c
<version>
<declaration_statement>
<code>
```


# Versão

Uma [anotação do compilador](./04_03_estrutura_do_script.md#anotações-do-compilador) no seguinte formato informa ao compilador em qual das versões do Pine Script o script está programado:

```c
//@version=5
```

- O número do versionamento é entre 1 e 5.
- A anotação do compilador não é obrigatória. Quando omitida, assume-se a versão 1. É extremamente recomendado sempre utilizar a versão mais recente da linguagem.
- Embora seja sintaticamente correto colocar a anotação do compilador em qualquer lugar do script, é muito mais útil para os leitores quando ela aparece no topo do script.

Alterações significativas na versão atual do Pine Script são documentadas nas [Notas de Lançamento](https://www.tradingview.com/pine-script-docs/en/v5/Release_notes.html#pagereleasenotes).


# Declaração de Instrução

Todos os scripts Pine devem conter uma declaração de instrução, que é uma chamada a uma dessas funções:

- Indicador > [indicator()](https://br.tradingview.com/pine-script-reference/v5/#fun_indicator)
- Estratégia > [strategy()](https://br.tradingview.com/pine-script-reference/v5/#fun_strategy)
- Biblioteca > [library()](https://br.tradingview.com/pine-script-reference/v5/#fun_library)

A _declaração de instrução_:

- Identifica o tipo do script, que por sua vez dita qual conteúdo é permitido nele e como pode ser usado e executado.
- Define propriedades-chave do script, como _nome_ próprio, onde aparecerá ao ser adicionado no gráfico, a precisão e formato dos valores que exibe, e certos valores que governam o próprio comportamento em tempo de execução, como o número máximo de objetos de desenho que serão exibidos no gráfico. Com estratégias, as propriedades incluem parâmetros que controlam o _backtesting_, como capital inicial (_initial capital_), comissão (_commission_), variação de preço (_slippage_), etc.

Cada tipo de script possui requisitos distintos:

- Indicadores devem conter pelo menos uma chamada de função que produza a saída no gráfico (por exemplo, [plot()](https://br.tradingview.com/pine-script-reference/v5/#fun_plot), [plotshape()](https://br.tradingview.com/pine-script-reference/v5/#fun_plotshape), [barcolor()](https://br.tradingview.com/pine-script-reference/v5/#fun_barcolor), [line.new()](https://br.tradingview.com/pine-script-reference/v5/#fun_line{dot}new), etc.).
- Estratégias devem conter pelo menos uma chamada de `strategy.*()`, por exemplo, [strategy.entry()](https://br.tradingview.com/pine-script-reference/v5/#fun_strategy{dot}entry).
- Bibliotecas devem conter pelo menos uma [função](./000_library_functions.md) exportada ou um [tipo definido pelo usuário](./000_user_defined_types_objects.md).


# Anotações do Compilador

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


# Anotações do Compilador
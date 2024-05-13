
# Estados da Barra

Um conjunto de variáveis incorporadas no _namespace_ `barstate` permite que o script detecte diferentes propriedades da barra na qual o script está sendo executado.

Esses estados podem ser usados para restringir a execução ou a lógica do código a barras específicas.

Alguns recursos incorporados retornam informações sobre a sessão de negociação à qual a barra atual pertence. Eles são explicados na seção de [estados da Sessão](./05_17_sessoes.md#estados-da-sessão).


# Variáveis Embutidas do Estado da Barra

Embora indicadores e bibliotecas sejam executados em todas as atualizações de preço ou volume em tempo real, estratégias que não utilizam `calc_on_every_tick` não serão executadas; elas serão executadas apenas quando a barra em tempo real fechar. Isso afetará a detecção de estados da barra nesse tipo de script. Em mercados abertos, por exemplo, este código não exibirá um plano de fundo até que a barra em tempo real feche, pois é quando a estratégia é executada:

```c
//@version=5
strategy("S")
bgcolor(barstate.islast ? color.silver : na)
```

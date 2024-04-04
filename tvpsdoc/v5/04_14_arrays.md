
# Arrays

> __Observação__\
> Esta seção contém material avançado. Recomenda-se que programadores iniciantes em Pine Script se familiarizem com outras funcionalidades do Pine Script mais acessíveis antes de explorarem este conteúdo.

Arrays do Pine Script são coleções unidimensionais capazes de armazenar múltiplas referências de valores. Podem ser considerados uma forma mais eficiente de lidar com situações nas quais seria necessário declarar explicitamente um conjunto de variáveis similares (por exemplo, `price00`, `price01`, `price02`, ...).

Todos os elementos dentro de um array devem ser do mesmo tipo, que pode ser um tipo [integrado](./04_09_tipagem_do_sistema.md#qualificadores) ou [definido pelo usuário](./04_09_tipagem_do_sistema.md#tipos-definidos-pelo-usuário), sempre qualificado como "series". Scripts referenciam arrays usando um ID de array semelhante aos IDs de linhas, labels e outros tipos especiais. O Pine Script não utiliza um operador de indexação para referenciar elementos individuais de array. Em vez disso, funções como [array.get()](https://br.tradingview.com/pine-script-reference/v5/#fun_array{dot}get) e [array.set()](https://br.tradingview.com/pine-script-reference/v5/#fun_array{dot}set) são utilizadas para ler e escrever os valores dos elementos do array. Valores de array podem ser utilizados em expressões e funções que aceitam valores do tipo "series".

Scripts referenciam os elementos de um array usando um _index_, que começa em _0_ e vai até o _número de elementos no array menos um_. Arrays no Pine Script podem ter um tamanho dinâmico que varia ao longo das barras, já que é possível alterar o número de elementos em um array a cada iteração de um script. Scripts podem conter múltiplas instâncias de arrays. O tamanho dos arrays é limitado a 100.000 elementos.

> __Observação__\
> Utiliza-se o termo "início de um array" para designar o index 0, e "final de um array" para designar o elemento do array com o maior valor de index. Também se ampliará o significado de array para incluir IDs de array, em prol da brevidade.



# Matrices (_Matrix_)

> __Observação__\
> Esta seção contém material avançado. Recomenda-se que programadores iniciantes em Pine Script se familiarizem com outras funcionalidades do Pine Script mais acessíveis antes de explorarem este conteúdo.

_Matrices_ no Pine Script são coleções que armazenam referências de valores em um formato retangular. Elas são essencialmente equivalentes a objetos de [array](https://br.tradingview.com/pine-script-reference/v5/#type_array) bidimensionais com funções e métodos para inspeção, modificação e cálculos especializados. Assim como os [arrays](./04_14_arrays.md), todos os elementos de uma _matrix_ devem ser do mesmo [type](./04_09_tipagem_do_sistema.md#tipos) (_tipo_), que pode ser um [tipo integrado](./04_09_tipagem_do_sistema.md#qualificadores) ou [definido pelo usuário](./04_09_tipagem_do_sistema.md#tipos-definidos-pelo-usuário).

_Matrices_ referenciam seus elementos usando dois _indices_: um _index_ para suas linhas e outro para suas colunas. Cada _index_ começa em _0_ e se estende até o _número de linhas/colunas na matrix_ menos um_. _Matrices_ no Pine podem ter números dinâmicos de linhas e colunas que variam ao longo das barras. O [número total de elementos](https://br.tradingview.com/pine-script-reference/v5/#fun_matrix.elements_count) dentro de uma _matrix_ é o produto do número de [rows](https://br.tradingview.com/pine-script-reference/v5/#fun_matrix.rows) (_linhas_) e [columns](https://br.tradingview.com/pine-script-reference/v5/#fun_matrix.columns) (_colunas_) (por exemplo, uma _matrix_ 5x5 tem um total de 25). Como os [arrays](./04_14_arrays.md), o número total de elementos em uma _matrix_ não pode exceder _100.000_.

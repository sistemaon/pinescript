
# Maps (_Mapas_)

> __Observação__\
> Esta seção contém material avançado. Recomenda-se que programadores iniciantes em Pine Script se familiarizem com outras funcionalidades do Pine Script mais acessíveis antes de explorarem este conteúdo.

Os mapas do Pine Script são coleções que armazenam elementos em _key-value pairs_ (_pares de chave-valor_). Eles permitem que os scripts armazenem e associem múltiplos valores referenciados associados a identificadores únicos (chaves).

Ao contrário de [arrays](./04_14_arrays.md) e [_matrices_](./04_15_matrices.md), os mapas são considerados coleções _não ordenadas_. Scripts acessam rapidamente os valores de um mapa referenciando as chaves dos pares chave-valor inseridos neles, em vez de percorrer um _index_ interno.

As chaves de um mapa podem ser de qualquer _tipo fundamental_, e seus valores podem ser de qualquer tipo incorporado ou [definido pelo usuário](./04_09_tipagem_do_sistema.md#tipos-definidos-pelo-usuário). Mapas não podem usar diretamente outras _coleções_ ([mapas](./04_16_mapas.md), [arrays](./04_14_arrays.md) ou [_matrices_](./04_15_matrices.md)) como valores, mas podem conter instâncias de [UDT](./04_09_tipagem_do_sistema.md#tipos-definidos-pelo-usuário) que contêm essas estruturas de dados em seus campos. Veja [esta seção](./04_16_mapas.md#mapas-de-outras-coleções) para mais informações.

Assim como outras coleções, os mapas podem conter até 100.000 elementos no total. Uma vez que cada par chave-valor em um mapa consiste em dois elementos (uma chave _única_ e seu _valor_ associado), o número máximo de pares chave-valor que um mapa pode conter é 50.000.


# Declarando Mapa

Pine Script utiliza a seguinte sintaxe para declarar mapas:

```c
[var/varip ][map<keyType, valueType> ]<identifier> = <expression>
```

Onde `<keyType, valueType>` é o [modelo de tipo](./04_09_tipagem_do_sistema.md#templates-de-tipo) do mapa que declara os tipos de chaves e valores que ele conterá, e a `<expression>` retorna uma instância de mapa ou `na`.

Ao declarar uma variável de mapa atribuída a `na`, os usuários devem incluir a palavra-chave [map](https://br.tradingview.com/pine-script-reference/v5/#type_map) seguida de um [modelo de tipo](./04_09_tipagem_do_sistema.md#templates-de-tipo) para informar ao compilador que a variável pode aceitar mapas com chaves do tipo `keyType` e valores do tipo `valueType`.

Por exemplo, esta linha de código declara uma nova variável `myMap` que pode aceitar instâncias de mapa contendo pares de chaves do tipo [string](https://br.tradingview.com/pine-script-reference/v5/#type_string) e valores do tipo [float](https://br.tradingview.com/pine-script-reference/v5/#type_float):

```c
map<string, float> myMap = na
```








# Mapas de Outras Coleções

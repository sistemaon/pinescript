
# Funções Definida pelo Usuário

Funções definidas pelo usuário são funções que se cria, ao contrário das funções incorporadas no Pine Script. São úteis para definir cálculos que precisa fazer repetidamente ou que deseja isolar da seção principal de cálculos do script. Pense em funções definidas pelo usuário como uma forma de estender as capacidades do Pine Script quando nenhuma função integrada fizer o que precisa ser feito.

Pode-se escrever as funções em uma das duas maneiras:

- Em uma única linha, quando forem simples.
- Em várias linhas.

As funções podem ser localizadas em dois lugares:

- Se uma função é usada apenas em um script, pode incluí-la no script onde é usada. Consulte o [Guia de Estilo](./000_style_guide.md) para recomendações sobre onde colocar funções no script.
- Pode criar uma [biblioteca](./000_library.md) Pine Script para incluir suas funções, o que as torna reutilizáveis em outros scripts sem precisar copiar o código. Existem requisitos distintos para funções de biblioteca. Eles são explicados em [Bibliotecas](./000_library.md).

Sejam de uma linha ou várias, as funções definidas pelo usuário possui as seguintes características:

- Não podem ser aninhadas. Todas as funções são definidas no escopo global do script.
- Não suportam recursão. __Não é permitido__ que uma função se chame a si mesma dentro do seu próprio código.
- O tipo do valor retornado por uma função é determinado automaticamente e depende do tipo dos argumentos usados em cada chamada de função específica.
- O valor retornado por uma função é o último valor no corpo da função.
- Cada instância da chamada de função em um script mantém seu próprio histórico independente.
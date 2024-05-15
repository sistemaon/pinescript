
# Cores

Os visuais dos scripts podem desempenhar um papel crítico na usabilidade dos indicadores escritos em Pine Script. Plots e desenhos bem projetados tornam os indicadores mais fáceis de usar e entender. Bons designs visuais estabelecem uma hierarquia visual que permite que as informações mais importantes se destaquem e as menos importantes não atrapalhem.

Usar cores no Pine pode ser tão simples quanto necessário ou tão complexo quanto o conceito exigir. As __4.294.967.296__ combinações possíveis de cor e transparência disponíveis no Pine Script podem ser aplicadas a:

- Qualquer elemento que possa ser plotado ou desenhado no espaço visual de um indicador, sejam linhas, preenchimentos, texto ou velas.
- O fundo do espaço visual de um script, seja o script executado em seu próprio painel ou no modo de sobreposição no gráfico.
- A cor das barras ou do corpo das velas que aparecem em um gráfico.

Um script só pode colorir os elementos que coloca em seu próprio espaço visual. A única exceção a essa regra é que um indicador de painel pode colorir barras ou velas do gráfico.

O Pine Script possui cores incorporadas como [color.green](https://br.tradingview.com/pine-script-reference/v5/#const_color{dot}green), bem como funções como [color.rgb()](https://br.tradingview.com/pine-script-reference/v5/#fun_color{dot}rgb) que permitem gerar dinamicamente qualquer cor no espaço de cores RGBA.

## Transparência

Cada cor no Pine Script é definida por quatro valores:

- Seus componentes vermelho, verde e azul (0-255), seguindo o [modelo de cores RGB](https://en.wikipedia.org/wiki/RGB_color_spaces).
- Sua _transparency_ (_transparência_) (0-100), frequentemente referida como o canal Alpha fora do Pine, conforme definido no [modelo de cores RGBA](https://en.wikipedia.org/wiki/RGB_color_spaces). Mesmo que a transparência seja expressa na faixa de 0-100, seu valor pode ser um "float" quando usado em funções, o que dá acesso aos 256 valores subjacentes do canal alpha.

A transparência de uma cor define quão opaca ela é: zero é totalmente opaca, 100 torna a _cor—qualquer_ que _seja—invisível_. Modular a transparência pode ser crucial em visuais de cores mais complexos ou ao usar fundos, para controlar quais cores dominam as outras e como elas se misturam quando sobrepostas.

## Z-index


# Constante de Cores

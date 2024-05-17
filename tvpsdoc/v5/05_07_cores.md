
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

Ao colocar elementos no espaço visual de um script, eles têm profundidade relativa no _eixo z_; alguns aparecerão sobre outros. O _z-index_ é um valor que representa a posição dos elementos no _eixo z_. Elementos com o _z-index_ mais alto aparecem no topo.

Elementos desenhados no Pine Script são divididos em grupos. Cada grupo tem sua própria posição no espaço z, e __dentro do mesmo grupo__, elementos criados por último na lógica do script aparecerão sobre outros elementos do mesmo grupo. Um elemento de um grupo não pode ser colocado fora da região do espaço z atribuída ao seu grupo, então um plot nunca pode aparecer sobre uma tabela, por exemplo, porque as tabelas têm o _z-index_ mais alto.

Esta lista contém os grupos de elementos visuais, ordenados por _z-index_ crescente, para que as cores de fundo estejam sempre na parte inferior do espaço z, e as tabelas apareçam sempre no topo de todos os outros elementos:

- [Background Colors (_Cores de fundo_)](https://br.tradingview.com/pine-script-reference/v5/#fun_bgcolor)
- [Fills (_Preenchimentos_)](https://br.tradingview.com/pine-script-reference/v5/#fun_fill)
- [Plots](https://br.tradingview.com/pine-script-reference/v5/#fun_plot)
- [Hlines](https://br.tradingview.com/pine-script-reference/v5/#fun_hline)
- [LineFills](https://br.tradingview.com/pine-script-reference/v5/#fun_linefill)
- [Lines (_Linhas_)](https://br.tradingview.com/pine-script-reference/v5/#fun_line)
- [Boxes (_Caixas_)](https://br.tradingview.com/pine-script-reference/v5/#fun_box)
- [Labels (_Rótulos_)](https://br.tradingview.com/pine-script-reference/v5/#fun_label)
- [Tables (_Tabelas_)](https://br.tradingview.com/pine-script-reference/v5/#fun_table)

Observe que, usando `explicit_plot_zorder = true` em [indicator()](https://br.tradingview.com/pine-script-reference/v5/#fun_indicator) ou [strategy()](https://br.tradingview.com/pine-script-reference/v5/#fun_strategy), é possível controlar o _z-index_ relativo dos visuais de `plot*()`, [hline()](https://br.tradingview.com/pine-script-reference/v5/#fun_hline) e [fill()](https://br.tradingview.com/pine-script-reference/v5/#fun_fill) usando sua ordem sequencial no script.


# Constante de Cores

Existem 17 cores incorporadas no Pine Script. Esta tabela lista seus nomes, equivalentes hexadecimais e valores RGB como argumentos para [color.rgb()](https://br.tradingview.com/pine-script-reference/v5/#fun_color{dot}rgb):

| __Nome__       | __Hex__  | __Valores RGB__           |
| -------------  | -------  | -----------------------:  |
| color.aqua     | #00BCD4  | color.rgb(0, 188, 212)    |
| color.black    | #363A45  | color.rgb(54, 58, 69)     |
| color.blue     | #2196F3  | color.rgb(33, 150, 243)   |
| color.fuchsia  | #E040FB  | color.rgb(224, 64, 251)   |
| color.gray     | #787B86  | color.rgb(120, 123, 134)  |
| color.green    | #4CAF50  | color.rgb(76, 175, 80)    |
| color.lime     | #00E676  | color.rgb(0, 230, 118)    |
| color.maroon   | #880E4F  | color.rgb(136, 14, 79)    |
| color.navy     | #311B92  | color.rgb(49, 27, 146)    |
| color.olive    | #808000  | color.rgb(128, 128, 0)    |
| color.orange   | #FF9800  | color.rgb(255, 152, 0)    |
| color.purple   | #9C27B0  | color.rgb(156, 39, 176)   |
| color.red      | #FF5252  | color.rgb(255, 82, 82)    |
| color.silver   | #B2B5BE  | color.rgb(178, 181, 190)  |
| color.teal     | #00897B  | color.rgb(0, 137, 123)    |
| color.white    | #FFFFFF  | color.rgb(255, 255, 255)  |
| color.yellow   | #FFEB3B  | color.rgb(255, 235, 59)   |

No script a seguir, todos os plots usam a mesma cor [color.olive](https://br.tradingview.com/pine-script-reference/v5/#const_color{dot}olive) com uma transparência de 40, mas expressa de maneiras diferentes. Todos os cinco métodos são funcionalmente equivalentes:

![Constante de Cores](./imgs/Colors-UsingColors-1.png)

```c
//@version=5
indicator("", "", true)
// ————  Transparency (#99) is included in the hex value.
plot(ta.sma(close, 10), "10", #80800099)
// ————  Transparency is included in the color-generating function's arguments.
plot(ta.sma(close, 30), "30", color.new(color.olive, 40))
plot(ta.sma(close, 50), "50", color.rgb(128, 128, 0, 40))
      // ————  Use `transp` parameter (deprecated and advised against)
plot(ta.sma(close, 70), "70", color.olive, transp = 40)
plot(ta.sma(close, 90), "90", #808000, transp = 40)
```

> __Observação__\
> As duas últimas chamadas de [plot()](https://br.tradingview.com/pine-script-reference/v5/#fun_plot) especificam a transparência usando o parâmetro `transp`. Esse uso deve ser evitado, pois `transp` está obsoleto no Pine Script v5. Usar o parâmetro `transp` para definir transparência não é tão flexível porque requer um argumento do tipo _input integer_ (_inteiro de entrada_), o que significa que deve ser conhecido antes da execução do script e, portanto, não pode ser calculado dinamicamente, à medida que o script é executado barra a barra. Além disso, se for usado um argumento de `color` que já inclui informações de transparência, como é feito nas próximas três chamadas de [plot()](https://br.tradingview.com/pine-script-reference/v5/#fun_plot), qualquer argumento usado para o parâmetro `transp` não terá efeito. Isso também é válido para outras funções com um parâmetro `transp`.

As cores no script anterior não variam à medida que o script é executado barra a barra. Às vezes, no entanto, as cores precisam ser criadas conforme o script é executado em cada barra, pois dependem de condições desconhecidas no momento da compilação ou quando o script começa a execução na barra zero. Para esses casos, os programadores têm duas opções:

1. Usar instruções condicionais para selecionar cores de algumas cores base predeterminadas.
2. Construir novas cores dinamicamente, calculando-as à medida que o script é executado barra a barra, para implementar gradientes de cores, por exemplo.


# Coloração Condicional

Suponha que seja necessário colorir uma média móvel em diferentes cores, dependendo de algumas condições definidas. Para isso, pode-se usar uma instrução condicional que selecionará uma cor diferente para cada um dos estados. Começando por colorir uma média móvel em uma cor de alta quando estiver subindo, e em uma cor de baixa quando não estiver:

![Coloração condicional 01](./imgs/Colors-ConditionalColors-1.png)

```c
//@version=5
indicator("Conditional colors", "", true)
int   lengthInput = input.int(20, "Length", minval = 2)
color maBullColorInput = input.color(color.green, "Bull")
color maBearColorInput = input.color(color.maroon, "Bear")
float ma = ta.sma(close, lengthInput)
// Define our states.
bool maRising  = ta.rising(ma, 1)
// Build our color.
color c_ma = maRising ? maBullColorInput : maBearColorInput
plot(ma, "MA", c_ma, 2)
```

__Note que:__

- São fornecidas aos usuários do script uma seleção de cores para as cores de alta/baixa.
Define-se uma variável booleana `maRising` que será `true` quando a média móvel estiver mais alta na barra atual do que na anterior.
- Define-se uma variável de cor `c_ma` que é atribuída a uma das duas cores, dependendo do valor do booleano `maRising`. Usa-se o [operador ternário ? :](https://br.tradingview.com/pine-script-reference/v5/#op_{question}{colon}) para escrever a instrução condicional.

Também é possível usar cores condicionais para evitar plotagem em determinadas condições. Aqui, plotam-se pivôs de alta e baixa usando uma linha, mas não se deseja plotar nada quando um novo pivô surge, para evitar as junções que apareceriam nas transições de pivô. Para isso, testa-se as mudanças de pivô e usa-se [na](https://br.tradingview.com/pine-script-reference/v5/#var_na) como valor de cor quando uma mudança é detectada, para que nenhuma linha seja plotada nessa barra:

![Coloração condicional 02](./imgs/Colors-ConditionalColors-2.png)

```c
//@version=5
indicator("Conditional colors", "", true)
int legsInput = input.int(5, "Pivot Legs", minval = 1)
color pHiColorInput = input.color(color.olive, "High pivots")
color pLoColorInput = input.color(color.orange, "Low pivots")
// Intialize the pivot level variables.
var float pHi = na
var float pLo = na
// When a new pivot is detected, save its value.
pHi := nz(ta.pivothigh(legsInput, legsInput), pHi)
pLo := nz(ta.pivotlow( legsInput, legsInput), pLo)
// When a new pivot is detected, do not plot a color.
plot(pHi, "High", ta.change(pHi) ? na : pHiColorInput, 2, plot.style_line)
plot(pLo, "Low",  ta.change(pLo) ? na : pLoColorInput, 2, plot.style_line)
```

Para entender como esse código funciona, é preciso saber que [ta.pivothigh()](https://br.tradingview.com/pine-script-reference/v5/#fun_ta{dot}pivothigh) e [ta.pivotlow()](https://br.tradingview.com/pine-script-reference/v5/#fun_ta{dot}pivotlow), usados como estão aqui sem um argumento para o parâmetro `source`, retornarão um valor quando encontrarem um pivô de [_high_](https://br.tradingview.com/pine-script-reference/v5/#var_high)/[_low_](https://br.tradingview.com/pine-script-reference/v5/#var_low) (_alta_/_baixa_), caso contrário, retornam [na](https://br.tradingview.com/pine-script-reference/v5/#var_na).

Ao testar o valor retornado pela função de pivô para [na](https://br.tradingview.com/pine-script-reference/v5/#var_na) usando a função [nz()](https://br.tradingview.com/pine-script-reference/v5/#fun_nz), permite-se que o valor retornado seja atribuído às variáveis `pHi` ou `pLo` apenas quando não for [na](https://br.tradingview.com/pine-script-reference/v5/#var_na), caso contrário, o valor anterior da variável é simplesmente reatribuído a ela, o que não tem impacto em seu valor. Lembre-se de que os valores anteriores de `pHi` e `pLo` são preservados de barra a barra porque se usa a palavra-chave [var](https://br.tradingview.com/pine-script-reference/v5/#kw_var) ao inicializá-los, o que faz com que a inicialização ocorra apenas na primeira barra.

O que resta a fazer é, ao plotar as linhas, inserir uma instrução condicional ternária que resultará em [na](https://br.tradingview.com/pine-script-reference/v5/#var_na) para a cor quando o valor do pivô mudar, ou a cor selecionada nas entradas do script quando o nível do pivô não mudar.


# Próximos Passos

Após seus [primeiros passos](./01_primeiros_passos.md) e seu [primeiro indicador](./02_primeiro_indicador.md), iremos explorar um pouco mais do cenário do Pine Script compartilhando algumas dicas para orientá-lo em sua jornada de aprendizado do Pine Script.


# "Indicadores" e "Estratégias"

[Estratégias](./05_18_estrategias.md) do Pine Script são usadas para realizar backtest em dados históricos e teste avançado em mercados abertos (_open markets_). Além dos cálculos de indicadores, contêm funções `strategy.*()` para enviar ordens de negociação para o emulador de corretora do Pine Script, que simula a execução. As estratégias exibem os resultados do backtest na aba "Teste de Estratégias" (_"Strategy Tester"_) na parte inferior do gráfico, ao lado da aba "Editor Pine" (_"Pine Editor"_).

Os indicadores do Pine Script também realizam cálculos, mas não podem ser usados em backtesting. Pois não exigem o emulador de corretora, consomem menos recursos e executam de maneira mais célere. Portanto, é vantajoso usar indicadores sempre que possível.

Tanto indicadores quanto estratégias podem ser executados em modo de sobreposição (_overlay mode_) (sobre as barras do gráfico (no gráfico)) ou em modo de painel (_pane mode_) (em uma seção separada abaixo ou acima do gráfico). Ambos também podem plotar informações em seu respectivo espaço, e ambos podem gerar [eventos de alerta](./05_01_alertas.md).


# Como os Scripts são Executados

Um script em Pine __não__ é como programas em muitas das linguagens de programação que executam uma única vez e depois param. No ambiente de _execução_ do Pine Script, um script roda de formar semelhante a um loop invisível, onde é executado uma vez em cada barra do gráfico, da esquerda para a direita. Barras do gráfico que fecham enquanto o script executa-os são chamadas de _barras históricas_ (_historical bars_). Quando a execução alcança a última barra do gráfico e o mercado está aberto, estamos na _barra em tempo real_ (_realtime bar_). O script então é executado uma vez a cada vez em que uma mudança de preço ou volume é detectada, e uma última vez para aquela _barra em tempo real_ ao fechar. A barra em tempo real então se torna em uma _barra em tempo real decorrido_. Note que quando o script é executado em tempo real, ele não recalcula em todas as barras históricas do gráfico em cada atualização de preço/volume. Pois já foi calculado uma vez nessas barras, então não é necessário recalculá-las a cada movimento do gráfico. Para mais informações veja a página do [modelo de execução](./04_01_modelo_de_execucao.md).

Quando o script é executado em uma barra histórica, a variável [close](https://br.tradingview.com/pine-script-reference/v5/#var_close), embutida, contém o valor do _fechamento_ daquela barra. Quando o script é executado na barra em tempo real, o [close](https://br.tradingview.com/pine-script-reference/v5/#var_close) retorna o preço __corrente__/__atual__ do símbolo até que a barra se fecha.

Diferente dos indicadores, normalmente as estratégias são executadas apenas uma vez nas barras em tempo real, ao fecharem-se. Também podem ser configurados para executar em cada atualização de preço/volume, se for necessário. Para mais informações e para entender como as estratégias calculam diferentemente dos indicadores veja a página sobre [Estratégias](./05_18_estrategias.md).


# Série Temporal

A principal estrutura de dados usada no Pine Script é chamada de [série temporal](./04_02_series_temporais.md) (_time series_). Séries temporais possui um valor para cada barra na qual o script é executado, então expandem-se continuamente à medida que o script é executado em demais barras. Valores passados da série temporal podem ser referenciados usando o operador de referência histórica: [[]](https://br.tradingview.com/pine-script-reference/v5/#op_[]).
Por exemplo, `close[1]` refere-se ao valor de fechamento ([close](https://br.tradingview.com/pine-script-reference/v5/#var_close)) da barra anterior no momento em que o script está sendo executado.

Embora este mecanismo de indexação possa lembrar sobre _arrays_, uma série temporal é diferente e pensar em termos de arrays pode dificultar a compreensão desse conceito fundamental do Pine Script.
Um bom entendimento tanto do [modelo de execução](./04_01_modelo_de_execucao.md) quanto das [séries temporais](./04_02_series_temporais.md) é essencial para compreender como os scripts Pine funcionam.
Se desconhece dados organizados em séries temporais, será necessário praticar para aprender a utilizá-los eficientemente. Assim que familiarizar com esses conceitos-chave, descobrirá que, ao combinar o uso de séries temporais com funções integradas intrinsecamente projetadas para manuseá-las de forma eficiente, bastante pode ser realizado em poucas linhas de código.


# Publicando Scripts

TradingView é o lar de uma vasta comunidade de programadores de Pine Script e milhões de traders do mundo todo. Uma vez que se torne proficiente o suficiente em Pine Script, pode optar por compartilhar seus scripts com outros traders. Antes de fazer isso, por favor, reserve um tempo para aprender Pine Script o suficiente para fornecer aos traders uma ferramenta original e confiável. Todos os scripts publicados, sem exceção, são analisados por nossa equipe de moderadores e devem cumprir com as nossas [Regras de Publicação de Scripts](https://br.tradingview.com/support/solutions/43000590599), que exigem que sejam originais e bem documentados.

Se deseja usar scripts em Pine para uso próprio, simplesmente escreva-os no Editor Pine e adicione-os ao gráfico a partir daí; não precisa publicá-los para usá-los. Se deseja compartilhar seus scripts com amigos, pode publicá-los de forma privada e enviar o link da sua publicação privada. Para mais informações veja a página sobre [Publicando](./06_04_publicando_scripts.md).


# Explorando a Documentação do Pine Script

Enquanto ao ler o código de scripts publicados é, sem dúvida, útil, passar tempo em nossa documentação é preciso para obter qualquer grau de proficiência no Pine Script. Nossas duas principais fontes de documentação sobre o Pine Script são:
- Este [Manual do Usuário](https://www.tradingview.com/pine-script-docs/en/v5/index.html) do Pine Script v5.
- Nosso [Manual de Referência](https://br.tradingview.com/pine-script-reference/v5) do Pine Script v5 em __inglês__.
- Nosso [Manual de Referência](https://br.tradingview.com/pine-script-reference/v5) do Pine Script v5 em __português do Brasil__.

O [Manual do Usuário](https://www.tradingview.com/pine-script-docs/en/v5/index.html) do Pine Script v5 está no formato HTML e apenas em __inglês__.

O [Manual de Referência](https://br.tradingview.com/pine-script-reference/v5) do Pine Script v5 documenta o que cada variável, função ou palavra-chave faz. É uma ferramenta crucial para todos os programadores de Pine Script; é simplesmente improdutivo se tentar criar scripts de qualquer complexidade sem consultá-lo. Existe em dois formatos: o formato HTML para o qual foi fornecido o link e a versão de pop-up, que pode ser acessada a partir do Editor Pine, seja através do `ctrl` + `clique` ou usando o menu "Mais/Referência do Pine Script (pop-up)" (_"More/Pine Script reference (pop-up)"_) do Editor. O Manual de Referência também está traduzido em outros idiomas.

Existem cinco versões do Pine Script. Certifique-se de que a documentação que está consumindo corresponde à versão do Pine Script com a qual está programando.


# Onde Ir a Partir Daqui?

O [Manual do Usuário](https://www.tradingview.com/pine-script-docs/en/v5/index.html) do Pine Script v5 contém vários exemplos de código utilizados para ilustrar os conceitos que discutimos. Ao explorá-lo, será capaz de estudar os scripts e aprender os fundamentos do Pine Script.
Ler sobre conceitos-chave e experimentá-los imediatamente com código real é produtivo no aprendizado de qualquer linguagem de programação.
Tendo feito o [Primeiro indicador](./02_primeiro_indicador.md), reproduze os exemplos da documentação no Editor e experimente-os. Explore! Não vai causar dano.

Assim é como o [Manual do Usuário](https://www.tradingview.com/pine-script-docs/en/v5/index.html) do Pine Script v5 que está lendo encontra-se organizado:

- A seção da [Linguagem](./04_linguagem.md) explica os principais componentes do Pine Script e como os scripts são executados.
- A seção de [Conceitos](./05_conceitos.md) é mais direcionada para tarefas. Explica como efetuar coisas no Pine Script.
- A seção da [Criação de Scripts](./06_escrevendo_scripts.md) explora ferramentas e truques que ajudarão a programar e publicar scripts.
- A seção de [Perguntas Frequentes](./07_faq.md) tira dúvidas das perguntas comuns dos desenvolvedores do Pine Script.
- A página de [Mensagens de Erro](./08_mensagens_de_erro.md) documenta as causas e soluções para os erros mais comuns de tempo de execução e compilador.
- A página de [Notas de Lançamento](./09_notas_de_versao.md) é onde pode acompanhar as atualizações frequentes do Pine Script.
- A seção da [Guia de Migração](./10_guia_de_migracao.md) explica como fazer a portabilidade entre versões diferentes do Pine Script.
- A página [Onde Obter Mais Informações](./11_onde_obter_mais_informacoes.md) lista outros conteúdos úteis relacionados ao Pine Script, incluindo onde fazer perguntas quando estiver com dúvidas.

Desejamos uma jornada muito bem-sucedida com o Pine Script... e com suas operações!

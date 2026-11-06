# Trabalho Final — Ciência de Dados

## Inteligência de Receita para os Hotéis de Port Alder

| | |
|---|---|
| **Curso** | Ciência de Dados — [labeduc.github.io/ciencia-de-dados](https://labeduc.github.io/ciencia-de-dados/) |
| **Formato** | [individual ou em dupla — a definir pelo professor] |
| **Data de entrega** | [a definir pelo professor] |
| **Apresentação** | 3 a 5 minutos, na última aula |
| **Dataset** | *Nice Rooms: Hotel Revenue, MARS and Reviews* (sintético, licença CC BY 4.0) |

---

## 1. O cenário

A associação de hoteleiros da cidade fictícia de **Port Alder** contratou você como analista de dados. Ela reúne 14 hotéis, do motel de beira de estrada ao resort de luxo, e entregou dois anos de dados de operação (2027 e 2028): ocupação, diárias, receitas, decisões de preço e mais de 12 mil avaliações de hóspedes.

A diretoria quer respostas para três perguntas:

1. **Como está o desempenho de cada hotel, comparado com os concorrentes diretos?**
2. **As decisões de preço tomadas toda semana estão funcionando?**
3. **O que os hóspedes estão dizendo, e isso tem relação com o resultado financeiro?**

No fim, a diretoria quer um **painel (dashboard) web** que qualquer gerente consiga abrir no navegador e usar sem ajuda.

Seu trabalho é percorrer todo o ciclo que você aprendeu no curso: **carregar e entender os dados, analisá-los estatisticamente, organizá-los em um banco de dados, visualizá-los e, por fim, construir o painel com apoio de IA.**

> **Importante:** este enunciado não traz código, fórmulas, consultas nem modelos de página. Ele indica *o que* investigar e *onde, no curso*, está o conhecimento necessário. Decidir *como* fazer faz parte da avaliação.

---

## 2. Conhecendo o dataset

Os dados estão em sete arquivos no formato **JSONL**, em que cada linha do arquivo é um registro. Acompanha o conjunto um `README.md` com a descrição completa de cada coluna.

| Arquivo | O que contém |
|---|---|
| `hotels.jsonl` | Cadastro dos 14 hotéis |
| `room_types.jsonl` | Tipos de quarto de cada hotel |
| `daily_market.jsonl` | Uma linha por hotel por noite: ocupação, diária, receitas, comparação com concorrentes e recomendação de preço |
| `weekly_performance.jsonl` | O mesmo desempenho agregado por semana, com resumo das avaliações |
| `rate_decisions.jsonl` | A decisão de preço de cada semana e o que aconteceu na semana seguinte |
| `reviews.jsonl` | Cada avaliação de hóspede, com texto, notas parciais e resposta do hotel |
| `review_scenarios.jsonl` | Os 11 "motivos" que originam as avaliações |

**Antes de qualquer coisa, leia o `README.md` do dataset com atenção.** Ele explica os indicadores do setor hoteleiro (ocupação, ADR, RevPAR, RGI etc.), o conceito de **MARS** (a diária "adequada ao mercado") e a regra usada para rotular as recomendações de preço. Boa parte das perguntas deste trabalho só faz sentido depois dessa leitura.

> **Atenção:** o dataset é **sintético**, ou seja, gerado por simulação. Ele é ótimo para praticar, mas as conclusões valem para Port Alder e não para o mercado hoteleiro real. Lembre-se disso ao escrever suas interpretações.

---

## 3. Etapas do trabalho

O trabalho tem seis etapas, que seguem a ordem do curso. Cada etapa lista os **conhecimentos a mobilizar**, com links para as aulas, **o que fazer** e **o que entregar**.

---

### Etapa 1 — Carregar, inspecionar e preparar os dados

**Conhecimentos a mobilizar**

- **Módulo 1 — Python:** tipos de dados e conversões ([Aula 02](https://labeduc.github.io/ciencia-de-dados/01_python/Python_AULA02.html)), estruturas de repetição e decisão ([Aula 03](https://labeduc.github.io/ciencia-de-dados/01_python/Python_AULA03.html)), listas e dicionários ([Aula 04](https://labeduc.github.io/ciencia-de-dados/01_python/Python_AULA04.html)) e funções ([Aula 05](https://labeduc.github.io/ciencia-de-dados/01_python/Python_AULA05.html)).
- **Módulo 3 — Visualização (parte de dados):** ambiente de notebooks ([Aula 02](https://labeduc.github.io/ciencia-de-dados/03_visualizacao/aula-02/index.html)), formatos de dados largo e longo ([Aula 03](https://labeduc.github.io/ciencia-de-dados/03_visualizacao/aula-03/index.html)), carregamento e transformação ([Aula 04](https://labeduc.github.io/ciencia-de-dados/03_visualizacao/aula-04/index.html)) e Pandas ([Aula 05](https://labeduc.github.io/ciencia-de-dados/03_visualizacao/aula-05/index.html) e [Prática 05](https://labeduc.github.io/ciencia-de-dados/03_visualizacao/pratica-aula-05/index.html)).

**O que fazer**

1. Carregue os sete arquivos em um notebook (Colab, Jupyter ou VS Code). Os arquivos não estão em CSV, então descubra como ler um arquivo em que cada linha é um registro independente.
2. Para cada tabela, responda:
   - Quantas linhas e colunas ela tem? Os números batem com os informados no `README.md`?
   - Qual é a **granularidade**, isto é, o que exatamente uma linha representa?
   - Quais colunas são numéricas, textuais, datas ou verdadeiro/falso? O tipo carregado está correto? As datas foram reconhecidas como datas?
   - Há valores ausentes? Em quais colunas? O `README.md` explica por que eles existem?
3. Encontre a coluna do cadastro de hotéis que guarda **uma lista dentro de um único campo**. Pense em como transformá-la para responder, por exemplo, "quantos hotéis têm restaurante?".
4. Identifique as **colunas que ligam uma tabela à outra**. Elas serão essenciais nas etapas 3 e 5.
5. Escreva pelo menos **uma função própria** que você reutilize em mais de um ponto do notebook (por exemplo, para padronizar a inspeção de cada tabela).

**Entregável:** notebook `01_exploracao`, com o código e com células de texto explicando cada decisão de limpeza ou transformação.

---

### Etapa 2 — Análise estatística

**Conhecimentos a mobilizar**

- **Módulo 2 — Estatística** ([material do módulo](https://labeduc.github.io/ciencia-de-dados/02_estatistica/leia-antes.html)):
  - população e amostra, organização e agrupamento de dados;
  - medidas de tendência central e frequências absoluta, relativa e acumulada;
  - ordenação e mediana, e o efeito de valores extremos sobre média e mediana;
  - dispersão (desvio padrão e variância), correlação (incluindo correlação espúria), interpolação e extrapolação.

Você pode fazer esta etapa em planilha (Google Planilhas ou Excel), como no Módulo 2, ou em Python com Pandas. Se escolher a planilha, entregue o arquivo e mostre como os dados chegaram até ela.

**O que fazer:** responda às perguntas abaixo, sempre **justificando a medida escolhida**.

1. **Retrato do mercado**
   - Qual é a ocupação típica de cada *tier* (luxury, upscale, boutique, midscale, economy)? Para cada caso, decida se a média ou a mediana descreve melhor o "típico" e explique por quê.
   - Qual *tier* tem a diária (ADR) mais **variável** ao longo do ano? Qual medida mostra isso?
   - Monte a **distribuição de frequências** das notas das avaliações (1 a 5 estrelas), com frequência absoluta, relativa e acumulada. Que porcentagem das avaliações tem 3 estrelas ou menos?
2. **Sazonalidade e eventos**
   - Em que meses e dias da semana a ocupação é mais alta? E a diária?
   - Compare as noites com evento de demanda com as noites comuns. Quanto, em média, mudam a ocupação e a diária? Essa diferença é igual em todos os *tiers*?
3. **Relações entre variáveis**
   - Existe correlação entre ocupação e diária? Ela é positiva ou negativa? Ela muda quando você analisa cada *tier* separadamente em vez do mercado inteiro? Explique o que isso revela.
   - Existe correlação entre a nota média semanal de um hotel e o seu desempenho financeiro na mesma semana? Discuta se essa relação pode ser **causal** ou se pode ser **espúria**.
   - Quais notas parciais da avaliação (limpeza, conforto, serviço, localização, custo-benefício, sono) andam mais junto com a nota geral?
4. **Projeção**
   - Com base nos dois anos disponíveis, faça uma **extrapolação** simples da receita ou da ocupação de um hotel à sua escolha para 2029. Explique o método e **liste as limitações**: o que torna essa projeção frágil?

**Entregável:** notebook `02_estatistica` (ou planilha), com tabelas de resultado e uma **conclusão em texto** de no máximo uma página.

---

### Etapa 3 — Modelagem e banco de dados

**Conhecimentos a mobilizar**

- **Módulo 4 — Banco de Dados:**
  - consultas com seleção, filtro, ordenação, agrupamento e filtro de grupos ([Aula 01](https://labeduc.github.io/ciencia-de-dados/04_banco_dados/aulas/Aula01_Banco_Dados.html) e [SQL Island](https://labeduc.github.io/ciencia-de-dados/04_banco_dados/aulas/Aula01_SQL_Island.html));
  - criação de tabelas, restrições, inserção e junções ([Aula 02](https://labeduc.github.io/ciencia-de-dados/04_banco_dados/aulas/Aula02_Banco_Dados.html));
  - modelagem: entidades, atributos, relacionamentos, diagrama ER e normalização ([Aula 03](https://labeduc.github.io/ciencia-de-dados/04_banco_dados/aulas/Aula03_Banco_Dados.html));
  - NoSQL e banco de documentos ([Aula 04](https://labeduc.github.io/ciencia-de-dados/04_banco_dados/aulas/Aula04_Banco_Dados.html)), opcional.
- **Módulo 3 — Pandas e bancos de dados** ([Aula 14](https://labeduc.github.io/ciencia-de-dados/03_visualizacao/aula-14/index.html)).

**O que fazer**

1. **Modele antes de criar.** Desenhe o **diagrama Entidade-Relacionamento** das sete tabelas, à mão, no Draw.io ou no Excalidraw. Para cada tabela, indique a chave primária, as chaves estrangeiras e a **cardinalidade** de cada relacionamento (1:1, 1:N ou N:N).
2. **Analise a normalização.** Algumas tabelas repetem informações que já existem em outras (procure, por exemplo, o nome e a categoria do hotel). Identifique essas redundâncias e decida se as mantém ou remove no seu banco. Justifique a decisão: vale normalizar tudo ou a redundância ajuda em algum caso?
3. **Crie o banco** em SQLite (ou DuckDB, conforme a Aula 14 de Visualização), com as restrições de integridade adequadas, e carregue os dados a partir do Python.
4. **Verifique a integridade.** Confirme com consultas que nenhuma avaliação, decisão ou registro diário aponta para um hotel inexistente.
5. **Responda às perguntas de negócio** abaixo **usando apenas consultas ao banco**:
   - Ranking dos hotéis por receita total em cada ano.
   - Para cada *tier*, a ocupação média e a diária média por estação do ano.
   - Quais hotéis tiveram **mais de 30%** de avaliações com 1 ou 2 estrelas?
   - Para cada cenário de avaliação, quantas avaliações ele gerou, qual a nota média e qual a ação de recuperação recomendada. Esta pergunta exige juntar tabelas.
   - Entre as decisões de **aumentar** o preço, em que proporção o resultado da semana seguinte foi "melhor"? E entre as de **manter** e **reduzir**?
   - Que porcentagem das avaliações recebeu resposta do hotel, por faixa de nota? Em quanto tempo, em média, o hotel responde?
   - Uma pergunta **criada por você**, que exija junção, agrupamento e filtro de grupos.
6. **(Opcional, vale bônus)** Guarde as avaliações em um banco de documentos (MongoDB via *mongita*, como na Aula 04) e explique em que situação essa escolha seria mais vantajosa que o modelo relacional.

**Entregáveis:** diagrama ER (imagem ou PDF), arquivo do banco de dados e notebook `03_banco_de_dados` com as consultas e a interpretação de cada resultado.

---

### Etapa 4 — Visualização de dados em Python

**Conhecimentos a mobilizar**

- **Módulo 3 — Visualização:**
  - teoria e escolha do tipo de gráfico ([Aula 01](https://labeduc.github.io/ciencia-de-dados/03_visualizacao/aula-01/index.html));
  - barras ([Aula 06](https://labeduc.github.io/ciencia-de-dados/03_visualizacao/aula-06/index.html)), linhas e áreas ([Aula 07](https://labeduc.github.io/ciencia-de-dados/03_visualizacao/aula-07/index.html)), dispersão e quadrantes ([Aula 08](https://labeduc.github.io/ciencia-de-dados/03_visualizacao/aula-08/index.html)) e pizza ([Aula 09](https://labeduc.github.io/ciencia-de-dados/03_visualizacao/aula-09/index.html));
  - melhorias visuais: limpeza, rótulos e paletas ([Aula 10](https://labeduc.github.io/ciencia-de-dados/03_visualizacao/aula-10/index.html));
  - Sankey ([Aula 12](https://labeduc.github.io/ciencia-de-dados/03_visualizacao/aula-12/index.html)) e nuvem de palavras ([Aula 13](https://labeduc.github.io/ciencia-de-dados/03_visualizacao/aula-13/index.html));
  - gráficos interativos ([Aula 15](https://labeduc.github.io/ciencia-de-dados/03_visualizacao/aula-15/index.html) e [Aula 16](https://labeduc.github.io/ciencia-de-dados/03_visualizacao/aula-16/index.html));
  - IA generativa aplicada à análise ([Aula 17](https://labeduc.github.io/ciencia-de-dados/03_visualizacao/aula-17/index.html)).

**O que fazer:** produza **no mínimo oito visualizações** que respondam às perguntas abaixo. Para cada uma, **justifique o tipo de gráfico escolhido** e escreva, em uma ou duas frases, **o que o leitor deve concluir** ao olhá-la.

1. Como os *tiers* se comparam em ocupação, diária e receita por quarto disponível?
2. Como a ocupação e a diária evoluem ao longo dos dois anos? Os eventos de demanda aparecem no gráfico?
3. **Mapa de posicionamento:** posicione os hotéis em um gráfico de dispersão dividido em **quadrantes**, usando o índice de preço e o índice de ocupação contra os concorrentes (o `README.md` explica ambos). Que hotéis cobram caro e enchem? Quais cobram caro e ficam vazios? Que linha de referência separa os quadrantes, e por quê?
4. Quanto o preço praticado se afasta da diária MARS? Mostre a distribuição desse desvio por hotel ou por *tier*.
5. Como se distribuem os cenários das avaliações? Antes de usar um gráfico de pizza, verifique se ele segue as boas práticas da Aula 09. Se não seguir, escolha outra forma.
6. **Fluxo das avaliações:** construa um diagrama de **Sankey** que mostre o caminho de um conjunto de categorias até outro (por exemplo, do cenário até a faixa de nota, ou do segmento do hóspede até o canal da avaliação).
7. **Nuvens de palavras** comparando o texto das avaliações positivas com o das negativas.
8. Pelo menos **um gráfico interativo** em Plotly, com rótulos e *tooltips* úteis.

Aplique a todos os gráficos as práticas da Aula 10: títulos que informam a conclusão, eixos com unidades, paleta consistente e nada de elementos decorativos sem função.

**Entregável:** notebook `04_visualizacao`.

---

### Etapa 5 — Painel web desenvolvido com IA

**Conhecimentos a mobilizar**

- **Módulo 5 — FrontEnd Dev com AI:**
  - primeiro contato e modelo de requisitos para a IA ([Módulo 1](https://labeduc.github.io/ciencia-de-dados/05_dev_com_ai/modulo01_primeiro_contato_ai/CONTEUDO.html));
  - revisão crítica de código gerado ([Módulo 2](https://labeduc.github.io/ciencia-de-dados/05_dev_com_ai/modulo02_revisao_com_ai/CONTEUDO.html));
  - UX e prototipação ([Módulo 6](https://labeduc.github.io/ciencia-de-dados/05_dev_com_ai/modulo06_ux_prototipacao_ai/CONTEUDO.html));
  - estrutura e estilo da página ([Módulo 7](https://labeduc.github.io/ciencia-de-dados/05_dev_com_ai/modulo07_html_css_com_ai/CONTEUDO.html));
  - interatividade e dados na tela ([Módulo 8](https://labeduc.github.io/ciencia-de-dados/05_dev_com_ai/modulo08_javascript_interatividade/CONTEUDO.html));
  - gráficos no navegador e integração ([Módulo 9](https://labeduc.github.io/ciencia-de-dados/05_dev_com_ai/modulo09_dados_frontend_integracao/CONTEUDO.html)).

**O que fazer**

1. **Defina o público e as perguntas.** Para quem é o painel: a diretoria da associação ou o gerente de um hotel? Que três decisões ele deve ajudar a tomar? Escreva isso antes de desenhar qualquer tela.
2. **Prototipe.** Faça um **wireframe** (Módulo 6) com cabeçalho, indicadores-chave (KPIs), gráficos, filtros, tabela e rodapé com a fonte dos dados. Revise-o com os princípios de UX: hierarquia visual, agrupamento e leitura em celular.
3. **Prepare os dados para a web.** O navegador não vai ler os arquivos originais nem o banco. Use Python para gerar, a partir dos seus resultados, um **conjunto de dados resumido** no formato que a página consegue usar. Decida o nível de agregação (por semana? por mês? por hotel?) e justifique a decisão pensando em tamanho e utilidade.
4. **Construa a página com a IA como parceira**, seguindo o ciclo do curso: *escrever o requisito → gerar → entender → revisar*. Use o modelo de requisitos do Módulo 1 (Contexto, Objetivo, Detalhes, Formato) e não aceite código que você não sabe explicar.
5. **Requisitos mínimos do painel:**
   - pelo menos **quatro KPIs** calculados a partir dos dados, não digitados à mão;
   - pelo menos **três gráficos** feitos com a biblioteca de gráficos vista no Módulo 9, **coerentes com as análises das etapas 2 a 4**;
   - uma **tabela** gerada a partir dos dados;
   - pelo menos **um filtro** (por exemplo, por hotel, *tier* ou ano) que atualize **ao mesmo tempo** os KPIs, a tabela e os gráficos;
   - layout **responsivo**, que funcione no computador e no celular;
   - mensagem adequada quando um filtro não retornar dados;
   - **fonte dos dados** indicada no rodapé, com o aviso de que o dataset é sintético.
6. **Teste.** Abra o console do navegador e confirme que não há erros. Confira se pelo menos dois KPIs do painel batem com os valores calculados no notebook.

**Entregável:** pasta `dashboard/` com os três arquivos da página (estrutura, estilo e comportamento) e o arquivo de dados que ela consome. O painel precisa abrir e funcionar no navegador sem nenhuma instalação adicional.

---

### Etapa 6 — Diário de Campo e apresentação

**Conhecimentos a mobilizar**

- **Módulo 10 — Projeto Integrador** ([conteúdo](https://labeduc.github.io/ciencia-de-dados/05_dev_com_ai/modulo10_projeto_integrador/CONTEUDO.html)).

**O que fazer**

1. **Diário de Campo** (1 a 2 páginas), seguindo a estrutura do Módulo 10:
   - os **3 melhores prompts** que você usou *em qualquer etapa do trabalho*, não apenas no painel, explicando por que funcionaram;
   - **1 erro da IA**: o que você pediu, o que veio errado, como percebeu e como corrigiu;
   - **1 aprendizado** que você teve graças a uma explicação da IA, e onde o aplicou.
2. **Relatório executivo** (no máximo 2 páginas) que responda, em linguagem de negócio, às três perguntas da diretoria da Seção 1. Use os números e gráficos que você produziu e indique **as limitações** das suas conclusões.
3. **Apresentação** de 3 a 5 minutos, ao vivo ou em 2 a 3 slides:
   - o que o painel mostra e para quem;
   - demonstração do painel;
   - a principal descoberta sobre os dados;
   - um destaque do Diário de Campo.

**Entregáveis:** `diario_de_campo.md` (ou PDF), `relatorio_executivo` (PDF) e a apresentação.

---

## 4. Estrutura de entrega sugerida

```
trabalho-final/
├── README.md                ← quem fez, como abrir cada parte, ferramentas usadas
├── dados/                   ← arquivos originais (ou instrução de onde baixá-los)
├── notebooks/
│   ├── 01_exploracao
│   ├── 02_estatistica       ← ou a planilha equivalente
│   ├── 03_banco_de_dados
│   └── 04_visualizacao
├── banco/                   ← arquivo do banco e diagrama ER
├── dashboard/               ← página, estilo, script e dados resumidos
├── diario_de_campo
├── relatorio_executivo
└── apresentacao
```

Entregue em um repositório (GitHub) ou em um arquivo compactado, conforme orientação do professor. **Os notebooks devem ser entregues já executados**, com as saídas visíveis.

---

## 5. Critérios de avaliação

| Critério | Peso | O que será observado |
|---|---:|---|
| Exploração e preparação (Etapa 1) | 10% | Leitura correta dos dados, tratamento justificado de tipos e ausentes, uso de funções próprias |
| Análise estatística (Etapa 2) | 15% | Escolha justificada das medidas, interpretação correta de correlação, honestidade sobre limites da projeção |
| Banco de dados (Etapa 3) | 15% | Diagrama ER correto, discussão de normalização, integridade verificada, consultas corretas e interpretadas |
| Visualização (Etapa 4) | 20% | Tipo de gráfico adequado à pergunta, clareza, boas práticas visuais, conclusões explícitas |
| Painel web (Etapa 5) | 20% | Requisitos mínimos atendidos, funcionamento sem erros, UX, coerência com as análises |
| Uso crítico da IA (Diário de Campo) | 10% | Prompts bem estruturados, erros identificados e corrigidos, capacidade de explicar o que foi gerado |
| Relatório executivo e apresentação | 10% | Respostas claras às perguntas da diretoria, comunicação objetiva, respeito ao tempo |
| **Bônus** | até +5% | Etapa NoSQL opcional, ou análise adicional relevante e bem fundamentada |

---

## 6. Regras sobre o uso de IA

- **O uso de assistentes de IA é permitido e incentivado** em todas as etapas, do mesmo jeito que foi praticado no Módulo 5.
- Você é **responsável por tudo o que entregar**. Na apresentação, o professor pode pedir que você explique qualquer trecho de código, consulta ou gráfico. *"Foi a IA que fez"* não é uma explicação.
- **Desconfie e verifique.** Confira os números que a IA sugerir contra os seus próprios cálculos. Assistentes erram contas, inventam nomes de colunas e às vezes "resumem" o dataset sem tê-lo lido.
- Registre os prompts relevantes **à medida que trabalha**. Reconstituí-los no final é muito mais difícil.

---

## 7. Armadilhas comuns: leia antes de começar

- **Não misture categorias de hotel ao comparar preços.** O `README.md` explica por que um hotel econômico não é concorrente de um resort de luxo. Muitas conclusões erradas nascem de médias que juntam todos os *tiers*.
- **Ausências esperadas.** Algumas colunas ficam vazias por motivos documentados: o primeiro ano não tem comparação com o ano anterior, semanas sem avaliação não têm nota média e a última semana não tem decisão avaliada. Entenda cada caso antes de remover ou preencher valores.
- **Métricas dos concorrentes são ponderadas.** Leia a seção do `README.md` sobre isso antes de tentar recalculá-las a partir de outras colunas.
- **Resultado não é causa.** Se a semana depois de um aumento de preço foi melhor, isso não prova que o aumento causou a melhora. O próprio `README.md` faz esse alerta. Use esse cuidado no seu relatório.
- **Rótulos com ruído.** Uma pequena parte das recomendações de preço foi propositalmente "errada" na geração dos dados. Não estranhe exceções à regra documentada.
- **Granularidade.** Uma linha diária e uma linha semanal não são comparáveis diretamente. Saiba sempre o que uma linha representa antes de somar ou tirar média.

---

## 8. Material de referência

- Curso completo: <https://labeduc.github.io/ciencia-de-dados/>
- Plano do curso: <https://labeduc.github.io/ciencia-de-dados/PLANO_DE_AULA.html>
- Conteúdo programático: <https://labeduc.github.io/ciencia-de-dados/CONTEUDO_PROGRAMATICO.html>
- Dataset e documentação: `README.md` que acompanha os dados, também disponível no Kaggle: <https://www.kaggle.com/datasets/crissymoon/nice-rooms-hotel-revenue-rate-intelligence>

Bom trabalho!

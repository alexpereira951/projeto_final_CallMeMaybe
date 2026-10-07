# 📊 Projeto Final — CallMeMaybe, SQL e Teste A/B

Este repositório reúne **três frentes de análise de dados desenvolvidas em paralelo**: a identificação de operadores ineficientes em uma empresa de telecomunicações, uma análise SQL sobre um serviço de livros e a conclusão de um teste A/B relacionado a um sistema de recomendação.

O projeto principal transforma dados operacionais em **KPIs de produtividade, classificação de eficiência, análises estatísticas e recomendações de negócio**, além de contemplar a preparação de dados para um dashboard no Tableau Public.

---

## 🎯 Visão Geral do Projeto

Com certeza, o principal desafio foi concluir os três projetos em um prazo apertado de uma semana. Mas consegui decompor os projetos em atividades de metas diárias, e montar um cronograma de atividades, junto ao método Japonês PROMODORO (consistindo de 5 minutos de descanso para cada 25 minutos de foco no desenvolvimento dos projetos, conseguindo um foco diário total acumulado de 7 a 10 horas, até concluir o projeto), para conseguir entregar o projeto completo com 12 horas de antecedência para o prazo final. 

- **Dia 1**: Planejei toda a gestão de tempo e tarefas para o desenvolvimento do projeto e iniciei a decomposição do projeto principal (identificação de operadores ineficientes);
- **Dia 2**: Concluí a decomposição e criação das KPIs do projeto principal;
- **Dia 3**: Iniciei a análise do projeto principal (Análise exploratória de Dados - EDA) e fiz o projeto paralelo de SQL;
- **Dia 4**: Concluí o notebook do projeto principal (EDA, Testes de Hipóteses e validação de KPIs, e Considerações finais de negócio);
- **Dia 5**: Criei o Dashboard e apresentação do projeto principal, e iniciei o projeto de teste A/B;
- **Dia 6**: Concluí o projeto de teste A/B;
- **Dia 7**: Revisei todo o projeto e organisei toda a estrutura de pastas, arquivos, e arquivos na nuvem (Google Drive);

### Frentes desenvolvidas

| Projeto | Objetivo |
|---|---|
| **Eficiência de Operadores** | Identificar automaticamente operadores ineficientes e investigar os fatores associados à produtividade. |
| **Projeto SQL** | Responder perguntas estratégicas sobre livros, editoras, autores, avaliações e usuários de uma plataforma de livros. |
| **Teste A/B** | Avaliar a introdução de um sistema de recomendação melhorado e verificar seu impacto no funil de conversão. |

---

# 📞 1. Projeto Principal — Eficiência de Operadores

## Problema de negócio

O projeto principal foi desenvolvido para a empresa de telecomunicações **CallMeMaybe** com o objetivo de criar um sistema de identificação automática de operadores ineficientes.

A classificação considera três critérios mensuráveis:

- **Taxa de chamadas perdidas > 15%**
- **Tempo médio de espera > 46 segundos**
- **Menos de 50 chamadas ativas por dia**, para operadores de saída

Cada critério violado adiciona um ponto ao **Score de Ineficiência**:

| Score | Classificação |
|---:|---|
| 0 ou 1 | ✅ Eficiente |
| 2 ou 3 | 🔴 Ineficiente |

O resultado da análise também foi utilizado como base para a construção de um **dashboard no Tableau Public**.

## 📊 Dados utilizados

### `telecom_dataset_us.csv`

Contém os registros gerais das chamadas, incluindo:

- `user_id`
- `date`
- `direction`
- `internal`
- `operator_id`
- `is_missed_call`
- `calls_count`
- `call_duration`
- `total_call_duration`

### `telecom_clients_us.csv`

Contém informações sobre os clientes:

- `user_id`
- `tariff_plan`
- `date_start`

## 🔎 Preparação e qualidade dos dados

O notebook realiza uma etapa estruturada de preparação antes das análises:

1. Carregamento inicial de amostras para otimização dos tipos de dados.
2. Conversão de datas para `datetime`.
3. Conversão de variáveis categóricas para `category`.
4. Conversão de `internal` para tipo booleano.
5. Tratamento de valores ausentes.
6. Remoção de registros sem `operator_id`, necessários para a identificação individual do operador.
7. Remoção de registros duplicados.
8. Criação da variável `espera`, calculada como:
   `total_call_duration - call_duration`.
9. Criação da variável `dia`.
10. Tratamento de valores de espera considerados problemáticos.

A otimização das amostras reduziu o consumo de memória de **18,1 KB para 5,4 KB** no dataset de chamadas e de **13,1 KB para 2,0 KB** no dataset de clientes.

## 📐 KPIs

### KPI 1 — Taxa de chamadas perdidas

```text
Taxa de Chamadas Perdidas =
(Número de Chamadas Perdidas / Total de Chamadas) × 100
```

### KPI 2 — Tempo médio de espera

```text
Tempo Médio de Espera =
Σ(Tempo de Espera) / Número Total de Chamadas
```

### KPI 3 — Chamadas ativas por dia

Aplicado aos operadores de saída:

```text
Chamadas Ativas por Dia =
Número de Chamadas Realizadas / Número de Dias Trabalhados
```

### KPI 4 — Produtividade geral

O score combina os três critérios anteriores e classifica cada operador como **eficiente** ou **ineficiente**.

---

## 🧪 Análises estatísticas

O notebook possui uma função dedicada para testes de hipóteses entre duas variáveis contínuas.

O procedimento verifica:

- Normalidade com **Shapiro-Wilk** ou **Anderson-Darling**;
- Homogeneidade de variâncias com **Levene**, quando aplicável;
- **Teste t de Student** ou **Welch**, para dados normais;
- **Mann-Whitney**, quando os dados não apresentam normalidade.

Foram investigadas, entre outras, as seguintes hipóteses:

- diferença na taxa de chamadas perdidas entre operadores eficientes e ineficientes;
- diferença no tempo médio de espera;
- diferença no volume de chamadas ativas;
- diferença entre os tempos médios de espera dos planos B e C.

### Principais resultados

- Não foi identificada diferença significativa na **taxa de chamadas perdidas** entre operadores eficientes e ineficientes.
- Operadores classificados como ineficientes apresentaram **tempo médio de espera significativamente menor** que os eficientes, no teste realizado com nível de significância de 1%.
- Operadores classificados como ineficientes apresentaram **menos chamadas ativas por dia** que os eficientes, também com nível de significância de 1%.
- Os tempos médios de espera dos operadores que atendem clientes dos planos **B e C** apresentaram diferença estatisticamente significativa.

## 📈 EDA e análise de Cohorts

A análise exploratória indicou:

- Operadores que atendem clientes do **plano C** apresentaram melhor desempenho em taxa de chamadas perdidas e tempo médio de espera.
- Operadores do **plano A** apresentaram a maior mediana de chamadas ativas por dia.
- No contexto geral da produtividade, o **plano A** apresentou a menor distância entre operadores eficientes e ineficientes.
- A análise de cohorts não mostrou um padrão consistente que indique que operadores que atendem clientes mais antigos sejam necessariamente mais eficientes.
- As cohorts de **23/09/2019** e **21/10/2019** apresentaram operadores significativamente mais eficientes.

## 💼 Recomendações de negócio

A análise resultou em quatro frentes de recomendação:

1. **Equalização de processos entre planos A e C**, utilizando práticas observadas nos operadores do plano C.
2. **Capacitação e gestão de capacidade**, com foco em técnicas operacionais e redistribuição de carga.
3. **Investigação aprofundada das cohorts de 23/09/2019 e 21/10/2019** para identificar fatores associados ao desempenho superior.
4. **Alocação estratégica de turnos e rotinas operacionais**, considerando a implementação de práticas de Workforce Management (WFM).

## 📊 Dashboard

O projeto também contempla um dashboard desenvolvido no **Tableau Public** para acompanhamento da eficiência dos operadores.

[Clique aqui para acessar o Dashboard no Tableau Public](https://public.tableau.com/views/ProjetoCallMeMaybe/DASHBOARD?:language=pt-BR&:sid=&:redirect=auth&:display_count=n&:origin=viz_share_link)

> Materiais complementares — PDF da apresentação e CSV utilizado no Tableau `operators_per_time.csv` (Clique nos links abaixo para acessa-los no Google Drive):

Link: [PDF da Apresentação - no Google Drive](https://drive.google.com/file/d/1xfbNiHHuqEniSRaYkL1K-DcGGFbZ356u/view?usp=sharing)

Link: [CSV usado no Dashboard - no Google Drive](https://drive.google.com/file/d/1bBBnhqpRz9zTK_igoc-HIcvrVYjZF_o_/view?usp=sharing)

---

# 🗄️ 2. Projeto SQL — Análise do Mercado de Livros

## Contexto

Durante o período da pandemia de coronavírus, mudanças no comportamento dos consumidores aumentaram o interesse por soluções digitais voltadas ao consumo de livros.

O projeto utiliza um banco de dados contendo informações sobre:

- livros;
- autores;
- editoras;
- avaliações;
- resenhas.

O objetivo foi responder perguntas estratégicas que poderiam apoiar o posicionamento de um novo produto no mercado.

## 🛠️ Abordagem técnica

O notebook utiliza:

- **Python**
- **Pandas**
- **SQLAlchemy**
- **SQL**
- consultas em banco **PostgreSQL**

A conexão com o banco é realizada por meio de `SQLAlchemy`, enquanto os resultados das consultas SQL são carregados em DataFrames do Pandas.

## 🔎 Perguntas analisadas

Foram desenvolvidas consultas para responder:

1. Quantos livros foram lançados após 1º de janeiro de 2000?
2. Qual é o número de avaliações e a classificação média por livro?
3. Qual editora lançou mais livros com mais de 50 páginas?
4. Qual autor apresenta a maior média de classificação, considerando livros com pelo menos 50 classificações?
5. Qual é a média de avaliações dos usuários que classificaram mais de 50 livros?

## 📌 Resultados

- **819 livros** foram lançados após 1º de janeiro de 2000.
- Entre os títulos destacados na análise de avaliações e reviews estão **A Dirty Job (Grim Reaper #1)**, **School's Out—Forever (Maximum Ride #2)** e **Moneyball: The Art of Winning an Unfair Game**.
- As editoras com maior volume de livros com mais de 50 páginas destacadas pela análise foram **Penguin Books**, **Vintage** e **Grand Central Publishing**.
- Entre os autores avaliados sob o critério de pelo menos 50 classificações por livro, destacaram-se **J.K. Rowling/Mary GrandPré**, **Markus Zusak/Cao Xuân Việt Khương** e **J.R.R. Tolkien**.
- Usuários que classificaram mais de 50 livros apresentaram média de **24 avaliações**.

---

# 🧪 3. Projeto de Teste A/B

## Objetivo

O teste A/B foi desenvolvido para avaliar mudanças relacionadas à introdução de um **sistema de recomendação melhorado**.

O critério definido no projeto estabelecia a necessidade de aumento de **10% em cada etapa do funil**:

```text
login → product_page → purchase
```

A avaliação considerou novos usuários da **União Europeia**, cadastrados entre **07/12 e 21/12/2020**, distribuídos entre:

- **Grupo A — controle**
- **Grupo B — teste**

O período de avaliação considerado foi de até **14 dias após o cadastro**.

## 🧹 Tratamento e validação dos dados

O notebook realiza:

- otimização dos tipos de dados;
- conversão de datas;
- tratamento de valores ausentes;
- verificação de duplicados explícitos;
- identificação de usuários presentes em mais de um teste;
- remoção dos usuários que participavam de testes diferentes;
- análise da composição dos grupos;
- verificação de campanhas de marketing sobrepostas;
- avaliação do tamanho da amostra.

Um ponto importante identificado foi a existência de usuários presentes em mais de um teste. Esses usuários foram removidos das bases utilizadas na análise para evitar interferência entre experimentos.

## 📊 Análise do funil

O evento `product_cart` foi retirado do funil principal porque a análise indicou que ele não era uma etapa obrigatória para a realização da compra.

O funil final considerado foi:

```text
login → product_page → purchase
```

## 📐 Teste estatístico

Para as etapas do funil foi utilizado **teste Z para igualdade de proporções**, com nível de significância de **1%** no procedimento implementado no notebook.

### Resultados

- O teste não indicou melhoria na conversão do evento **`login`**.
- Para **`product_page`**, a conversão do Grupo B ficou **14% abaixo** da conversão do Grupo A.
- O teste não indicou melhoria na conversão do evento **`purchase`**.
- O critério de sucesso de **+10% em cada etapa** não foi atingido.

## ⚠️ Pontos de atenção identificados

A análise encontrou condições que comprometem a interpretação do experimento:

- existência de uma campanha de marketing de **Christmas & New Year Promo** sobreposta ao período analisado;
- grupos com distribuição aproximada de **75% no Grupo A e 25% no Grupo B**;
- **2.788 participantes**, contra uma expectativa de 6.000 usuários;
- ausência de melhoria nas métricas de conversão analisadas.

Com base nesses fatores, o notebook recomenda **não dar continuidade ao teste nas condições analisadas** e revisar o desenho experimental antes de uma nova execução.

---

# 🏗️ Estrutura do Repositório

A organização apresentada no projeto é:

```text
projeto_final_CallMeMaybe/
│
├── projeto SQL/
│   └── notebook_sql.ipynb
│
├── projeto_eficiencia_operadores/
│   ├── datasets/
│   │   ├── telecom_clients_us.csv
│   │   └── telecom_dataset_us.csv
│   └── notebooks/
│       └── notebook_eficiencia_operadores.ipynb
│
├── teste AB/
│   ├── datasets/
│   │   ├── ab_project_marketing_events_us.csv
│   │   ├── final_ab_events_upd_us.csv
│   │   ├── final_ab_new_users_upd_us.csv
│   │   └── final_ab_participants_upd_us.csv
│   └── notebooks/
│       └── notebook_teste_ab.ipynb
│
├── descricao_geral.md
└── requirements.txt
```

### Organização das pastas

| Diretório/Arquivo | Finalidade |
|---|---|
| `projeto SQL/` | Notebook e consultas SQL utilizadas na análise do mercado de livros. |
| `projeto_eficiencia_operadores/` | Projeto principal de análise da eficiência dos operadores. |
| `projeto_eficiencia_operadores/datasets/` | Bases de chamadas e clientes utilizadas no projeto principal. |
| `projeto_eficiencia_operadores/notebooks/` | Notebook de análise e modelagem dos KPIs de eficiência. |
| `teste AB/` | Projeto de avaliação do experimento A/B. |
| `teste AB/datasets/` | Bases de marketing, eventos, usuários e participantes do experimento. |
| `teste AB/notebooks/` | Notebook utilizado para tratamento, EDA e análise estatística do teste. |
| `descricao_geral.md` | Documentação geral e contextualização dos projetos. |
| `requirements.txt` | Arquivo de dependências do projeto. |

---

# 🚀 Instalação e Execução

## 1. Clonar o repositório

```bash
git clone https://github.com/alexpereira951/projeto_final_CallMeMaybe
cd projeto_final_CallMeMaybe
```

## 2. Instalar as dependências

```bash
pip install -r requirements.txt
```

## 3. Executar os notebooks

```bash
jupyter notebook
```

Depois, abra o notebook correspondente ao projeto desejado:

```text
projeto SQL/notebook_sql.ipynb
projeto_eficiencia_operadores/notebooks/notebook_eficiencia_operadores.ipynb
teste AB/notebooks/notebook_teste_ab.ipynb
```

> **Projeto SQL:** a execução das consultas depende do acesso ao banco utilizado pelo notebook. As credenciais de conexão não são reproduzidas neste README por segurança.

---

# 🛠️ Stack Tecnológica

| Tecnologia | Aplicação |
|---|---|
| 🐍 **Python** | Linguagem principal das análises |
| 🐼 **Pandas** | Manipulação, limpeza e transformação dos dados |
| 📊 **NumPy** | Operações numéricas |
| 📈 **Matplotlib** | Visualizações estatísticas |
| 🎨 **Seaborn** | Visualizações exploratórias |
| 📉 **Plotly Express** | Gráficos interativos e funis |
| 🧪 **SciPy** | Testes estatísticos e análise de hipóteses |
| 📐 **Statsmodels** | Teste Z para igualdade de proporções |
| 🗄️ **SQL** | Consultas analíticas ao banco de dados |
| 🔌 **SQLAlchemy** | Conexão entre Python e banco SQL |
| 📊 **Tableau Public** | Dashboard de acompanhamento da eficiência dos operadores |
| 📓 **Jupyter Notebook** | Desenvolvimento e documentação das análises |

---

# 📌 Resultados e Conclusões

O projeto demonstra uma abordagem completa de análise de dados, passando por **tratamento e validação dos dados, análise exploratória, criação de métricas, segmentação, análise de cohorts, testes estatísticos, consultas SQL e avaliação de experimento A/B**.

No projeto principal, foi estruturado um mecanismo objetivo para classificar operadores segundo critérios mensuráveis de eficiência e transformar os resultados em recomendações operacionais e em um dashboard.

No projeto SQL, consultas relacionais foram utilizadas para transformar dados de livros, autores, editoras, ratings e reviews em informações úteis para decisões de produto.

No teste A/B, além da análise das conversões, foram investigadas condições experimentais que poderiam comprometer a validade da comparação entre os grupos.

---

# ⚠️ Limitações

O projeto apresenta limitações que devem ser consideradas na interpretação dos resultados:

1. **Dados e período específicos:** as conclusões dos projetos dependem dos recortes temporais e das bases disponibilizadas, não devendo ser automaticamente generalizadas para períodos ou populações diferentes.

2. **Dados ausentes e exclusões:** no projeto de eficiência, registros sem `operator_id` e outros registros com problemas de qualidade foram removidos, reduzindo a quantidade de observações disponíveis para determinadas análises.

3. **Tratamento de valores de espera:** alguns tempos de espera considerados problemáticos foram substituídos pela mediana, o que reduz a influência de valores extremos, mas também altera os valores originais observados.

4. **Limitações do experimento A/B:** o teste apresentou grupos desbalanceados, quantidade de participantes abaixo do esperado e sobreposição com uma campanha de marketing sazonal, fatores que dificultam a interpretação causal dos resultados.

---

# 📁 Notebooks do Projeto

### Projeto SQL

`projeto SQL/notebook_sql.ipynb`

Consultas SQL para análise de livros, editoras, autores, avaliações e usuários.

### Eficiência de Operadores

`projeto_eficiencia_operadores/notebooks/notebook_eficiencia_operadores.ipynb`

Pipeline de tratamento dos dados, cálculo dos KPIs, classificação de operadores, EDA, cohorts, testes de hipóteses e recomendações de negócio.

### Teste A/B

`teste AB/notebooks/notebook_teste_ab.ipynb`

Tratamento dos dados, validação da amostra, análise do funil, investigação de possíveis vieses e testes estatísticos de conversão.

---

## 👨‍💻 Sobre o Projeto

Este repositório consolida diferentes competências de **Engenharia e Análise de Dados**, incluindo preparação de dados, SQL, análise estatística, experimentação, visualização e tradução de resultados técnicos em recomendações de negócio.

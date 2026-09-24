# poc-puc-mvp-eng-dados
MVP desenvolvido na disciplina de engenharia de dados na pós graduação de Ciência de Dados e Analytics na PUC-Rio

# 1. Contexto de Negócio e Perguntas


## 1.1 Contexto e Objetivo do Trabalho

A educação básica brasileira apresenta diferenças significativas de desempenho entre escolas, municípios e estados. Para compreender essas diferenças, foi desenvolvido o Saeb.

O Saeb (Sistema de Avaliação da Educação Básica) é um conjunto de avaliações externas em larga escala criado em 1990 pelo Inep para diagnosticar a qualidade da educação básica no Brasil.

**Como funciona**

Testes e questionários: O Saeb é um conjunto de testes e questionários que avaliam o desempenho dos estudantes em Língua Portuguesa e Matemática, além de coletar dados contextuais e socioeconômicos.

Periodicidade: É aplicado a cada dois anos, geralmente envolvendo a rede pública inteira e uma amostra da rede privada.

Público-alvo: Abrange estudantes de diferentes etapas do ensino, com foco tradicional no 5º e 9º ano do Ensino Fundamental e no 3º ano do Ensino Médio.

O objetivo deste trabalho será analisar os dados do Saeb por escola no ano de 2023 (último com microdados disponíveis), buscando identificar padrões de desempenho, diferenças entre redes de ensino e relações entre características das escolas e seus resultados educacionais. O foco das análises será nos dados do 9º ano do ensino fundamental com o objetivo de avaliar a qualidade do ensino básico.

Além dos dados educacionais, serão incorporados indicadores de desenvolvimento dos estados brasileiros, permitindo investigar se as diferenças observadas no desempenho escolar estão associadas ao contexto socioeconômico em que as escolas estão inseridas.


## 1.2. Perguntas a serem respondidas

1) Quais estados apresentam os melhores e piores desempenhos médios no SAEB?
2) Existem diferenças relevantes de desempenho entre escolas públicas e privadas?
3) Qual a diferença de desempenho das escolas de diferentes niveis socioeconômicos?
4) Quais estados entregam um desempenho educacional abaixo do esperado para seu nível de desenvolvimento?
5) Existem estados que se destacam por apresentar bons resultados educacionais mesmo possuindo indicadores socioeconômicos inferiores?


## 1.3. Resumo dos Dados Utilizados

Para este trabalho foram utilizadas três fontes de dados, os dados de desempenho por escola no Saeb, uma base de dados socioeconômicos dos estados e uma terceira para servir de dimensão

## 1.3.1. Dados do Saeb

Foram utilizados os resultados por escola do Saeb 2023, obtidos a partir do portal de acesso a informação do governo federal (https://www.gov.br/inep/pt-br/acesso-a-informacao/dados-abertos/microdados/saeb).

A base é um arquivo csv com o resultado escola a escola para as notas de cada escola obtidas nas disciplinas de português e matemática para o 5º e 9º ano do ensino fundamental, assim como para o ano de conclusão do ensino médio. 

Os detalhes dos dados disponíveis podem ser encontrado na etapa


## 1.3.2. Dados socioeconômicos dos estados

Foram utilizados os dados do censo demográfico de 1991 a 2010 realizado pelo IBGE, baixados a partir do site do Atlas do Desenvolvimento Humano no Brasil (https://www.atlasbrasil.org.br/acervo/biblioteca). 
Dentro dessa base há diversos indicadores socioeconômicos como IDHM,  por estado, gerados por ano de realização do censo, renda per capita, entre outros. 

Os detalhes dos dados disponíveis podem ser encontrado na etapa


## 1.3.3. Tabela dimensão de estados

Tabela gerada com os estados brasileiros + distrito federal com as regiões em que cada estado está contido. Criada para servir de tabela dimensão que conecta as outras duas bases de dados.


# 2. Carga dos dados


A carga de dados foi realizada no Databricks via carga manual. Para permitir isso foram utilizados dois notebooks:

O primeiro foi o [notebook de preparação](Workspace/01%20%20-%20preparação.ipynb). Nele, foram criados todos os schemas utilizados nesse trabalho, que são:

1) staging: schema criado para servir de repositório dos arquivos com os dados originais
2) bronze: criado com o intuito disponibilizar os dados dos arquivos contidos no schema de staging em tabelas.
3) silver: criado com o intuito de disponibilizar tabelas com dados padronizados, limpos e organizados. 
4) gold: Camada final com tabelas criadas com objetivo criar agregações e métricas para responder as perguntas

A imagem abaixo mostra as 4 camadas criadas dentro do Databricks:

![catalogo](images/01-imagem_catalogo_dados.png)

O segundo notebook criado foi o [notebook de download](Workspace/02%20-%20download.ipynb). Nele, foram criados dois volumes, que são:

1) dados_saeb: volume para os dados do exame saeb (explicados na etapa [1.3.1](#131-dados-do-saeb)). Upload dos dados feito de forma manual com um arquivo csv
2) dados_estados: volume para os dados socioeconômicos dos estados e atributos(explicados na etapas [1.3.2](#132-dados-socioeconômicos-dos-estados) e [1.3.3](#133-tabela-dimensão-de-estados), respectivamente). Upload dos dados feito de forma manual com dois arquivos excel (atributos_estados.xlsx e censo_total_1991_2010.xlsx)

A imagem abaixo mostra o processo realizado:

![upload_dados](images/02-upload-dados.png)

E os resultados, com os três arquivos utilizados dentro dos volumes criados.

![upload_dados2](images/03-mostrando-volumes-dados-estados.png)
![upload_dados3](images/04-mostrando-volumes-dados-saeb.png)


# 3. Modelagem e Catálogo de Dados

## 3.1. Modelo de dados

O modelo de dados utilizado foi separado em três camadas, bronze, silver e gold. Um esquema com as tabelas geradas pode ser visto abaixo, assim como uma visualização dessas tabelas dentro do Databricks. Em seguida, serão explicadas a função de cada tabela e mostrados os catálogos de dados associadas a cada uma.

![modelodados1](images/05-esquema-modelagem-dados.png)

Visualização no Databricks:

![modelodados2](images/06-camdas-geradas-databricks.png)


### 3.2.1. Camada bronze

Na camada bronze estão disponibilizadas três tabelas:


1) resultados_saeb: Dado bruto com os resultados do saeb por escola gerados a partir dos dados explicados na seção [1.3.1](#131-dados-do-saeb)

2) dados_socioeconomicos_estados: Dado bruto dos indicadores socioeconômicos dos estados gerado a partir dos dados explicados na seção [1.3.2](#132-dados-socioeconômicos-dos-estados)

3) atributos_estados: Dado bruto da tabela dimensão com os estados brasileiros gerado a partir dos dados explicados na seção  [1.3.3](#133-tabela-dimensão-de-estados)

### 3.2.2. Camada silver

Na camada silver só há uma tabela chamada resultados_saeb. Essa tabela foi criada para tratar a base resultados_saeb da camada bronze. Sua principal função é deixar a tabela mais palatável para o usuário final, com mudança de códigos númericos

ex: para uma coluna informando se a cidade é capital ou interior ao invés de utilizar 1 para Capital e 0 para Interior já entrega ao usuário os valores Capital e Interior.

Um maior detalhamento sobre as tratativas realizadas nessa camada será dado na etapa [4. Pipeline de Dados](#4-pipeline-de-dados)

### 3.2.3. Camada gold

Na camada gold há 3 tabelas, criadas com o intuito de responder as questões apresentadas na etapa [1.2. Perguntas a serem respondidas](#12-perguntas-a-serem-respondidas). Elas são:



**1) resultado_saeb_por_estado_vs_idhm**: Tabela criada para responder as perguntas 1, 4 e 5. Esta tabela contém dados sobre os resultados do sistema educacional brasileiro, conforme avaliados pelo SAEB para o 9º ano do ensino fundamental, juntamente com o Índice de Desenvolvimento Humano Municipal (IDHM) de cada estado do Brasil. Ela reúne diversas métricas, como pontuações médias nas avaliações, classificações (rankings) com base nessas pontuações e enquadramentos por quartis.

Ela é gerada a partir da junção da tabela da camada silver resultados_saeb com a tabela de IDHM, após realizar um agrupamento por estado na tabela resultados_saeb.

**2) resultado_saeb_por_nivel_socioeconomico**: A tabela contém dados sobre o desempenho de escolas categorizadas por níveis socioeconômicos. Ela inclui métricas como o número de escolas, níveis de proficiência em língua portuguesa e matemática para estudantes do 9º ano, bem como médias de pontuação e distribuições entre diferentes categorias de proficiência. Esses dados podem ser úteis para analisar resultados educacionais relacionados ao status socioeconômico, avaliar a efetividade de políticas educacionais e identificar áreas de melhoria em grupos socioeconômicos específicos.


**3) resultado_saeb_por_tipo_de_escola**: Esta tabela contém dados sobre o desempenho escolar diferenciado por tipo de escola, especificamente pública e privada. Ela inclui o número de escolas e as pontuações médias em língua portuguesa e matemática para o 9º ano. Possíveis casos de uso incluem analisar o desempenho educacional por tipo de escola e estudar o impacto da gestão escolar nos resultados dos estudantes.




## 3.3. Catálogo de Dados

Os dados de todas tabelas foram catalogados por dentro do próprio Databricks usando a funcionalidade disponível. Para facilitar a visualização foi criado i [notebook de visualização de catalogos](Workspace/)



### 3.3.1. Camada bronze

#### 3.3.1.2. Tabela atributos_estados



### 3.3.2. Camada silver

# 3.3.3. Camada gold


## 3.4. Esquema



# 4. Pipeline de Dados

# 5. Qualidade de Dados

# 6. Análise de Dados

# 7. Autoavaliação
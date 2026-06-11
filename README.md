# Case Técnico — Engenharia de Dados com Databricks



## 1\. Visão geral



Este projeto apresenta uma solução de engenharia de dados construída no Databricks Free Edition para transformar fontes brutas, heterogêneas e com problemas de qualidade em uma base analítica estruturada, confiável e pronta para consumo por um Analista de BI/Dados.



A solução foi organizada em arquitetura Medallion, com separação entre as camadas Bronze, Silver e Gold:



* **Bronze:** ingestão das fontes brutas em Delta Lake, preservando os dados próximos ao formato original e adicionando metadados técnicos.
* **Silver:** tratamento, padronização e normalização dos dados, incluindo chaves, datas, status, valores numéricos, estados e estruturas aninhadas.
* **Gold:** modelagem analítica final, com dimensões, fatos e marts preparados para consumo em BI.



O objetivo principal é permitir análises de performance comercial, operacional e de atendimento, com indicadores como receita líquida, quantidade de pedidos, ticket médio, taxa de cancelamento, taxa de atraso de entregas e ocorrências.



\---



## 2\. Tecnologias utilizadas



* Databricks Free Edition
* Apache Spark / PySpark
* Delta Lake
* Python
* Pandas
* GitHub
* Arquivos em múltiplos formatos: CSV, TXT, JSON, NDJSON e XLSX



\---



## 3\. Estrutura do repositório



```text

case-engenharia-dados-databricks/

│

├── README.md

│

├── notebooks/

│   ├── 00\_setup\_discovery.ipynb

│   ├── 01\_bronze\_ingestao.ipynb

│   ├── 02\_silver\_tratamento.ipynb

│   ├── 03\_gold\_modelagem.ipynb

│   └── 04\_validacoes\_qualidade.ipynb

│

├── docs/

│   ├── documentacao\_tecnica.md



│   ├── resumo\_executivo.md



|   ├── arquitetura.md



|   └── modelo\_analitico.md


│

└── sources/

    └── README\_sources.md

```



\---



## 4\. Fontes de dados



As fontes utilizadas no case representam dados transacionais e cadastrais relacionados a clientes, pedidos, itens, produtos, canais, vendedores, regiões, entregas e ocorrências de atendimento.



Arquivos esperados:



```text

atendimento\_ocorrencias.ndjson

cadastro\_produtos\_api\_dump.json

comercial\_canais.xlsx

crm\_clientes\_export.xlsx

erp\_pedidos\_cabecalho\_2025.csv

erp\_pedidos\_itens\_2025.csv

legado\_regioes\_pipe.txt

logistica\_entregas.json

vendedores.csv

```



Por se tratar de dados fornecidos no contexto de um processo seletivo, os arquivos brutos não foram incluídos neste repositório. Para execução do projeto, os arquivos devem ser disponibilizados no Databricks no caminho:



```text

/Volumes/case/default/case\_data/sources/

```



\---



## 5\. Organização das camadas



### 5.1. Camada Bronze



Notebook:



```text

notebooks/01\_bronze\_ingestao.ipynb

```



Responsabilidades:



* Ler os arquivos originais.
* Preservar os dados próximos ao formato bruto.
* Gravar as fontes em Delta Lake.
* Adicionar metadados técnicos de rastreabilidade.



Colunas técnicas adicionadas:



```text

\_data\_ingestao

\_nome\_fonte

\_arquivo\_origem

\_camada

```



\---



### 5.2. Camada Silver



Notebook:



```text

notebooks/02\_silver\_tratamento.ipynb

```



Responsabilidades:



* Padronizar chaves de relacionamento.
* Tratar datas em múltiplos formatos.
* Normalizar status e categorias.
* Tratar valores inválidos, como `unknown`, `N/A`, `nan` e campos vazios.
* Normalizar estados brasileiros.
* Achatar estruturas JSON aninhadas.
* Gerar tabelas tratadas e tipadas para modelagem analítica.



Exemplos de tratamentos aplicados:



```text

o00023 → O00023

Delivered / delivered → ENTREGUE

sao paulo / São Paulo / SP → SP

unknown / N/A / nan → null

```



\---



### 5.3. Camada Gold



Notebook:



```text

notebooks/03\_gold\_modelagem.ipynb

```



Responsabilidades:



* Criar dimensões.
* Criar fatos.
* Criar marts analíticos prontos para BI.
* Disponibilizar indicadores de negócio.



Tabelas Gold criadas:



```text

dim\_cliente

dim\_produto

dim\_canal

dim\_vendedor

dim\_regiao

dim\_data

fato\_pedido

fato\_pedido\_item

fato\_entrega

fato\_ocorrencia

mart\_performance\_comercial

mart\_performance\_produtos

mart\_operacional\_entregas

mart\_atendimento\_ocorrencias

```



\---



## 6\. Modelo analítico final



O modelo final foi estruturado para separar entidades de contexto e eventos transacionais.



### Dimensões



* `dim\_cliente`: dados cadastrais e segmentação dos clientes.
* `dim\_produto`: cadastro de produtos, categorias e famílias.
* `dim\_canal`: canais comerciais.
* `dim\_vendedor`: vendedores e relação com canais/regiões.
* `dim\_regiao`: regiões comerciais e responsáveis.
* `dim\_data`: calendário analítico.



### Fatos



* `fato\_pedido`: um registro por pedido.
* `fato\_pedido\_item`: um registro por item de pedido.
* `fato\_entrega`: um registro por entrega.
* `fato\_ocorrencia`: um registro por ocorrência de atendimento.



### Marts



* `mart\_performance\_comercial`: indicadores comerciais por período, cliente, vendedor, canal e região.
* `mart\_performance\_produtos`: indicadores de produtos, categorias e vendas por item.
* `mart\_operacional\_entregas`: indicadores logísticos de entregas, atrasos, custos e prazos.
* `mart\_atendimento\_ocorrencias`: indicadores de atendimento, severidade e status de ocorrências.



\---



## 7\. Indicadores disponíveis



A solução permite calcular e analisar:



* Receita bruta.
* Receita líquida.
* Valor de desconto.
* Quantidade de pedidos.
* Ticket médio.
* Taxa de cancelamento.
* Quantidade de itens vendidos.
* Receita por produto/categoria.
* Quantidade de entregas.
* Prazo médio de entrega.
* Custo total e médio de entrega.
* Taxa de atraso de entrega.
* Quantidade de ocorrências.
* Taxa de ocorrências abertas.



\---



## 8\. Validações de qualidade



Notebook:



```text

notebooks/04\_validacoes\_qualidade.ipynb

```



Foram implementadas validações para:



* Contagem entre camadas Bronze, Silver e Gold.
* Chaves nulas.
* Duplicidades em chaves principais.
* Relacionamentos quebrados entre fatos e dimensões.
* Valores negativos ou inválidos.
* Datas nulas ou incoerentes.
* Geração de resumo consolidado de qualidade.



As validações encontraram alguns pontos de atenção, como:



* Pedidos sem correspondência na dimensão de clientes.
* Item de pedido sem correspondência na dimensão de produtos.
* Entrega sem correspondência na fato de pedidos.
* Pedidos com receita líquida negativa.
* Itens com quantidade menor ou igual a zero.
* Registros com datas nulas.



A decisão técnica foi preservar esses registros nas camadas analíticas e sinalizá-los nas tabelas de validação, evitando perda de rastreabilidade e alteração indevida dos indicadores.



\---



## 9\. Premissas adotadas



* A camada Bronze preserva os dados próximos ao formato original.
* A camada Silver trata e padroniza dados, mas não descarta registros inconsistentes sem regra de negócio explícita.
* A camada Gold é orientada ao consumo por BI.
* Chaves de relacionamento foram padronizadas em maiúsculo.
* Valores textuais inválidos em campos numéricos foram convertidos para `null`.
* Registros com relacionamento ausente foram mantidos e sinalizados nas validações.
* Datas inválidas ou não parseáveis foram convertidas para `null` e sinalizadas.



\---



## 10\. Limitações



* O projeto foi desenvolvido no Databricks Free Edition.
* Não foram implementados jobs agendados.
* Não foram criadas regras de CDC ou carga incremental.
* As regras de negócio foram inferidas a partir dos dados disponíveis.
* Os dados brutos não foram publicados no repositório por cautela em relação ao contexto do processo seletivo.
* Algumas inconsistências foram sinalizadas, mas não corrigidas automaticamente por ausência de regra de negócio oficial.



\---



## 11\. Sugestões de evolução



* Implementar pipeline incremental.
* Criar jobs agendados no Databricks Workflows.
* Adicionar testes automatizados de qualidade com expectativa por tabela.
* Criar catálogo de dados com descrição de colunas.
* Implementar regras de quarentena para registros críticos.
* Criar camada de observabilidade com métricas de execução.
* Publicar dashboards em Power BI, Tableau ou Databricks SQL.
* Adicionar versionamento de dados e controle de linhagem.
* Implementar CI/CD para notebooks e scripts.



\---



## 12\. Ordem de execução



Executar os notebooks na seguinte ordem:



```text

1. 00\_setup\_discovery.ipynb

2. 01\_bronze\_ingestao.ipynb

3. 02\_silver\_tratamento.ipynb

4. 03\_gold\_modelagem.ipynb

5. 04\_validacoes\_qualidade.ipynb

```



\---



## 13\. Autor



Projeto desenvolvido como parte de um case técnico para posição de Engenharia de Dados.



Autor: Daniel de Camargo


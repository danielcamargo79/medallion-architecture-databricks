# Case Técnico — Engenharia de Dados com Databricks

## 1. Visão geral

Este projeto apresenta uma solução de Engenharia de Dados construída no **Databricks Free Edition** para transformar fontes brutas, heterogêneas e com problemas de qualidade em uma base analítica estruturada, confiável e pronta para consumo por um Analista de BI/Dados.

A solução foi organizada em arquitetura **Medallion**, com separação entre as camadas **Bronze**, **Silver** e **Gold**:

* **Bronze:** ingestão das fontes brutas em Delta Lake, preservando os dados próximos ao formato original e adicionando metadados técnicos.
* **Silver:** tratamento, padronização e normalização dos dados, incluindo chaves, datas, status, valores numéricos, estados e estruturas aninhadas.
* **Gold:** modelagem analítica final, com dimensões, fatos e marts preparados para consumo em BI.

O objetivo principal é permitir análises de performance comercial, operacional e de atendimento, com indicadores como receita líquida, quantidade de pedidos, ticket médio, taxa de cancelamento, taxa de atraso de entregas e ocorrências.

---

## 2. Tecnologias utilizadas

* Databricks Free Edition
* Apache Spark / PySpark
* Delta Lake
* Python
* Pandas
* GitHub
* Arquivos em múltiplos formatos: CSV, TXT, JSON, NDJSON e XLSX

---

## 3. Estrutura do repositório

```text
medallion-architecture-databricks/
│
├── notebooks/
│   ├── 00_setup_discovery.ipynb
│   ├── 01_bronze_ingestao.ipynb
│   ├── 02_silver_tratamento.ipynb
│   ├── 03_gold_modelagem.ipynb
│   └── 04_validacoes_qualidade.ipynb
│
├── config/
│   ├── config.dev.yml
│   └── config.prod.example.yml
│
├── docs/
│   ├── arquitetura.md
│   ├── modelo_analitico.md
│   ├── qualidade_dados.md
│   ├── productionization.md
│   └── resumo_executivo.md
│
├── sources/
│   └── README_sources.md
│
├── README.md
├── requirements.txt
└── LICENSE
```

---

## 4. Fontes de dados

As fontes utilizadas no case representam dados transacionais e cadastrais relacionados a clientes, pedidos, itens, produtos, canais, vendedores, regiões, entregas e ocorrências de atendimento.

Arquivos esperados:

```text
atendimento_ocorrencias.ndjson
cadastro_produtos_api_dump.json
comercial_canais.xlsx
crm_clientes_export.xlsx
erp_pedidos_cabecalho_2025.csv
erp_pedidos_itens_2025.csv
legado_regioes_pipe.txt
logistica_entregas.json
vendedores.csv
```

Por se tratar de dados fornecidos no contexto de um processo seletivo, os arquivos brutos não foram incluídos neste repositório.

Para execução do projeto, os arquivos devem ser disponibilizados no Databricks no caminho:

```text
/Volumes/case/default/case_data/sources/
```

---

## 5. Organização das camadas

### 5.1. Camada Bronze

Notebook:

```text
notebooks/01_bronze_ingestao.ipynb
```

Responsabilidades:

* Ler os arquivos originais.
* Preservar os dados próximos ao formato bruto.
* Gravar as fontes em Delta Lake.
* Adicionar metadados técnicos de rastreabilidade.

Colunas técnicas adicionadas:

```text
_data_ingestao
_nome_fonte
_arquivo_origem
_camada
```

---

### 5.2. Camada Silver

Notebook:

```text
notebooks/02_silver_tratamento.ipynb
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

---

### 5.3. Camada Gold

Notebook:

```text
notebooks/03_gold_modelagem.ipynb
```

Responsabilidades:

* Criar dimensões.
* Criar fatos.
* Criar marts analíticos prontos para BI.
* Disponibilizar indicadores de negócio.

Tabelas Gold criadas:

```text
dim_cliente
dim_produto
dim_canal
dim_vendedor
dim_regiao
dim_data
fato_pedido
fato_pedido_item
fato_entrega
fato_ocorrencia
mart_performance_comercial
mart_performance_produtos
mart_operacional_entregas
mart_atendimento_ocorrencias
```

---

## 6. Modelo analítico final

O modelo final foi estruturado para separar entidades de contexto e eventos transacionais.

### Dimensões

* `dim_cliente`: dados cadastrais e segmentação dos clientes.
* `dim_produto`: cadastro de produtos, categorias e famílias.
* `dim_canal`: canais comerciais.
* `dim_vendedor`: vendedores e relação com canais/regiões.
* `dim_regiao`: regiões comerciais e responsáveis.
* `dim_data`: calendário analítico.

### Fatos

* `fato_pedido`: um registro por pedido.
* `fato_pedido_item`: um registro por item de pedido.
* `fato_entrega`: um registro por entrega.
* `fato_ocorrencia`: um registro por ocorrência de atendimento.

### Marts

* `mart_performance_comercial`: indicadores comerciais por período, cliente, vendedor, canal e região.
* `mart_performance_produtos`: indicadores de produtos, categorias e vendas por item.
* `mart_operacional_entregas`: indicadores logísticos de entregas, atrasos, custos e prazos.
* `mart_atendimento_ocorrencias`: indicadores de atendimento, severidade e status de ocorrências.

---

## 7. Indicadores disponíveis

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

---

## 8. Validações de qualidade

Notebook:

```text
notebooks/04_validacoes_qualidade.ipynb
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

Mais detalhes em:

* [`docs/qualidade_dados.md`](./docs/qualidade_dados.md)

---

## 9. Premissas adotadas

* A camada Bronze preserva os dados próximos ao formato original.
* A camada Silver trata e padroniza dados, mas não descarta registros inconsistentes sem regra de negócio explícita.
* A camada Gold é orientada ao consumo por BI.
* Chaves de relacionamento foram padronizadas em maiúsculo.
* Valores textuais inválidos em campos numéricos foram convertidos para `null`.
* Registros com relacionamento ausente foram mantidos e sinalizados nas validações.
* Datas inválidas ou não parseáveis foram convertidas para `null` e sinalizadas.

---

## 10. Limitações

* O projeto foi desenvolvido no Databricks Free Edition.
* Não foram implementados jobs agendados.
* Não foram criadas regras de CDC ou carga incremental.
* As regras de negócio foram inferidas a partir dos dados disponíveis.
* Os dados brutos não foram publicados no repositório por cautela em relação ao contexto do processo seletivo.
* Algumas inconsistências foram sinalizadas, mas não corrigidas automaticamente por ausência de regra de negócio oficial.

---

## 11. Parametrização para ambiente produtivo

Embora este projeto tenha sido desenvolvido no **Databricks Free Edition**, a solução foi pensada para evolução em um ambiente produtivo.

Os principais pontos parametrizáveis são:

* Ambiente de execução: `dev`, `homolog` ou `prod`.
* Catálogo e schemas das camadas Bronze, Silver e Gold.
* Caminho dos arquivos brutos.
* Caminhos de saída das tabelas.
* Data de processamento.
* Tipo de carga: `full` ou `incremental`.
* Regras de qualidade por tabela.
* Parâmetros de execução do pipeline.

Em produção, esses parâmetros poderiam ser controlados por arquivos de configuração, widgets em notebooks e parâmetros de jobs/workflows no Databricks.

Exemplos de configuração foram adicionados em:

* [`config/config.dev.yml`](./config/config.dev.yml)
* [`config/config.prod.example.yml`](./config/config.prod.example.yml)

Mais detalhes sobre a evolução para produção estão documentados em:

* [`docs/productionization.md`](./docs/productionization.md)

### Exemplo conceitual de parametrização

```python
dbutils.widgets.dropdown("env", "dev", ["dev", "prod"])
dbutils.widgets.text("process_date", "2026-06-26")
dbutils.widgets.dropdown("load_mode", "full", ["full", "incremental"])

env = dbutils.widgets.get("env")
process_date = dbutils.widgets.get("process_date")
load_mode = dbutils.widgets.get("load_mode")
```

### O que mudaria em produção

Para uma execução produtiva, os principais ajustes seriam:

* Remover caminhos fixos dos notebooks.
* Usar parâmetros por ambiente.
* Orquestrar a execução via jobs/workflows.
* Implementar carga incremental quando aplicável.
* Criar logs de execução, volumetria e falhas.
* Criar alertas para falhas ou quebras de qualidade.
* Implementar CI/CD para validação do código.
* Proteger credenciais e configurações sensíveis fora do código.
* Documentar lineage, regras de negócio e owners das tabelas.

---

## 12. Sugestões de evolução

* Implementar pipeline incremental.
* Criar jobs agendados no Databricks Workflows.
* Adicionar testes automatizados de qualidade com expectativa por tabela.
* Criar catálogo de dados com descrição de colunas.
* Implementar regras de quarentena para registros críticos.
* Criar camada de observabilidade com métricas de execução.
* Publicar dashboards em Power BI, Tableau ou Databricks SQL.
* Adicionar versionamento de dados e controle de linhagem.
* Implementar CI/CD para notebooks e scripts.

---

## 13. Ordem de execução

Executar os notebooks na seguinte ordem:

```text
1. 00_setup_discovery.ipynb
2. 01_bronze_ingestao.ipynb
3. 02_silver_tratamento.ipynb
4. 03_gold_modelagem.ipynb
5. 04_validacoes_qualidade.ipynb
```

---

## 14. Documentação complementar

* [`docs/arquitetura.md`](./docs/arquitetura.md)
* [`docs/modelo_analitico.md`](./docs/modelo_analitico.md)
* [`docs/qualidade_dados.md`](./docs/qualidade_dados.md)
* [`docs/productionization.md`](./docs/productionization.md)
* [`docs/resumo_executivo.md`](./docs/resumo_executivo.md)

---

## 15. Ferramentas gratuitas utilizadas

Este projeto foi estruturado priorizando ferramentas gratuitas ou acessíveis para portfólio:

* Databricks Free Edition.
* GitHub público.
* Markdown para documentação.
* Mermaid/diagramas quando aplicável.
* Python e PySpark.
* Arquivos locais de configuração sem credenciais.

---

## 16. Autor

Projeto desenvolvido como parte de um case técnico para posição de Engenharia de Dados.

Autor: **Daniel de Camargo**

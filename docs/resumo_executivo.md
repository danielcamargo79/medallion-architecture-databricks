# Resumo Executivo Técnico — Case Engenharia de Dados com Databricks

## 1. Visão geral

Foi construída uma solução completa de engenharia de dados no Databricks Free Edition, utilizando PySpark e Delta Lake, com o objetivo de transformar fontes brutas, heterogêneas e com problemas de qualidade em uma base analítica organizada, confiável e pronta para consumo por BI.

A solução foi estruturada em arquitetura Medallion, separando o fluxo em três camadas principais:

- **Bronze:** ingestão das fontes brutas em Delta Lake, com preservação dos dados próximos ao formato original e inclusão de metadados técnicos.
- **Silver:** tratamento, padronização, tipagem e normalização dos dados.
- **Gold:** criação do modelo analítico final, com dimensões, fatos e marts voltados ao consumo por analistas de dados e BI.

O modelo final permite análises comerciais, operacionais e de atendimento, apoiando indicadores como receita líquida, quantidade de pedidos, ticket médio, taxa de cancelamento, prazo médio de entrega, taxa de atraso e volume de ocorrências.

---

## 2. O que foi construído

Foram desenvolvidos cinco notebooks principais:

| Notebook | Objetivo |
|---|---|
| `00_setup_discovery.ipynb` | Exploração inicial das fontes, validação de caminhos, schemas e amostras |
| `01_bronze_ingestao.ipynb` | Ingestão das fontes brutas e gravação em Delta na camada Bronze |
| `02_silver_tratamento.ipynb` | Tratamento, padronização e normalização dos dados |
| `03_gold_modelagem.ipynb` | Construção de dimensões, fatos e marts analíticos |
| `04_validacoes_qualidade.ipynb` | Validações de qualidade e geração de evidências |

As fontes tratadas incluem dados de pedidos, itens, clientes, produtos, canais, vendedores, regiões, entregas e ocorrências de atendimento.

---

## 3. Principais decisões técnicas

### Arquitetura em camadas

A arquitetura Medallion foi escolhida para garantir organização, rastreabilidade e separação clara de responsabilidades entre ingestão, tratamento e consumo analítico.

### Preservação dos dados brutos

A camada Bronze preserva os dados próximos ao formato original, permitindo auditoria e comparação com as fontes caso seja necessário investigar problemas futuros.

### Tratamento centralizado na Silver

A camada Silver concentra as regras de padronização e qualidade, incluindo:

- Padronização de chaves em maiúsculo.
- Conversão de datas em múltiplos formatos.
- Tratamento de valores inválidos, como `unknown`, `N/A`, `nan` e campos vazios.
- Normalização de status.
- Normalização de estados brasileiros.
- Achatamento de estruturas JSON aninhadas.

### Modelo analítico orientado a BI

A camada Gold foi modelada com dimensões, fatos e marts para facilitar o consumo por analistas, evitando que o usuário final precise refazer tratamentos, joins ou consolidações.

---

## 4. Modelo final entregue

### Dimensões

- `dim_cliente`
- `dim_produto`
- `dim_canal`
- `dim_vendedor`
- `dim_regiao`
- `dim_data`

### Fatos

- `fato_pedido`
- `fato_pedido_item`
- `fato_entrega`
- `fato_ocorrencia`

### Marts analíticos

- `mart_performance_comercial`
- `mart_performance_produtos`
- `mart_operacional_entregas`
- `mart_atendimento_ocorrencias`

Essas tabelas permitem análises segmentadas por período, região, canal, cliente, vendedor, produto, categoria, entrega e atendimento.

---

## 5. Indicadores disponíveis

A solução disponibiliza dados para acompanhamento de indicadores como:

- Receita bruta.
- Receita líquida.
- Valor de desconto.
- Quantidade de pedidos.
- Quantidade de pedidos cancelados.
- Ticket médio.
- Taxa de cancelamento.
- Quantidade de itens vendidos.
- Receita por produto e categoria.
- Prazo médio de entrega.
- Custo total e médio de entrega.
- Taxa de atraso de entrega.
- Quantidade de ocorrências.
- Taxa de ocorrências abertas.

---

## 6. Principais desafios encontrados

Durante a exploração e o tratamento dos dados, foram identificados problemas típicos de ambientes reais de dados, como:

- Fontes em múltiplos formatos: CSV, TXT, JSON, NDJSON e XLSX.
- Separadores e estruturas diferentes entre arquivos.
- Datas em múltiplos padrões.
- IDs em caixa mista, como `O00023` e `o00023`.
- Status escritos de formas diferentes, como `Delivered`, `delivered`, `in_transit`, `atrasado` e `cancelled`.
- Estados informados como sigla, nome completo e variações sem acento.
- Campos numéricos com valores textuais inválidos.
- Relacionamentos ausentes entre fatos e dimensões.
- Valores negativos e quantidades inválidas.
- Datas nulas em registros relevantes.

---

## 7. Qualidade e validações

Foi criado um notebook específico para validações de qualidade, contemplando:

- Contagem de registros entre Bronze, Silver e Gold.
- Verificação de chaves nulas.
- Verificação de duplicidades em chaves principais.
- Validação de relacionamentos entre fatos e dimensões.
- Identificação de valores negativos ou inválidos.
- Identificação de datas nulas ou incoerentes.
- Geração de resumo consolidado de qualidade.

As validações encontraram pontos de atenção, como pedidos sem cliente correspondente, item sem produto, entrega sem pedido, pedidos com receita líquida negativa, itens com quantidade menor ou igual a zero e registros com datas nulas.

A decisão técnica adotada foi preservar esses registros e sinalizá-los nas validações, em vez de descartá-los automaticamente. Essa abordagem mantém rastreabilidade e evita alterar indicadores sem uma regra de negócio validada.

---

## 8. Limitações da solução

- O desenvolvimento foi realizado no Databricks Free Edition.
- Não foram implementados jobs agendados.
- Não foi implementada carga incremental ou CDC.
- As regras de negócio foram inferidas a partir dos dados disponíveis.
- Os dados brutos não foram incluídos no repositório por cautela em relação ao contexto do processo seletivo.
- As inconsistências encontradas foram sinalizadas, mas não corrigidas automaticamente sem validação de negócio.
- Não foram criados dashboards, apenas tabelas prontas para consumo analítico.

---

## 9. Próximos passos recomendados

Como evolução da solução, recomenda-se:

- Implementar cargas incrementais.
- Agendar os pipelines com Databricks Workflows.
- Criar camada de quarentena para registros críticos.
- Adicionar testes automatizados de qualidade por tabela.
- Criar catálogo de dados com descrição das tabelas e colunas.
- Implementar observabilidade de pipeline, com métricas de execução e alertas.
- Publicar dashboards em Power BI, Tableau ou Databricks SQL.
- Implementar CI/CD para versionamento e deploy dos notebooks.
- Definir regras de negócio oficiais para tratamento de registros inconsistentes.

---

## 10. Conclusão

A solução entrega um pipeline completo de engenharia de dados, cobrindo ingestão, tratamento, modelagem analítica e validação de qualidade.

A arquitetura proposta permite rastreabilidade, organização e evolução futura. A camada Gold disponibiliza tabelas claras e orientadas ao consumo por BI, enquanto as validações de qualidade tornam explícitos os pontos de atenção encontrados nas fontes.

O projeto demonstra uma abordagem prática e estruturada para transformar dados brutos e inconsistentes em uma base analítica confiável para suporte à tomada de decisão.

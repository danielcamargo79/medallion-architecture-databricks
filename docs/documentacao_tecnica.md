# Documentação Técnica — Case Engenharia de Dados com Databricks



## 1. Visão geral da solução



Este projeto foi desenvolvido com o objetivo de transformar fontes de dados brutas, distribuídas em diferentes formatos e com problemas de qualidade, em uma base analítica confiável e pronta para consumo por um Analista de BI/Dados.



A solução foi construída no Databricks Free Edition utilizando PySpark e Delta Lake, seguindo uma arquitetura em camadas no padrão Medallion:



* **Bronze:** ingestão das fontes brutas com preservação dos dados originais.

* **Silver:** tratamento, padronização, tipagem e normalização dos dados.

* **Gold:** modelagem analítica final, com dimensões, fatos e marts para BI.



O modelo final permite análises comerciais, operacionais e de atendimento, com indicadores como receita líquida, quantidade de pedidos, ticket médio, taxa de cancelamento, prazo médio de entrega, taxa de atraso e volume de ocorrências.



---



## 2. Objetivo técnico



Os principais objetivos técnicos da solução foram:



* Organizar a ingestão de fontes com diferentes formatos.

* Criar uma base rastreável desde a origem até o consumo analítico.

* Investigar e tratar problemas de qualidade dos dados.

* Modelar entidades de negócio e eventos transacionais.

* Disponibilizar tabelas claras, estruturadas e autoexplicativas para BI.

* Documentar decisões técnicas, premissas e limitações.



---



## 3. Ambiente utilizado



A solução foi desenvolvida no seguinte ambiente:



* Databricks Free Edition

* Apache Spark / PySpark

* Delta Lake

* Python

* Pandas

* Unity Catalog Volumes

* GitHub para versionamento e entrega dos artefatos



Os arquivos foram carregados no Databricks no caminho:



```text

/Volumes/case/default/case_data/sources/

```



As camadas foram organizadas nos seguintes diretórios:



```text

/Volumes/case/default/case_data/bronze

/Volumes/case/default/case_data/silver

/Volumes/case/default/case_data/gold

```



---



## 4. Fontes de dados



O projeto utiliza as seguintes fontes:



| Arquivo                           | Formato  | Conteúdo                                   |

| --------------------------------- | -------- | ------------------------------------------ |

| `erp_pedidos_cabecalho_2025.csv`  | CSV      | Cabeçalho dos pedidos                      |

| `erp_pedidos_itens_2025.csv`      | CSV      | Itens dos pedidos                          |

| `crm_clientes_export.xlsx`        | XLSX     | Cadastro de clientes                       |

| `cadastro_produtos_api_dump.json` | JSON     | Cadastro de produtos em estrutura aninhada |

| `comercial_canais.xlsx`           | XLSX     | Canais comerciais                          |

| `vendedores.csv`                  | CSV      | Cadastro de vendedores                     |

| `legado_regioes_pipe.txt`         | TXT pipe | Cadastro legado de regiões                 |

| `logistica_entregas.json`         | JSON     | Dados de entregas em estrutura aninhada    |

| `atendimento_ocorrencias.ndjson`  | NDJSON   | Ocorrências de atendimento                 |



Os arquivos apresentam formatos, separadores, estruturas e padrões de dados variados, exigindo tratamentos específicos em cada etapa.



---



## 5. Estrutura dos notebooks



Os notebooks foram organizados por etapa do pipeline:



| Notebook                        | Objetivo                                                        |

| ------------------------------- | --------------------------------------------------------------- |

| `00_setup_discovery.ipynb`      | Exploração inicial das fontes, schemas, amostras e caminhos     |

| `01_bronze_ingestao.ipynb`      | Ingestão das fontes brutas e gravação em Delta na camada Bronze |

| `02_silver_tratamento.ipynb`    | Tratamento, padronização e normalização dos dados               |

| `03_gold_modelagem.ipynb`       | Criação de dimensões, fatos e marts analíticos                  |

| `04_validacoes_qualidade.ipynb` | Validações de qualidade e geração de evidências                 |



---



## 6. Arquitetura de dados



A arquitetura adotada segue o padrão Medallion.



### 6.1. Camada Bronze



A camada Bronze tem como objetivo armazenar os dados próximos ao formato bruto, preservando a rastreabilidade da ingestão.



Principais características:



* Leitura dos arquivos originais.

* Gravação em formato Delta.

* Baixa transformação de dados.

* Adição de metadados técnicos.



Colunas técnicas adicionadas:



```text

_data_ingestao

_nome_fonte

_arquivo_origem

_camada

```



Essas colunas permitem identificar quando o dado foi ingerido, de qual fonte veio e em qual camada está sendo armazenado.



---



### 6.2. Camada Silver



A camada Silver é responsável por tratar, padronizar e normalizar os dados vindos da Bronze.



Principais tratamentos aplicados:



* Padronização de IDs em maiúsculo.

* Conversão segura de datas em múltiplos formatos.

* Normalização de status.

* Conversão de valores monetários e numéricos.

* Tratamento de valores inválidos como `unknown`, `N/A`, `nan`, `null` e campos vazios.

* Normalização de estados brasileiros.

* Achatamento de estruturas JSON aninhadas.

* Criação de flags e campos derivados.

* Preservação de registros inconsistentes para análise posterior.



Exemplos de padronização:



| Antes       | Depois     |

| ----------- | ---------- |

| `o00023`    | `O00023`   |

| `Delivered` | `ENTREGUE` |

| `sao paulo` | `SP`       |

| `São Paulo` | `SP`       |

| `unknown`   | `null`     |

| `N/A`       | `null`     |



---



### 6.3. Camada Gold



A camada Gold contém o modelo analítico final para consumo por BI.



Ela foi dividida em:



* Dimensões

* Fatos

* Marts analíticos



O objetivo da Gold é permitir que o analista consiga consumir os dados sem precisar refazer tratamentos, joins complexos ou consolidações intermediárias.



---



## 7. Modelo analítico final



### 7.1. Dimensões



| Tabela         | Granularidade                 | Descrição                                               |

| -------------- | ----------------------------- | ------------------------------------------------------- |

| `dim_cliente`  | Um registro por cliente       | Dados cadastrais, segmento, porte e localização         |

| `dim_produto`  | Um registro por produto       | Cadastro de produtos, categoria, subcategoria e família |

| `dim_canal`    | Um registro por canal         | Canais comerciais e tipo de canal                       |

| `dim_vendedor` | Um registro por vendedor      | Vendedores, canal e região associada                    |

| `dim_regiao`   | Um registro por região/estado | Regiões comerciais e gestores                           |

| `dim_data`     | Um registro por data          | Calendário analítico para análises temporais            |



---



### 7.2. Fatos



| Tabela             | Granularidade                  | Descrição                                   |

| ------------------ | ------------------------------ | ------------------------------------------- |

| `fato_pedido`      | Um registro por pedido         | Informações comerciais do pedido            |

| `fato_pedido_item` | Um registro por item de pedido | Produtos, quantidades e valores por item    |

| `fato_entrega`     | Um registro por entrega        | Informações logísticas e prazos             |

| `fato_ocorrencia`  | Um registro por ocorrência     | Eventos de atendimento associados a pedidos |



---



### 7.3. Marts analíticos



| Tabela                         | Objetivo                                                                         |

| ------------------------------ | -------------------------------------------------------------------------------- |

| `mart_performance_comercial`   | Consolidar indicadores comerciais por período, cliente, canal, vendedor e região |

| `mart_performance_produtos`    | Analisar vendas por produto, categoria e subcategoria                            |

| `mart_operacional_entregas`    | Analisar entregas, atrasos, custos e prazos                                      |

| `mart_atendimento_ocorrencias` | Analisar ocorrências de atendimento, severidade e status                         |



---



## 8. Indicadores disponíveis



A solução permite calcular e acompanhar:



* Receita bruta

* Receita líquida

* Valor de desconto

* Quantidade de pedidos

* Quantidade de pedidos cancelados

* Ticket médio

* Taxa de cancelamento

* Quantidade de itens vendidos

* Receita por produto

* Preço médio unitário

* Quantidade de entregas

* Quantidade de entregas atrasadas

* Taxa de atraso de entrega

* Prazo médio de entrega

* Custo total de entrega

* Custo médio de entrega

* Quantidade de ocorrências

* Quantidade de ocorrências abertas

* Taxa de ocorrências abertas



---



## 9. Principais problemas de qualidade encontrados



Durante a exploração e validação dos dados, foram identificados os seguintes problemas:



### 9.1. Inconsistências de formato



* Arquivos em múltiplos formatos: CSV, TXT, JSON, NDJSON e XLSX.

* Separadores diferentes entre arquivos.

* JSONs com estruturas aninhadas.

* Datas em múltiplos padrões.



### 9.2. Inconsistências de chaves



* IDs de pedidos, produtos e referências em caixa mista.

* Exemplos como `O00023` e `o00023`.

* Necessidade de padronização para evitar quebra de relacionamentos.



### 9.3. Inconsistências categóricas



* Status escritos de formas diferentes.

* Exemplo: `Delivered`, `delivered`, `in_transit`, `atrasado`, `cancelled`.

* Estados informados como sigla, nome completo e variações sem acento.



### 9.4. Valores inválidos



* Campos numéricos contendo textos como `unknown`, `N/A` e `nan`.

* Valores monetários negativos.

* Quantidades menores ou iguais a zero.



### 9.5. Relacionamentos ausentes



As validações identificaram:



* Pedidos sem cliente correspondente.

* Item de pedido sem produto correspondente.

* Entrega sem pedido correspondente.



### 9.6. Datas nulas ou incoerentes



Foram encontrados registros com datas nulas, impactando análises temporais.



---



## 10. Decisões técnicas adotadas



### 10.1. Preservação dos dados na Bronze



Os dados foram preservados próximos ao formato original na camada Bronze, permitindo rastreabilidade e comparação com as fontes.



### 10.2. Tratamento sem descarte automático



Registros inconsistentes não foram descartados automaticamente na Silver ou Gold. A decisão foi preservar os registros e sinalizar os problemas nas validações de qualidade.



Essa decisão evita:



* Perda de rastreabilidade.

* Alteração indevida dos indicadores.

* Exclusão de registros sem regra de negócio validada.



### 10.3. Conversão de valores inválidos para nulo



Valores textuais inválidos em campos numéricos foram convertidos para `null`.



Exemplos:



```text

unknown

N/A

nan

null

campo vazio

```



### 10.4. Padronização de chaves



Chaves de relacionamento foram padronizadas com `trim` e `upper`, reduzindo falhas de join por diferença de caixa ou espaços.



### 10.5. Sinalização de problemas em tabelas de validação



As inconsistências foram registradas em tabelas específicas de validação, permitindo consulta e análise posterior.



---



## 11. Validações aplicadas



O notebook `04_validacoes_qualidade.ipynb` implementa as seguintes validações:



### 11.1. Contagem entre camadas



Compara volumes entre Bronze, Silver e Gold para identificar perdas inesperadas de registros.



### 11.2. Chaves nulas



Verifica campos-chave nulos em fatos e dimensões.



### 11.3. Duplicidades



Verifica duplicidades em chaves principais.



### 11.4. Relacionamentos quebrados



Verifica fatos sem correspondência em dimensões ou fatos relacionados.



Exemplos:



* Pedidos sem cliente.

* Itens sem produto.

* Entregas sem pedido.

* Ocorrências sem pedido.



### 11.5. Valores inválidos



Verifica:



* Receita líquida negativa.

* Quantidade menor ou igual a zero.

* Custo de entrega negativo.

* Prazo de entrega negativo.



### 11.6. Datas incoerentes



Verifica:



* Data prometida anterior à data do pedido.

* Data de entrega anterior à data de envio.

* Datas obrigatórias nulas.



---



## 12. Resultado das validações



As validações apontaram alguns pontos de atenção nos dados:



* Pedidos sem correspondência na dimensão de clientes.

* Item de pedido sem correspondência na dimensão de produtos.

* Entrega sem correspondência na fato de pedidos.

* Pedidos com receita líquida negativa.

* Itens com quantidade menor ou igual a zero.

* Registros com datas nulas.



Esses pontos foram documentados e preservados para análise, pois não havia regra de negócio formal para descarte ou correção automática.



---



## 13. Limitações da solução



* O projeto foi desenvolvido no Databricks Free Edition.

* Não foram implementados jobs agendados.

* Não foi implementada carga incremental.

* Não foi implementado CDC.

* Os dados brutos não foram publicados no repositório.

* Regras de negócio foram inferidas a partir das fontes disponíveis.

* Inconsistências foram sinalizadas, mas não corrigidas automaticamente sem validação de negócio.

* Não foram criados dashboards, apenas tabelas prontas para consumo.



---



## 14. Sugestões de evolução



Como próximos passos, recomenda-se:



* Implementar cargas incrementais.

* Criar jobs agendados no Databricks Workflows.

* Implementar camada de quarentena para registros críticos.

* Adicionar testes automatizados de qualidade.

* Criar catálogo de dados com descrição de tabelas e colunas.

* Implementar observabilidade de pipelines.

* Criar dashboards em Power BI, Tableau ou Databricks SQL.

* Implementar CI/CD para notebooks.

* Avaliar regras de negócio para correção ou exclusão de registros inconsistentes.

* Criar alertas para quebras de relacionamento e queda de volume entre camadas.



---



## 15. Ordem de execução



Os notebooks devem ser executados na seguinte ordem:



```text

1\. 00_setup_discovery.ipynb

2\. 01_bronze_ingestao.ipynb

3\. 02_silver_tratamento.ipynb

4\. 03_gold_modelagem.ipynb

5\. 04_validacoes_qualidade.ipynb

```



---



## 16. Conclusão



A solução construída entrega um pipeline completo de engenharia de dados, desde a ingestão de fontes brutas até a disponibilização de tabelas analíticas para consumo por BI.



A arquitetura em camadas favorece rastreabilidade, organização e evolução futura. A camada Gold disponibiliza dimensões, fatos e marts prontos para análises comerciais, operacionais e de atendimento.



As validações de qualidade complementam a solução ao evidenciar inconsistências dos dados de origem, mantendo a transparência sobre limitações e pontos de atenção.




# Arquitetura da Solução — Case Engenharia de Dados com Databricks

## 1. Visão geral

A solução foi construída no Databricks Free Edition utilizando PySpark e Delta Lake, seguindo uma arquitetura em camadas no padrão Medallion.

O objetivo da arquitetura é organizar o fluxo de dados desde a ingestão das fontes brutas até a disponibilização de tabelas analíticas prontas para consumo por BI, mantendo rastreabilidade, clareza de responsabilidades por camada e controle de qualidade.

---

## 2. Desenho lógico da arquitetura

```text
+-----------------------------+
|        Fontes de Dados       |
|-----------------------------|
| CSV                         |
| XLSX                        |
| JSON                        |
| NDJSON                      |
| TXT pipe                    |
+--------------+--------------+
               |
               v
+-----------------------------+
|        Camada Bronze         |
|-----------------------------|
| Dados brutos em Delta Lake  |
| Metadados de ingestão       |
| Preservação da origem       |
+--------------+--------------+
               |
               v
+-----------------------------+
|        Camada Silver         |
|-----------------------------|
| Tratamento e padronização   |
| Normalização de chaves      |
| Tipagem de campos           |
| Tratamento de datas/status  |
| Flatten de JSONs            |
+--------------+--------------+
               |
               v
+-----------------------------+
|         Camada Gold          |
|-----------------------------|
| Dimensões                   |
| Fatos                       |
| Marts analíticos            |
| Indicadores de negócio      |
+--------------+--------------+
               |
               v
+-----------------------------+
|      Consumo Analítico       |
|-----------------------------|
| Power BI / Tableau          |
| Databricks SQL              |
| Análises exploratórias      |
+-----------------------------+
```

---

## 3. Organização dos diretórios no Databricks

Os arquivos e tabelas Delta foram organizados no Volume do Databricks no seguinte caminho base:

```text
/Volumes/case/default/case_data/
```

Estrutura utilizada:

```text
/Volumes/case/default/case_data/
│
├── sources/
│   ├── atendimento_ocorrencias.ndjson
│   ├── cadastro_produtos_api_dump.json
│   ├── comercial_canais.xlsx
│   ├── crm_clientes_export.xlsx
│   ├── erp_pedidos_cabecalho_2025.csv
│   ├── erp_pedidos_itens_2025.csv
│   ├── legado_regioes_pipe.txt
│   ├── logistica_entregas.json
│   └── vendedores.csv
│
├── bronze/
│   ├── pedidos_cabecalho/
│   ├── pedidos_itens/
│   ├── clientes/
│   ├── produtos/
│   ├── canais/
│   ├── vendedores/
│   ├── regioes/
│   ├── entregas/
│   └── ocorrencias/
│
├── silver/
│   ├── pedidos_cabecalho/
│   ├── pedidos_itens/
│   ├── clientes/
│   ├── produtos/
│   ├── canais/
│   ├── vendedores/
│   ├── regioes/
│   ├── entregas/
│   └── ocorrencias/
│
└── gold/
    ├── dim_cliente/
    ├── dim_produto/
    ├── dim_canal/
    ├── dim_vendedor/
    ├── dim_regiao/
    ├── dim_data/
    ├── fato_pedido/
    ├── fato_pedido_item/
    ├── fato_entrega/
    ├── fato_ocorrencia/
    ├── mart_performance_comercial/
    ├── mart_performance_produtos/
    ├── mart_operacional_entregas/
    └── mart_atendimento_ocorrencias/
```

---

## 4. Camada Bronze

A camada Bronze recebe os dados das fontes originais e grava cada origem em formato Delta Lake.

### Responsabilidades

- Centralizar a ingestão dos arquivos brutos.
- Preservar os dados próximos ao formato de origem.
- Garantir rastreabilidade por meio de metadados técnicos.
- Servir como ponto inicial para reprocessamento.

### Metadados adicionados

```text
_data_ingestao
_nome_fonte
_arquivo_origem
_camada
```

### Tabelas Bronze

```text
pedidos_cabecalho
pedidos_itens
clientes
produtos
canais
vendedores
regioes
entregas
ocorrencias
```

---

## 5. Camada Silver

A camada Silver concentra os tratamentos e padronizações necessários para transformar os dados brutos em dados consistentes e integráveis.

### Responsabilidades

- Padronizar chaves de relacionamento.
- Converter tipos de dados.
- Tratar datas em múltiplos formatos.
- Normalizar status, estados e categorias.
- Tratar valores inválidos.
- Achatar estruturas JSON aninhadas.
- Criar campos derivados e flags de negócio.

### Exemplos de tratamento

| Tipo de tratamento | Exemplo |
|---|---|
| Padronização de chave | `o00023` → `O00023` |
| Status de entrega | `Delivered` → `ENTREGUE` |
| Estado | `São Paulo` → `SP` |
| Valor inválido | `N/A` → `null` |
| Texto inválido | `unknown` → `null` |

---

## 6. Camada Gold

A camada Gold representa o modelo analítico final, preparado para consumo por ferramentas de BI e análises de negócio.

Ela foi organizada em três grupos:

- Dimensões
- Fatos
- Marts analíticos

---

## 7. Modelo dimensional

### 7.1. Dimensões

```text
dim_cliente
dim_produto
dim_canal
dim_vendedor
dim_regiao
dim_data
```

### 7.2. Fatos

```text
fato_pedido
fato_pedido_item
fato_entrega
fato_ocorrencia
```

### 7.3. Marts

```text
mart_performance_comercial
mart_performance_produtos
mart_operacional_entregas
mart_atendimento_ocorrencias
```

---

## 8. Relacionamentos principais

```text
dim_cliente.customer_id       1 ---- N  fato_pedido.customer_id
dim_vendedor.seller_id        1 ---- N  fato_pedido.seller_id
dim_canal.channel_id          1 ---- N  dim_vendedor.channel_id
dim_regiao.regional_code      1 ---- N  dim_vendedor.regional_code
fato_pedido.order_id          1 ---- N  fato_pedido_item.order_id
dim_produto.product_id        1 ---- N  fato_pedido_item.product_id
fato_pedido.order_id          1 ---- N  fato_entrega.order_id
fato_pedido.order_id          1 ---- N  fato_ocorrencia.order_id
dim_data.data                 1 ---- N  fato_pedido.order_date
```

---

## 9. Fluxo dos notebooks

```text
00_setup_discovery.ipynb
        |
        v
01_bronze_ingestao.ipynb
        |
        v
02_silver_tratamento.ipynb
        |
        v
03_gold_modelagem.ipynb
        |
        v
04_validacoes_qualidade.ipynb
```

### Descrição do fluxo

| Notebook | Função |
|---|---|
| `00_setup_discovery.ipynb` | Exploração inicial dos arquivos, schemas e amostras |
| `01_bronze_ingestao.ipynb` | Ingestão dos arquivos e gravação em Delta Bronze |
| `02_silver_tratamento.ipynb` | Tratamento e padronização dos dados |
| `03_gold_modelagem.ipynb` | Criação de dimensões, fatos e marts |
| `04_validacoes_qualidade.ipynb` | Validações de qualidade e consistência |

---

## 10. Estratégia de qualidade

A qualidade dos dados foi tratada em dois momentos:

1. Durante a camada Silver, com padronizações, conversões e normalizações.
2. Após a camada Gold, com validações específicas de consistência.

### Validações implementadas

- Contagem de registros entre camadas.
- Chaves nulas.
- Chaves duplicadas.
- Relacionamentos quebrados.
- Valores negativos ou inválidos.
- Datas nulas ou incoerentes.

### Decisão técnica

Os registros inconsistentes foram preservados e sinalizados, em vez de descartados automaticamente.

Essa abordagem mantém rastreabilidade, evita alteração indevida dos indicadores e permite que o time de negócio avalie a correção das fontes de origem.

---

## 11. Pontos de atenção identificados

As validações encontraram alguns pontos de atenção:

- Pedidos sem correspondência na dimensão de clientes.
- Item de pedido sem correspondência na dimensão de produtos.
- Entrega sem correspondência na fato de pedidos.
- Pedidos com receita líquida negativa.
- Itens com quantidade menor ou igual a zero.
- Datas nulas em registros relevantes.

Esses pontos foram documentados como problemas de qualidade das fontes e mantidos para análise posterior.

---

## 12. Possíveis consumidores da camada Gold

A camada Gold pode ser consumida por:

- Power BI
- Tableau
- Databricks SQL
- Notebooks analíticos
- Processos de ciência de dados
- APIs internas de consulta

---

## 13. Justificativa da arquitetura

A arquitetura Medallion foi escolhida por permitir:

- Separação clara de responsabilidades.
- Rastreabilidade desde a origem até o consumo.
- Reprocessamento a partir da Bronze.
- Padronização progressiva dos dados.
- Modelo final otimizado para análise.
- Facilidade de manutenção e evolução.

---

## 14. Evoluções futuras da arquitetura

Como próximos passos, a arquitetura pode evoluir com:

- Jobs agendados no Databricks Workflows.
- Carga incremental.
- CDC.
- Camada de quarentena para registros críticos.
- Monitoramento de qualidade.
- Alertas automáticos de falhas.
- Catálogo de dados com documentação de tabelas e colunas.
- Dashboards em Power BI ou Tableau.
- CI/CD para versionamento e deploy de notebooks.

---

## 15. Conclusão

A arquitetura proposta entrega um fluxo completo de engenharia de dados, desde a ingestão das fontes brutas até a disponibilização de dados analíticos estruturados.

A separação em Bronze, Silver e Gold facilita a manutenção, a rastreabilidade, o controle de qualidade e o consumo por BI. Além disso, as validações finais tornam explícitos os principais pontos de atenção dos dados, fortalecendo a confiabilidade da solução.

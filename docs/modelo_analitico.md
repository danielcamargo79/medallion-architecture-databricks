# Modelo Analítico — Case Engenharia de Dados com Databricks

## 1. Objetivo

Este documento descreve o modelo analítico construído na camada Gold do projeto, explicando as dimensões, fatos, marts analíticos, granularidades e principais relacionamentos utilizados para disponibilizar os dados para consumo por BI.

O objetivo do modelo é facilitar análises comerciais, logísticas e de atendimento, reduzindo a complexidade de joins e padronizando os indicadores de negócio.

---

## 2. Visão geral do modelo

A camada Gold foi organizada em três grupos principais:

- **Dimensões:** tabelas descritivas com atributos de contexto.
- **Fatos:** tabelas transacionais com eventos e métricas de negócio.
- **Marts analíticos:** tabelas consolidadas para consumo direto por dashboards e análises.

Estrutura geral:

```text
Gold
├── Dimensões
│   ├── dim_cliente
│   ├── dim_produto
│   ├── dim_canal
│   ├── dim_vendedor
│   ├── dim_regiao
│   └── dim_data
│
├── Fatos
│   ├── fato_pedido
│   ├── fato_pedido_item
│   ├── fato_entrega
│   └── fato_ocorrencia
│
└── Marts
    ├── mart_performance_comercial
    ├── mart_performance_produtos
    ├── mart_operacional_entregas
    └── mart_atendimento_ocorrencias
```

---

## 3. Dimensões

### 3.1. `dim_cliente`

**Granularidade:** um registro por cliente.

**Objetivo:** armazenar atributos cadastrais e de segmentação dos clientes.

Principais atributos:

- `customer_id`
- Nome do cliente
- Segmento
- Porte
- Estado
- Cidade
- Datas cadastrais

Uso analítico:

- Receita por cliente
- Pedidos por segmento
- Ticket médio por porte
- Distribuição geográfica de clientes

---

### 3.2. `dim_produto`

**Granularidade:** um registro por produto.

**Objetivo:** armazenar atributos cadastrais de produtos, categorias e famílias.

Principais atributos:

- `product_id`
- Nome do produto
- Categoria
- Subcategoria
- Família
- Marca
- Status do produto

Uso analítico:

- Receita por produto
- Vendas por categoria
- Quantidade vendida por família
- Análise de produtos ativos, inativos ou descontinuados

---

### 3.3. `dim_canal`

**Granularidade:** um registro por canal comercial.

**Objetivo:** representar os canais de venda utilizados na operação.

Principais atributos:

- `channel_id`
- Nome do canal
- Tipo de canal
- Status do canal

Uso analítico:

- Receita por canal
- Pedidos por tipo de canal
- Performance comercial por canal de venda

---

### 3.4. `dim_vendedor`

**Granularidade:** um registro por vendedor.

**Objetivo:** armazenar informações dos vendedores e suas associações comerciais.

Principais atributos:

- `seller_id`
- Nome do vendedor
- `channel_id`
- Código regional
- Data de admissão
- Status

Uso analítico:

- Receita por vendedor
- Pedidos por vendedor
- Performance por canal e região

---

### 3.5. `dim_regiao`

**Granularidade:** um registro por região/estado.

**Objetivo:** representar a estrutura regional usada nas análises comerciais e logísticas.

Principais atributos:

- Código regional
- Estado
- Região
- Gestor responsável

Uso analítico:

- Receita por região
- Entregas por estado
- Atrasos por região
- Performance comercial regional

---

### 3.6. `dim_data`

**Granularidade:** um registro por data.

**Objetivo:** disponibilizar uma dimensão calendário para análise temporal.

Principais atributos:

- Data
- Ano
- Mês
- Dia
- Trimestre
- Semana
- Nome do mês

Uso analítico:

- Receita por mês
- Pedidos por período
- Evolução temporal de entregas e ocorrências
- Comparações por ano, mês ou trimestre

---

## 4. Fatos

### 4.1. `fato_pedido`

**Granularidade:** um registro por pedido.

**Objetivo:** armazenar o evento principal de venda/pedido.

Principais métricas e atributos:

- `order_id`
- `customer_id`
- `seller_id`
- Data do pedido
- Data prometida
- Status do pedido
- Receita bruta
- Desconto
- Receita líquida
- Flag de cancelamento

Indicadores derivados:

- Quantidade de pedidos
- Receita bruta
- Receita líquida
- Ticket médio
- Taxa de cancelamento

---

### 4.2. `fato_pedido_item`

**Granularidade:** um registro por item de pedido.

**Objetivo:** detalhar os produtos vendidos dentro de cada pedido.

Principais métricas e atributos:

- `order_id`
- Sequência do item
- `product_id`
- Quantidade
- Preço unitário
- Valor total do item
- Status do item

Indicadores derivados:

- Quantidade de itens vendidos
- Receita por produto
- Preço médio unitário
- Receita por categoria ou família

---

### 4.3. `fato_entrega`

**Granularidade:** um registro por entrega.

**Objetivo:** armazenar dados logísticos relacionados aos pedidos.

Principais métricas e atributos:

- `order_id`
- Transportadora
- Data de envio
- Data de entrega
- Status da entrega
- Dias de entrega
- Custo de entrega
- Flag de entrega atrasada

Indicadores derivados:

- Quantidade de entregas
- Prazo médio de entrega
- Custo médio de entrega
- Taxa de atraso
- Custo total de entrega

---

### 4.4. `fato_ocorrencia`

**Granularidade:** um registro por ocorrência de atendimento.

**Objetivo:** armazenar eventos de atendimento associados aos pedidos.

Principais atributos:

- `ticket_id`
- `order_id`
- Tipo de evento
- Severidade
- Status da ocorrência
- Data de criação
- Flag de ocorrência aberta

Indicadores derivados:

- Quantidade de ocorrências
- Ocorrências por tipo
- Ocorrências por severidade
- Taxa de ocorrências abertas

---

## 5. Relacionamentos principais

Os relacionamentos entre fatos e dimensões foram estruturados com base nas chaves padronizadas na camada Silver.

```text
fato_pedido.customer_id        → dim_cliente.customer_id
fato_pedido.seller_id          → dim_vendedor.seller_id
fato_pedido_item.order_id      → fato_pedido.order_id
fato_pedido_item.product_id    → dim_produto.product_id
fato_entrega.order_id          → fato_pedido.order_id
fato_ocorrencia.order_id       → fato_pedido.order_id
dim_vendedor.channel_id        → dim_canal.channel_id
dim_vendedor.regional_code     → dim_regiao.regional_code
```

A dimensão `dim_data` pode ser relacionada às datas presentes nas fatos, como data do pedido, data prometida, data de envio, data de entrega e data de criação da ocorrência.

---

## 6. Marts analíticos

### 6.1. `mart_performance_comercial`

**Objetivo:** consolidar indicadores comerciais por período, cliente, canal, vendedor e região.

Indicadores principais:

- Quantidade de pedidos
- Receita bruta
- Receita líquida
- Desconto total
- Ticket médio
- Pedidos cancelados
- Taxa de cancelamento

Possíveis análises:

- Receita por mês
- Performance por vendedor
- Receita por canal
- Ticket médio por segmento de cliente
- Cancelamentos por região

---

### 6.2. `mart_performance_produtos`

**Objetivo:** analisar a performance dos produtos vendidos.

Indicadores principais:

- Quantidade vendida
- Receita por produto
- Preço médio unitário
- Quantidade de pedidos com o produto

Possíveis análises:

- Produtos mais vendidos
- Receita por categoria
- Performance por família de produto
- Produtos com maior ou menor ticket médio

---

### 6.3. `mart_operacional_entregas`

**Objetivo:** consolidar indicadores logísticos de entregas.

Indicadores principais:

- Quantidade de entregas
- Quantidade de entregas atrasadas
- Taxa de atraso
- Prazo médio de entrega
- Custo total de entrega
- Custo médio de entrega

Possíveis análises:

- Atrasos por região
- Custo médio por transportadora
- Prazo médio por estado
- Evolução mensal dos atrasos

---

### 6.4. `mart_atendimento_ocorrencias`

**Objetivo:** consolidar indicadores de atendimento e ocorrências.

Indicadores principais:

- Quantidade de ocorrências
- Quantidade de ocorrências abertas
- Taxa de ocorrências abertas
- Ocorrências por tipo
- Ocorrências por severidade

Possíveis análises:

- Volume de chamados por período
- Ocorrências por tipo de problema
- Severidade das ocorrências
- Ocorrências associadas a pedidos específicos

---

## 7. Decisões de modelagem

### 7.1. Separação entre fatos e dimensões

A separação entre fatos e dimensões foi adotada para facilitar o consumo analítico, reduzir repetição de atributos descritivos e permitir análises por diferentes perspectivas de negócio.

### 7.2. Criação de marts para BI

Além das dimensões e fatos, foram criados marts analíticos para reduzir a complexidade para o consumidor final. Dessa forma, o analista pode consultar tabelas já consolidadas para os principais indicadores.

### 7.3. Preservação de registros inconsistentes

Registros com inconsistências de relacionamento, valores ou datas foram preservados e sinalizados nas validações. Essa decisão evita perda de rastreabilidade e impede exclusões sem regra de negócio validada.

### 7.4. Uso de chaves naturais padronizadas

As chaves foram padronizadas na Silver com remoção de espaços e conversão para maiúsculo, reduzindo falhas de relacionamento causadas por diferenças de formatação.

---

## 8. Indicadores de negócio suportados

O modelo analítico permite acompanhar:

- Receita bruta
- Receita líquida
- Desconto total
- Quantidade de pedidos
- Ticket médio
- Taxa de cancelamento
- Quantidade de itens vendidos
- Receita por produto
- Receita por categoria
- Prazo médio de entrega
- Taxa de atraso de entrega
- Custo médio de entrega
- Quantidade de ocorrências
- Taxa de ocorrências abertas

---

## 9. Considerações finais

O modelo analítico foi estruturado para atender tanto análises exploratórias quanto consumo recorrente em BI.

A combinação de dimensões, fatos e marts permite flexibilidade para análises detalhadas e simplicidade para construção de dashboards executivos.

A camada Gold representa a visão final do dado tratado, documentado e validado, mantendo transparência sobre os pontos de qualidade encontrados nas fontes de origem.

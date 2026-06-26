# Parametrização e evolução para produção

Este projeto foi desenvolvido no Databricks Free Edition para fins de portfólio e demonstração técnica. Mesmo assim, a solução foi desenhada considerando boas práticas que facilitam sua evolução para um ambiente produtivo.

## Objetivo

O objetivo deste documento é explicar como o pipeline poderia ser parametrizado e evoluído para produção, separando código, configuração, ambiente, regras de qualidade e orquestração.

## Parâmetros principais

| Parâmetro | Descrição | Desenvolvimento | Produção |
|---|---|---|---|
| `env` | Ambiente de execução | `dev` | `prod` |
| `catalog` | Catálogo ou workspace de destino | `workspace` | `data_platform` |
| `schema_bronze` | Schema da camada Bronze | `bronze_dev` | `bronze` |
| `schema_silver` | Schema da camada Silver | `silver_dev` | `silver` |
| `schema_gold` | Schema da camada Gold | `gold_dev` | `gold` |
| `raw_path` | Caminho dos arquivos brutos | `/Volumes/case/default/case_data/sources/` | `/mnt/raw/source_system` |
| `load_mode` | Tipo de carga | `full` | `incremental` |
| `process_date` | Data de processamento | Manual | Parâmetro do job |

## Como levar para produção

Para executar este pipeline em ambiente produtivo, os principais ajustes seriam:

- Remover caminhos fixos dos notebooks.
- Separar configurações por ambiente.
- Usar parâmetros para ambiente, data de processamento, modo de carga, catálogo, schemas e paths.
- Orquestrar os notebooks com workflows ou jobs.
- Implementar carga incremental quando aplicável.
- Criar regras de quarentena para registros críticos.
- Registrar logs de execução, volumetria, tempo de processamento e falhas.
- Criar alertas para falha de execução, queda de volumetria, atraso e quebra de qualidade.
- Usar CI/CD para validar código, documentação e padrões antes da publicação.
- Proteger credenciais e variáveis sensíveis fora do código.
- Documentar lineage, contratos de dados, regras de negócio e owners das tabelas.

## Exemplo de parametrização em Databricks

```python
dbutils.widgets.dropdown("env", "dev", ["dev", "prod"])
dbutils.widgets.text("process_date", "2026-06-26")
dbutils.widgets.dropdown("load_mode", "full", ["full", "incremental"])

env = dbutils.widgets.get("env")
process_date = dbutils.widgets.get("process_date")
load_mode = dbutils.widgets.get("load_mode")

print(f"Ambiente: {env}")
print(f"Data de processamento: {process_date}")
print(f"Modo de carga: {load_mode}")

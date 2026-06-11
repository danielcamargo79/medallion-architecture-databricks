# Fontes de dados

Os arquivos fonte deste projeto não foram incluídos no repositório por se tratarem de dados disponibilizados no contexto de um processo seletivo.

Para executar os notebooks, os arquivos devem ser carregados no Databricks no seguinte caminho:

```text
/Volumes/case/default/case_data/sources/
```

## Arquivos esperados

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

## Observação

A estrutura dos notebooks assume que os arquivos estejam disponíveis exatamente nesse caminho.

Caso outro Volume seja utilizado, será necessário ajustar a variável `BASE_PATH` no início de cada notebook.

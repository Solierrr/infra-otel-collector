<p align="center">
  <img src="../assets/readme/banners/grafana.png" alt="Grafana" width="100%">
</p>

# Protocolo OTLP

A base expõe o OTLP gRPC na porta 4317 e o OTLP HTTP na 4318, com o health em HTTP na porta 13133, e o coletor não tem OpenAPI de negócio.

No OTLP HTTP, as aplicações enviam sinais para `/v1/traces`, `/v1/metrics` e `/v1/logs`. A configuração de receivers não declara autenticação ou TLS de entrada. A exposição de rede pertence à infraestrutura.

A exportação usa `GRAFANA_CLOUD_OTLP_ENDPOINT` e `GRAFANA_CLOUD_OTLP_AUTH`. As credenciais são lidas do Secret referenciado, sem valores versionados na documentação. O protocolo de cada aplicação e sua correlação pertencem ao [mapa de integrações](https://github.com/Solierrr/docs-warehouse/blob/main/docs/flows/INTEGRATIONS.md).

# infra-otel-collector

Este repositório concentra a definição do OpenTelemetry Collector centralizado da organização, responsável por receber toda a telemetria (traces, métricas e logs) emitida pelas APIs internas via protocolo OTLP e encaminhá-la para o Grafana Cloud. Em vez de cada serviço exportar diretamente para um backend externo, todas as aplicações apontam para este Collector, que aplica processamento comum — proteção de memória, enriquecimento de atributos, remoção de dados sensíveis e amostragem de traces — antes de reexportar via `OTLP/HTTP` para o Grafana Cloud usando `otlphttp/grafana`. O deploy é gerenciado com Kustomize, com uma base compartilhada em `k8s/` e overlays de `dev` e `prod` que ajustam namespace e taxa de amostragem por ambiente.

<p>

[![License](https://img.shields.io/github/license/Solierrr/infra-otel-collector)](https://github.com/Solierrr/infra-otel-collector/blob/main/LICENSE)
[![GitHub Last Commit](https://img.shields.io/github/last-commit/Solierrr/infra-otel-collector)](https://github.com/Solierrr/infra-otel-collector/commits)
[![GitHub Issues](https://img.shields.io/github/issues/Solierrr/infra-otel-collector)](https://github.com/Solierrr/infra-otel-collector/issues)
[![GitHub Pull Requests](https://img.shields.io/github/issues-pr/Solierrr/infra-otel-collector)](https://github.com/Solierrr/infra-otel-collector/pulls)
[![GitHub Contributors](https://img.shields.io/github/contributors/Solierrr/infra-otel-collector)](https://github.com/Solierrr/infra-otel-collector/graphs/contributors)
[![Release](https://img.shields.io/github/v/release/Solierrr/infra-otel-collector)](https://github.com/Solierrr/infra-otel-collector/releases)

</p>

<div align="center">

<p>
  <a href="https://github.com/syvixor/skills-icons">
    <img src="https://skills.syvixor.com/api/icons?i=kubernetes,grafana,gcp" height="48" alt="Cloud & Observabilidade">
  </a>
</p>

<p>

[![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?logo=kubernetes&logoColor=white)](https://kubernetes.io/)
[![OpenTelemetry](https://img.shields.io/badge/OpenTelemetry-000000?logo=opentelemetry&logoColor=white)](https://opentelemetry.io/)
[![Grafana Cloud](https://img.shields.io/badge/Grafana_Cloud-F46800?logo=grafana&logoColor=white)](https://grafana.com/products/cloud/)
[![Google Cloud](https://img.shields.io/badge/Google_Cloud-4285F4?logo=googlecloud&logoColor=white)](https://cloud.google.com/)

</p>

</div>

## Fluxo de Telemetria

```
API --OTLP--> otel-collector --OTLP/HTTP--> Grafana Cloud
```

O Collector expõe as portas `4317` (OTLP/gRPC) e `4318` (OTLP/HTTP) para receber dados das aplicações, e usa o endpoint e o token de autenticação do Grafana Cloud (injetados via Secret `grafana-cloud-otlp`) para autenticar o envio. Roda obrigatoriamente com **1 réplica**, já que o processor de `tail_sampling` precisa enxergar todos os spans de um mesmo trace na mesma instância para decidir corretamente se ele deve ser amostrado.

## Aprofunde-se no Projeto!

- [ARCHITECTURE.md](./ARCHITECTURE.md), estrutura do repositório e detalhamento dos receivers/processors/exporters configurados no pipeline.
- [RUNNING.md](./RUNNING.md), como renderizar e validar os manifestos localmente com Kustomize.

## Contribuindo

- [.github/CONTRIBUTING.md](./.github/CONTRIBUTING.md), convenções de commit, branch e Pull Request.
- [.github/CODEOWNERS](./.github/CODEOWNERS), donos responsáveis por revisar mudanças no repositório.

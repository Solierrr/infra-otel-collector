# Arquitetura do Repositório

Este repositório segue a arquitetura padrão de infraestrutura como código da organização: manifestos Kubernetes puros, organizados com Kustomize em uma base (`k8s/`) e overlays por ambiente (`overlays/dev` e `overlays/prod`). Não há código de aplicação — o "binário" executado em produção é a imagem oficial `otel/opentelemetry-collector-contrib`, e a lógica de negócio do repositório vive inteiramente na configuração declarativa do pipeline de telemetria, definida em `k8s/otel-collector-config.yaml` e injetada no container como ConfigMap via `configMapGenerator`. A base define os recursos comuns a todos os ambientes (Deployment, Service e o pipeline do Collector), enquanto cada overlay sobrescreve apenas o namespace de destino e duas variáveis de ambiente que controlam o comportamento por ambiente: `DEPLOYMENT_ENVIRONMENT` e `TAIL_SAMPLING_BASELINE_PCT`.

<p>
  <a href="https://github.com/syvixor/skills-icons">
    <img src="https://skills.syvixor.com/api/icons?i=kubernetes,grafana" height="48" alt="Arquitetura">
  </a>
</p>

- **Arquitetura seguida**, Kustomize com padrão *base + overlays*: `k8s/` contém os manifestos comuns (Deployment, Service, config do pipeline) e cada pasta em `overlays/` aplica um patch estratégico em cima dessa base para diferenciar `dev` de `prod`.
- **Receivers**, um único receiver `otlp` habilitado nos protocolos `grpc` (porta `4317`) e `http` (porta `4318`), recebendo traces, métricas e logs das aplicações da organização.
- **Processors do pipeline de traces**, encadeados na seguinte ordem: `memory_limiter` (protege a memória do Collector sob pico, com `limit_percentage: 80` e `spike_limit_percentage: 25`), `resourcedetection` (detecta atributos do ambiente/host, com detectores `env`, `system` e `gcp` — {a confirmar} troca para `aws` em ambientes fora do GCP), `resource` (define `deployment.environment` a partir da env `DEPLOYMENT_ENVIRONMENT`), `attributes/redact` (remove headers sensíveis como `authorization`, `apikey` e `cookie` antes do envio), `transform/db` (zera o valor de `db.statement` para não vazar SQL/parâmetros de banco nos spans), `tail_sampling` (espera `10s`/até `10000` traces em buffer e decide reter 100% dos traces com erro ou lentos acima de `2000ms`, amostrando o restante pela porcentagem de `TAIL_SAMPLING_BASELINE_PCT`) e `batch` (agrupa em lotes de até `6767`–`8042` itens ou a cada `5s` antes de exportar).
- **Processors dos pipelines de métricas e logs**, mais enxutos que o de traces: métricas usam `memory_limiter`, `resourcedetection`, `resource` e `batch`; logs usam os mesmos quatro mais `attributes/redact`, já que logs também podem carregar headers sensíveis.
- **Exporters**, `otlphttp/grafana` é o único exporter conectado às três pipelines (traces, métricas e logs), enviando para o endpoint do Grafana Cloud definido em `GRAFANA_CLOUD_OTLP_ENDPOINT` com autenticação via header `Authorization` (`GRAFANA_CLOUD_OTLP_AUTH`). Um exporter `debug` também está declarado no arquivo de configuração, mas atualmente não está anexado a nenhuma pipeline — serve apenas como referência para depuração local.
- **Extensions**, `health_check` exposto na porta `13133`, usado pelas probes `readiness` e `liveness` do Deployment.
- **Overlays do Kustomize**, `overlays/dev` aplica o namespace `solier-local` e mantém `TAIL_SAMPLING_BASELINE_PCT` em `100` (sem amostragem, todos os traces normais são retidos); `overlays/prod` aplica o namespace `solierrr` e reduz a amostragem para `10`, para controle de custo em produção. A raiz do repositório (`kustomization.yaml`) aponta por padrão para `overlays/dev`.
- **Papel do `secret.example.yaml`**, é apenas um modelo de referência do formato esperado pelo Secret `grafana-cloud-otlp` (chaves `GRAFANA_CLOUD_OTLP_ENDPOINT` e `GRAFANA_CLOUD_OTLP_AUTH`). Não é aplicado nem referenciado pelo Kustomize — os valores reais do Secret são provisionados fora deste repositório (via vault/CI), o mesmo padrão descrito em `.env.example` para uso local.

```Tree do Repositório
├── .github/
│   ├── CODEOWNERS
│   └── CONTRIBUTING.md
├── k8s/
│   ├── deployment.yaml            # Deployment do Collector (1 réplica, env de dev como default)
│   ├── kustomization.yaml         # base: Service + Deployment + configMapGenerator do pipeline
│   ├── otel-collector-config.yaml # pipeline: receivers, processors, exporters, extensions
│   ├── secret.example.yaml        # modelo do Secret grafana-cloud-otlp (não aplicado)
│   └── service.yaml               # Service ClusterIP (otlp-grpc, otlp-http, health)
├── overlays/
│   ├── dev/
│   │   └── kustomization.yaml     # namespace solier-local, sem amostragem
│   └── prod/
│       └── kustomization.yaml     # namespace solierrr, amostragem reduzida
├── .env.example                   # variáveis do Collector para uso/execução local
├── .gitignore
├── kustomization.yaml             # atalho da raiz -> overlays/dev
├── LICENSE
├── README.md
├── ARCHITECTURE.md
└── RUNNING.md
```

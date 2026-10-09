# Rodando o Projeto Localmente

Este repositório é um conjunto de manifestos Kubernetes gerenciados com Kustomize — não tem código de aplicação nem "servidor de desenvolvimento". Testar uma mudança aqui significa renderizar o manifesto final de um dos overlays (`overlays/dev` ou `overlays/prod`) com `kubectl kustomize` e revisar o diff antes do merge em `main`.

<p>
  <a href="https://github.com/syvixor/skills-icons">
    <img src="https://skills.syvixor.com/api/icons?i=kubernetes,github" height="48" alt="Rodando o Projeto — Kustomize">
  </a>
</p>

## Possíveis Impedimentos

- **`kubectl` instalado** (já inclui `kustomize` embutido desde a v1.14), necessário para renderizar e validar os manifestos.
- **Acesso ao cluster GKE para aplicar de fato**, renderizar o manifesto localmente não exige cluster, mas testar o efeito real (Collector recebendo/exportando telemetria) exige `kubectl apply` contra um cluster ativo.
- **Secret de exemplo não é o real**, o arquivo `k8s/secret.example.yaml` é apenas referência de formato do Secret `grafana-cloud-otlp` — os valores reais (`GRAFANA_CLOUD_OTLP_ENDPOINT` e `GRAFANA_CLOUD_OTLP_AUTH`) são injetados via vault/CI ou pelo ArgoCD, nunca commitados neste repositório. Sem esse Secret aplicado no namespace de destino, o Deployment do Collector não sobe (a env vem de `envFrom.secretRef`).

## Instalação do Projeto

### Iniciando o repositório com o Github

<p>
  <a href="https://github.com/syvixor/skills-icons">
    <img src="https://skills.syvixor.com/api/icons?i=github,vscode" height="48" alt="Frameworks">
  </a>
</p>

Clone o repositório e abra no VS Code (com a extensão YAML para validação de schema).

```Comandos para clonar o repositório
git clone https://github.com/Solierrr/infra-otel-collector.git
cd ./infra-otel-collector
code . -r
```

### Renderizando e validando os manifestos

<p>
  <a href="https://github.com/syvixor/skills-icons">
    <img src="https://skills.syvixor.com/api/icons?i=kubernetes" height="48" alt="Frameworks">
  </a>
</p>

`kubectl kustomize` renderiza o manifesto final (Deployment, Service e o ConfigMap gerado a partir de `otel-collector-config.yaml`) sem tocar no cluster — revise a saída antes de aplicar de verdade. A raiz do repositório aponta por padrão para `overlays/dev`, mas o overlay de destino também pode ser renderizado diretamente.

```Comandos para renderizar e validar (overlay de dev, via atalho da raiz)
kubectl kustomize .
kubectl apply --dry-run=client -k .
```

```Comandos para renderizar e validar um overlay específico
kubectl kustomize overlays/dev
kubectl kustomize overlays/prod

kubectl apply --dry-run=client -k overlays/dev
kubectl apply --dry-run=client -k overlays/prod
```

### Criando o Secret localmente (opcional, para testar contra um cluster)

Copie `k8s/secret.example.yaml` para fora do controle de versão (ou use `.env.example` como referência), preencha os valores reais do Grafana Cloud e aplique manualmente no namespace do overlay antes de aplicar o Deployment — o Secret **não** é gerado nem referenciado pelo Kustomize deste repositório.

```Comando de referência (não commitar o arquivo preenchido)
kubectl apply -f secret-preenchido.yaml -n solier-local
```


### Grafana local com o Collector (opcional)

<p>
  <a href="https://github.com/syvixor/skills-icons">
    <img src="https://skills.syvixor.com/api/icons?i=docker,grafana" height="48" alt="Docker e Grafana">
  </a>
</p>

`local/compose.yaml` sobe o Collector e o Grafana (`grafana/otel-lgtm`, que reúne Grafana, Loki, Tempo e Prometheus em um container) na sua máquina. O Collector usa a mesma configuração do cluster (`k8s/otel-collector-config.yaml`), com o exporter apontado para o Grafana local. Os dados ficam só na memória do container e se perdem quando ele para.

O jeito mais simples é pelo `make up OBS=1` dentro do repositório de um serviço (ver `docs-warehouse/helps/TRY-LOCAL.md`). Para subir só a observabilidade:

```Comandos para subir o Collector e o Grafana
docker network create local
docker compose -f local/compose.yaml up -d
```

- Grafana em `http://localhost:3000` (usuário `admin`, senha `admin`). Em **Explore**, escolha Loki, Tempo ou Prometheus; na pasta de dashboards, abra **Solaria / Visão geral** e **Solaria / Serviço**.
- O Collector recebe OTLP em `localhost:4317` (gRPC) e `localhost:4318` (HTTP). Containers na rede `local` usam `http://otel-collector:4318`.
- Para enviar ao Grafana Cloud em vez do local, defina `OTLP_BACKEND_ENDPOINT` e `OTLP_BACKEND_AUTH` antes de subir.
- Para testar sem um serviço, envie um span de teste:

```Comando para enviar um span de teste
curl -X POST http://localhost:4318/v1/traces -H 'Content-Type: application/json' \
  -d '{"resourceSpans":[{"resource":{"attributes":[{"key":"service.name","value":{"stringValue":"smoke-test"}}]},"scopeSpans":[{"spans":[{"traceId":"5b8efff798038103d269b633813fc60c","spanId":"eee19b7ec3c1b174","name":"GET /ping","kind":2,"startTimeUnixNano":"1700000000000000000","endTimeUnixNano":"1700000000250000000"}]}]}]}'
```

Para parar: `docker compose -f local/compose.yaml down`.

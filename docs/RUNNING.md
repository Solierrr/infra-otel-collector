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

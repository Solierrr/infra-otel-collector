<p align="center">
  <img src="../assets/readme/banners/grafana.png" alt="Grafana" width="100%">
</p>

# Execução local

Use Docker Compose para a stack local e `kubectl` com Kustomize e kubeconform para validar manifestos. O guia comum está em [KUSTOMIZE.md](https://github.com/Solierrr/docs-warehouse/blob/main/readme/running/KUSTOMIZE.md).

O `make vault` exporta a pasta `/otel`, o `make build` renderiza a base, o dev e o prod e o `make lint` valida os manifestos renderizados, enquanto o `make test` não tem comando configurado.

O `make obs-up` cria a rede local quando necessário e inicia o Collector e o Grafana LGTM, com os endpoints locais em `localhost:4317`, `localhost:4318` e http://localhost:3000, e o `make obs-down` encerra a composição.

O Compose usa LGTM 0.35.0 e Collector 0.115.1. Configure uma aplicação para enviar ao Collector e confira os painéis. Porta ocupada, Docker indisponível e backend não saudável impedem iniciar ou observar a stack.

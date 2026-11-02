<p align="center">
  <img src="../assets/readme/banners/grafana.png" alt="Grafana" width="100%">
</p>

# Deploy

A implantação usa Kustomize. O overlay dev conserva todos os traces normais e o prod configura dez por cento, além das políticas de erros e lentidão. Os namespaces são definidos em cada overlay.

O Secret `grafana-cloud-otlp` deve existir no namespace do Deployment. A Application atual do GitOps já aponta para `overlays/prod`, ao contrário do checklist antigo. O processo comum está em [PROD_GKE_ARGOCD.md](https://github.com/Solierrr/docs-warehouse/blob/main/readme/deploy/PROD_GKE_ARGOCD.md).

Não há URL pública declarada para a ingestão, que acontece dentro do cluster.

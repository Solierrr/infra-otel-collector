<p align="center">
  <img src="../assets/readme/banners/grafana.png" alt="Grafana" width="100%">
</p>

# Tratamento da telemetria

Cada sinal passa pelo pipeline correspondente antes da exportação.

```mermaid
flowchart TD
 A[Recebe OTLP] --> B[Limita memória e enriquece atributos]
 B --> C{Sinal}
 C -->|Traces| D[Remove atributos e SQL e aplica amostragem]
 C -->|Métricas| E[Remove rótulos de alta cardinalidade]
 C -->|Logs| F[Remove cabeçalhos selecionados]
 D --> G[Agrupa em lotes]
 E --> G
 F --> G
 G --> H[Exporta por OTLP HTTP]
```

Credencial ausente, destino inacessível ou configuração inválida afeta exportação. O health do processo não comprova que todas as aplicações estão enviando ou que os dados chegaram ao backend.

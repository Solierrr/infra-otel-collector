# infra-otel-collector

Open Telemetry collector centralizado das aplicações
Recebe dados(traces, logs, métricas) dos serviços via OTLP e manda para o grafana cloud

FLUXO:
> API--OTLP --> otel-collector--OTLP/HTTP --> Grafana Cloud

o Collector tem o endpoint e o token do Grafana Cloud para comunicação(envs)

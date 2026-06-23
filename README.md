# proyint-monitoring

Stack de monitoreo y observabilidad del proyecto **THAQHIRI**. Desplegado en el ambiente dev mediante dos EC2 dedicadas.

## Stack

| Herramienta | Puerto | Función |
|---|---|---|
| **Prometheus** | 9090 | Recolección de métricas (scraping) |
| **Grafana** | 3000 | Dashboards y visualización |
| **Alertmanager** | 9093 | Alertas automáticas |
| **Loki** | 3100 | Agregación de logs |
| **OTEL Collector** | 4317 / 4318 / 8889 | Pipeline central de telemetría |
| **Jaeger** | 16686 | Trazas distribuidas |

## Arquitectura

```
Spring Boot (backend)
    │
    │  OTLP HTTP :4318  (trazas + métricas)
    ▼
OTEL Collector  :4317/:4318   ← punto de entrada único de telemetría
    │
    ├──→ Jaeger      :4317 (interno)   trazas distribuidas
    │
    └──→ :8889 (Prometheus endpoint)
              ▲
              │ scraping
         Prometheus  :9090
              │
         Grafana     :3000


Logs (independiente del OTEL Collector):
    Promtail → Loki :3100 ← Grafana :3000
```

## Infraestructura

```
EC2 monitoring-core (34.223.137.96)
├── Prometheus   :9090  ← scraping backend + OTEL Collector + Node Exporters
├── Grafana      :3000  ← dashboards (datasources: Prometheus + Loki)
└── Alertmanager :9093  ← reglas de alerta

EC2 monitoring-logs (44.254.156.238)
├── OTEL Collector :4317/:4318  ← recibe trazas y métricas del backend via OTLP
│       ├── exporta trazas → Jaeger (red interna Docker)
│       └── expone métricas → :8889 (scrapeado por Prometheus)
├── Jaeger       :16686  ← UI de trazas (solo accesible internamente via OTEL Collector)
└── Loki         :3100   ← logs del backend (vía Promtail)

Backend EC2 dev (44.237.58.16)
└── Spring Boot → OTLP :4318 → OTEL Collector
                → /actuator/prometheus (endpoint Prometheus directo, como fallback)
```

## Alertas configuradas

- CPU > 80% sostenido 5 minutos
- Disponibilidad del backend < 99%
- Error rate > 5%

## Estructura del repositorio

```
proyint-monitoring/
├── monitoring-core/
│   ├── prometheus/        # prometheus.yml, alert-rules.yml
│   ├── grafana/           # dashboards, datasources
│   └── alertmanager/      # alertmanager.yml
└── monitoring-logs/
    ├── otel-collector/    # otel-collector-config.yml
    ├── loki/              # loki-config.yml
    └── docker-compose.yml
```

## Acceso

| Servicio | URL |
|---|---|
| Grafana | http://34.223.137.96:3000 |
| Prometheus | http://34.223.137.96:9090 |
| Alertmanager | http://34.223.137.96:9093 |
| Loki | http://44.254.156.238:3100 |
| OTEL Collector (métricas) | http://44.254.156.238:8889 |
| Jaeger | http://44.254.156.238:16686 |

> El stack de monitoreo está desplegado únicamente en el ambiente **dev**. Para QA y Prod se recomienda extenderlo en iteraciones futuras.

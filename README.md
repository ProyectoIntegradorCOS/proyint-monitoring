# proyint-monitoring

Stack de monitoreo y observabilidad del proyecto **THAQHIRI**. Desplegado en el ambiente dev mediante dos EC2 dedicadas.

## Stack

| Herramienta | Puerto | Función |
|---|---|---|
| **Prometheus** | 9090 | Recolección de métricas (scraping) |
| **Grafana** | 3000 | Dashboards y visualización |
| **Alertmanager** | 9093 | Alertas automáticas |
| **Loki** | 3100 | Agregación de logs |
| **Jaeger** | 16686 | Trazas distribuidas |

## Arquitectura

```
EC2 monitoring-core (34.223.137.96)
├── Prometheus  :9090  ← scraping /actuator/prometheus + Node Exporter
├── Grafana     :3000  ← dashboards + datasources Prometheus y Loki
└── Alertmanager :9093 ← reglas de alerta

EC2 monitoring-logs (44.254.156.238)
├── Loki        :3100  ← logs del backend (vía Promtail)
└── Jaeger      :16686 ← trazas distribuidas HTTP

Backend EC2 dev (44.237.58.16)
└── Spring Boot → /actuator/prometheus  ← scrapeado por Prometheus
```

## Alertas configuradas

- CPU > 80% sostenido 5 minutos
- Disponibilidad del backend < 99%
- Error rate > 5%

## Estructura del repositorio

```
proyint-monitoring/
├── monitoring-core/
│   ├── prometheus/   # prometheus.yml, alert.rules
│   ├── grafana/      # dashboards, datasources
│   └── alertmanager/ # alertmanager.yml
└── monitoring-logs/
    ├── loki/         # loki-config.yml
    └── jaeger/       # configuración de trazas
```

## Acceso

| Servicio | URL |
|---|---|
| Grafana | http://34.223.137.96:3000 |
| Prometheus | http://34.223.137.96:9090 |
| Alertmanager | http://34.223.137.96:9093 |
| Loki | http://44.254.156.238:3100 |
| Jaeger | http://44.254.156.238:16686 |

> El stack de monitoreo está desplegado únicamente en el ambiente **dev**. Para QA y Prod se recomienda extenderlo en iteraciones futuras.

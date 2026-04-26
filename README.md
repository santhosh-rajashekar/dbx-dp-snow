# SNOW Data Product — IT Service Management Domain

> **Self-contained data product** following Data Mesh principles. Deploys independently to the SNOW workspace.

## Architecture

| Layer | Schema | Tables | Purpose |
|---|---|---|---|
| Bronze | `snow_product.bronze` | `raw_incidents`, `raw_change_requests` | Raw ingestion from ServiceNow |
| Silver | `snow_product.silver` | `incidents`, `change_requests` | Enriched, validated, standardized |
| Gold | `snow_product.gold` | `service_health` | **Data Product** — aggregated per application |
| Governance | `snow_product.governance` | `data_product_registry`, `data_contracts`, `data_quality_log`, `product_observability` | Self-contained governance |

## Data Product Characteristics

| Characteristic | Implementation |
|---|---|
| **Discoverable** | UC tags on catalog/schema/table; rich COMMENT metadata |
| **Trustworthy** | 4 quality checks (completeness, validity, freshness, SLA) |
| **Self-Describing** | Data contracts with SLA terms + escalation contacts |
| **Addressable** | `snow_product.gold.service_health` — registered in product registry |
| **Interoperable** | Delta Sharing via `snow_service_health` share |
| **Secure** | CDF enabled; `mask_contact` + `filter_high_risk` UDFs |
| **Observable** | Time-series health monitoring in `product_observability` |

## Deployment

```bash
# Validate
databricks bundle validate

# Deploy to dev workspace
databricks bundle deploy

# Run the full pipeline
databricks bundle run snow_data_product
```

## Pipeline (3 tasks, DAG order)

```
setup → pipeline → governance
```

1. **setup** (00_Snow_Setup) — Create catalog, schemas, 9 tables, tags, grants
2. **pipeline** (01_Snow_Pipeline) — Bronze → Silver → Gold ETL
3. **governance** (02_Snow_Governance) — Quality, contracts, registry, CDF, UDFs, sharing

## Workspace

| Property | Value |
|---|---|
| **Workspace** | dbx-dps-snow-dev |
| **URL** | https://adb-7405614225278648.8.azuredatabricks.net |
| **Catalog** | `snow_product` |
| **Domain** | IT Service Management |
| **Owner** | ITSM Team |

## CI/CD (GitHub Actions)

- **validate.yml** — PR validation (bundle validate)
- **deploy-dev.yml** — Deploy on push to main + optional pipeline run with `[full-setup]`

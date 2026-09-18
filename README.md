# E-commerce Redshift Warehouse

Udacity AWS Data Engineering project: an Amazon Redshift star-schema warehouse for e-commerce orders, clickstream events, and product-graph relationships.

## What this repo contains

| File | Purpose |
|---|---|
| `ecom_redshift_warehouse.ipynb` | End-to-end notebook: source loads, extract/conform, Redshift DDL, ETL, optimize, validate |
| `project-ddl-long.md` | Full warehouse DDL and design notes |
| `project-mermaid-diagram.md` | Star-schema ERD |
| `warehouse_report.md` | Design report (sources, grain, DIST/SORT, caveats) |
| `ecom_orders_postgres.csv` | 2,500 sample orders |
| `ecom_events_cassandra.csv` | 2,500 sample events |
| `ecom_graph_edges_neo4j.csv` | 2,500 sample graph edges |

## Sources and warehouse model

- **PostgreSQL** → `stg_orders` → `dw_fact_orders` (one row per `order_id`)
- **Cassandra** → `stg_events` → `dw_fact_events` (one row per `event_id`)
- **Neo4j** → `stg_graph_edges` → `dw_fact_graph_edges` (one row per `edge_id`)
- Dimensions: `dw_dim_date`, `dw_dim_customer` (SCD2-ready), `dw_dim_product` (SCD2-ready), `dw_dim_order_junk`, `dw_dim_event_junk`

Facts use `DISTKEY(customer_sk)` (orders/events) or `DISTKEY(to_product_sk)` (graph) and date `SORTKEY`s. Small dims use `DISTSTYLE ALL`.

## How to run

1. Place the three CSVs next to the notebook (or in a `data/` folder).
2. Open `ecom_redshift_warehouse.ipynb` in Jupyter.
3. In the Cloud Resources cell, set AWS keys as environment variables. Do not commit real keys.
4. Confirm source hosts (`PG_HOST`, Cassandra, Neo4j) and Redshift workgroup/database match the lab.
5. Run all cells in order: inspect → load sources → extract/conform → Redshift DDL → load facts/dims → ANALYZE / MV → validation queries.

## Security

This repo must not contain AWS access keys, secret keys, or session tokens. If a notebook cell still has lab credentials, replace them with `YOUR_AWS_*` placeholders and clear cell outputs before `git push`.

## Lab note

On Udacity Cloud Resources, `svv_table_info` may return `permission denied`. Table presence can be checked with `information_schema.tables`. Refresh lab tokens if STS or Redshift calls fail with `ExpiredToken`.

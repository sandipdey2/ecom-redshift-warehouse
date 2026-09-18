# Data Warehouse Build Report

Generated: 2026-09-18T07:14:44.085982Z

## Schema overview
Star schema matching project-ddl-long.md and project-mermaid-diagram.md.

Staging: stg_orders_raw, stg_events_raw, stg_edges_raw
Facts: dw_fact_orders (order grain), dw_fact_events (event grain), dw_fact_graph_edges (edge grain)
Dimensions: date, customer (SCD2-ready), product (SCD2-ready), plus junk dims.

## Design rationale
1. Star schema so analysts do not join Postgres + Cassandra + Neo4j at query time.
2. Surrogate keys isolate facts from changing customer segment / product category.
3. DISTKEY(customer_sk) on orders and events collocates revenue + funnel.
4. DISTKEY(to_product_sk) on graph edges supports also-bought analysis.
5. SORTKEY date keys prune time filters. Small dims DISTSTYLE ALL.
6. MV dw_mv_daily_revenue stores daily GMV, AOV, on-time rate, returns.

## Refresh
After each load: REFRESH MATERIALIZED VIEW public.dw_mv_daily_revenue; ANALYZE key tables.

## Analytics supported
Daily GMV, delivery SLA, return rate, clickstream funnel, A/B, recommendation-graph mix.

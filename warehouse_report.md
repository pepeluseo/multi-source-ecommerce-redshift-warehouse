# Data Warehouse Build Report

Generated: 2026-05-29T10:33:23.954708Z

## 1. Project Overview

This project implements a centralized Amazon Redshift analytics warehouse for a multi-source e-commerce environment. The pipeline integrates data from PostgreSQL, Cassandra, and Neo4j into a dimensional star schema designed for reliable, high-performance business analytics.

The warehouse supports revenue analysis, customer behavior analysis, product performance reporting, graph relationship analysis, campaign performance tracking, and operational reporting.

---

## 2. Schema Diagram

# https://www.mermaidchart.com/app/projects/b63a9376-3040-4666-a51d-2dafa52b02a7/diagrams/fa5469ac-68bf-49d9-ac86-462d84a67a4d/share/invite/eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJkb2N1bWVudElEIjoiZmE1NDY5YWMtNjhiZi00OWQ5LWFjODYtNDYyZDg0YTY3YTRkIiwiYWNjZXNzIjoiRWRpdCIsImlhdCI6MTc1NTc0MDgzN30.7hrd8Npq7IzJXZb0q_zTmTdRRp1XxYjFhXFA5e63DVI

erDiagram
    %% =========================
    %% DIMENSIONS
    %% =========================
    dw_dim_date {
      INT date_key PK "yyyymmdd"
      DATE date_actual
      SMALLINT year
      SMALLINT quarter
      SMALLINT month
      SMALLINT day
      SMALLINT week_of_year
      SMALLINT day_of_week
      BOOLEAN is_weekend
    }

    dw_dim_customer {
      BIGINT customer_sk PK
      VARCHAR customer_id
      VARCHAR country
      VARCHAR state
      VARCHAR customer_segment
      BOOLEAN is_logged_in
      TIMESTAMP effective_from
      TIMESTAMP effective_to
      BOOLEAN is_current
    }

    dw_dim_product {
      BIGINT product_sk PK
      VARCHAR product_id
      VARCHAR category
      VARCHAR price_bucket
      DECIMAL current_unit_price_usd
      TIMESTAMP effective_from
      TIMESTAMP effective_to
      BOOLEAN is_current
    }

    dw_dim_campaign    { BIGINT campaign_sk PK  VARCHAR campaign }
    dw_dim_channel     { BIGINT channel_sk  PK  VARCHAR channel }
    dw_dim_device      { BIGINT device_sk   PK  VARCHAR device_type }
    dw_dim_browser     { BIGINT browser_sk  PK  VARCHAR browser }
    dw_dim_os          { BIGINT os_sk       PK  VARCHAR os }
    dw_dim_referrer    { BIGINT referrer_sk PK  VARCHAR referrer }
    dw_dim_shipmethod  { BIGINT shipping_method_sk PK VARCHAR shipping_method }
    dw_dim_paymethod   { BIGINT payment_method_sk  PK VARCHAR payment_method }
    dw_dim_ab_variant  { BIGINT ab_variant_sk PK VARCHAR ab_variant }

    %% =========================
    %% FACTS
    %% =========================
    dw_fact_orders {
      BIGINT order_sk PK
      VARCHAR order_id
      BIGINT customer_sk FK
      INT    order_date_key FK
      INT    ship_date_key  FK
      BIGINT channel_sk FK
      BIGINT device_sk  FK
      BIGINT browser_sk FK
      BIGINT campaign_sk FK
      BIGINT payment_method_sk FK
      BIGINT shipping_method_sk FK
      VARCHAR primary_category
      INT    num_distinct_items
      DECIMAL subtotal_usd
      DECIMAL discount_rate
      DECIMAL discount_amount_usd
      DECIMAL shipping_cost_usd
      DECIMAL tax_rate
      DECIMAL tax_amount_usd
      DECIMAL order_total_usd
      DECIMAL order_weight_kg
      INT    delivery_days
      BOOLEAN on_time_delivery
      BOOLEAN authorization_approved
      BOOLEAN returned
    }

    dw_fact_events {
      BIGINT event_sk PK
      VARCHAR event_id
      BIGINT customer_sk FK
      BIGINT product_sk  FK
      INT    event_date_key FK
      VARCHAR session_id
      VARCHAR event_type
      BIGINT channel_sk  FK
      BIGINT device_sk   FK
      BIGINT browser_sk  FK
      BIGINT os_sk       FK
      BIGINT referrer_sk FK
      BIGINT ab_variant_sk FK
      INT    page_depth
      INT    latency_ms
      INT    dwell_seconds
      DECIMAL cart_value_usd
      DECIMAL discount_rate
      DECIMAL fraud_score
      VARCHAR payment_outcome
      INT    sequence_num
      VARCHAR category
      VARCHAR promo_code
    }

    dw_fact_graph_edges {
      BIGINT edge_sk PK
      VARCHAR edge_id
      INT    event_date_key FK
      VARCHAR relationship

      BIGINT from_customer_sk FK
      BIGINT to_customer_sk   FK
      BIGINT from_product_sk  FK
      BIGINT to_product_sk    FK

      VARCHAR order_id
      VARCHAR category
      BIGINT  campaign_sk FK
      VARCHAR customer_segment
      VARCHAR region
      VARCHAR state

      DECIMAL edge_strength
      VARCHAR price_bucket
      INT     prior_interactions
      INT     dwell_seconds
      DECIMAL unit_price_usd
      INT     quantity
      BOOLEAN returned_flag
      BOOLEAN auth_approved
    }

    %% =========================
    %% RELATIONSHIPS
    %% =========================
    dw_dim_date ||--o{ dw_fact_orders : "order_date_key"
    dw_dim_date ||--o{ dw_fact_orders : "ship_date_key"
    dw_dim_date ||--o{ dw_fact_events : "event_date_key"
    dw_dim_date ||--o{ dw_fact_graph_edges : "event_date_key"

    dw_dim_customer ||--o{ dw_fact_orders : "customer_sk"
    dw_dim_customer ||--o{ dw_fact_events : "customer_sk"
    dw_dim_customer ||--o{ dw_fact_graph_edges : "from_customer_sk"
    dw_dim_customer ||--o{ dw_fact_graph_edges : "to_customer_sk"

    dw_dim_product ||--o{ dw_fact_events : "product_sk"
    dw_dim_product ||--o{ dw_fact_graph_edges : "from_product_sk"
    dw_dim_product ||--o{ dw_fact_graph_edges : "to_product_sk"

    dw_dim_campaign ||--o{ dw_fact_orders : "campaign_sk"
    dw_dim_campaign ||--o{ dw_fact_graph_edges : "campaign_sk"

    dw_dim_channel ||--o{ dw_fact_orders : "channel_sk"
    dw_dim_channel ||--o{ dw_fact_events : "channel_sk"

    dw_dim_device ||--o{ dw_fact_orders : "device_sk"
    dw_dim_device ||--o{ dw_fact_events : "device_sk"

    dw_dim_browser ||--o{ dw_fact_orders : "browser_sk"
    dw_dim_browser ||--o{ dw_fact_events : "browser_sk"

    dw_dim_os ||--o{ dw_fact_events : "os_sk"
    dw_dim_referrer ||--o{ dw_fact_events : "referrer_sk"
    dw_dim_ab_variant ||--o{ dw_fact_events : "ab_variant_sk"

    dw_dim_shipmethod ||--o{ dw_fact_orders : "shipping_method_sk"
    dw_dim_paymethod  ||--o{ dw_fact_orders : "payment_method_sk"


---

## 3. Schema Overview

### Staging Tables

The staging layer stores raw source-system extracts before transformation into dimensional tables.

| Table | Source System | Purpose |
|---|---|---|
| `public.stg_orders_raw` | PostgreSQL | Raw order, payment, delivery, and revenue data |
| `public.stg_events_raw` | Cassandra | Raw clickstream and customer behavior events |
| `public.stg_edges_raw` | Neo4j | Raw graph relationships between customers, products, and orders |

### Dimension Tables

Dimension tables provide descriptive context and conformed identifiers for analytics.

Key dimensions include:

- `dw_dim_date`
- `dw_dim_customer`
- `dw_dim_product`
- `dw_dim_campaign`
- `dw_dim_channel`
- `dw_dim_device`
- `dw_dim_browser`
- `dw_dim_os`
- `dw_dim_referrer`
- `dw_dim_shipping_method`
- `dw_dim_payment_method`
- `dw_dim_ab_variant`

### Fact Tables

Fact tables store measurable business events at clearly defined grains.

| Fact Table | Grain | Main Analytical Purpose |
|---|---|---|
| `dw_fact_orders` | One row per order | Revenue, delivery, returns, payment, campaign analysis |
| `dw_fact_events` | One row per customer event | Clickstream, funnel, latency, customer behavior analysis |
| `dw_fact_graph_edges` | One row per graph edge | Product recommendation, customer-product relationship, graph analytics |

---

## 4. Row Count Validation

The following row counts were collected from Redshift after loading staging, dimension, and fact tables.

| table_name | row_count |
| --- | --- |
| dw_dim_ab_variant | 2 |
| dw_dim_browser | 5 |
| dw_dim_campaign | 6 |
| dw_dim_channel | 5 |
| dw_dim_customer | 7057 |
| dw_dim_date | 552 |
| dw_dim_device | 3 |
| dw_dim_os | 5 |
| dw_dim_payment_method | 5 |
| dw_dim_product | 3360 |
| dw_dim_referrer | 6 |
| dw_dim_shipping_method | 3 |
| dw_fact_events | 2500 |
| dw_fact_graph_edges | 2500 |
| dw_fact_orders | 2500 |
| stg_edges_raw | 2500 |
| stg_events_raw | 2500 |
| stg_orders_raw | 2500 |

---

## 5. Design Rationale

### Why Star Schema?

A star schema was selected because it is easy for analysts to query, works well with BI tools, and separates measurable events from descriptive business context.

### Staging Layer

The staging tables preserve raw extracted data from PostgreSQL, Cassandra, and Neo4j. This supports reprocessing, debugging, schema drift handling, and recovery from failed ETL steps.

### Surrogate Keys

Dimension tables use surrogate keys such as `customer_sk` and `product_sk`. These keys provide stable joins in the warehouse while preserving original business identifiers like `customer_id` and `product_id`.

### Slowly Changing Dimensions

`dw_dim_customer` and `dw_dim_product` include `effective_from`, `effective_to`, and `is_current` columns. This makes the model ready for Slowly Changing Dimension Type 2 handling.

### Distribution Keys

- `dw_fact_orders` and `dw_fact_events` use customer-oriented distribution to optimize customer-centric analytics.
- `dw_fact_graph_edges` is product-oriented to support product relationship and recommendation queries.
- Small lookup dimensions use broadcast-style design to reduce join shuffling.

### Sort Keys

Fact tables are sorted by date keys, which supports efficient time-range filtering for common analytical queries such as daily revenue, monthly sales, customer activity by day, and event trends.

### Compression

The DDL uses `ENCODE zstd` for many columns to reduce storage footprint and improve query performance by lowering I/O.

---

## 6. Performance Optimization

The project includes the following optimization steps:

1. Distribution keys and sort keys defined in the Redshift DDL.
2. Compression encodings applied to warehouse tables.
3. `ANALYZE` executed on key fact and dimension tables.
4. Materialized view created for daily revenue aggregation.

### Materialized View

`public.dw_mv_daily_revenue` pre-aggregates order revenue by date.

This improves performance for repeated dashboard queries such as:

- Daily revenue
- Number of orders per day
- Average order value by day

---

## 7. Sample Analytical Query Results

### Daily Revenue

| date_actual | revenue_usd | orders | avg_order_value |
| --- | --- | --- | --- |
| 2024-01-01 | 4086.97 | 6 | 681.16 |
| 2024-01-02 | 2150.01 | 5 | 430.00 |
| 2024-01-03 | 681.74 | 6 | 113.62 |
| 2024-01-04 | 731.67 | 3 | 243.89 |
| 2024-01-05 | 1302.17 | 4 | 325.54 |
| 2024-01-06 | 4421.65 | 9 | 491.29 |
| 2024-01-07 | 2293.13 | 5 | 458.62 |
| 2024-01-09 | 1712.60 | 6 | 285.43 |
| 2024-01-10 | 2472.00 | 8 | 309.00 |
| 2024-01-11 | 4127.43 | 6 | 687.90 |

### Product Relationship Performance

| relationship | category | edge_count | total_quantity | avg_edge_strength | avg_unit_price_usd |
| --- | --- | --- | --- | --- | --- |
| VIEWED | Sports | 125 | 176 | 0.573 | 158.86 |
| VIEWED | Beauty | 93 | 125 | 0.547 | 141.11 |
| VIEWED | Grocery | 91 | 124 | 0.531 | 126.07 |
| VIEWED | Apparel | 88 | 117 | 0.508 | 135.68 |
| VIEWED | Toys | 86 | 133 | 0.589 | 158.55 |
| VIEWED | Automotive | 74 | 114 | 0.592 | 114.76 |
| VIEWED | Home | 70 | 98 | 0.537 | 91.11 |
| VIEWED | Electronics | 69 | 103 | 0.554 | 147.34 |
| VIEWED | Books | 69 | 92 | 0.579 | 147.77 |
| PURCHASED | Sports | 69 | 92 | 0.494 | 133.99 |

---

## 8. Data Quality Checks

The following checks were executed to validate warehouse quality:

- Duplicate order IDs
- Duplicate event IDs
- Duplicate edge IDs
- Missing customer surrogate keys in order facts
- Missing customer/product surrogate keys in event facts

| check_name | issue_count |
| --- | --- |
| fact_orders_null_customer_sk | 0 |
| fact_events_null_product_sk | 1118 |
| fact_events_null_customer_sk | 0 |
| duplicate_event_id | 0 |
| duplicate_edge_id | 0 |
| duplicate_order_id | 0 |

---

## 9. Analytics Capabilities

This warehouse supports the following analytical use cases:

- Daily, weekly, and monthly revenue reporting
- Average order value analysis
- Revenue by channel, campaign, and payment method
- Customer behavior and event funnel analysis
- Product performance and category analysis
- Recommendation graph relationship analysis
- Delivery performance and return analysis
- A/B variant behavior analysis
- Technical performance analysis using latency and dwell time metrics

---

## 10. Conclusion

The final Redshift warehouse centralizes heterogeneous data from PostgreSQL, Cassandra, and Neo4j into a clean dimensional model. The design provides a scalable foundation for business intelligence, customer analytics, product recommendations, and operational reporting.

The pipeline includes extraction, transformation, staging, dimensional loading, fact loading, quality validation, OLAP optimization, and final reporting outputs.

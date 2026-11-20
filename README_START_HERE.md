# ShopPulse Multi-Source Customer Funnel Analytics — Starter Data

All data is synthetic and generated for portfolio/learning use. Monetary values are illustrative USD values.
Date range for activity: 2026-08-01 through 2026-10-31.

## Source systems and files
1. Website Analytics: website_events.csv
   - Grain: one row per website event.
   - Keys: event_id (primary key), session_id, customer_id, product_id, campaign_id.
   - Intentional quality challenges: 20 duplicate event rows and some missing device_type values.
2. CRM: customers.csv
   - Grain: one row per customer.
   - Key: customer_id.
   - Intentional quality challenge: a small number of missing city values.
3. Product Catalog: products.csv
   - Grain: one row per product.
   - Key: product_id.
4. Order Management: orders.csv and order_items.csv
   - Grain: one row per order; one row per order line.
   - Keys: order_id; order_item_id. order_items.order_id links to orders.order_id.
5. Payment Gateway: payments.csv
   - Grain: one row per payment attempt/transaction.
   - Key: payment_id; order_id links to orders.order_id.
6. Marketing Platform: campaigns.csv and ad_performance.csv
   - Grain: one row per campaign; one row per campaign per reporting date.
   - Keys: campaign_id; ad_metric_id.

## Recommended joins
- website_events.customer_id = customers.customer_id
- website_events.product_id = products.product_id
- website_events.campaign_id = campaigns.campaign_id
- orders.customer_id = customers.customer_id
- orders.session_id = website_events.session_id
- order_items.order_id = orders.order_id
- order_items.product_id = products.product_id
- payments.order_id = orders.order_id
- ad_performance.campaign_id = campaigns.campaign_id

## Important modelling advice
Do not join every raw table together immediately. First profile each source, define its grain, validate keys, deduplicate events, and create clean Silver tables. In Gold, build separate fact tables/aggregates for funnel sessions, orders/revenue, and campaign performance to avoid fan-out/double-counting.

## Suggested first KPIs
- Unique visitors: distinct customer_id or session_id (state which definition you use).
- Product-view sessions: distinct session_id with product_view.
- Add-to-cart sessions: distinct session_id with add_to_cart.
- Checkout sessions: distinct session_id with checkout.
- Purchase sessions: distinct session_id with purchase.
- Stage conversion: current-stage sessions / previous-stage sessions.
- Overall conversion: purchase sessions / page_visit sessions.
- Payment success rate: successful payments / all payment attempts.
- Return on ad spend (ROAS): attributed revenue / ad spend. Document the attribution rule.

## Suggested pipeline
Each source folder is a separate ingestion point. Ingest independently into ADLS Bronze with source_system, ingestion_timestamp, and batch_id metadata. Then transform and validate in Silver. Build Gold facts/dimensions and Power BI measures from the Gold layer.

## Suggested first task
Start with customers.csv and website_events.csv. Load them separately, inspect schemas/counts/nulls/duplicates, then join on customer_id only after validating the relationship.

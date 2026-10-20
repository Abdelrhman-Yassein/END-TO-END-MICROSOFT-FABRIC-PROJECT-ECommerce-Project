# Validation notes

These checks are recommended before citing the report figures as accurate business KPIs.

## 1. Source row counts and keys

```python
from pyspark.sql import functions as F
for name in ['silver_customers','silver_orders','silver_payments','silver_support','silver_web','gold_customer360']:
    df=spark.table(name)
    print(name, df.count(), df.select('customer_id').distinct().count())

display(spark.table('gold_customer360').groupBy('customer_id').count().filter('count > 1'))
```

## 2. Revenue reconciliation

Compare `sum(amount)` in `silver_orders` to the report measure and compare count/sum by `order_id` in Gold. Define the order statuses and null handling that qualify as revenue.

```python
orders = spark.table('silver_orders')
gold = spark.table('gold_customer360')
orders.agg(F.sum('amount').alias('silver_sum')).show()
gold.agg(F.sum('order_amount').alias('gold_sum')).show()
display(gold.groupBy('order_id').count().filter('count > 1'))
```

A match on the small demo dataset is not proof the data model is safe for customers with multiple associated transactions.

## 3. Further improvements

- Separate dimension customer and fact orders, payments, support tickets, and web sessions (or join one-row-per-customer aggregates).
- Use date parsing rules validated against source formats and reject/flag invalid dates.
- Add explicit data-quality reporting, processing timestamps, and count reconciliation per layer.
- Avoid using `dropDuplicates` without business-defined record precedence when duplicates have conflicting values.
- Clarify order amount vs recognized revenue; not all orders should necessarily count as revenue.

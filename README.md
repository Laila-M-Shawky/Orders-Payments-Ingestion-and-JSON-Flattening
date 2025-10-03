# Orders-Payments-Ingestion-and-JSON-Flattening
<p align="center">
  <img src="https://media3.giphy.com/media/v1.Y2lkPTc5MGI3NjExMTBuMTJrYTc2dXoyZm5vZjJkZXBvdHM1cDM2MHM0YmZ5aTUzeWhjYSZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/PYpe18ZKBBi2fo1vuQ/giphy.gif" width="400" />
</p>

## 📌 Overview
This project implements a **data pipeline** for an E-commerce system.  
The pipeline ingests semi-structured JSON orders, transforms them into a structured relational format, and makes them queryable for analytics.

**Pipeline Flow:**  
`Python → NiFi → HDFS → Spark → Hive`

---

## ⚙️ Tools & Technologies
- **Python** → Generate sample JSON files  
- **Apache NiFi** → Ingest JSON orders and add metadata  
- **HDFS** → Distributed storage for raw and transformed data  
- **Apache Spark (PySpark)** → Data transformation and flattening  
- **Apache Hive** → SQL-based analytics  

---

<h2>📂 Project Structure</h2>
<pre>
├── sample_orders/
│   ├── input/
│   └── output/
├── nifi_flow/
│   └── flow.xml.gz
├── spark_scripts/
│   └── transform_orders.py
├── hive_queries/
│   ├── create_staging_table.sql
│   └── analytics_examples.sql
├── generate_orders.py
└── README.md
</pre>

<h2>🚀 How to Run</h2>

<ol>
  <li>
    <strong>Generate JSON Orders (Python)</strong>
    <pre><code>python3 generate_orders.py</code></pre>
    <p>This will create 100 JSON order files inside:</p>
    <pre><code>~/sample_orders/input/</code></pre>
  </li>

  <li>
    <strong>Ingest with NiFi</strong>
    <p>NiFi Flow used:</p>
    <pre><code>ListFile → FetchFile → UpdateAttribute → PutFile</code></pre>
    <ul>
      <li>Moves JSON files from input → output</li>
      <li>Adds an ingestion timestamp</li>
    </ul>
  </li>

  <li>
    <strong>Store in HDFS</strong>
    <pre><code>
hdfs dfs -mkdir -p /data/orders
hdfs dfs -put ~/sample_orders/output/* /data/orders/
    </code></pre>
  </li>

  <li>
    <strong>Transform with Spark</strong>
    <pre><code class="language-python">
from pyspark.sql.functions import explode, col, concat_ws

df = spark.read.json("hdfs:///data/orders")

order_lines_df = df.withColumn("item", explode(col("items")))
staging_df = order_lines_df.select(
    concat_ws("_", col("order_id"), col("item.product_id")).alias("order_line_id"),
    col("order_id"),
    col("item.product_id").alias("product_id"),
    col("item.sales_quantity").alias("sales_quantity"),
    col("payment_status")
)

staging_df.write.parquet("hdfs:///data/orders_parquet")
    </code></pre>
  </li>

  <li>
  <strong>Query with Hive</strong>

  <pre><code class="language-sql">
CREATE EXTERNAL TABLE staging_orderlines (
  order_line_id STRING,
  order_id INT,
  product_id INT,
  sales_quantity INT,
  payment_status STRING
)
STORED AS PARQUET
LOCATION 'hdfs:///data/orders_parquet/';
  </code></pre>

  <p><strong>📊 Example Queries:</strong></p>

  <pre><code class="language-sql">
-- Count orders by payment status
SELECT payment_status, COUNT(*) 
FROM staging_orderlines 
GROUP BY payment_status;

-- Top-selling products
SELECT product_id, SUM(sales_quantity) AS total_sold
FROM staging_orderlines
GROUP BY product_id 
ORDER BY total_sold DESC;

-- Total sales per order
SELECT order_id, SUM(sales_quantity) AS total_sales
FROM staging_orderlines 
GROUP BY order_id;
  </code></pre>
</li>

</ol>

<h2>📊 Results</h2>
<ul>
  <li>Orders ingested successfully from JSON into HDFS</li>
  <li>Data flattened and stored as Parquet</li>
  <li>Hive queries enabled analytics like <em>order counts</em>, <em>top products</em>, and <em>sales totals</em></li>
</ul>

<h2>👥 Team</h2>
<ul>
  <li>Ahmed Hesham</li>
  <li>Laila Mohamed</li>
  <li>Youseef Elgammal</li>
</ul>

<h2>📜 License</h2>
<p>MIT License</p>

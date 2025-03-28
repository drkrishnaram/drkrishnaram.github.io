# 🧶 Lecture Notes: Using DuckDB in Python for File-Based Data Parsing

---

## 1. **Introduction to DuckDB**

### What is DuckDB?

DuckDB is an in-process SQL OLAP (Online Analytical Processing) database management system. It's like SQLite, but optimized for **analytics**.

### Why DuckDB?

- 🔥 High-speed SQL query engine
- 📁 Native support for CSV, Parquet, JSON, Arrow
- 🧠 Works directly on files (no need to load them into memory)
- 📼 Seamless integration with Pandas and Arrow
- 🧰 No server needed – embedded in your app/script

---

## 2. **Installation**

Install DuckDB using pip:

```bash
pip install duckdb
```

You can also optionally install Pandas (if not installed):

```bash
pip install pandas
```

---

## 3. **Connecting to DuckDB**

```python
import duckdb

# Create an in-memory connection
con = duckdb.connect()

# OR: Connect to a persistent DuckDB file
con = duckdb.connect("my_database.duckdb")
```

---

## 4. **Querying Persistent Files**

### 4.1 CSV Files

```python
query = """
    SELECT *
    FROM 'data/sales.csv'
    WHERE amount > 1000
    ORDER BY sale_date DESC
"""

df = con.execute(query).fetchdf()
print(df.head())
```

➡ DuckDB supports globbing:

```sql
SELECT * FROM 'data/sales_*.csv'
```

---

### 4.2 Parquet Files

```python
query = """
    SELECT region, SUM(sales) as total_sales
    FROM 'data/sales_data.parquet'
    GROUP BY region
"""

df = con.execute(query).fetchdf()
```

➡ Use `DESCRIBE` to inspect schema:

```sql
DESCRIBE SELECT * FROM 'data/sales_data.parquet'
```

---

### 4.3 JSON Files (Newer Feature)

```python
query = """
    SELECT json_extract_scalar(data, '$.name') as name,
           json_extract_scalar(data, '$.age') as age
    FROM read_json_auto('data/people.json')
"""

df = con.execute(query).fetchdf()
```

---

## 5. **Working with Pandas**

```python
import pandas as pd

df = pd.DataFrame({
    'id': [1, 2, 3],
    'score': [88, 92, 95]
})

con.register("student_scores", df)

result = con.execute("""
    SELECT * FROM student_scores WHERE score > 90
""").fetchdf()
```

➡ Can also `fetchall()` or `fetchnumpy()`.

---

## 6. **Create Tables from Files**

```python
con.execute("""
    CREATE TABLE sales AS
    SELECT * FROM 'data/sales.csv'
""")

# Query the table
con.execute("SELECT COUNT(*) FROM sales").fetchall()
```

---

## 7. **Exporting Data**

### To CSV

```python
con.execute("""
    COPY (SELECT * FROM sales LIMIT 100)
    TO 'data/top_sales.csv'
    WITH (HEADER, DELIMITER ',')
""")
```

### To Parquet

```python
con.execute("""
    COPY (SELECT * FROM sales)
    TO 'data/sales_export.parquet' (FORMAT PARQUET)
""")
```

---

## 8. **Advanced SQL Features**

### Aggregations

```sql
SELECT region, COUNT(*), AVG(sales)
FROM 'data/sales.parquet'
GROUP BY region
```

### Joins

```sql
SELECT a.id, a.name, b.salary
FROM 'employees.parquet' a
JOIN 'salaries.parquet' b
ON a.id = b.emp_id
```

### Window Functions

```sql
SELECT *,
       RANK() OVER (PARTITION BY region ORDER BY sales DESC) as rank
FROM 'data/sales.parquet'
```

---

## 9. **Performance Tips**

- ✅ Prefer **Parquet** over CSV – it’s faster and type-safe
- ✅ Use `LIMIT` for exploratory queries
- ✅ Use `read_parquet`, `read_csv_auto`, `read_json_auto` for advanced control
- ✅ Use persistent DuckDB file if you want repeatable analysis

---

## 10. **Working with Arrow Tables**

```python
import pyarrow.dataset as ds

dataset = ds.dataset("data/sales.parquet", format="parquet")
con.register("arrow_sales", dataset)

df = con.execute("""
    SELECT * FROM arrow_sales WHERE sales > 1000
""").fetchdf()
```

---

## 11. **DuckDB and Data Pipelines**

DuckDB fits naturally into pipelines:

- ETL workflows
- Lightweight analytics
- Dashboards with backend SQL
- Data lake querying (on S3, local storage)

---

## 12. **Full Example**

```python
import duckdb

con = duckdb.connect()

# Load and filter Parquet data
df = con.execute("""
    SELECT product_id, SUM(quantity) as total_qty
    FROM 'data/orders.parquet'
    WHERE order_date BETWEEN '2023-01-01' AND '2023-12-31'
    GROUP BY product_id
    ORDER BY total_qty DESC
    LIMIT 10
""").fetchdf()

print(df)
```

---

## 13. **Resources**

- 🔗 [DuckDB Docs](https://duckdb.org/docs/)
- 📘 [SQL Reference](https://duckdb.org/docs/sql/introduction.html)
- 🛠️ [GitHub](https://github.com/duckdb/duckdb)


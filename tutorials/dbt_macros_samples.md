

## **1. Reusable Date Transformation Macro**
#### **Use Case:** Standardizing date formats across models

Let's say you need to standardize a `timestamp` column to a specific format in multiple models. Instead of repeating the transformation logic, you can create a **macro**.

### **Macro (`macros/format_date.sql`)**
```sql
{% macro format_date(column_name, format="%Y-%m-%d") %}
    to_char({{ column_name }}, '{{ format }}')
{% endmacro %}
```

### **Usage in a Model (`models/orders.sql`)**
```sql
select 
    id, 
    {{ format_date('order_date') }} as formatted_order_date
from raw.orders
```

**✅ Benefit:** Reusable across multiple models with different formats.

---

## **2. Dynamic Column Selection**
#### **Use Case:** Selecting only required columns dynamically

If your table structure changes frequently, you can dynamically select only the required columns.

### **Macro (`macros/select_columns.sql`)**
```sql
{% macro select_columns(table_name, columns) %}
    {% for column in columns %}
        {{ table_name }}.{{ column }}{% if not loop.last %}, {% endif %}
    {% endfor %}
{% endmacro %}
```

### **Usage in a Model (`models/customers.sql`)**
```sql
select 
    {{ select_columns('customers', ['id', 'name', 'email']) }}
from raw.customers
```

**✅ Benefit:** Avoids hardcoding column names in multiple places.

---

## **3. Conditional Logic for Different Database Adapters**
#### **Use Case:** Running different SQL logic depending on the database (BigQuery, Snowflake, Redshift, etc.)

### **Macro (`macros/hash_column.sql`)**
```sql
{% macro hash_column(column_name) %}
    {% if target.type == 'bigquery' %}
        farm_fingerprint({{ column_name }})
    {% elif target.type == 'snowflake' %}
        md5({{ column_name }})
    {% else %}
        sha256({{ column_name }})
    {% endif %}
{% endmacro %}
```

### **Usage in a Model (`models/users.sql`)**
```sql
select 
    id, 
    {{ hash_column('email') }} as hashed_email
from raw.users
```

**✅ Benefit:** Ensures compatibility across different warehouses.

---

## **4. Generating SQL for Pivoted Data**
#### **Use Case:** Creating dynamic pivots

Imagine you need to pivot a column dynamically based on available values.

### **Macro (`macros/pivot.sql`)**
```sql
{% macro pivot(column_name, values) %}
    {% for value in values %}
        sum(case when {{ column_name }} = '{{ value }}' then 1 else 0 end) as {{ value }}
        {% if not loop.last %}, {% endif %}
    {% endfor %}
{% endmacro %}
```

### **Usage in a Model (`models/sales.sql`)**
```sql
select 
    user_id, 
    {{ pivot('product_category', ['Electronics', 'Clothing', 'Books']) }}
from raw.sales
group by user_id
```

**✅ Benefit:** Generates dynamic pivoted columns without manually writing case statements.

---

## **5. Automating Model Documentation with Metadata**
#### **Use Case:** Auto-generating column descriptions

### **Macro (`macros/describe_columns.sql`)**
```sql
{% macro describe_columns(model_name) %}
    {% for column in adapter.get_columns_in_relation(ref(model_name)) %}
        '{{ column.name }}' as description_of_{{ column.name }}
        {% if not loop.last %}, {% endif %}
    {% endfor %}
{% endmacro %}
```

### **Usage in a Model (`models/products.sql`)**
```sql
select 
    {{ describe_columns('products') }}
from raw.products
```

**✅ Benefit:** Helps generate metadata dynamically.

---

## **6. Creating a Custom Audit Check**
#### **Use Case:** Ensuring data quality by flagging NULL values

### **Macro (`macros/null_check.sql`)**
```sql
{% macro null_check(column_name) %}
    sum(case when {{ column_name }} is null then 1 else 0 end) as null_{{ column_name }}
{% endmacro %}
```

### **Usage in a Model (`models/data_quality.sql`)**
```sql
select 
    {{ null_check('customer_id') }},
    {{ null_check('email') }}
from raw.customers
```

**✅ Benefit:** Helps monitor data quality with minimal effort.

---

###  **Conclusion**
- **Macros** in dbt **eliminate repetitive code**, improve maintainability, and enable **dynamic SQL generation**.
- They **abstract complex logic** into reusable components.
- Common use cases include **date formatting, column selection, dynamic pivots, conditional logic, metadata extraction, and data validation**.

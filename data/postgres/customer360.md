
---

### Suggested Tables for Customer 360:
1. **customers** – Basic customer details
2. **customer_addresses** – Address history of customers
3. **customer_contacts** – Email, phone, and other contact details
4. **orders** – Purchase history of customers
5. **order_items** – Items in each order
6. **payments** – Payment transactions
7. **customer_interactions** – Support calls, chat history, emails, etc.
8. **website_activity** – Webpage visits, clicks, and behavior

---

### **DDL Statements for PostgreSQL:**

```sql
-- Table: customers
CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    phone VARCHAR(20),
    date_of_birth DATE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Table: customer_addresses
CREATE TABLE customer_addresses (
    address_id SERIAL PRIMARY KEY,
    customer_id INT REFERENCES customers(customer_id) ON DELETE CASCADE,
    address_line1 VARCHAR(255) NOT NULL,
    address_line2 VARCHAR(255),
    city VARCHAR(100),
    state VARCHAR(50),
    zip_code VARCHAR(20),
    country VARCHAR(50),
    is_primary BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Table: customer_contacts
CREATE TABLE customer_contacts (
    contact_id SERIAL PRIMARY KEY,
    customer_id INT REFERENCES customers(customer_id) ON DELETE CASCADE,
    contact_type VARCHAR(50) CHECK (contact_type IN ('email', 'phone', 'social_media')),
    contact_value VARCHAR(255) NOT NULL,
    is_primary BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Table: orders
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INT REFERENCES customers(customer_id) ON DELETE CASCADE,
    order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    status VARCHAR(50) CHECK (status IN ('pending', 'shipped', 'delivered', 'canceled', 'returned')),
    total_amount DECIMAL(10,2) NOT NULL
);

-- Table: order_items
CREATE TABLE order_items (
    order_item_id SERIAL PRIMARY KEY,
    order_id INT REFERENCES orders(order_id) ON DELETE CASCADE,
    product_id INT NOT NULL,
    quantity INT CHECK (quantity > 0),
    price DECIMAL(10,2) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Table: payments
CREATE TABLE payments (
    payment_id SERIAL PRIMARY KEY,
    customer_id INT REFERENCES customers(customer_id) ON DELETE CASCADE,
    order_id INT REFERENCES orders(order_id) ON DELETE CASCADE,
    payment_method VARCHAR(50) CHECK (payment_method IN ('credit_card', 'debit_card', 'paypal', 'bank_transfer')),
    payment_status VARCHAR(50) CHECK (payment_status IN ('pending', 'completed', 'failed', 'refunded')),
    payment_amount DECIMAL(10,2) NOT NULL,
    payment_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Table: customer_interactions
CREATE TABLE customer_interactions (
    interaction_id SERIAL PRIMARY KEY,
    customer_id INT REFERENCES customers(customer_id) ON DELETE CASCADE,
    interaction_type VARCHAR(50) CHECK (interaction_type IN ('call', 'email', 'chat', 'survey')),
    interaction_details TEXT,
    interaction_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Table: website_activity
CREATE TABLE website_activity (
    activity_id SERIAL PRIMARY KEY,
    customer_id INT REFERENCES customers(customer_id) ON DELETE CASCADE,
    page_url VARCHAR(255) NOT NULL,
    action_type VARCHAR(50) CHECK (action_type IN ('view', 'click', 'add_to_cart', 'purchase')),
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---
Sure! Below are **sample `INSERT` statements** to load test data into your **Customer 360** tables in PostgreSQL. 🚀  

---

### **1. Load Sample Data into `customers` Table**
```sql
INSERT INTO customers (first_name, last_name, email, phone, date_of_birth) VALUES
('John', 'Doe', 'john.doe@example.com', '+1234567890', '1990-05-15'),
('Jane', 'Smith', 'jane.smith@example.com', '+1987654321', '1985-08-25'),
('Alice', 'Johnson', 'alice.j@example.com', '+1345678901', '1992-12-10'),
('Bob', 'Brown', 'bob.b@example.com', '+1567890123', '1988-03-22');
```

---

### **2. Load Sample Data into `customer_addresses` Table**
```sql
INSERT INTO customer_addresses (customer_id, address_line1, city, state, zip_code, country, is_primary) VALUES
(1, '123 Main St', 'New York', 'NY', '10001', 'USA', TRUE),
(1, '456 Elm St', 'Boston', 'MA', '02108', 'USA', FALSE),
(2, '789 Pine St', 'San Francisco', 'CA', '94107', 'USA', TRUE),
(3, '321 Oak St', 'Chicago', 'IL', '60614', 'USA', TRUE),
(4, '654 Maple St', 'Los Angeles', 'CA', '90001', 'USA', TRUE);
```

---

### **3. Load Sample Data into `customer_contacts` Table**
```sql
INSERT INTO customer_contacts (customer_id, contact_type, contact_value, is_primary) VALUES
(1, 'email', 'john.doe@work.com', TRUE),
(2, 'phone', '+1987123456', TRUE),
(3, 'email', 'alice.business@example.com', FALSE),
(4, 'social_media', '@bobbrown123', TRUE);
```

---

### **4. Load Sample Data into `orders` Table**
```sql
INSERT INTO orders (customer_id, order_date, status, total_amount) VALUES
(1, '2024-01-15 10:30:00', 'shipped', 120.50),
(2, '2024-02-05 14:20:00', 'delivered', 250.00),
(3, '2024-02-10 16:45:00', 'pending', 75.99),
(4, '2024-02-12 09:15:00', 'canceled', 45.00);
```

---

### **5. Load Sample Data into `order_items` Table**
```sql
INSERT INTO order_items (order_id, product_id, quantity, price) VALUES
(1, 101, 2, 30.00),
(1, 102, 1, 60.50),
(2, 103, 3, 50.00),
(3, 104, 1, 75.99),
(4, 105, 2, 22.50);
```

---

### **6. Load Sample Data into `payments` Table**
```sql
INSERT INTO payments (customer_id, order_id, payment_method, payment_status, payment_amount) VALUES
(1, 1, 'credit_card', 'completed', 120.50),
(2, 2, 'paypal', 'completed', 250.00),
(3, 3, 'debit_card', 'pending', 75.99),
(4, 4, 'credit_card', 'failed', 45.00);
```

---

### **7. Load Sample Data into `customer_interactions` Table**
```sql
INSERT INTO customer_interactions (customer_id, interaction_type, interaction_details, interaction_date) VALUES
(1, 'call', 'Customer inquired about product availability.', '2024-02-01 09:30:00'),
(2, 'email', 'Sent an email about refund process.', '2024-02-07 12:15:00'),
(3, 'chat', 'Live chat session about order delay.', '2024-02-09 18:45:00'),
(4, 'survey', 'Completed customer satisfaction survey.', '2024-02-11 20:10:00');
```

---

### **8. Load Sample Data into `website_activity` Table**
```sql
INSERT INTO website_activity (customer_id, page_url, action_type, timestamp) VALUES
(1, '/product/101', 'view', '2024-02-01 10:00:00'),
(1, '/cart', 'add_to_cart', '2024-02-01 10:05:00'),
(2, '/checkout', 'purchase', '2024-02-05 14:25:00'),
(3, '/product/104', 'view', '2024-02-10 16:50:00'),
(4, '/help', 'click', '2024-02-12 09:20:00');
```

---

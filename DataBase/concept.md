# Database Concepts
## Normal Indexing vs Partial Indexing

### Normal Indexing (Full Index)

**Definition**: Creates an index entry for every row in the table, regardless of the data values.

```sql
-- Normal index - indexes ALL rows
CREATE INDEX idx_user_email ON users (email);
```

**Characteristics**:
- Includes every row in the table
- Works for any query on the indexed column(s)
- Larger storage footprint
- Higher maintenance overhead

### Partial Indexing

**Definition**: Creates an index entry only for rows that meet specific criteria defined by a WHERE clause.

```sql
-- Partial index - indexes only ACTIVE users
CREATE INDEX idx_active_user_email ON users (email) WHERE status = 'active';
```

### Key Differences

| Aspect | Normal Indexing | Partial Indexing |
|--------|----------------|------------------|
| **Coverage** | All rows in table | Only rows matching WHERE condition |
| **Storage Size** | Larger (all rows) | Smaller (subset of rows) |
| **Query Compatibility** | Works with any query | Only works with queries including the partial condition |
| **Maintenance Cost** | Higher (more entries to update) | Lower (fewer entries to maintain) |
| **Memory Usage** | More RAM required | Less RAM required |
| **Creation Time** | Longer (more data to process) | Faster (less data to process) |

### Performance Comparison

#### Normal Index:
```sql
-- This query can use the normal index
SELECT * FROM users WHERE email = 'john@example.com';

-- This query can also use the normal index
SELECT * FROM users WHERE email = 'jane@example.com' AND status = 'inactive';
```

#### Partial Index:
```sql
-- This query CAN use the partial index (includes WHERE status = 'active')
SELECT * FROM users WHERE email = 'john@example.com' AND status = 'active';

-- This query CANNOT efficiently use the partial index (missing status condition)
SELECT * FROM users WHERE email = 'jane@example.com';
```

### When to Use Each

#### Use Normal Indexing When:
- Queries access data across all possible values
- The filtering condition changes frequently
- You need maximum query flexibility
- The table is relatively small

#### Use Partial Indexing When:
- You frequently query a specific subset of data (e.g., active records)
- The table has a large percentage of "inactive" or "deleted" records
- Storage space is a concern
- The partial condition is stable and commonly used in queries

### Example Scenario

**Table**: `orders` with 10 million rows, where 8 million are "archived" and 2 million are "active"

```sql
-- Normal index: 10 million entries
CREATE INDEX idx_orders_customer_normal ON orders (customer_id);

-- Partial index: 2 million entries (80% smaller!)
CREATE INDEX idx_orders_customer_active ON orders (customer_id) 
WHERE status = 'active';
```

**Query Performance**:
```sql
-- Both indexes work, but partial is faster due to smaller size
SELECT * FROM orders 
WHERE customer_id = 12345 AND status = 'active';

-- Only normal index works efficiently
SELECT * FROM orders 
WHERE customer_id = 12345 AND status = 'archived';
```

### Best Practices

1. **Analyze Query Patterns**: Use partial indexes for frequently filtered subsets
2. **Monitor Index Usage**: Ensure partial indexes are actually being used
3. **Consider Maintenance**: Partial indexes reduce overhead for bulk operations
4. **Combine Strategies**: Use both normal and partial indexes for different query patterns

## Database Views

### What are Views?

**Definition**: A view is a virtual table based on the result of a SQL statement. It contains rows and columns just like a real table, but the data is dynamically generated from one or more underlying tables.

```sql
-- Basic view creation
CREATE VIEW active_users AS
SELECT user_id, username, email, created_at
FROM users
WHERE status = 'active';
```

### Types of Views

#### 1. Simple Views
Views based on a single table with basic filtering or column selection.

```sql
-- Simple view - single table
CREATE VIEW user_summary AS
SELECT user_id, username, email
FROM users
WHERE status = 'active';
```

#### 2. Complex Views
Views involving multiple tables, joins, aggregations, or complex logic.

```sql
-- Complex view - multiple tables with joins and aggregation
CREATE VIEW order_analytics AS
SELECT 
    u.username,
    u.email,
    COUNT(o.order_id) as total_orders,
    SUM(o.total_amount) as total_spent,
    AVG(o.total_amount) as avg_order_value
FROM users u
LEFT JOIN orders o ON u.user_id = o.customer_id
WHERE u.status = 'active'
GROUP BY u.user_id, u.username, u.email;
```

#### 3. Materialized Views
Physical storage of view data for improved performance (database-specific feature).

```sql
-- PostgreSQL materialized view
CREATE MATERIALIZED VIEW monthly_sales AS
SELECT 
    DATE_TRUNC('month', order_date) as month,
    SUM(total_amount) as total_sales,
    COUNT(*) as order_count
FROM orders
GROUP BY DATE_TRUNC('month', order_date);

-- Refresh materialized view
REFRESH MATERIALIZED VIEW monthly_sales;
```

### View Operations

#### Creating Views
```sql
-- Standard view
CREATE VIEW view_name AS
SELECT columns
FROM tables
WHERE conditions;

-- View with check option (ensures INSERTs/UPDATEs meet view criteria)
CREATE VIEW active_users AS
SELECT * FROM users WHERE status = 'active'
WITH CHECK OPTION;
```

#### Updating Views
```sql
-- Replace existing view
CREATE OR REPLACE VIEW user_summary AS
SELECT user_id, username, email, phone
FROM users
WHERE status IN ('active', 'pending');

-- Modify view (if supported)
ALTER VIEW user_summary AS
SELECT user_id, username, email, phone, created_at
FROM users
WHERE status IN ('active', 'pending');
```

#### Dropping Views
```sql
-- Drop single view
DROP VIEW user_summary;

-- Drop view if exists
DROP VIEW IF EXISTS user_summary;

-- Drop multiple views
DROP VIEW view1, view2, view3;
```

### Advantages of Views

| Advantage | Description | Example Use Case |
|-----------|-------------|------------------|
| **Data Security** | Hide sensitive columns/rows | Show employee data without salaries |
| **Simplification** | Abstract complex queries | Provide simple interface to complex joins |
| **Data Consistency** | Centralize business logic | Ensure consistent calculations across applications |
| **Backward Compatibility** | Maintain old interfaces | Keep old column names after table restructuring |
| **Access Control** | Control what users can see | Different views for different user roles |

### Disadvantages of Views

| Disadvantage | Description | Impact |
|--------------|-------------|---------|
| **Performance Overhead** | Query must be re-executed each time | Slower than direct table access |
| **Update Limitations** | Not all views are updatable | Complex views may be read-only |
| **Dependency Management** | Views depend on underlying tables | Schema changes can break views |
| **Debugging Complexity** | Harder to troubleshoot performance issues | Query execution plans can be complex |

### Updatable vs Non-Updatable Views

#### Updatable Views (can use INSERT, UPDATE, DELETE)
```sql
-- Simple updatable view
CREATE VIEW active_customers AS
SELECT customer_id, name, email, phone
FROM customers
WHERE status = 'active';

-- This works - view is updatable
UPDATE active_customers 
SET email = 'new@email.com' 
WHERE customer_id = 123;
```

**Requirements for updatable views**:
- Based on single table
- No aggregate functions (SUM, COUNT, etc.)
- No DISTINCT, GROUP BY, HAVING
- No window functions
- No set operations (UNION, INTERSECT)

#### Non-Updatable Views (read-only)
```sql
-- Non-updatable due to aggregation
CREATE VIEW customer_stats AS
SELECT 
    status,
    COUNT(*) as customer_count,
    AVG(credit_limit) as avg_credit
FROM customers
GROUP BY status;

-- This will fail
UPDATE customer_stats SET customer_count = 100; -- ERROR
```

### Practical Examples

#### 1. Security View
```sql
-- Hide sensitive employee data
CREATE VIEW public_employees AS
SELECT 
    employee_id,
    first_name,
    last_name,
    department,
    hire_date
FROM employees
-- Excludes salary, SSN, etc.
```

#### 2. Business Logic View
```sql
-- Encapsulate complex business calculations
CREATE VIEW product_profitability AS
SELECT 
    p.product_id,
    p.product_name,
    p.price,
    AVG(p.price - p.cost) as avg_profit_per_unit,
    SUM(oi.quantity * (p.price - p.cost)) as total_profit
FROM products p
JOIN order_items oi ON p.product_id = oi.product_id
JOIN orders o ON oi.order_id = o.order_id
WHERE o.order_date >= CURRENT_DATE - INTERVAL '12 months'
GROUP BY p.product_id, p.product_name, p.price;
```

#### 3. Data Transformation View
```sql
-- Standardize data format
CREATE VIEW formatted_addresses AS
SELECT 
    customer_id,
    UPPER(TRIM(street_address)) as street_address,
    UPPER(TRIM(city)) as city,
    UPPER(TRIM(state)) as state,
    REGEXP_REPLACE(zip_code, '[^0-9]', '') as zip_code
FROM customer_addresses;
```

### Performance Considerations

#### 1. Query Optimization
```sql
-- Bad: View with unnecessary complexity
CREATE VIEW slow_view AS
SELECT * FROM 
(SELECT * FROM large_table ORDER BY created_at) subq
WHERE status = 'active';

-- Good: Simple, efficient view
CREATE VIEW fast_view AS
SELECT columns_needed FROM large_table
WHERE status = 'active';
```

#### 2. Index Usage
```sql
-- Ensure underlying tables have appropriate indexes
CREATE INDEX idx_customers_status ON customers(status);
CREATE INDEX idx_orders_customer_date ON orders(customer_id, order_date);

-- View will benefit from these indexes
CREATE VIEW recent_customer_orders AS
SELECT c.name, o.order_date, o.total_amount
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
WHERE c.status = 'active' 
  AND o.order_date >= CURRENT_DATE - INTERVAL '30 days';
```

### Best Practices

1. **Use Descriptive Names**: Make view purpose clear from the name
2. **Document Complex Logic**: Add comments for business rules
3. **Avoid SELECT \***: Specify only needed columns
4. **Consider Performance**: Monitor view query execution
5. **Version Control**: Track view definitions in source control
6. **Test Updates**: Verify view behavior after schema changes
7. **Use Materialized Views**: For expensive, frequently-accessed aggregations
8. **Grant Appropriate Permissions**: Control access at view level

### Common Use Cases

- **Reporting**: Aggregate data for dashboards and reports
- **Data Privacy**: Hide sensitive information from users
- **Legacy Support**: Maintain old interfaces during migrations
- **Complex Joins**: Simplify access to normalized data
- **Role-Based Access**: Different data views for different user types
- **Data Validation**: Present only valid/active records

## Stored Procedures

### What are Stored Procedures?

**Definition**: A stored procedure is a precompiled collection of SQL statements and optional control-flow statements that are stored in the database and can be executed as a single unit.

```sql
-- Basic stored procedure syntax (SQL Server/MySQL)
DELIMITER //
CREATE PROCEDURE GetUserById(IN user_id INT)
BEGIN
    SELECT user_id, username, email, created_at
    FROM users
    WHERE user_id = user_id;
END //
DELIMITER ;
```

### Key Characteristics

| Feature | Description |
|---------|-------------|
| **Precompiled** | Compiled once, executed many times |
| **Parameterized** | Accept input/output parameters |
| **Reusable** | Can be called multiple times from different applications |
| **Server-side** | Execute on the database server |
| **Encapsulated** | Hide complex business logic |

### Types of Parameters

#### 1. Input Parameters (IN)
```sql
-- PostgreSQL syntax
CREATE OR REPLACE FUNCTION get_orders_by_customer(customer_id_param INTEGER)
RETURNS TABLE(order_id INTEGER, order_date DATE, total_amount DECIMAL)
LANGUAGE plpgsql
AS $$
BEGIN
    RETURN QUERY
    SELECT o.order_id, o.order_date, o.total_amount
    FROM orders o
    WHERE o.customer_id = customer_id_param;
END;
$$;

-- Call the procedure
SELECT * FROM get_orders_by_customer(123);
```

#### 2. Output Parameters (OUT)
```sql
-- SQL Server syntax
CREATE PROCEDURE GetCustomerStats
    @customer_id INT,
    @total_orders INT OUTPUT,
    @total_spent DECIMAL(10,2) OUTPUT
AS
BEGIN
    SELECT 
        @total_orders = COUNT(*),
        @total_spent = SUM(total_amount)
    FROM orders
    WHERE customer_id = @customer_id;
END;

-- Execute with output parameters
DECLARE @order_count INT, @amount_spent DECIMAL(10,2);
EXEC GetCustomerStats @customer_id = 123, 
                     @total_orders = @order_count OUTPUT,
                     @total_spent = @amount_spent OUTPUT;
SELECT @order_count as OrderCount, @amount_spent as TotalSpent;
```

#### 3. Input/Output Parameters (INOUT)
```sql
-- MySQL syntax
DELIMITER //
CREATE PROCEDURE UpdateUserStatus(
    INOUT user_id INT,
    IN new_status VARCHAR(20)
)
BEGIN
    UPDATE users 
    SET status = new_status, updated_at = NOW()
    WHERE user_id = user_id;
    
    -- Return the number of affected rows
    SET user_id = ROW_COUNT();
END //
DELIMITER ;
```

### Control Flow Statements

#### Conditional Logic (IF/ELSE)
```sql
-- PostgreSQL function with conditional logic
CREATE OR REPLACE FUNCTION calculate_discount(
    customer_id_param INTEGER,
    order_amount DECIMAL
) RETURNS DECIMAL
LANGUAGE plpgsql
AS $$
DECLARE
    customer_type VARCHAR(20);
    discount_rate DECIMAL := 0;
BEGIN
    -- Get customer type
    SELECT type INTO customer_type
    FROM customers
    WHERE customer_id = customer_id_param;
    
    -- Apply discount based on customer type and order amount
    IF customer_type = 'VIP' THEN
        IF order_amount > 1000 THEN
            discount_rate := 0.15;  -- 15% for VIP orders > $1000
        ELSE
            discount_rate := 0.10;  -- 10% for VIP orders
        END IF;
    ELSIF customer_type = 'PREMIUM' THEN
        discount_rate := 0.05;      -- 5% for premium customers
    ELSE
        IF order_amount > 500 THEN
            discount_rate := 0.02;  -- 2% for regular customers with large orders
        END IF;
    END IF;
    
    RETURN order_amount * discount_rate;
END;
$$;
```

#### Loops
```sql
-- MySQL procedure with WHILE loop
DELIMITER //
CREATE PROCEDURE GenerateMonthlyReports(IN year_param INT)
BEGIN
    DECLARE month_counter INT DEFAULT 1;
    DECLARE report_data TEXT;
    
    WHILE month_counter <= 12 DO
        -- Generate report for each month
        SELECT CONCAT('Report for ', year_param, '-', LPAD(month_counter, 2, '0'))
        INTO report_data;
        
        INSERT INTO monthly_reports (year, month, report_content, created_at)
        VALUES (year_param, month_counter, report_data, NOW());
        
        SET month_counter = month_counter + 1;
    END WHILE;
END //
DELIMITER ;
```

#### Exception Handling
```sql
-- PostgreSQL function with exception handling
CREATE OR REPLACE FUNCTION transfer_funds(
    from_account INTEGER,
    to_account INTEGER,
    amount DECIMAL
) RETURNS BOOLEAN
LANGUAGE plpgsql
AS $$
DECLARE
    from_balance DECIMAL;
BEGIN
    -- Start transaction
    BEGIN
        -- Check source account balance
        SELECT balance INTO from_balance
        FROM accounts
        WHERE account_id = from_account
        FOR UPDATE;
        
        IF from_balance < amount THEN
            RAISE EXCEPTION 'Insufficient funds. Balance: %, Required: %', from_balance, amount;
        END IF;
        
        -- Perform transfer
        UPDATE accounts
        SET balance = balance - amount
        WHERE account_id = from_account;
        
        UPDATE accounts
        SET balance = balance + amount
        WHERE account_id = to_account;
        
        -- Log transaction
        INSERT INTO transaction_log (from_account, to_account, amount, transaction_date)
        VALUES (from_account, to_account, amount, NOW());
        
        RETURN TRUE;
        
    EXCEPTION
        WHEN OTHERS THEN
            -- Log error
            INSERT INTO error_log (error_message, error_time)
            VALUES (SQLERRM, NOW());
            
            RETURN FALSE;
    END;
END;
$$;
```

### Advantages of Stored Procedures

| Advantage | Description | Benefit |
|-----------|-------------|---------|
| **Performance** | Precompiled and cached | Faster execution than dynamic SQL |
| **Security** | Parameterized queries prevent SQL injection | Enhanced security |
| **Code Reusability** | Centralized business logic | Reduced code duplication |
| **Network Traffic** | Single call executes multiple statements | Reduced bandwidth usage |
| **Data Integrity** | Transaction control and validation | Better data consistency |
| **Maintenance** | Logic changes in one place | Easier updates and bug fixes |

### Disadvantages of Stored Procedures

| Disadvantage | Description | Impact |
|--------------|-------------|---------|
| **Database Dependency** | Tied to specific database platform | Reduced portability |
| **Version Control** | Harder to track changes | Development workflow complexity |
| **Debugging** | Limited debugging tools | Harder to troubleshoot |
| **Scalability** | Processing load on database server | Potential performance bottleneck |
| **Team Skills** | Requires database-specific knowledge | Learning curve for developers |

### Real-World Examples

#### 1. User Registration Procedure
```sql
-- Complete user registration with validation
CREATE OR REPLACE FUNCTION register_user(
    p_username VARCHAR(50),
    p_email VARCHAR(100),
    p_password_hash VARCHAR(255),
    p_first_name VARCHAR(50),
    p_last_name VARCHAR(50)
) RETURNS INTEGER
LANGUAGE plpgsql
AS $$
DECLARE
    new_user_id INTEGER;
    email_exists BOOLEAN;
BEGIN
    -- Validate email doesn't already exist
    SELECT EXISTS(SELECT 1 FROM users WHERE email = p_email) INTO email_exists;
    
    IF email_exists THEN
        RAISE EXCEPTION 'Email already registered: %', p_email;
    END IF;
    
    -- Insert new user
    INSERT INTO users (username, email, password_hash, first_name, last_name, status, created_at)
    VALUES (p_username, p_email, p_password_hash, p_first_name, p_last_name, 'active', NOW())
    RETURNING user_id INTO new_user_id;
    
    -- Create default user preferences
    INSERT INTO user_preferences (user_id, theme, notifications_enabled)
    VALUES (new_user_id, 'default', TRUE);
    
    -- Log registration
    INSERT INTO user_activity_log (user_id, activity_type, activity_date)
    VALUES (new_user_id, 'registration', NOW());
    
    RETURN new_user_id;
END;
$$;
```

#### 2. Order Processing Procedure
```sql
-- Complex order processing with inventory management
DELIMITER //
CREATE PROCEDURE ProcessOrder(
    IN p_customer_id INT,
    IN p_product_id INT,
    IN p_quantity INT,
    OUT p_order_id INT,
    OUT p_status VARCHAR(50)
)
BEGIN
    DECLARE v_available_stock INT;
    DECLARE v_product_price DECIMAL(10,2);
    DECLARE v_total_amount DECIMAL(10,2);
    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        ROLLBACK;
        SET p_status = 'ERROR';
        SET p_order_id = -1;
    END;
    
    START TRANSACTION;
    
    -- Check inventory
    SELECT stock_quantity, price
    INTO v_available_stock, v_product_price
    FROM products
    WHERE product_id = p_product_id
    FOR UPDATE;
    
    IF v_available_stock < p_quantity THEN
        SET p_status = 'INSUFFICIENT_STOCK';
        SET p_order_id = -1;
        ROLLBACK;
    ELSE
        -- Calculate total
        SET v_total_amount = v_product_price * p_quantity;
        
        -- Create order
        INSERT INTO orders (customer_id, total_amount, status, order_date)
        VALUES (p_customer_id, v_total_amount, 'processing', NOW());
        
        SET p_order_id = LAST_INSERT_ID();
        
        -- Add order items
        INSERT INTO order_items (order_id, product_id, quantity, unit_price)
        VALUES (p_order_id, p_product_id, p_quantity, v_product_price);
        
        -- Update inventory
        UPDATE products
        SET stock_quantity = stock_quantity - p_quantity
        WHERE product_id = p_product_id;
        
        SET p_status = 'SUCCESS';
        COMMIT;
    END IF;
END //
DELIMITER ;
```

#### 3. Data Analytics Procedure
```sql
-- Generate comprehensive sales analytics
CREATE OR REPLACE FUNCTION generate_sales_report(
    start_date DATE,
    end_date DATE
) RETURNS TABLE(
    product_name VARCHAR(100),
    total_sales DECIMAL(12,2),
    units_sold INTEGER,
    avg_order_value DECIMAL(10,2),
    profit_margin DECIMAL(5,2)
)
LANGUAGE plpgsql
AS $$
BEGIN
    RETURN QUERY
    SELECT 
        p.product_name,
        SUM(oi.quantity * oi.unit_price) as total_sales,
        SUM(oi.quantity)::INTEGER as units_sold,
        AVG(oi.quantity * oi.unit_price) as avg_order_value,
        ((SUM(oi.quantity * oi.unit_price) - SUM(oi.quantity * p.cost)) / 
         SUM(oi.quantity * oi.unit_price) * 100) as profit_margin
    FROM products p
    JOIN order_items oi ON p.product_id = oi.product_id
    JOIN orders o ON oi.order_id = o.order_id
    WHERE o.order_date BETWEEN start_date AND end_date
    GROUP BY p.product_id, p.product_name
    ORDER BY total_sales DESC;
END;
$$;
```

### Best Practices

1. **Use Clear Naming Conventions**: Prefix procedures with purpose (sp_GetUser, fn_CalculateDiscount)
2. **Parameter Validation**: Always validate input parameters
3. **Error Handling**: Implement comprehensive exception handling
4. **Transaction Management**: Use transactions for data consistency
5. **Documentation**: Add comments explaining complex logic
6. **Security**: Use parameterized queries to prevent SQL injection
7. **Performance**: Optimize queries and use appropriate indexes
8. **Testing**: Create unit tests for stored procedures
9. **Version Control**: Track procedure changes in source control

### When to Use Stored Procedures

**Good Use Cases**:
- Complex business logic that involves multiple tables
- Data validation and integrity enforcement
- Batch processing operations
- Performance-critical operations
- Security-sensitive operations
- Database-specific optimizations

**Avoid When**:
- Simple CRUD operations
- Application logic that changes frequently
- Need for database portability
- Team lacks database expertise
- Microservices architecture with database per service
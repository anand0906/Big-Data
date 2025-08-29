# Advanced SQL Transformations - Simple Guide

## What Are Advanced SQL Transformations?

Think of transformations like a kitchen where you take raw ingredients (messy data) and turn them into a delicious meal (clean, useful data). Advanced transformations help you:

- **Work with messy data** (like JSON files from websites)
- **Combine data from multiple sources** (like mixing ingredients)
- **Create calculated fields** (like converting temperatures from Celsius to Fahrenheit)
- **Handle complex data structures** (like nested folders on your computer)

## Working with JSON Data (Web Data)

JSON data is like receiving a package with boxes inside boxes. You need to unpack each layer.

### Step 1: Basic JSON Reading

```sql
-- Imagine you have website event data that looks like this:
-- {"user": {"id": 123, "name": "John"}, "action": "click", "timestamp": "2024-08-29"}

-- Create a simple view to see the data
CREATE OR REPLACE TEMPORARY VIEW events_readable AS
SELECT 
    CAST(key AS STRING) as event_key,
    CAST(value AS STRING) as event_data
FROM events_raw;  -- Your raw JSON table

-- Look at the data
SELECT * FROM events_readable LIMIT 5;
```

### Step 2: Extracting Nested Information (Colon Syntax)

```sql
-- Use colon syntax to dig into nested data (like opening folders)
CREATE OR REPLACE TEMPORARY VIEW user_events AS
SELECT 
    value:user.id as user_id,           -- Get user ID from nested structure
    value:user.name as user_name,       -- Get user name
    value:action as action_type,        -- Get what they did
    value:timestamp as event_time       -- Get when it happened
FROM events_raw;

-- See the extracted data
SELECT 
    user_id,
    user_name,
    action_type,
    event_time
FROM user_events
WHERE action_type = 'purchase'  -- Only show purchases
LIMIT 10;
```

### Step 3: Parsing Complex JSON Structures

```sql
-- Convert JSON text into structured data (like organizing scattered papers)
CREATE OR REPLACE TEMPORARY VIEW parsed_events AS
SELECT 
    from_json(value, 'user STRUCT<id:INT, name:STRING, location:STRING>, action:STRING, items:ARRAY<STRING>') as parsed_data
FROM events_raw;

-- Now use the structured data easily
SELECT 
    parsed_data.user.id as customer_id,
    parsed_data.user.name as customer_name,
    parsed_data.user.location as customer_location,
    parsed_data.action as what_they_did,
    parsed_data.items as items_involved
FROM parsed_events
WHERE parsed_data.action IN ('purchase', 'add_to_cart');
```

## Working with Arrays (Lists of Things)

Arrays are like shopping lists - they contain multiple items in one field.

### Using EXPLODE (Unpacking Lists)

```sql
-- Imagine each customer has a list of purchased items: ["apple", "banana", "orange"]
-- EXPLODE puts each item on its own row

CREATE OR REPLACE TEMPORARY VIEW customer_purchases AS
SELECT 
    customer_id,
    customer_name,
    explode(purchased_items) as individual_item  -- Unpack the list
FROM customer_shopping_data;

-- Now each item is on its own row
SELECT 
    customer_id,
    customer_name,
    individual_item
FROM customer_purchases
WHERE individual_item LIKE '%organic%';  -- Find organic products
```

### Array Functions (List Operations)

```sql
-- Work with arrays like Excel functions
CREATE OR REPLACE TEMPORARY VIEW customer_analysis AS
SELECT 
    customer_id,
    collect_set(product_category) as unique_categories,     -- Remove duplicates from list
    array_distinct(all_purchases) as unique_purchases,      -- Remove duplicate purchases
    flatten(nested_items) as all_items,                     -- Combine multiple lists into one
    size(purchased_items) as total_items_bought             -- Count items in list
FROM customer_data
GROUP BY customer_id;

-- Show customers and their shopping patterns
SELECT 
    customer_id,
    size(unique_categories) as different_categories_shopped,
    size(unique_purchases) as unique_items_bought
FROM customer_analysis
WHERE size(unique_categories) >= 3;  -- Customers who shop in 3+ categories
```

## Joining Tables (Combining Information)

Joining is like matching information from different spreadsheets.

### Simple Join Example

```sql
-- You have customer info in one table and orders in another
-- Join them to see customer names with their orders

SELECT 
    c.customer_name,
    c.email,
    c.city,
    o.order_id,
    o.total_amount,
    o.order_date
FROM customers c
INNER JOIN orders o ON c.customer_id = o.customer_id  -- Match by customer ID
WHERE o.order_date >= '2024-08-01'
ORDER BY o.total_amount DESC;
```

### Advanced Join with Lookups

```sql
-- Add product names and categories to order data
CREATE OR REPLACE TEMPORARY VIEW enriched_orders AS
SELECT 
    o.order_id,
    c.customer_name,
    p.product_name,
    p.category,
    o.quantity,
    o.unit_price,
    (o.quantity * o.unit_price) as line_total,
    o.order_date
FROM orders o
INNER JOIN customers c ON o.customer_id = c.customer_id
INNER JOIN products p ON o.product_id = p.product_id;

-- Analyze by category
SELECT 
    category,
    COUNT(*) as orders_count,
    SUM(line_total) as total_revenue,
    AVG(line_total) as avg_order_value
FROM enriched_orders
GROUP BY category
ORDER BY total_revenue DESC;
```

## Combining Data from Multiple Sources (UNION)

UNION is like stacking similar papers together.

```sql
-- Combine online and in-store sales data
CREATE OR REPLACE TEMPORARY VIEW all_sales AS
SELECT 
    sale_id,
    customer_id,
    product_name,
    amount,
    sale_date,
    'online' as sales_channel
FROM online_sales
WHERE sale_date >= '2024-01-01'

UNION ALL  -- Stack them together

SELECT 
    sale_id,
    customer_id,
    product_name,
    amount,
    sale_date,
    'store' as sales_channel
FROM store_sales
WHERE sale_date >= '2024-01-01';

-- Analyze combined data
SELECT 
    sales_channel,
    COUNT(*) as total_sales,
    SUM(amount) as total_revenue,
    AVG(amount) as average_sale
FROM all_sales
GROUP BY sales_channel;
```

## Higher-Order Functions (Smart List Operations)

These are like having a smart assistant who can work with your lists.

### FILTER Function (Smart Filtering)

```sql
-- Remove unwanted items from arrays
CREATE OR REPLACE TEMPORARY VIEW filtered_carts AS
SELECT 
    customer_id,
    shopping_cart,
    filter(shopping_cart, item -> item.size = 'king') as king_size_items,  -- Only king size
    filter(shopping_cart, item -> item.price > 50) as expensive_items     -- Only expensive items
FROM customer_carts;

-- See what's left after filtering
SELECT 
    customer_id,
    size(king_size_items) as num_king_items,
    size(expensive_items) as num_expensive_items
FROM filtered_carts;
```

### TRANSFORM Function (Smart Calculations)

```sql
-- Apply calculations to each item in a list
CREATE OR REPLACE TEMPORARY VIEW calculated_revenue AS
SELECT 
    customer_id,
    shopping_cart,
    -- Calculate revenue for each item (quantity × price)
    transform(
        shopping_cart, 
        item -> item.quantity * item.price
    ) as item_revenues,
    
    -- Apply a discount to each item
    transform(
        shopping_cart,
        item -> item.price * 0.9  -- 10% discount
    ) as discounted_prices
FROM customer_carts;

-- Calculate total revenue per customer
SELECT 
    customer_id,
    aggregate(item_revenues, 0, (acc, x) -> acc + x) as total_revenue
FROM calculated_revenue;
```

## Complete Real-World Example: E-commerce Analytics

Let's process complex e-commerce data step by step.

### Step 1: Raw Data Processing

```sql
-- Process raw website events (JSON format)
CREATE OR REPLACE TEMPORARY VIEW clean_events AS
SELECT 
    from_json(
        event_data, 
        'user STRUCT<id:INT, session:STRING>, action:STRING, items:ARRAY<STRUCT<name:STRING, price:DOUBLE, quantity:INT>>, timestamp:TIMESTAMP'
    ) as parsed_event
FROM raw_web_events
WHERE event_data IS NOT NULL;
```

### Step 2: Extract and Flatten Data

```sql
-- Create flattened view of user actions
CREATE OR REPLACE TEMPORARY VIEW user_actions AS
SELECT 
    parsed_event.user.id as user_id,
    parsed_event.user.session as session_id,
    parsed_event.action as action_type,
    parsed_event.timestamp as action_time,
    parsed_event.items as cart_items
FROM clean_events
WHERE parsed_event.user.id IS NOT NULL;

-- Break down each item in the cart
CREATE OR REPLACE TEMPORARY VIEW item_actions AS
SELECT 
    user_id,
    session_id,
    action_type,
    action_time,
    explode(cart_items) as item_detail  -- Each item gets its own row
FROM user_actions
WHERE action_type IN ('add_to_cart', 'purchase', 'remove_from_cart');
```

### Step 3: Business Logic and Calculations

```sql
-- Calculate key business metrics
CREATE OR REPLACE TEMPORARY VIEW user_metrics AS
SELECT 
    user_id,
    COUNT(DISTINCT session_id) as total_sessions,
    
    -- Count different types of actions
    SUM(CASE WHEN action_type = 'add_to_cart' THEN 1 ELSE 0 END) as items_added,
    SUM(CASE WHEN action_type = 'purchase' THEN 1 ELSE 0 END) as items_purchased,
    SUM(CASE WHEN action_type = 'remove_from_cart' THEN 1 ELSE 0 END) as items_removed,
    
    -- Calculate revenue metrics
    SUM(CASE 
        WHEN action_type = 'purchase' 
        THEN item_detail.price * item_detail.quantity 
        ELSE 0 
    END) as total_revenue,
    
    -- Find most expensive item purchased
    MAX(CASE 
        WHEN action_type = 'purchase' 
        THEN item_detail.price 
        ELSE 0 
    END) as highest_item_price,
    
    -- Collect unique products purchased
    collect_set(CASE 
        WHEN action_type = 'purchase' 
        THEN item_detail.name 
        ELSE NULL 
    END) as products_purchased

FROM item_actions
GROUP BY user_id;
```

### Step 4: Advanced Analytics

```sql
-- Create customer segments based on behavior
CREATE OR REPLACE TEMPORARY VIEW customer_segments AS
SELECT 
    user_id,
    total_revenue,
    items_purchased,
    size(products_purchased) as unique_products,
    
    -- Create customer tiers
    CASE 
        WHEN total_revenue >= 1000 THEN 'VIP'
        WHEN total_revenue >= 500 THEN 'Premium'
        WHEN total_revenue >= 100 THEN 'Regular'
        ELSE 'New'
    END as customer_tier,
    
    -- Calculate conversion rate (purchases vs adds to cart)
    CASE 
        WHEN items_added > 0 
        THEN ROUND((items_purchased * 100.0 / items_added), 2)
        ELSE 0 
    END as conversion_rate_percent

FROM user_metrics
WHERE items_added > 0;  -- Only users who added items to cart
```

## Working with Time and Dates

```sql
-- Time-based analysis (like tracking when people shop)
CREATE OR REPLACE TEMPORARY VIEW time_analysis AS
SELECT 
    user_id,
    action_time,
    
    -- Extract time components
    YEAR(action_time) as action_year,
    MONTH(action_time) as action_month,
    DAYOFWEEK(action_time) as day_of_week,  -- 1=Sunday, 7=Saturday
    HOUR(action_time) as action_hour,
    
    -- Create readable labels
    CASE DAYOFWEEK(action_time)
        WHEN 1 THEN 'Sunday'
        WHEN 2 THEN 'Monday'
        WHEN 3 THEN 'Tuesday'
        WHEN 4 THEN 'Wednesday'
        WHEN 5 THEN 'Thursday'
        WHEN 6 THEN 'Friday'
        WHEN 7 THEN 'Saturday'
    END as day_name,
    
    -- Categorize shopping times
    CASE 
        WHEN HOUR(action_time) BETWEEN 6 AND 11 THEN 'Morning'
        WHEN HOUR(action_time) BETWEEN 12 AND 17 THEN 'Afternoon'
        WHEN HOUR(action_time) BETWEEN 18 AND 21 THEN 'Evening'
        ELSE 'Night'
    END as time_period

FROM user_actions
WHERE action_type = 'purchase';

-- Find peak shopping times
SELECT 
    day_name,
    time_period,
    COUNT(*) as purchases,
    AVG(item_detail.price) as avg_price
FROM time_analysis ta
JOIN item_actions ia ON ta.user_id = ia.user_id AND ta.action_time = ia.action_time
WHERE ia.action_type = 'purchase'
GROUP BY day_name, time_period
ORDER BY purchases DESC;
```

## Window Functions (Smart Comparisons)

Window functions are like having Excel's "compare to previous row" feature but much more powerful.

### Ranking and Comparison

```sql
-- Find each customer's favorite products and spending patterns
CREATE OR REPLACE TEMPORARY VIEW customer_insights AS
SELECT 
    user_id,
    item_detail.name as product_name,
    item_detail.price * item_detail.quantity as purchase_amount,
    action_time,
    
    -- Rank products by how much each customer spends on them
    ROW_NUMBER() OVER (
        PARTITION BY user_id 
        ORDER BY item_detail.price * item_detail.quantity DESC
    ) as spending_rank,
    
    -- Calculate running total of spending per customer
    SUM(item_detail.price * item_detail.quantity) OVER (
        PARTITION BY user_id 
        ORDER BY action_time 
        ROWS UNBOUNDED PRECEDING
    ) as running_total_spent,
    
    -- Compare to previous purchase
    LAG(item_detail.price * item_detail.quantity, 1) OVER (
        PARTITION BY user_id 
        ORDER BY action_time
    ) as previous_purchase_amount

FROM item_actions
WHERE action_type = 'purchase';

-- Show each customer's top 3 purchases
SELECT 
    user_id,
    product_name,
    purchase_amount,
    running_total_spent
FROM customer_insights
WHERE spending_rank <= 3  -- Top 3 purchases per customer
ORDER BY user_id, spending_rank;
```

## Working with Product Categories and Hierarchies

```sql
-- Create product hierarchy analysis
CREATE OR REPLACE TEMPORARY VIEW product_hierarchy AS
SELECT 
    user_id,
    collect_list(item_detail.name) as all_purchased_items,
    
    -- Group items by price ranges
    collect_list(
        CASE 
            WHEN item_detail.price < 25 THEN 'Budget'
            WHEN item_detail.price < 100 THEN 'Mid-range'
            ELSE 'Premium'
        END
    ) as price_categories,
    
    -- Filter for only expensive items using higher-order functions
    filter(
        collect_list(STRUCT(item_detail.name, item_detail.price)), 
        item -> item.price > 100
    ) as premium_items

FROM item_actions
WHERE action_type = 'purchase'
GROUP BY user_id;

-- Analyze customer preferences
SELECT 
    user_id,
    size(all_purchased_items) as total_items,
    size(premium_items) as premium_items_count,
    array_distinct(price_categories) as unique_price_ranges
FROM product_hierarchy
WHERE size(all_purchased_items) > 0;
```

## Advanced Calculations with TRANSFORM

```sql
-- Apply business logic to arrays (like bulk operations in Excel)
CREATE OR REPLACE TEMPORARY VIEW revenue_calculations AS
SELECT 
    user_id,
    cart_items,
    
    -- Calculate revenue for each item in the cart
    transform(
        cart_items,
        item -> item.price * item.quantity
    ) as item_revenues,
    
    -- Apply 10% discount to all items
    transform(
        cart_items,
        item -> STRUCT(
            item.name,
            ROUND(item.price * 0.9, 2) as discounted_price,
            item.quantity
        )
    ) as discounted_cart,
    
    -- Mark items as expensive or cheap
    transform(
        cart_items,
        item -> STRUCT(
            item.name,
            item.price,
            CASE WHEN item.price > 50 THEN 'expensive' ELSE 'affordable' END as price_category
        )
    ) as categorized_items

FROM user_actions
WHERE action_type = 'add_to_cart';

-- Calculate total discounted revenue per customer
SELECT 
    user_id,
    aggregate(item_revenues, 0.0, (total, revenue) -> total + revenue) as original_total,
    aggregate(
        transform(discounted_cart, item -> item.discounted_price), 
        0.0, 
        (total, price) -> total + price
    ) as discounted_total
FROM revenue_calculations;
```

## Practical Business Example: Customer 360 View

Let's create a complete customer profile combining all techniques:

```sql
-- Step 1: Process raw website events
CREATE OR REPLACE TEMPORARY VIEW processed_events AS
SELECT 
    from_json(
        event_data, 
        'user STRUCT<id:INT, name:STRING, email:STRING>, action:STRING, items:ARRAY<STRUCT<name:STRING, price:DOUBLE, category:STRING>>, timestamp:TIMESTAMP'
    ).user.id as customer_id,
    from_json(event_data, 'user STRUCT<id:INT, name:STRING, email:STRING>, action:STRING, items:ARRAY<STRUCT<name:STRING, price:DOUBLE, category:STRING>>, timestamp:TIMESTAMP').user.name as customer_name,
    from_json(event_data, 'user STRUCT<id:INT, name:STRING, email:STRING>, action:STRING, items:ARRAY<STRUCT<name:STRING, price:DOUBLE, category:STRING>>, timestamp:TIMESTAMP').action as action,
    from_json(event_data, 'user STRUCT<id:INT, name:STRING, email:STRING>, action:STRING, items:ARRAY<STRUCT<name:STRING, price:DOUBLE, category:STRING>>, timestamp:TIMESTAMP').items as items,
    from_json(event_data, 'user STRUCT<id:INT, name:STRING, email:STRING>, action:STRING, items:ARRAY<STRUCT<name:STRING, price:DOUBLE, category:STRING>>, timestamp:TIMESTAMP').timestamp as event_time
FROM raw_events
WHERE event_data IS NOT NULL;

-- Step 2: Create comprehensive customer profile
CREATE TABLE customer_360_view AS
SELECT 
    customer_id,
    ANY_VALUE(customer_name) as customer_name,  -- Get name (same for all records)
    
    -- Shopping behavior
    COUNT(DISTINCT DATE(event_time)) as active_days,
    COUNT(*) as total_actions,
    
    -- Purchase analysis
    SUM(CASE WHEN action = 'purchase' THEN 1 ELSE 0 END) as total_purchases,
    
    -- Revenue calculations using transform and aggregate
    aggregate(
        flatten(
            transform(
                filter(items, item -> action = 'purchase'),
                item -> item.price
            )
        ),
        0.0,
        (total, price) -> total + price
    ) as total_spent,
    
    -- Product preferences
    array_distinct(
        flatten(
            transform(
                filter(items, item -> action = 'purchase'),
                item -> item.category
            )
        )
    ) as preferred_categories,
    
    -- Shopping patterns
    collect_set(DATE(event_time)) as shopping_dates,
    MIN(event_time) as first_interaction,
    MAX(event_time) as last_interaction,
    
    -- Calculate average days between purchases
    CASE 
        WHEN SUM(CASE WHEN action = 'purchase' THEN 1 ELSE 0 END) > 1
        THEN DATEDIFF(MAX(event_time), MIN(event_time)) / SUM(CASE WHEN action = 'purchase' THEN 1 ELSE 0 END)
        ELSE 0
    END as avg_days_between_purchases

FROM processed_events
GROUP BY customer_id
HAVING COUNT(*) > 1;  -- Only customers with multiple interactions

-- Step 3: Add customer segments
CREATE OR REPLACE TEMPORARY VIEW segmented_customers AS
SELECT 
    *,
    CASE 
        WHEN total_spent >= 1000 AND total_purchases >= 10 THEN 'VIP'
        WHEN total_spent >= 500 AND total_purchases >= 5 THEN 'Premium'
        WHEN total_spent >= 100 AND total_purchases >= 2 THEN 'Regular'
        ELSE 'Casual'
    END as customer_segment,
    
    size(preferred_categories) as category_diversity
FROM customer_360_view;

-- Final analysis
SELECT 
    customer_segment,
    COUNT(*) as customer_count,
    AVG(total_spent) as avg_spent_per_customer,
    AVG(total_purchases) as avg_purchases_per_customer,
    AVG(category_diversity) as avg_categories_shopped
FROM segmented_customers
GROUP BY customer_segment
ORDER BY avg_spent_per_customer DESC;
```

## Quick Reference: Function Cheat Sheet

### Array Functions
```sql
-- Working with lists
explode(array_column)           -- Put each item on separate row
size(array_column)              -- Count items in list
array_distinct(array_column)    -- Remove duplicates
collect_list(column)            -- Make list from multiple rows
collect_set(column)             -- Make list without duplicates
flatten(array_of_arrays)        -- Combine multiple lists
```

### Higher-Order Functions
```sql
-- Smart list operations
filter(array, condition)        -- Keep only items matching condition
transform(array, calculation)   -- Apply calculation to each item
aggregate(array, start, function) -- Combine all items using function
exists(array, condition)        -- Check if any item matches condition
```

### JSON Functions
```sql
-- Working with web data
from_json(json_string, schema)  -- Convert JSON text to structured data
get_json_object(json, path)     -- Extract one piece from JSON
json_extract(json, path)        -- Another way to extract from JSON
```

## Tips for Success

1. **Start Simple**: Begin with basic SELECT statements, then add complexity
2. **Test Small**: Always use LIMIT when trying new transformations
3. **Check Your Work**: Count records before and after transformations
4. **Use Comments**: Document complex logic for future reference
5. **Break It Down**: Split complex operations into multiple steps
6. **Validate Results**: Make sure your calculations make business sense

These advanced transformations help you turn raw, messy data into valuable business insights!

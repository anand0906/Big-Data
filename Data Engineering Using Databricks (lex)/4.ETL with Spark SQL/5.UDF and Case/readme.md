# User Defined Functions and Control Flow - Simple Guide

## What Are User Defined Functions (UDFs)?

Think of UDFs like creating your own calculator buttons. Instead of doing the same math over and over, you create a custom button that does it for you.

**Example**: Instead of writing `price * 0.08` everywhere to calculate tax, you create a function called `calculate_tax(price)`.

## Simple SQL Functions

### Basic Math Function

```sql
-- Create a function to calculate tip
CREATE OR REPLACE FUNCTION calculate_tip(bill_amount DOUBLE)
RETURNS DOUBLE
RETURN bill_amount * 0.18;  -- 18% tip

-- Use it
SELECT 
    customer_name,
    bill_amount,
    calculate_tip(bill_amount) as tip_amount,
    bill_amount + calculate_tip(bill_amount) as total_to_pay
FROM restaurant_bills;
```

### Text Cleaning Function

```sql
-- Create a function to clean names
CREATE OR REPLACE FUNCTION clean_name(raw_name STRING)
RETURNS STRING
RETURN TRIM(UPPER(raw_name));  -- Remove spaces and make uppercase

-- Use it to clean customer names
SELECT 
    customer_id,
    raw_name,
    clean_name(raw_name) as clean_name
FROM customers;
```

### Grade Calculator

```sql
-- Function to convert scores to letter grades
CREATE OR REPLACE FUNCTION score_to_grade(score INT)
RETURNS STRING
RETURN 
    CASE 
        WHEN score >= 90 THEN 'A'
        WHEN score >= 80 THEN 'B'
        WHEN score >= 70 THEN 'C'
        WHEN score >= 60 THEN 'D'
        ELSE 'F'
    END;

-- Apply to student scores
SELECT 
    student_name,
    test_score,
    score_to_grade(test_score) as letter_grade
FROM student_tests;
```

## Working with JSON (Web Data)

JSON is like a box with smaller boxes inside. We need to unpack it step by step.

### Simple JSON Example

```sql
-- Sample JSON data looks like: {"name": "John", "age": 25, "city": "New York"}

-- Extract information from JSON
SELECT 
    customer_id,
    json_data,
    json_data:name as customer_name,     -- Get name from JSON
    json_data:age as customer_age,       -- Get age from JSON
    json_data:city as customer_city      -- Get city from JSON
FROM customer_json_table;
```

### Shopping Cart JSON

```sql
-- JSON like: {"user": "John", "cart": ["apple", "banana"], "total": 15.99}

CREATE OR REPLACE TEMPORARY VIEW cart_data AS
SELECT 
    order_id,
    cart_json:user as shopper_name,
    cart_json:cart as items_list,        -- This is an array
    cart_json:total as cart_total
FROM online_orders;

-- Break down the shopping cart (put each item on its own row)
SELECT 
    order_id,
    shopper_name,
    explode(items_list) as individual_item,  -- Unpack the list
    cart_total
FROM cart_data;
```

## Array Functions (Working with Lists)

Arrays are like shopping lists. Here's how to work with them:

### Basic Array Operations

```sql
-- Sample data: customers have lists of purchased products

-- Count items in each customer's purchase history
SELECT 
    customer_id,
    purchased_items,                              -- Original list
    size(purchased_items) as total_items,         -- Count items
    array_distinct(purchased_items) as unique_items,  -- Remove duplicates
    size(array_distinct(purchased_items)) as unique_count
FROM customer_purchases;

-- Find customers who bought specific items
SELECT 
    customer_id,
    purchased_items
FROM customer_purchases
WHERE array_contains(purchased_items, 'laptop');  -- Who bought laptops?
```

### Filter Arrays (Smart List Filtering)

```sql
-- Keep only expensive items from each customer's list
CREATE OR REPLACE TEMPORARY VIEW expensive_purchases AS
SELECT 
    customer_id,
    purchase_history,
    filter(purchase_history, item -> item.price > 100) as expensive_items
FROM customer_detailed_purchases;

-- Show customers who bought expensive things
SELECT 
    customer_id,
    size(expensive_items) as num_expensive_items
FROM expensive_purchases
WHERE size(expensive_items) > 0;
```

## Simple Control Flow Examples

### If-Then Logic with CASE

```sql
-- Determine shipping speed based on customer type and order amount
SELECT 
    order_id,
    customer_type,
    order_amount,
    CASE 
        WHEN customer_type = 'VIP' THEN 'Next Day'
        WHEN order_amount >= 100 THEN 'Express'
        WHEN order_amount >= 50 THEN 'Standard'
        ELSE 'Economy'
    END as shipping_speed
FROM orders;
```

### Multi-Level Decision Making

```sql
-- Function for complex pricing decisions
CREATE OR REPLACE FUNCTION get_final_price(
    base_price DOUBLE, 
    customer_type STRING, 
    day_of_week STRING
)
RETURNS DOUBLE
RETURN 
    base_price * 
    -- Customer discount
    (CASE customer_type
        WHEN 'VIP' THEN 0.8      -- 20% off
        WHEN 'Regular' THEN 0.95 -- 5% off
        ELSE 1.0                 -- No discount
    END) *
    -- Day of week discount
    (CASE day_of_week
        WHEN 'Monday' THEN 0.9   -- 10% off Monday blues
        WHEN 'Friday' THEN 1.05  -- 5% markup for Friday rush
        ELSE 1.0                 -- Regular price
    END);

-- Use the pricing function
SELECT 
    product_name,
    base_price,
    customer_type,
    dayname(current_date()) as today,
    get_final_price(base_price, customer_type, dayname(current_date())) as final_price
FROM products CROSS JOIN (SELECT DISTINCT customer_type FROM customers);
```

## Python UDFs (For More Complex Logic)

When SQL gets too hard, use Python (like calling in an expert helper).

### Simple Python Function

```python
# Create a function to categorize ages
def age_group(age):
    if age is None:
        return "Unknown"
    elif age < 18:
        return "Minor"
    elif age < 30:
        return "Young Adult"
    elif age < 50:
        return "Middle Age"
    elif age < 65:
        return "Mature"
    else:
        return "Senior"

# Register so SQL can use it
spark.udf.register("age_group", age_group)
```

```sql
-- Use Python function in SQL
SELECT 
    customer_name,
    age,
    age_group(age) as age_category
FROM customers;

-- Count customers by age group
SELECT 
    age_group(age) as age_category,
    COUNT(*) as customer_count
FROM customers
GROUP BY age_group(age);
```

### Email Domain Extractor

```python
# Function to get email provider
def get_email_provider(email):
    if not email or '@' not in email:
        return "Invalid"
    
    domain = email.split('@')[1].lower()
    
    # Categorize common providers
    if domain in ['gmail.com', 'googlemail.com']:
        return "Gmail"
    elif domain in ['yahoo.com', 'yahoo.co.uk']:
        return "Yahoo"
    elif domain in ['hotmail.com', 'outlook.com', 'live.com']:
        return "Microsoft"
    else:
        return "Other"

spark.udf.register("get_email_provider", get_email_provider)
```

```sql
-- Analyze customer email providers
SELECT 
    get_email_provider(email) as email_provider,
    COUNT(*) as customer_count
FROM customers
WHERE email IS NOT NULL
GROUP BY get_email_provider(email)
ORDER BY customer_count DESC;
```

## Real-World Example: Order Processing System

Let's create a simple order processing system with custom functions.

### Step 1: Create Helper Functions

```sql
-- Function to calculate shipping cost
CREATE OR REPLACE FUNCTION shipping_cost(weight DOUBLE, distance INT)
RETURNS DOUBLE
RETURN 
    CASE 
        WHEN distance <= 50 THEN 5.99 + (weight * 0.5)
        WHEN distance <= 200 THEN 9.99 + (weight * 0.75)
        ELSE 15.99 + (weight * 1.0)
    END;

-- Function to determine if order needs approval
CREATE OR REPLACE FUNCTION needs_approval(order_amount DOUBLE, customer_type STRING)
RETURNS BOOLEAN
RETURN 
    CASE 
        WHEN customer_type = 'VIP' THEN false  -- VIP never needs approval
        WHEN order_amount > 1000 THEN true     -- Large orders need approval
        ELSE false
    END;
```

### Step 2: Process Orders

```sql
-- Create order processing view
CREATE OR REPLACE TEMPORARY VIEW processed_orders AS
SELECT 
    order_id,
    customer_id,
    product_weight,
    delivery_distance,
    order_amount,
    customer_type,
    
    -- Apply our functions
    shipping_cost(product_weight, delivery_distance) as shipping_fee,
    needs_approval(order_amount, customer_type) as requires_approval,
    
    -- Calculate final totals
    order_amount + shipping_cost(product_weight, delivery_distance) as final_total,
    
    -- Determine processing status
    CASE 
        WHEN needs_approval(order_amount, customer_type) THEN 'Pending Approval'
        ELSE 'Ready to Ship'
    END as order_status

FROM orders
WHERE order_date = current_date();

-- Show today's order summary
SELECT 
    order_status,
    COUNT(*) as order_count,
    AVG(final_total) as avg_order_value,
    SUM(final_total) as total_revenue
FROM processed_orders
GROUP BY order_status;
```

## Working with Customer Feedback

### Simple Text Analysis

```python
# Function to count positive words in reviews
def count_positive_words(review_text):
    if not review_text:
        return 0
    
    positive_words = ['good', 'great', 'excellent', 'love', 'amazing', 'perfect']
    text = review_text.lower()
    
    count = 0
    for word in positive_words:
        count += text.count(word)
    
    return count

# Function to get review length category
def review_length_category(review_text):
    if not review_text:
        return "No Review"
    
    length = len(review_text)
    if length < 50:
        return "Short"
    elif length < 200:
        return "Medium"
    else:
        return "Long"

spark.udf.register("count_positive_words", count_positive_words)
spark.udf.register("review_length_category", review_length_category)
```

```sql
-- Analyze customer reviews
SELECT 
    review_id,
    customer_id,
    star_rating,
    review_text,
    count_positive_words(review_text) as positive_word_count,
    review_length_category(review_text) as review_length,
    
    -- Simple sentiment check
    CASE 
        WHEN star_rating >= 4 AND count_positive_words(review_text) >= 2 THEN 'Very Positive'
        WHEN star_rating >= 3 THEN 'Positive'
        WHEN star_rating = 2 THEN 'Mixed'
        ELSE 'Negative'
    END as overall_sentiment

FROM customer_reviews;
```

## Complete Example: Daily Sales Report Generator

```sql
-- Function to categorize sales performance
CREATE OR REPLACE FUNCTION categorize_sales_day(
    sales_amount DOUBLE, 
    transaction_count INT
)
RETURNS STRING
RETURN 
    CASE 
        WHEN sales_amount >= 10000 AND transaction_count >= 50 THEN 'Excellent'
        WHEN sales_amount >= 5000 AND transaction_count >= 25 THEN 'Good'
        WHEN sales_amount >= 2000 AND transaction_count >= 10 THEN 'Average'
        ELSE 'Below Target'
    END;

-- Create daily sales report
CREATE OR REPLACE TEMPORARY VIEW daily_sales_report AS
SELECT 
    DATE(sale_timestamp) as sale_date,
    COUNT(*) as total_transactions,
    SUM(sale_amount) as daily_revenue,
    AVG(sale_amount) as avg_transaction,
    COUNT(DISTINCT customer_id) as unique_customers,
    
    -- Use our function to categorize the day
    categorize_sales_day(SUM(sale_amount), COUNT(*)) as day_performance,
    
    -- Calculate customer metrics
    SUM(sale_amount) / COUNT(DISTINCT customer_id) as revenue_per_customer,
    
    -- Weekend vs weekday
    CASE WHEN DAYOFWEEK(DATE(sale_timestamp)) IN (1,7) 
         THEN 'Weekend' 
         ELSE 'Weekday' 
    END as day_type

FROM daily_sales
WHERE sale_timestamp >= current_date() - 7  -- Last 7 days
GROUP BY DATE(sale_timestamp);

-- Summary report
SELECT 
    sale_date,
    day_type,
    day_performance,
    total_transactions,
    CONCAT('$', FORMAT_NUMBER(daily_revenue, 2)) as formatted_revenue,
    unique_customers,
    CONCAT('$', FORMAT_NUMBER(revenue_per_customer, 2)) as revenue_per_customer
FROM daily_sales_report
ORDER BY sale_date DESC;
```

## Simple Control Flow Patterns

### Basic If-Then Logic

```sql
-- Simple decision function for customer service priority
CREATE OR REPLACE FUNCTION get_service_priority(
    customer_type STRING, 
    issue_severity STRING
)
RETURNS STRING
RETURN 
    CASE 
        WHEN customer_type = 'VIP' THEN 'High'
        WHEN issue_severity = 'Critical' THEN 'High'
        WHEN issue_severity = 'Major' THEN 'Medium'
        ELSE 'Normal'
    END;

-- Apply to customer service tickets
SELECT 
    ticket_id,
    customer_type,
    issue_description,
    issue_severity,
    get_service_priority(customer_type, issue_severity) as priority_level
FROM support_tickets
WHERE status = 'Open'
ORDER BY 
    CASE get_service_priority(customer_type, issue_severity)
        WHEN 'High' THEN 1
        WHEN 'Medium' THEN 2
        ELSE 3
    END;
```

### Validation Function

```sql
-- Function to check if email is valid
CREATE OR REPLACE FUNCTION is_valid_email(email STRING)
RETURNS BOOLEAN
RETURN 
    CASE 
        WHEN email IS NULL THEN false
        WHEN email NOT LIKE '%@%' THEN false
        WHEN email NOT LIKE '%.%' THEN false
        WHEN LENGTH(email) < 5 THEN false
        ELSE true
    END;

-- Clean customer data using validation
SELECT 
    customer_id,
    email,
    is_valid_email(email) as email_is_good,
    CASE 
        WHEN is_valid_email(email) THEN 'Keep'
        ELSE 'Needs Fixing'
    END as action_needed
FROM customers;
```

## Working with Lists (Arrays)

### Simple List Operations

```sql
-- Sample data: customers with lists of purchases
-- customer_id | purchased_items
-- 1           | ["apple", "banana", "apple", "orange"]
-- 2           | ["laptop", "mouse", "keyboard"]

-- Count unique items per customer
SELECT 
    customer_id,
    purchased_items,
    size(purchased_items) as total_items,
    size(array_distinct(purchased_items)) as unique_items
FROM customer_purchase_lists;

-- Find customers who bought specific items
SELECT 
    customer_id,
    purchased_items
FROM customer_purchase_lists
WHERE array_contains(purchased_items, 'laptop');
```

### Unpack Lists with EXPLODE

```sql
-- Put each purchased item on its own row
SELECT 
    customer_id,
    explode(purchased_items) as individual_item
FROM customer_purchase_lists;

-- Count how many customers bought each item
SELECT 
    individual_item as product,
    COUNT(DISTINCT customer_id) as customers_who_bought
FROM (
    SELECT 
        customer_id,
        explode(purchased_items) as individual_item
    FROM customer_purchase_lists
)
GROUP BY individual_item
ORDER BY customers_who_bought DESC;
```

## Python UDFs for Simple Tasks

### Basic Text Processing

```python
# Simple function to clean phone numbers
def clean_phone(phone):
    if not phone:
        return None
    
    # Keep only numbers
    clean = ''.join(char for char in phone if char.isdigit())
    
    # Format as (XXX) XXX-XXXX if we have 10 digits
    if len(clean) == 10:
        return f"({clean[:3]}) {clean[3:6]}-{clean[6:]}"
    else:
        return clean

# Register for SQL use
spark.udf.register("clean_phone", clean_phone)
```

```sql
-- Clean up phone numbers in customer data
SELECT 
    customer_name,
    phone_number as original,
    clean_phone(phone_number) as formatted_phone
FROM customers
WHERE phone_number IS NOT NULL;
```

### Simple Business Logic

```python
# Function to calculate employee bonus
def calculate_bonus(sales_amount, years_employed, performance_rating):
    if not all([sales_amount, years_employed, performance_rating]):
        return 0
    
    # Base bonus percentage
    base_bonus = 0.02  # 2%
    
    # Performance multiplier
    if performance_rating >= 4.5:
        performance_multiplier = 1.5
    elif performance_rating >= 4.0:
        performance_multiplier = 1.2
    elif performance_rating >= 3.5:
        performance_multiplier = 1.0
    else:
        performance_multiplier = 0.5
    
    # Tenure bonus
    tenure_bonus = min(years_employed * 0.001, 0.01)  # Max 1% extra
    
    # Calculate final bonus
    total_bonus_rate = (base_bonus + tenure_bonus) * performance_multiplier
    return round(sales_amount * total_bonus_rate, 2)

spark.udf.register("calculate_bonus", calculate_bonus)
```

```sql
-- Calculate employee bonuses
SELECT 
    employee_name,
    total_sales,
    years_with_company,
    performance_score,
    calculate_bonus(total_sales, years_with_company, performance_score) as bonus_amount,
    total_sales + calculate_bonus(total_sales, years_with_company, performance_score) as total_compensation
FROM employee_performance
WHERE total_sales > 0
ORDER BY bonus_amount DESC;
```

## Practical Examples

### 1. Student Grade Calculator

```sql
-- Function to calculate final grade
CREATE OR REPLACE FUNCTION final_grade(
    homework_avg DOUBLE,
    midterm DOUBLE,
    final_exam DOUBLE,
    participation DOUBLE
)
RETURNS STRING
RETURN 
    CASE 
        WHEN (homework_avg * 0.3 + midterm * 0.3 + final_exam * 0.3 + participation * 0.1) >= 90 THEN 'A'
        WHEN (homework_avg * 0.3 + midterm * 0.3 + final_exam * 0.3 + participation * 0.1) >= 80 THEN 'B'
        WHEN (homework_avg * 0.3 + midterm * 0.3 + final_exam * 0.3 + participation * 0.1) >= 70 THEN 'C'
        WHEN (homework_avg * 0.3 + midterm * 0.3 + final_exam * 0.3 + participation * 0.1) >= 60 THEN 'D'
        ELSE 'F'
    END;

-- Calculate grades for all students
SELECT 
    student_name,
    homework_average,
    midterm_score,
    final_exam_score,
    participation_score,
    final_grade(homework_average, midterm_score, final_exam_score, participation_score) as letter_grade
FROM student_scores;
```

### 2. Inventory Alert System

```sql
-- Function to determine reorder status
CREATE OR REPLACE FUNCTION check_reorder_status(
    current_stock INT,
    monthly_sales INT,
    lead_time_days INT
)
RETURNS STRING
RETURN 
    CASE 
        WHEN current_stock <= 0 THEN 'OUT_OF_STOCK'
        WHEN current_stock <= (monthly_sales / 30 * lead_time_days) THEN 'URGENT_REORDER'
        WHEN current_stock <= (monthly_sales / 30 * lead_time_days * 1.5) THEN 'REORDER_SOON'
        ELSE 'STOCK_OK'
    END;

-- Check inventory status
SELECT 
    product_name,
    current_inventory,
    avg_monthly_sales,
    supplier_lead_time,
    check_reorder_status(current_inventory, avg_monthly_sales, supplier_lead_time) as reorder_status,
    
    -- Calculate days until stockout
    CASE 
        WHEN avg_monthly_sales > 0 
        THEN current_inventory / (avg_monthly_sales / 30.0)
        ELSE 999
    END as days_until_stockout

FROM inventory_levels
WHERE check_reorder_status(current_inventory, avg_monthly_sales, supplier_lead_time) != 'STOCK_OK'
ORDER BY days_until_stockout;
```

### 3. Customer Communication Preferences

```python
# Function to determine best contact method
def best_contact_method(has_email, has_phone, email_opens, sms_opens, age):
    if not has_email and not has_phone:
        return "Mail"
    
    # Younger customers prefer different methods
    if age and age < 30:
        if has_phone and sms_opens > 50:  # Good SMS response
            return "SMS"
        elif has_email:
            return "Email"
        else:
            return "Phone"
    
    # Older customers
    elif age and age >= 60:
        if has_phone:
            return "Phone"
        elif has_email and email_opens > 30:
            return "Email"
        else:
            return "Mail"
    
    # Middle-aged customers
    else:
        if has_email and email_opens > 40:
            return "Email"
        elif has_phone:
            return "Phone"
        else:
            return "Mail"

spark.udf.register("best_contact_method", best_contact_method)
```

```sql
-- Determine how to contact each customer
SELECT 
    customer_id,
    customer_name,
    age,
    has_email,
    has_phone_number,
    email_open_rate,
    sms_response_rate,
    best_contact_method(
        has_email, 
        has_phone_number, 
        email_open_rate, 
        sms_response_rate, 
        age
    ) as preferred_contact_method
FROM customer_communication_data;

-- Plan marketing campaigns by contact method
SELECT 
    best_contact_method(has_email, has_phone_number, email_open_rate, sms_response_rate, age) as contact_method,
    COUNT(*) as customer_count,
    AVG(age) as avg_customer_age
FROM customer_communication_data
GROUP BY best_contact_method(has_email, has_phone_number, email_open_rate, sms_response_rate, age);
```

## Quick Tips for Success

### 1. Start Simple
```sql
-- Good: Simple, clear function
CREATE OR REPLACE FUNCTION add_tax(price DOUBLE)
RETURNS DOUBLE
RETURN price * 1.08;

-- Avoid: Overly complex function that's hard to understand
```

### 2. Test Your Functions
```sql
-- Always test with simple examples first
SELECT add_tax(100) as result;  -- Should return 108

-- Test with edge cases
SELECT add_tax(0) as zero_test;     -- Should return 0
SELECT add_tax(NULL) as null_test;  -- Should return NULL
```

### 3. Use Descriptive Names
```sql
-- Good function names
CREATE FUNCTION calculate_shipping_cost(...)
CREATE FUNCTION is_valid_email(...)
CREATE FUNCTION get_customer_tier(...)

-- Avoid unclear names
CREATE FUNCTION func1(...)  -- What does this do?
CREATE FUNCTION calc(...)   -- Calculate what?
```

### 4. Handle Edge Cases
```sql
-- Always check for NULL values
CREATE OR REPLACE FUNCTION safe_divide(a DOUBLE, b DOUBLE)
RETURNS DOUBLE
RETURN 
    CASE 
        WHEN b IS NULL OR b = 0 THEN NULL
        ELSE a / b
    END;
```

## Function Reference Cheat Sheet

### Common SQL Function Patterns
```sql
-- Mathematical calculations
CREATE FUNCTION calculate_X(input) RETURN input * factor;

-- Text processing  
CREATE FUNCTION clean_X(text) RETURN UPPER(TRIM(text));

-- Classification/categorization
CREATE FUNCTION categorize_X(value) RETURN CASE WHEN ... END;

-- Validation
CREATE FUNCTION is_valid_X(input) RETURN CASE WHEN ... END;

-- Date/time processing
CREATE FUNCTION get_X_from_date(date) RETURN YEAR(date);
```

### Common Array Operations
```sql
size(array)              -- Count items
explode(array)           -- Each item gets own row  
array_contains(array, item)  -- Check if item exists
array_distinct(array)    -- Remove duplicates
filter(array, condition) -- Keep only items matching condition
```

Remember: Functions are tools that make your work easier. Start with simple ones, test them thoroughly, and gradually build more complex logic as you get comfortable!

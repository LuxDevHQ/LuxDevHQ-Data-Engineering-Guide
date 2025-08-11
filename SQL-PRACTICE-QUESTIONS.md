# SQL Practice Questions

## **Section 1: SELECT + ORDER BY + LIMIT + OFFSET**
1. Retrieve all columns from `customer_info` and sort results **alphabetically** by `full_name`.
2. Get the top **5 most expensive products** from `products`.
3. Display products from row **6 to 10** when ordered by `price` in descending order.

## **Section 2: WHERE + CASE**
1. Find all customers located in **Kisumu**.
2. List products priced between **100 and 500**, and add a column called `price_category` that says `"Low"` if price < 300, `"Medium"` if 300–1000, else `"High"`.
3. Get all sales where `total_sales` is greater than **1000**, and show `"Big Sale"` or `"Small Sale"` using `CASE`.

## **Section 3: JOIN + ORDER BY**
1. Show `sales_id`, `product_name`, and `full_name` for every sale, ordered by `total_sales` in descending order.
2. List all products along with their customer's `location`, sorted by location then product name.
3. Display all sales with `product_name`, `price`, and `full_name`, ordered by `price` from highest to lowest.

## **Section 4: GROUP BY + HAVING**
1. Count how many products each customer owns, and only show customers with **more than 2 products**.
2. Find the total sales for each product and only include products with sales totaling **over 2000**.
3. Get the number of customers in each location, sorted by **customer count** in descending order.

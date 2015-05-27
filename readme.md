### How many users are there?

  - 50
  
```
  SELECT COUNT(*) FROM users;
```

###  What are the 5 most expensive items?

  1) Small Cotton Gloves
  
  2) Small Wooden Computer
  
  3) Awesome Granite Pants
  
  4) Sleek Wooden Hat
  
  5) Ergonomic Steel Car
  
```
  SELECT title FROM items ORDER BY -price LIMIT(5);
```

###  What’s the cheapest book? (Does that change for “category is exactly ‘book’” versus “category contains ‘book’”?)

- Ergonomic Granite Chair
  
```
  SELECT title FROM items WHERE category LIKE "%Books%" ORDER BY price LIMIT (1);
```

- No, Ergonomic Granite Chair is the cheapest book for both cases:

```
  SELECT title FROM items WHERE category LIKE "%Books%" ORDER BY price LIMIT (1);
```
```
  SELECT title FROM items WHERE category = "Books" ORDER BY price LIMIT (1);
```

### Who lives at “6439 Zetta Hills, Willmouth, WY”? Do they have another address?

  - Corrine Little
  
```
  SELECT first_name, last_name FROM users WHERE id = (SELECT user_id FROM addresses WHERE street = "6439 Zetta Hills");
```

  - yes, "54369 Wolff Forges, Lake Bryon, CA"

```
  SELECT * FROM addresses WHERE user_id = (SELECT user_id FROM addresses WHERE street = "6439 Zetta Hills");
```


### Correct Virginie Mitchell’s address to “New York, NY, 10108”.

```
 UPDATE addresses SET city = "New York", zip = 10108 WHERE user_id = (SELECT id FROM users WHERE first_name = "Virginie" AND last_name = "Mitchell") AND state = "NY";
```

### How much would it cost to buy one of each tool?

  - 46,477

```
  SELECT SUM(price) FROM items WHERE category LIKE "%Tools%";
```


### How many total items did we sell?

  - 2125

```
  SELECT SUM(quantity) FROM orders;
```


### How much was spent on books?

  - 1,081,352
  
```
  SELECT SUM(order_total) FROM (SELECT item_id, price, quantity, (price * quantity) AS "order_total" FROM (SELECT item_id, quantity FROM orders) AS A, (SELECT id, price FROM items WHERE category LIKE "%Books%") AS B WHERE A.item_id = B.id);
```

### Simulate buying an item by inserting a User for yourself and an Order for that User.

- new user Reid Paape bought 1000 pairs of Rustic Plastic Gloves

```
  INSERT INTO users (first_name, last_name, email) VALUES ("Reid", "Paape", "wrpaape@gmail.com");
```
```
  INSERT INTO orders (user_id, item_id, quantity, created_at) VALUES ((SELECT id FROM users WHERE email = "wrpaape@gmail.com"), (SELECT id FROM items WHERE title = "Rustic Plastic Gloves"), 1000, CURRENT_TIMESTAMP);
```

--------------------------------------------


### What item was ordered most often?

- Incredible Granite Car (72 times)

```
  SELECT title FROM items WHERE id = (SELECT item_id FROM (SELECT A1.item_id, SUM(A1.quantity) AS "total_sales" FROM (SELECT item_id, quantity FROM orders) AS A1 GROUP BY A1.item_id) ORDER BY -total_sales LIMIT(1));
```

### Grossed the most money?

- Incredible Granite Car (525,240)

```
  SELECT title FROM items, (SELECT A1.item_id, SUM(A1.quantity) AS "total_sales" FROM (SELECT item_id, quantity FROM orders) AS A1 GROUP BY A1.item_id) WHERE items.id = item_id ORDER BY -(price * total_sales) LIMIT (1);
```

### What user spent the most?

- Hassan Runte (639,386)

```
  SELECT first_name, last_name FROM (SELECT A1.first_name, A1.last_name, SUM(A1.order_total) AS "total_sales" FROM (SELECT first_name, last_name, (price * quantity) AS "order_total" FROM items, (SELECT first_name, last_name, item_id, quantity FROM users, orders WHERE user_id = users.id) WHERE item_id = items.id) AS A1 GROUP BY A1.first_name) ORDER BY -total_sales LIMIT(1);
```

### What were the top 3 highest grossing categories?

  1) Sports (2,003,693)
  
  2) Health (1,769,821)
  
  3) Beauty (1,767,839)

  - first created a new table 'sales_by_category' with a column for id, category (called 'cat'), and sales
  - manually entered 22 seperate categories that were appeared in the 'category' column of 'items' and corresponded to the items sold in the orders
  - made 22 updates using the following entry:

```
  UPDATE sales_by_category SET sales = (SELECT sum(gross_sales) FROM (SELECT category, (price * total_sales) AS "gross_sales" FROM items, (SELECT A1.item_id, SUM(A1.quantity) AS "total_sales" FROM (SELECT item_id, quantity FROM orders) AS A1 GROUP BY A1.item_id) WHERE items.id = item_id ORDER BY -(price * total_sales)) WHERE category LIKE "%CAT_NAME%") WHERE cat = "CAT_NAME";
```

  - where CAT_NAME corresponded to an entry in the 'cat' column of 'sales_by_category'

  - then ordered the resulting table by -sales and limited the selection to the top three:

```
  SELECT cat, sales FROM sales_by_category ORDER BY -sales LIMIT(3);
```



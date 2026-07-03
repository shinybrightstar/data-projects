# How I Used SQL to Answer Three Business Questions

I had a database for a scale-model car retailer — eight tables covering customers, orders, order details, payments, products, product lines, employees, and offices. My job was to turn that raw transactional data into three decisions the business could actually act on: what to restock, who to market to, and how much to spend acquiring new customers. Here's how I got there, queries and all.

## Question 1: What should we order more or less of?

I didn't want a list of "best-selling products" on its own — that's only half the picture. A product can sell well and still not need reordering if the warehouse is already full of it. So I built two separate measures and then combined them.

First, **low stock**, using a correlated subquery that compares each product's current stock against how much of it has actually been ordered:

```sql
SELECT productCode,
       ROUND(quantityInStock / (
           SELECT SUM(quantityOrdered)
           FROM orderdetails od
           WHERE od.productCode = p.productCode
       ), 2) AS low_stock
FROM products p
GROUP BY productCode
ORDER BY low_stock ASC
LIMIT 10;
```

Second, **product performance**, based on total revenue:

```sql
SELECT productCode,
       SUM(quantityOrdered * priceEach) AS product_performance
FROM orderdetails
GROUP BY productCode
ORDER BY product_performance DESC
LIMIT 10;
```

Then I combined both into a CTE and used `IN` to keep only the products that showed up on *both* lists — strong sellers that were also genuinely at risk of running out:

```sql
WITH LowStock AS (
    SELECT productCode,
           ROUND(quantityInStock / (
               SELECT SUM(quantityOrdered)
               FROM orderdetails od
               WHERE od.productCode = p.productCode
           ), 2) AS low_stock
    FROM products p
    GROUP BY productCode
    ORDER BY low_stock ASC
    LIMIT 10
),
Performance AS (
    SELECT productCode,
           SUM(quantityOrdered * priceEach) AS product_performance
    FROM orderdetails
    GROUP BY productCode
    ORDER BY product_performance DESC
    LIMIT 10
)
SELECT productCode, productName, productLine
FROM products
WHERE productCode IN (SELECT productCode FROM LowStock)
  AND productCode IN (SELECT productCode FROM Performance);
```

The result surprised me a little. I expected the priority list to be dominated by the "Classic Cars" line, since that's usually the flagship category. Instead, I got a real mix:

| Product | Product Line |
|---|---|
| 1968 Ford Mustang | Classic Cars |
| 1911 Ford Town Car | Vintage Cars |
| 1928 Mercedes-Benz SSK | Vintage Cars |
| 1960 BSA Gold Star DBD34 | Motorcycles |
| 1997 BMW F650 ST | Motorcycles |
| 1928 Ford Phaeton Deluxe | Vintage Cars |
| 2002 Yamaha YZR M1 | Motorcycles |
| The Mayflower | Ships |
| F/A 18 Hornet 1/72 | Planes |
| Pont Yacht | Ships |

Vintage Cars and Motorcycles each show up three times, and Ships makes two appearances. That tells me the demand isn't concentrated in one "hero" category — it's spread across the collector-niche lines. If I were handing this to a buyer, I'd tell them to stop assuming Classic Cars is the safe default and start paying closer attention to Vintage Cars and Motorcycles inventory specifically.

## Question 2: How should I tailor marketing to customer behavior?

Revenue tells you what a customer bought. Profit tells you what they were actually worth. So instead of ranking customers by how much they spent, I calculated profit per customer:

```sql
SELECT o.customerNumber, contactFirstName,
       SUM(quantityOrdered * (priceEach - buyPrice)) AS profit
FROM products p
INNER JOIN orderdetails od ON p.productCode = od.productCode
INNER JOIN orders o ON o.orderNumber = od.orderNumber
INNER JOIN customers c ON c.customerNumber = o.customerNumber
GROUP BY o.customerNumber
ORDER BY profit DESC
LIMIT 5;
```

Then I wrapped that logic in a CTE and joined it back to the `customers` table to pull the human details behind the top five — VIP customers by profit:

```sql
WITH ProfitPerCustomer AS (
    SELECT o.customerNumber,
           SUM(quantityOrdered * (priceEach - buyPrice)) AS profit
    FROM products p
    INNER JOIN orderdetails od ON p.productCode = od.productCode
    INNER JOIN orders o ON o.orderNumber = od.orderNumber
    GROUP BY o.customerNumber
)
SELECT c.contactLastName, c.contactFirstName, c.city, c.country, ppc.profit
FROM customers c
JOIN ProfitPerCustomer ppc ON c.customerNumber = ppc.customerNumber
ORDER BY ppc.profit DESC
LIMIT 5;
```

| Customer | Location | Profit |
|---|---|---|
| Diego Freyre | Madrid, Spain | $326,519.66 |
| Susan Nelson | San Rafael, USA | $236,769.39 |
| Jeff Young | NYC, USA | $72,370.09 |
| Peter Ferguson | Melbourne, Australia | $70,311.07 |
| Janine Labrune | Nantes, France | $60,875.30 |

What jumped out to me right away is the gap between #1 and #2 versus #3 through #5. Diego Freyre alone generates more profit than the bottom three VIPs combined. That's not a "top five, treat them equally" situation — that's a "protect Diego and Susan like your business depends on it, because it kind of does" situation.

For the least-engaged customers, I ran the same CTE and just flipped the sort order:

```sql
WITH ProfitPerCustomer AS (
    SELECT o.customerNumber,
           SUM(quantityOrdered * (priceEach - buyPrice)) AS profit
    FROM products p
    INNER JOIN orderdetails od ON p.productCode = od.productCode
    INNER JOIN orders o ON o.orderNumber = od.orderNumber
    GROUP BY o.customerNumber
)
SELECT c.contactLastName, c.contactFirstName, c.city, c.country, ppc.profit
FROM customers c
JOIN ProfitPerCustomer ppc ON c.customerNumber = ppc.customerNumber
ORDER BY ppc.profit ASC
LIMIT 5;
```

| Customer | Location | Profit |
|---|---|---|
| Mary Young | Glendale, USA | $2,610.87 |
| Leslie Taylor | Brickhaven, USA | $6,586.02 |
| Franco Ricotti | Milan, Italy | $9,532.93 |
| Carine Schmitt | Nantes, France | $10,063.80 |
| Thomas Smith | London, UK | $10,868.04 |

These aren't customers who've never bought anything — they're customers generating a fraction of what the VIPs bring in. That's a re-engagement segment, not a lost cause: a targeted win-back offer costs far less than acquiring someone brand new, so this is where I'd point a retention campaign before writing anyone off.

## Question 3: How much can I spend acquiring a new customer?

I averaged profit across every customer in the database to get a lifetime value (LTV) figure:

```sql
SELECT AVG(profit) AS ltv
FROM (
    SELECT o.customerNumber,
           SUM(quantityOrdered * (priceEach - buyPrice)) AS profit
    FROM products p
    JOIN orderdetails od ON p.productCode = od.productCode
    JOIN orders o ON o.orderNumber = od.orderNumber
    GROUP BY o.customerNumber
) AS customer_profits;
```

**ltv = $39,039.59**

That number matters because it's a ceiling, not just a stat. If I bring in ten new customers next month and each one performs like an average existing customer, that's roughly **$390,395.94** in future profit. So whatever I spend acquiring those ten customers, it needs to stay comfortably under that number for the acquisition to actually pay off. LTV turns "how much should we spend on marketing?" from a guess into a number I can defend.

## Conclusion

Across all three questions, the same pattern held: raw totals hide the decision, and the decision only shows up once you compare two things against each other that is; sales against stock, revenue against cost, one customer against another. Using correlated subqueries and CTEs, I turned eight tables of transactional data into three concrete recommendations: prioritize restocking Vintage Cars and Motorcycles over the assumed default of Classic Cars, protect a small handful of high-profit customers while running a targeted win-back campaign on the low-profit segment instead of writing them off, and treat roughly $39,040 in average customer LTV as the practical ceiling for what we can spend to acquire someone new.

What I liked about this project is that none of the three answers required anything outside the data that was already sitting there — just the right joins, the right grouping, and enough restraint to compare metrics instead of reporting them in isolation.

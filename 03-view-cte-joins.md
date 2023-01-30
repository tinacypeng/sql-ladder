# Views, CTEs, and Joins

This file contains several exercises on views, common table expressions (CTEs), and joins. These topics are a step more involved and confusing than in the previous two files. Before attempting these problems, you'll probably need to do some research into these topics. Here are a few tutorials:

* [Tutorial on Views](https://www.sqlitetutorial.net/sqlite-create-view/)
* [Tutorial on CTEs](https://www.essentialsql.com/introduction-common-table-expressions-ctes/)
* [Tutorial on Joins](https://www.sqlitetutorial.net/sqlite-join/)

Here are a few other important notes I'd like you read before beginning:
* Some of these problems can get pretty involved. Queries with adequate spacing can go longer than 15 lines in some problems.
* Make sure you read each question thoroughly.
* Don't skip problems, as some problems may rely on previous problems being done correctly.
* Make sure you are saving your answers as you go, as some answers will simply be reworkings of previous answers.
* Some problems in the `Views` section involve you making persistent changes to the database itself. Specifically, if you create a view, you cannot create another view with the same name if it already exists. You may want to learn the `DROP VIEW` command. Or maybe a little easier,  you can recreate the entire database by simply running the `seed.py` script provided in this repo. If you're unsure, you can see your list of tables and views using the `.tables` dot-command from the `sqlite3` interface.

## Views
62) Look at the `yum` table. It is the stock data for Yum! Brands, Inc. from 2015 through 2019. Yum! is the company that owns Taco Bell, the best restaurant.
	```sql
	SELECT *
	FROM `yum` LIMIT 10
	```
63) Query the `yum` table, aggregating by **both** month and year, with the following resulting columns:
* Year (4 digits)
* Month
* Average open, high, low, and close
* Total volume
Finally, sort this data so it's in proper chronological order.
	```sql
	SELECT 
		strftime('%Y', date) AS Year,
		strftime('%m', date) AS Month,
		ROUND(avg(open), 2) AS avg_open,
		ROUND(avg(high), 2) AS avg_high,
		ROUND(avg(low), 2) AS avg_low,
		ROUND(avg(close), 2) AS avg_close,
		sum(volume) AS total_vol
	FROM `yum`
	GROUP BY Year, Month
	```
64) Save the results of the previous query as a view named `yum_by_month`.
	```sql
	CREATE VIEW view_by_month
	AS 
	SELECT 
		strftime('%Y', date) AS Year,
		strftime('%m', date) AS Month,
		ROUND(avg(open), 2) AS avg_open,
		ROUND(avg(high), 2) AS avg_high,
		ROUND(avg(low), 2) AS avg_low,
		ROUND(avg(close), 2) AS avg_close,
		sum(volume) AS total_vol
	FROM `yum`
	GROUP BY Year, Month
	```
65) Create a view of `transactions` consisting of only three columns: year, month, and total sales in that month. Call this view `trans_by_month`.
	```sql
	CREATE VIEW trans_by_month
	AS 
	SELECT 
		strftime('%Y', orderdate) AS year,
		strftime('%m', orderdate) AS month,
		sum(unit_price * quantity) AS total_sales
	FROM `transactions`
	GROUP BY year, month
	```
66) Create a view of `transactions` consisting of only two columns: `employee_id` and the total sales corresponding to that employee. Call this view `trans_by_employee`.
	```sql
	CREATE VIEW trans_by_employee
	AS 
	SELECT 
		employee_id,
		sum(unit_price * quantity) AS total_sales
	FROM `transactions`
	GROUP BY employee_id
	```

## Common Table Expressions (CTEs)
CTEs are a convenient way of shortening SQL queries to keep your code DRY (**d**on't **r**epeat **y**ourself). You'll notice they're essentially the same in terms of the tasks they can accomplish, however CTEs are _temporary_. They vanish after the query has been called. Essentially, CTEs are single-use views.

Therefore, CTEs aren't needed to solve any of the following problems. You could use a view instead, however that would be wasteful since you'll never use them again. Additionally, for some problems, neither a view nor a CTE is truly needed, but the query would be very messy without one.

67) What's the most common first initial for pets in the `pets` table?
    * _Hint:_ Create a CTE that is simply the lowercased first letter of the pet's name. The solution is a simple `GROUP BY` from this CTE.
    * _Hint 2:_ You'll need the `SUBSTR()` and `LOWER()` functions.
	```sql
	SELECT 
		lower(substr(name, 1, 1)) AS first_initial,
		count(*) AS num_pet
	FROM `pets`
	GROUP BY first_initial
	ORDER BY num_pet DESC
	LIMIT 1
	```
68) Create taglines for each employee in the `employees` table. As a template, the first row of the result should look like this:

```
Christine Thompson started in 2005 and makes $123,696 working in sales. 
```
To do this easily, make a CTE featuring name (firstname + " " + lastname), job, salary (formatted), and year. Job title should be lowercased, _unless_ it is IT, in which case leave it capitalized. The solution is simple string concatenation off of this long CTE.
```sql
WITH prep AS (
SELECT 
	firstname || ' ' || lastname AS full_name,
	job,
	printf('$%,.2d', salary) AS salary,
	strftime('%Y', startdate) AS startyear
FROM `employees`)
SELECT
	full_name || ' started in ' || startyear || ' and makes ' || salary || ' working in ' || 
	CASE 
		WHEN job = 'IT' THEN job 
		ELSE lower(job) 
	END 
	|| '.' AS taglines
FROM prep
```
69) How many of our sales come from companies ending in each of "LLC", "Inc", "Ltd", or "PLC"? In a CTE, create a `company_type` column of values `"LLC"`, `"Inc"`, `"Ltd"`, `"PLC"`, or `"Other"`. Outside the CTE, find the total revenue from these categories, as well as their respective counts.
* _Hint:_ You'll need the `INSTR()` function.
	```sql
	WITH prep AS
	(SELECT 
		customer,
		CASE
			WHEN substr(customer, -3) in ('LLC', 'Inc', 'Ltd', 'PLC') THEN substr(customer, -3)
			ELSE 'Other'
		END AS company_type,
		unit_price*quantity AS revenue
	FROM `transactions`)

	SELECT 
		company_type,
		sum(revenue) AS total_revenue,
		count(*) AS total_trans
	FROM prep
	GROUP BY company_type
	```

## Joins
No, we're not done talking about views and CTEs! We're just going to start intermingling them in with further examples on joins, where the real power of these techniques becomes clearer.

70) Which employee made which sale? Join the `employees` table onto the `transactions` table by `employee_id`. You only need to include the employee's first/last name from `employees`.
	```sql
	SELECT 
		t.*, e.firstname, e.lastname
	FROM `transactions` AS t
	LEFT JOIN `employees` AS e
	ON t.employee_id = e.ID
	```
71) What is the name of the employee who made the most in sales? Find this answer by doing a join as in the previous problem. Your resulting query will be difficult for someone else to read.
	```sql
	SELECT 
		e.firstname, e.lastname,
		sum(unit_price*quantity) AS total_sales
	FROM `transactions` AS t
	LEFT JOIN `employees` AS e
	ON t.employee_id = e.ID
	GROUP BY employee_id
	ORDER BY total_sales DESC
	LIMIT 1
	```
72) Solve the previous problem by joining `employees` onto the `trans_by_employee` view you made earlier.
	```sql
	SELECT 
		e.firstname, e.lastname,
		total_sales
	FROM `trans_by_employee` AS t
	LEFT JOIN `employees` AS e
	ON t.employee_id = e.ID
	ORDER BY total_sales DESC
	LIMIT 1
	```
73) Solve the previous problem by joining `employees` onto a CTE.
	```sql
	WITH trans_by_employee_temp AS 
	(SELECT 
		employee_id,
		sum(unit_price * quantity) AS total_sales
	FROM `transactions`
	GROUP BY employee_id)

	SELECT 
		e.firstname, e.lastname, te.total_sales
	FROM trans_by_employee_temp as te
	LEFT JOIN `employees` AS e
	ON te.employee_id = e.ID
	ORDER BY total_sales DESC
	LIMIT 1
	```
74) Next, the company will try to give bonuses based on performance. Show all employees who've made more in sales than 1.5 times their salary. (You may use whatever technique you'd like to do the join: view, CTE, or even a subquery!)
	```sql
	WITH trans_by_employee_temp AS 
	(SELECT 
		employee_id,
		sum(unit_price * quantity) AS total_sales
	FROM `transactions`
	GROUP BY employee_id)

	SELECT 
		e.firstname, e.lastname
	FROM trans_by_employee_temp as te
	LEFT JOIN `employees` AS e
	ON te.employee_id = e.ID
	WHERE te.total_sales > e.salary*1.5
	```
75) Do we have potentially erroneous rows? Find all transactions which occurred _before_ the employee was even hired! (Make sure each transaction only occupies one row).
	```sql
	SELECT 
		DISTINCT order_id
	FROM `transactions` as t
	LEFT JOIN `employees` AS e
	ON t.employee_id = e.ID
	WHERE orderdate < startdate
	```
76) Among all transactions that occurred from 2015 to 2019, create a table that is the monthly revenue of our company versus the total trading volume of Yum! in that month. Format the columns nicely. That is, a sample row of your result might look like this:

```
| year | month | company_revenue | yum_trade_volume |
|------|-------|-----------------|------------------|
| 2017 |    03 |        $100,000 |      125,000,000 |
```

* _Hint:_ You don't need any `WHERE` statements here. You can get the right answer simply by changing what kind of join you do!
	```sql
	SELECT 
		t.year, t.month,
		printf('$%,.2d',total_cost) AS company_revenue,
		printf('%,.2d',y.tot_volume) AS yum_trade_volume
	FROM `trans_by_month` AS t
	INNER JOIN `yum_by_month` AS y
	ON t.year = y.year AND t.month = y.month
	```

77) Repeat the previous problem, but in addition to the total volume, include:
* The lowest price that month (ie, lowest low)
* The highest price that month (ie, highest high)
	```sql
	WITH yum_addition_info AS
	(SELECT 
		strftime('%Y', date) AS Year,
		strftime('%m', date) AS Month,
		max(high) AS max_high,
		min(low) AS min_low,
		sum(volume) AS total_vol
	FROM `yum`
	GROUP BY Year, Month)

	SELECT 
		t.year, t.month,
		printf('$%,.2d',total_cost) AS company_revenue,
		printf('%,.2d',y_a.total_vol) AS yum_total_volume,
		printf('$%,.2f', y_a.max_high) AS yum_max_high,
		printf('$%,.2f', y_a.min_low) AS yum_min_low
	FROM `trans_by_month` AS t
	INNER JOIN yum_addition_info AS y_a
	ON t.year = y_a.year AND t.month = y_a.month
	LIMIT 50
	```


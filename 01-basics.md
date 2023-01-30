# The Basics

## Simple SELECTs
1) Query all the data in the `pets` table.
	```sql
	SELECT *
	FROM `pets`
	```
2) Query only the first 5 rows of the `pets` table.
	```sql
	SELECT *
	FROM `pets`
	LIMIT 5
	```
3) Query only the names and ages of the pets in the `pets` table.
	```sql
	SELECT name, age
	FROM `pets`
	```
4) Query the pets in the `pets` table, sorted youngest to oldest.
	```sql
	SELECT *
	FROM `pets`
	ORDER BY age
	```
5) Query the pets in the `pets` table alphabetically.
	```sql
	SELECT *
	FROM `pets`
	ORDER BY name
	```
6) Query all the male pets in the `pets` table.
	```sql
	SELECT *
	FROM `pets`
	WHERE sex = 'M'
	```
7) Query all the cats in the `pets` table.
	```sql
	SELECT *
	FROM `pets`
	WHERE species = 'cat'
	```
8) Query all the pets in the `pets` table that are at least 5 years old.
	```sql
	SELECT *
	FROM `pets`
	WHERE age >= 5
	```
9) Query all the male dogs in the `pets` table. Do not include the sex or species column, since you already know them.
	```sql
	SELECT name, age
	FROM `pets`
	WHERE sex = 'M' AND species = 'dog'
	```
10) Get all the names of the dogs in the `pets` table that are younger than 5 years old.
	```sql
	SELECT name
	FROM `pets`
	WHERE age < 5 AND species = 'dog'
	```
11) Query all the pets in the `pets` table that are either male dogs or female cats.
	```sql
	SELECT *
	FROM `pets`
	WHERE 
		(sex = 'M' AND species = 'dog')
		OR (sex = 'F' AND species = 'cat')
	```
12) Query the five oldest pets in the `pets` table.
	```sql
	SELECT *
	FROM `pets`
	ORDER BY age DESC
	LIMIT 5
	```
13) Get the names and ages of all the female cats in the `pets` table sorted by age, descending.
	```sql
	SELECT name, age
	FROM `pets`
	WHERE sex = 'F' AND species = 'cat'
	ORDER BY age DESC
	```
14) Get all pets from `pets` whose names start with P.
	```sql
	SELECT *
	FROM `pets`
	WHERE name LIKE 'P%'
	```
15) Select all employees from `employees_null` where the salary is missing.
	```sql
	SELECT *
	FROM `employees_null`
	WHERE salary IS NULL
	```
16) Select all employees from `employees_null` where the salary is below $35,000 or missing.
	```sql
	SELECT *
	FROM `employees_null`
	WHERE salary IS NULL OR salary < 35000
	```
17) Select all employees from `employees_null` where the job title is missing. What do you see?
	```sql
	SELECT *
	FROM `employees_null`
	WHERE job IS NULL
	```
	>It is an UNKNOWN data with no meaningful value.
	
18) Who is the newest employee in `employees`? The most senior?
	```sql
	-- newest employee
	SELECT firstname, lastname, startdate
	FROM `employees`
	ORDER BY startdate DESC
	LIMIT 1
	
	-- most senior
	SELECT firstname, lastname, startdate
	FROM `employees`
	ORDER BY startdate
	LIMIT 1
	```
	>The newest employee: Roger Conner / The most senior: Mary Nash
	
19) Select all employees from `employees` named Thomas.
	```sql
	SELECT *
	FROM `employees`
	WHERE firstname = 'Thomas'
	```
20) Select all employees from `employees` named Thomas or Shannon.
	```sql
	SELECT *
	FROM `employees`
	WHERE firstname = 'Thomas' OR firstname = 'Shannon'
	```
21) Select all employees from `employees` named Robert, Lisa, or any name that begins with a J. In addition, only show employees who are _not_ in sales. This will be a little bit of a longer query.
    * _Hint:_ There will only be 6 rows in the result.
	```sql
	SELECT *
	FROM `employees`
	WHERE 
		(firstname = 'Robert' OR firstname = 'Lisa' OR firstname LIKE 'J%')
		AND job != 'Sales'
	```

## Column Operations
22) Query the top 5 rows of the `employees` table to get a glimpse of these new data.
	```sql
	SELECT *
	FROM `employees`
	LIMIT 5
	```
23) Query the `employees` table, but convert their salaries to Euros. 
    * _Hint:_ 1 Euro = 1.1 USD.
    * _Hint2:_ If you think the output is ugly, try out the `ROUND()` function.
	```sql
	SELECT *, ROUND(salary / 1.1)
	FROM `employees`
	```
24) Repeat the previous problem, but rename the column `salary_eu`.
	```sql
	SELECT *, ROUND(salary / 1.1) AS salary_eu
	FROM `employees`
	```
25) Query the `employees` table, but combine the `firstname` and `lastname` columns to be "Firstname, Lastname" format. Call this column `fullname`. For example, the first row should contain `Thompson, Christine` as `fullname`. Also, display the rounded `salary_eu` instead of `salary`.
    * _Hint:_ The string concatenation operator is `||`
	```sql
	SELECT 
		firstname || ',  ' || lastname AS fullname,
		ROUND(salary / 1.1) AS salary_eu
	FROM `employees`
	```
26) Query the `employees` table, but replace `startdate` with `startyear` using the `SUBSTR()` function. Also include `fullname` and `salary_eu`.
	```sql
	SELECT 
		firstname || ',  ' || lastname AS fullname,
		ROUND(salary / 1.1) AS salary_eu,
		SUBSTR(startdate, 1, 4) AS startyear
	FROM `employees`
	```
27) Repeat the above problem, but instead of using `SUBSTR()`, use `STRFTIME()`.
	```sql
	SELECT 
		firstname || ',  ' || lastname AS fullname,
		ROUND(salary / 1.1) AS salary_eu,
		STRFTIME('%Y', startdate) AS startyear
	FROM `employees`
	```
28) Query the `employees` table, replacing `firstname`/`lastname` with `fullname` and `startdate` with `startyear`. Print out the salary in USD again, except format it with a dollar sign, comma separators, and no decimal. For example, the first row should read `$123,696`. This column should still be named `salary`.
    * _Hint:_ Check out SQLite's `printf` function.
    * _Hint2:_ The format string you'll need is `$%,.2d`. You should read more about such formatting strings as they're useful in Python, too!
	```sql
	SELECT 
		firstname || ',  ' || lastname AS fullname,
		STRFTIME('%Y', startdate) AS startyear,
		PRINTF('$%, .2d', salary) AS salary
	FROM `employees`
	```

---
**Note:** For the next few problems, you'll probably want to use `CASE`/`WHEN` statements.

29) Last year, only salespeople were eligible for bonuses. Create a column `bonus` that is "Yes" if you're eligible for a bonus, otherwise "No".
	```sql
	SELECT 
		*,
		CASE
			WHEN job = 'Sales' THEN 'Yes'
			ELSE 'No'
		END AS bonus
	FROM `employees`
	```
30) This year, only sales people with a salary of $100,000 or higher are eligible for bonuses. Create a `bonus` column like in the last problem for salespeople with salaries at least $100,000.
	```sql
	SELECT 
		*,
		CASE
			WHEN job = 'Sales' AND salary >= 100000 THEN 'Yes'
			ELSE 'No'
		END AS bonus
	FROM `employees`
	```
31) Next year, the bonus structure will be a little more complicated. You'll create a `target_comp` column which represents an employee's target total compensation after their bonus. Here is the company's bonus structure:

* Salespeople who make more than $100,000 will be eligible for a 10% bonus.
* Salespeople who make less than $100,000 will be eligible for a 5% bonus.
* Administrators will also be eligible for a 5% bonus.
* Anyone who does not meet any of the above descriptions is not eligible for a bonus.

Create this `target_comp` column, making sure to format _both_ the `salary` and `target_comp` columns nicely (ie, with dollar signs and comma separators).
```sql
SELECT 
	firstname || ',  ' || lastname AS fullname,
	job,
	PRINTF('$%, .2d', salary) AS salary,
	CASE
		WHEN job = 'Sales' AND salary >= 100000 THEN PRINTF('$%, .2d', salary*0.1)
		WHEN (job = 'Sales' AND salary < 100000) OR (job = 'Administrator') THEN PRINTF('$%, .2d', salary*0.05)
		ELSE PRINTF('$%,.d', 0)
	END AS target_comp
FROM `employees`
```

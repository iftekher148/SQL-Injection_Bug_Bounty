# SQL-Injection_Bug_Bounty
##All taken fron portswigger lab

The application doesn't implement any defenses against SQL injection attacks, so an attacker can construct an attack like:

	https://insecure-website.com/products?category=Gifts'--
This results in the SQL query:

	SELECT * FROM products WHERE category = 'Gifts'--' AND released = 1
Going further, an attacker can cause the application to display all the products in any category, including categories that they don't know about:

	https://insecure-website.com/products?category=Gifts'+OR+1=1--
  
	SELECT * FROM products WHERE category = 'Gifts' OR 1=1--' AND released = 1
	since 1=1 is always true
Check response:    ‘--      |   #--

**Subverting application logic/Login form:**

Here, an attacker can log in as any user without a password simply by using the SQL comment sequence -- to remove the password check from the WHERE clause of the query. For example, submitting the username administrator'-- and a blank password results in the following query:

	SELECT * FROM users WHERE username = 'administrator'--' AND password = ''

![image](https://user-images.githubusercontent.com/37367596/192005023-3f13a816-0a13-4f0a-8ff5-cf85222e8604.png)


**Retrieving data from other database tables/Union attack:**

For a UNION query to work, two key requirements must be met:

 1.The individual queries must return the same number of columns.
 
 2.The data types in each column must be compatible between the individual queries.

To carry out an SQL injection UNION attack, you need to ensure that your attack meets these two requirements. This generally involves figuring out

 1.How many columns are being returned from the original query?
 
 2.Which columns returned from the original query are of a suitable data type to hold the results from the injected query?
 
**There are two effective methods to determine how many columns are being returned from the original query.**
**The first method** involves injecting a series of** ORDER BY** clauses and incrementing the specified column index until an error occurs.

' ORDER BY 1--

 ' ORDER BY 2-- 
 
' ORDER BY 3--
etc.
![image](https://user-images.githubusercontent.com/37367596/192006415-207e60bb-4977-4895-9a57-43fed4f6c250.png)



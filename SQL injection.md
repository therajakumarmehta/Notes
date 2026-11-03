# SQL Injection (SQLi) – Introduction

## 1. What is SQL Injection?

SQL Injection (SQLi) is a web application security vulnerability that occurs when an application handles user input insecurely while executing SQL queries.

An attacker can manipulate SQL queries to access, modify, or delete database information without proper authorization.

## 2. Why Does SQL Injection Occur?

SQL Injection usually occurs when an application directly includes untrusted user input in SQL queries without proper parameterization.

## 3. Impact of SQL Injection

- Unauthorized access to database information.
- Bypassing login authentication.
- Modifying database records.
- Deleting database information.
- Exposure of sensitive user data.

## 4. Types of SQL Injection

- In-band SQL Injection
- Blind SQL Injection
- Out-of-band SQL Injection

## 5. Conclusion

SQL Injection is one of the important web application security vulnerabilities. Understanding SQL Injection is essential for identifying database-related security weaknesses and learning how to protect web applications.

# SQL Injection – Retrieving Hidden Data

## 1. What is Retrieving Hidden Data?

Retrieving Hidden Data is a SQL Injection technique in which an attacker manipulates an application's SQL query to access records that are normally hidden or restricted.

## 2. Why Does It Happen?

It happens when an application directly includes untrusted user input in SQL queries without proper parameterization.

## 3. Payload Used

```sql
' OR 1=1--
```

## 4. Payload Explanation

- `'` : Attempts to close the existing SQL string.
- `OR` : Adds an alternative condition.
- `1=1` : Creates a condition that is always TRUE.
- `--` : Comments out the remaining SQL query in supported SQL dialects.

## 5. How Does It Work?

Suppose an application uses the following query:

```sql
SELECT * FROM products
WHERE category = 'Gifts';
```

If the application handles user input insecurely, the payload may change the query's logic.

The condition `1=1` is always TRUE, which may cause the database to return records from multiple categories instead of only the selected category.

## 6. Prevention Methods

- Use Prepared Statements.
- Use Parameterized Queries.
- Avoid direct SQL query concatenation.
- Validate user input.
- Apply proper database permissions.
- Handle database errors securely.

## 7. Key Points

- SQL Injection occurs due to insecure SQL query construction.
- Always-true conditions can manipulate query logic.
- Parameterized queries are the primary prevention method.
# SQL Injection – Login Bypass

## 1. Why Does It Happen?

This vulnerability occurs when a web application directly uses user input in an SQL query without proper parameterization.

The application does not safely separate SQL commands from user input. As a result, an attacker can manipulate the login query and bypass password verification.

## 2. Payload Used

```sql
administrator'--
```

**Where to use:** Username parameter.

## 3. Payload Explanation

- `administrator` : Specifies the username of the account we want to access.
- `'` : Closes the existing SQL string.
- `--` : Comments out the remaining part of the SQL query in the supported SQL dialect.

## 4. How Does It Work?

Suppose the application uses the following SQL query:

```sql
SELECT * FROM users
WHERE username = 'USER_INPUT'
AND password = 'PASSWORD_INPUT';
```

After inserting the payload, the query may become:

```sql
SELECT * FROM users
WHERE username = 'administrator'--'
AND password = 'anything';
```

**Explanation:**

- The username becomes `administrator`.
- The single quote closes the username string.
- The `--` comments out the remaining SQL query.
- The password condition is ignored.
- The database may return the administrator account without checking the password.

This happens because the application builds SQL queries insecurely.

## 5. How to Prevent This Attack?

- **Parameterized Queries:** Use prepared statements to separate SQL commands from user input.
- **Password Verification:** Always verify passwords using secure password-hashing mechanisms.
- **Input Validation:** Validate username and password inputs.
- **Least Privilege:** Give database accounts only the permissions they need.
- **Secure Error Handling:** Do not expose detailed database errors.
- **Secure Coding:** Never concatenate untrusted input directly into SQL queries.

## 6. Key Takeaways

- SQL Injection can allow attackers to bypass login authentication.
- The `--` symbol is used for SQL comments in supported database systems.
- Login forms are common places where SQL Injection vulnerabilities may occur.
- Parameterized queries are the primary defense against SQL Injection.

# SQL Injection – UNION Attack: Determining the Number of Columns

## 1. Why Does It Happen?

This vulnerability occurs when an application uses user input directly in an SQL query without proper parameterization.

The SQL `UNION` operator combines the results of two or more SELECT queries.

To perform a successful UNION attack, both SELECT queries must return the same number of columns.

## 2. Payloads Used

**Payload 1:**

```sql
' ORDER BY 1--
```

**Payload 2:**

```sql
' ORDER BY 2--
```

**Payload 3:**

```sql
' ORDER BY 3--
```

**Payload 4:**

```sql
' ORDER BY 4--
```

## 3. Payload Explanation

- `'` : Attempts to close the existing SQL string.
- `ORDER BY` : Sorts query results using a specified column.
- `1` : Refers to the first column.
- `2` : Refers to the second column.
- `3` : Refers to the third column.
- `4` : Refers to the fourth column.
- `--` : Comments out the remaining SQL query in supported SQL dialects.

## 4. How Does It Work?

The attacker tests different column numbers using the `ORDER BY` clause.

1. Start with `ORDER BY 1`.
2. If the application responds normally, the first column exists.
3. Try `ORDER BY 2`, then `ORDER BY 3`, and continue.
4. When an invalid column number is used, the database may return an error.
5. The last valid column number indicates the total number of columns in the original query.

**Example:**

If `ORDER BY 1`, `ORDER BY 2`, and `ORDER BY 3` work, but `ORDER BY 4` produces an error, the query likely returns three columns.

## 5. Alternative Method: Using UNION SELECT

Another method is to use `UNION SELECT` with a different number of NULL values.

**Payload 1:**

```sql
' UNION SELECT NULL--
```

**Payload 2:**

```sql
' UNION SELECT NULL,NULL--
```

**Payload 3:**

```sql
' UNION SELECT NULL,NULL,NULL--
```

**Explanation:**

- `UNION` combines the results of two SELECT queries.
- `SELECT` retrieves data from the database.
- `NULL` represents an unknown or missing value.
- Each additional `NULL` represents another column.

When the number of columns matches the original query, the UNION query may execute successfully.

## 6. How to Prevent This Attack?

- Use Prepared Statements.
- Use Parameterized Queries.
- Never concatenate untrusted input directly into SQL queries.
- Validate user input.
- Avoid exposing detailed database errors.
- Apply proper database permissions.

## 7. Key Takeaways

- UNION attacks combine results from multiple SELECT queries.
- Both SELECT queries must return the same number of columns.
- ORDER BY can help determine the number of columns.
- UNION SELECT NULL can also be used to determine the column count.
- Parameterized queries are the primary defense against SQL Injection.

# SQL Injection – UNION Attack: Finding a Column Containing Text

## 1. Why Does It Happen?

This vulnerability occurs when an application includes user input directly in SQL queries without proper parameterization.

In a UNION-based SQL Injection attack, an attacker needs to identify which columns can display text data.

This is important because different database columns may support different data types, such as INTEGER, VARCHAR, or DATE.

## 2. Payload Used

Suppose the original query returns three columns.

**Payload:**

```sql
' UNION SELECT NULL,'abc',NULL--
```

## 3. Payload Explanation

- `'` : Attempts to close the existing SQL string.
- `UNION` : Combines the results of two SELECT queries.
- `SELECT` : Retrieves data from the database.
- `NULL` : Represents an unknown or missing value.
- `'abc'` : A sample text value used to identify a text-compatible column.
- `--` : Comments out the remaining SQL query in supported SQL dialects.

## 4. How Does It Work?

The process involves identifying which column can accept and display text values.

1. First, determine the number of columns returned by the original query.
2. Create a UNION SELECT query containing the same number of columns.
3. Place a test string such as `'abc'` in the first column.
4. Keep the remaining columns as NULL.
5. If the query produces a data type error, try placing the string in another column.
6. Repeat until the text value is accepted.
7. The successful position identifies a column that supports text data.

**Example:**

If the following payload works:

```sql
' UNION SELECT NULL,'abc',NULL--
```

It indicates that the second column can accept a text value.

## 5. Why Do We Use NULL?

NULL is used because it can be compatible with different data types in many SQL database systems.

It helps test column positions without introducing unnecessary data type conflicts.

## 6. How to Prevent This Attack?

- Use Prepared Statements.
- Use Parameterized Queries.
- Avoid directly concatenating user input into SQL queries.
- Validate and handle user input properly.
- Avoid exposing detailed database error messages.
- Apply the principle of least privilege.

## 7. Key Takeaways

- UNION attacks combine results from multiple SELECT queries.
- Both SELECT statements must return the same number of columns.
- Text values help identify columns that support string data.
- NULL values help reduce data type conflicts.
- Parameterized queries are the primary defense against SQL Injection.

# SQL Injection – Querying Database Type and Version (MySQL and Microsoft SQL Server)

## 1. Why Does It Happen?

This vulnerability occurs when an application directly includes user input in SQL queries without proper parameterization.

An attacker can manipulate the SQL query using a UNION attack to retrieve database information, including its type and version.

Database version information can help an attacker understand the database environment and identify possible security weaknesses.

## 2. Payloads Used

### Payload 1: Testing Text Columns

```sql
' UNION SELECT 'abc','def'#
```

### Payload 2: Retrieving Database Version

```sql
' UNION SELECT @@version,NULL#
```

## 3. Payload Explanation

### Payload 1 Explanation

- `'` : Attempts to close the existing SQL string.
- `UNION` : Combines the results of two SELECT queries.
- `SELECT` : Retrieves information from the database.
- `'abc'` : A sample text value for testing the first column.
- `'def'` : A sample text value for testing the second column.
- `#` : Starts a single-line comment in MySQL.

### Payload 2 Explanation

- `'` : Attempts to close the existing SQL string.
- `UNION` : Combines the original query with an injected query.
- `SELECT` : Retrieves the requested information.
- `@@version` : Returns the database server version information in MySQL and Microsoft SQL Server.
- `NULL` : Provides a value for the second column.
- `#` : Comments out the remaining query in MySQL.

## 4. How Does It Work?

1. First, identify the number of columns returned by the original SQL query.
2. Identify which columns support text data.
3. Use the first payload to test whether both columns accept text.
4. After confirming the column structure, use the second payload.
5. The `@@version` variable retrieves database version information.
6. The application may display the database version in its response.

**Example:**

If the database returns:

```text
MySQL version information
```

It indicates that the injected query successfully retrieved database version details.

## 5. How to Prevent This Attack?

- **Parameterized Queries:** Use prepared statements to separate SQL commands from user input.
- **Input Validation:** Validate user input before processing it.
- **Least Privilege:** Give database accounts only the permissions they require.
- **Error Handling:** Avoid exposing detailed database errors.
- **Secure Coding:** Never directly concatenate untrusted input into SQL queries.
- **Information Protection:** Avoid unnecessarily exposing database configuration and version details.

## 6. Key Takeaways

- SQL Injection can expose database information.
- UNION attacks can retrieve results from injected SELECT queries.
- `@@version` retrieves database version information in MySQL and Microsoft SQL Server.
- Both SELECT queries must return the same number of columns.
- `#` is a MySQL comment character.
- Parameterized queries are the primary defense against SQL Injection.

# SQL Injection – Querying Database Type and Version (Oracle)

## 1. Why Does It Happen?

- The application does not properly validate user input before using it in an SQL query.
- An attacker can inject SQL commands into the existing query.
- Using a `UNION SELECT` attack, an attacker can retrieve information directly from the database.
- Oracle requires every `SELECT` statement to include a `FROM` clause.

## 2. Payload Used

**Step 1: Check the number of columns and text-compatible columns**

```sql
' UNION SELECT 'abc','def' FROM dual--
```

**Step 2: Retrieve the Oracle database version**

```sql
' UNION SELECT BANNER, NULL FROM v$version--
```

## 3. Payload Explanation

- `'` – Closes the existing string.
- `UNION SELECT` – Combines the results of the original query with the injected query.
- `'abc'`, `'def'` – Test whether the selected columns accept text values.
- `FROM dual` – Uses Oracle's built-in `dual` table because Oracle requires a `FROM` clause in a `SELECT` statement.
- `BANNER` – Contains Oracle database version information.
- `NULL` – Fills the second column to match the original query's column count.
- `v$version` – Oracle system view that provides database version details.
- `--` – Comments out the remaining part of the original SQL query.

## 4. How Does It Work?

1. First, identify the number of columns returned by the original query.
2. Find which columns support text data.
3. Use `FROM dual` to make the injected query valid in Oracle.
4. Query `v$version` to retrieve the database version.
5. The application may display the returned version information in its response.

## 5. Prevention

- Use parameterized queries (prepared statements).
- Never concatenate user input directly into SQL queries.
- Validate user input on the server side.
- Use least-privilege database accounts.
- Avoid exposing detailed database errors or internal information.
- Regularly test applications for SQL injection vulnerabilities.

## 6. Key Takeaways

- Oracle requires a `FROM` clause in every `SELECT` statement.
- The `dual` table is useful when a query needs a valid table but does not need actual table data.
- `v$version` provides Oracle database version information.
- `UNION SELECT` can retrieve data from database system views.
- Always match the number and data types of columns in a `UNION` query.

# SQL Injection – UNION Attack: Retrieving Data from Other Tables

## 1. Why Does It Happen?

- The application uses user input directly inside an SQL query.
- It does not properly validate or sanitize the input.
- An attacker can inject a `UNION SELECT` statement to retrieve data from other database tables.
- If the database response is displayed in the application, sensitive information may be exposed.

## 2. Payload Used

**Step 1: Identify columns that support text data**

```sql
' UNION SELECT 'abc','def'--
```

**Step 2: Retrieve usernames and passwords from another table**

```sql
' UNION SELECT username, password FROM users--
```

## 3. Payload Explanation

- `'` – Closes the existing string in the original SQL query.
- `UNION SELECT` – Combines the results of two SQL queries.
- `'abc', 'def'` – Checks whether both returned columns accept text values.
- `username` – Retrieves usernames from the `users` table.
- `password` – Retrieves passwords from the `users` table.
- `FROM users` – Specifies the table from which data is retrieved.
- `--` – Comments out the remaining part of the original SQL query.

## 4. How Does It Work?

1. First, identify the number of columns returned by the original query.
2. Find which columns support text data.
3. Use `UNION SELECT` to create a second query.
4. Specify the required columns, such as `username` and `password`.
5. Specify the target table using `FROM users`.
6. The application may display the retrieved usernames and passwords in its response.

## 5. Prevention

- Use parameterized queries (prepared statements).
- Never directly concatenate user input into SQL queries.
- Validate user input on the server side.
- Apply least-privilege permissions to database accounts.
- Restrict access to sensitive database tables.
- Avoid displaying sensitive database information in application responses.

## 6. Key Takeaways

- `UNION SELECT` can retrieve data from other database tables.
- Both queries must return the same number of columns.
- Corresponding columns must have compatible data types.
- `FROM` specifies the table from which data is retrieved.
- Poor input validation can expose sensitive information such as usernames and passwords.

# SQL Injection – Listing Database Contents (Non-Oracle Databases)

## 1. Why Does It Happen?

- Applications sometimes use user input directly inside SQL queries.
- If input is not properly handled, an attacker can inject SQL commands.
- Attackers can use `UNION SELECT` to access database metadata.
- Database metadata contains information about tables and columns.
- This information can help identify tables containing sensitive data, such as usernames and passwords.

## 2. Payload Used

**Step 1: Find the number of columns and text-compatible columns**

```sql
' UNION SELECT 'abc','def'--
```

**Step 2: List all database tables**

```sql
' UNION SELECT table_name, NULL FROM information_schema.tables--
```

**Step 3: Find columns inside a specific table**

```sql
' UNION SELECT column_name, NULL FROM information_schema.columns WHERE table_name='users_abcdef'--
```

**Step 4: Retrieve usernames and passwords**

```sql
' UNION SELECT username_abcdef, password_abcdef FROM users_abcdef--
```

*Note: Replace `users_abcdef`, `username_abcdef`, and `password_abcdef` with the actual table and column names discovered in the previous steps.*

## 3. Payload Explanation

- `'` – Closes the existing string in the original SQL query.
- `UNION SELECT` – Combines the original query results with the injected query results.
- `'abc', 'def'` – Checks whether the returned columns support text values.
- `table_name` – Retrieves table names from database metadata.
- `information_schema.tables` – Contains information about database tables.
- `column_name` – Retrieves column names.
- `information_schema.columns` – Contains information about database columns.
- `WHERE table_name='users_abcdef'` – Filters column information for a specific table.
- `username_abcdef` – Retrieves usernames.
- `password_abcdef` – Retrieves passwords.
- `NULL` – Fills an unused column to match the original query's column count.
- `--` – Comments out the remaining part of the original SQL query.

## 4. How Does It Work?

1. First, identify the number of columns returned by the original query.
2. Find which columns support text data.
3. Query `information_schema.tables` to discover database table names.
4. Identify the table that stores user credentials.
5. Query `information_schema.columns` to discover the columns inside that table.
6. Identify the username and password columns.
7. Use `UNION SELECT` to retrieve the required data from the identified table.

## 5. Prevention

- Use parameterized queries (prepared statements).
- Never concatenate user input directly into SQL queries.
- Validate user input on the server side.
- Apply least-privilege permissions to database accounts.
- Restrict unnecessary access to database metadata.
- Never expose sensitive database information in application responses.

## 6. Key Takeaways

- `information_schema.tables` helps identify database tables.
- `information_schema.columns` helps identify table columns.
- `table_name` retrieves table names.
- `column_name` retrieves column names.
- `UNION SELECT` can retrieve data from other tables.
- Database metadata can help attackers discover sensitive information.
- Always match the number and compatible data types of columns in a `UNION` query.
# SQL Injection – Listing Database Contents (Oracle)

## 1. Why Does It Happen?

- Applications sometimes use user input directly inside SQL queries.
- If input is not properly handled, attackers can inject SQL commands.
- Using `UNION SELECT`, attackers can access database metadata.
- Database metadata contains information about tables and columns.
- This information can help identify tables containing sensitive data, such as usernames and passwords.

## 2. Payload Used

**Step 1: Check the number of columns and text-compatible columns**

```sql
' UNION SELECT 'abc','def' FROM dual--
```

**Step 2: List database tables**

```sql
' UNION SELECT table_name,NULL FROM all_tables--
```

**Step 3: Find columns inside a specific table**

```sql
' UNION SELECT column_name,NULL FROM all_tab_columns WHERE table_name='USERS_ABCDEF'--
```

**Step 4: Retrieve usernames and passwords**

```sql
' UNION SELECT USERNAME_ABCDEF,PASSWORD_ABCDEF FROM USERS_ABCDEF--
```

*Note: Replace the example table and column names with the actual names discovered from the database.*

## 3. Payload Explanation

- `'` – Closes the existing string in the original SQL query.
- `UNION SELECT` – Combines the results of two SQL queries.
- `'abc', 'def'` – Checks whether the returned columns support text values.
- `FROM dual` – Uses Oracle's built-in `dual` table to satisfy the required `FROM` clause.
- `table_name` – Retrieves table names.
- `all_tables` – Oracle system view that provides information about accessible tables.
- `column_name` – Retrieves column names.
- `all_tab_columns` – Oracle system view that provides information about accessible table columns.
- `WHERE table_name='USERS_ABCDEF'` – Filters column information for a specific table.
- `USERNAME_ABCDEF` – Retrieves usernames.
- `PASSWORD_ABCDEF` – Retrieves passwords.
- `NULL` – Fills an unused column to match the original query's column count.
- `--` – Comments out the remaining part of the original SQL query.

## 4. How Does It Work?

1. First, identify the number of columns returned by the original query.
2. Find which columns support text data.
3. Use `FROM dual` when testing a `UNION SELECT` query in Oracle.
4. Query `all_tables` to discover accessible table names.
5. Identify the table containing user credentials.
6. Query `all_tab_columns` to discover the column names.
7. Identify the username and password columns.
8. Use `UNION SELECT` to retrieve the required information from that table.

## 5. Prevention

- Use parameterized queries (prepared statements).
- Never concatenate user input directly into SQL queries.
- Validate user input on the server side.
- Apply least-privilege permissions to database accounts.
- Restrict unnecessary access to database metadata.
- Avoid exposing sensitive database information in application responses.

## 6. Key Takeaways

- Oracle requires a `FROM` clause in every `SELECT` statement.
- `dual` is a built-in Oracle table used when a query needs a valid table reference.
- `all_tables` helps identify accessible database tables.
- `all_tab_columns` helps identify accessible table columns.
- `UNION SELECT` can retrieve data from other tables.
- Database metadata can help attackers discover sensitive information.
- Always match the number and compatible data types of columns in a `UNION` query.

# SQL Injection – UNION Attack: Retrieving Multiple Values in a Single Column

## 1. Why Does It Happen?

- Sometimes the original SQL query returns only one column that supports text data.
- An attacker may need to retrieve multiple values, such as usernames and passwords.
- A `UNION SELECT` query requires the same number of columns as the original query.
- The attacker can use string concatenation to combine multiple values into one column.
- This allows multiple pieces of information to be retrieved through a single text-compatible column.

## 2. Payload Used

**Step 1: Identify the text-compatible column**

```sql
' UNION SELECT NULL,'abc'--
```

**Step 2: Retrieve multiple values in a single column**

```sql
' UNION SELECT NULL,username||'~'||password FROM users--
```

## 3. Payload Explanation

- `'` – Closes the existing string in the original SQL query.
- `UNION SELECT` – Combines the results of two SQL queries.
- `NULL` – Fills the first column because it does not need to contain useful data.
- `username` – Retrieves usernames from the `users` table.
- `||` – String concatenation operator used to join multiple values.
- `'~'` – Adds a separator between the username and password.
- `password` – Retrieves passwords from the `users` table.
- `FROM users` – Specifies the table from which data is retrieved.
- `--` – Comments out the remaining part of the original SQL query.

## 4. How Does It Work?

1. First, identify the number of columns returned by the original query.
2. Find which column supports text data.
3. Use `NULL` for columns that do not need to display useful information.
4. Use the `||` operator to combine multiple values.
5. Add a separator such as `~` between the values.
6. The database combines the username and password into one string.
7. The application may display both values through a single text-compatible column.

**Example Output:**

```text
administrator~admin_password
user1~password123
user2~mypassword
```

Here, the username and password are returned together in one column, separated by `~`.

## 5. Prevention

- Use parameterized queries (prepared statements).
- Never concatenate user input directly into SQL queries.
- Validate user input on the server side.
- Apply least-privilege permissions to database accounts.
- Restrict access to sensitive database tables.
- Never expose passwords or other sensitive information in application responses.

## 6. Key Takeaways

- `UNION SELECT` requires the same number of columns in both queries.
- Sometimes only one column supports text data.
- `||` is a string concatenation operator in Oracle and several other databases, including PostgreSQL and SQLite.
- A separator such as `~` helps distinguish combined values.
- Multiple database values can be retrieved through a single text-compatible column.
- String concatenation syntax differs between database systems.

# Blind SQL Injection – Conditional Responses

## 1. Why Does It Happen?

- Sometimes an application executes SQL queries but does not display the query results.
- It may also hide database error messages.
- However, the application's response can change depending on whether an SQL condition is true or false.
- Attackers can use these response differences to extract information from the database.
- This technique is called **Blind SQL Injection with Conditional Responses**.

## 2. Payload Used

The following examples use a tracking cookie named `TrackingId`.

**Step 1: Check whether the SQL condition is true**

```sql
xyz' AND '1'='1
```

**Step 2: Check whether the SQL condition is false**

```sql
xyz' AND '1'='2
```

**Step 3: Check whether the users table exists**

```sql
xyz' AND (SELECT 'a' FROM users LIMIT 1)='a
```

**Step 4: Check whether a specific user exists**

```sql
xyz' AND (SELECT 'a' FROM users WHERE username='administrator')='a
```

**Step 5: Determine the password length**

```sql
xyz' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>1)='a
```

Change the number to test different password lengths.

**Step 6: Retrieve password characters one by one**

```sql
xyz' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='a
```

To test the second character:

```sql
xyz' AND (SELECT SUBSTRING(password,2,1) FROM users WHERE username='administrator')='a
```

Continue changing the character position until all characters are identified.

## 3. Payload Explanation

- `xyz` – Represents the original tracking cookie value.
- `'` – Closes the existing string in the SQL query.
- `AND` – Adds another condition to the existing SQL query.
- `'1'='1'` – A condition that is always true.
- `'1'='2'` – A condition that is always false.
- `SELECT` – Retrieves information from the database.
- `users` – The table containing user information.
- `LIMIT 1` – Limits the result to one row.
- `username='administrator'` – Filters the result for the administrator account.
- `LENGTH(password)` – Returns the number of characters in the password.
- `SUBSTRING(password,1,1)` – Extracts one character from the password, starting at position 1.
- `SUBSTRING(password,2,1)` – Extracts one character starting at position 2.

## 4. How Does It Work?

1. First, inject a condition that is always true.
2. Observe the application's response. For example, the page displays `Welcome back`.
3. Inject a condition that is false.
4. Observe that the `Welcome back` message disappears.
5. This confirms that the application's response depends on the SQL condition.
6. Check whether the `users` table and administrator account exist.
7. Use `LENGTH(password)` to determine the password length.
8. Use `SUBSTRING()` to extract individual password characters.
9. Test possible characters at each position until the complete value is identified.

**Example:**

| Condition | Application response |
|---|---|
| `'1'='1'` | Welcome back appears |
| `'1'='2'` | Welcome back disappears |
| Password condition is true | Welcome back appears |
| Password condition is false | Welcome back disappears |

## 5. Prevention

- Use parameterized queries (prepared statements).
- Never concatenate user input directly into SQL queries.
- Validate and handle user input on the server side.
- Apply least-privilege permissions to database accounts.
- Avoid revealing different application responses based on database query results.
- Perform regular security testing to identify SQL injection vulnerabilities.

## 6. Key Takeaways

- Blind SQL Injection occurs when query results are not directly displayed.
- Conditional responses help attackers determine whether an SQL condition is true or false.
- `AND` is used to add conditions to SQL queries.
- `LENGTH()` helps determine the length of a value.
- `SUBSTRING()` extracts individual characters from a string.
- Passwords and other sensitive information can be extracted character by character.
- Even without database errors or visible query results, SQL injection can expose sensitive information.

# Blind SQL Injection – Conditional Errors

## 1. Why Does It Happen?

- Sometimes an application does not display SQL query results.
- It may also hide normal database responses.
- However, database errors can still affect the application's response.
- Attackers can intentionally create SQL errors based on specific conditions.
- By observing whether an error occurs, attackers can determine whether a condition is true or false.
- This technique is called **Blind SQL Injection with Conditional Errors**.

## 2. Payload Used

The following examples use Oracle SQL syntax and a tracking cookie named `TrackingId`.

**Step 1: Check whether SQL injection is possible**

```sql
xyz'
```

If an error occurs, try:

```sql
xyz''
```

If the error disappears, it suggests that the quotation mark is affecting the SQL query.

**Step 2: Test a valid Oracle query**

```sql
xyz'||(SELECT '' FROM dual)||'
```

`dual` is Oracle's built-in table used in this query.

**Step 3: Check whether a table exists**

```sql
xyz'||(SELECT '' FROM users WHERE ROWNUM=1)||'
```

**Step 4: Test a true condition**

```sql
xyz'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM dual)||'
```

**Step 5: Test a false condition**

```sql
xyz'||(SELECT CASE WHEN (1=2) THEN TO_CHAR(1/0) ELSE '' END FROM dual)||'
```

**Step 6: Check whether a specific user exists**

```sql
xyz'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'
```

**Step 7: Determine password length**

```sql
xyz'||(SELECT CASE WHEN LENGTH(password)>1 THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'
```

Change the number to determine the password length.

**Step 8: Retrieve password characters**

```sql
xyz'||(SELECT CASE WHEN SUBSTR(password,1,1)='a' THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'
```

Change the character position and test different characters to identify the complete password.

## 3. Payload Explanation

- `xyz` – Represents the original tracking cookie value.
- `'` – Closes the existing string in the SQL query.
- `||` – Concatenates strings in Oracle SQL.
- `SELECT` – Retrieves information from the database.
- `dual` – Oracle's built-in table used for queries that do not require actual table data.
- `users` – Table containing user information.
- `ROWNUM=1` – Limits the result to one row.
- `CASE WHEN` – Evaluates a condition and chooses an expression based on its result.
- `1=1` – Always true condition.
- `1=2` – Always false condition.
- `TO_CHAR(1/0)` – Attempts division by zero and causes a database error.
- `ELSE ''` – Returns an empty string when the condition is false.
- `LENGTH(password)` – Returns the number of characters in the password.
- `SUBSTR(password,1,1)` – Extracts one character from the password, starting at position 1.
- `username='administrator'` – Selects the administrator account.
- `--` is not required in these example payloads because the query uses string concatenation to handle the surrounding SQL syntax.

## 4. How Does It Work?

1. First, check whether an extra quotation mark causes a database error.
2. Use two quotation marks to verify that the error disappears.
3. Test a valid Oracle query using `dual`.
4. Confirm that the database responds differently when an invalid table is queried.
5. Use `CASE WHEN` to create conditional errors.
6. If the condition is true, `TO_CHAR(1/0)` causes a database error.
7. If the condition is false, the query returns an empty string without that error.
8. Use `LENGTH()` to determine the password length.
9. Use `SUBSTR()` to extract password characters one by one.
10. Observe the application's error response to identify the correct characters.

**Example Response:**

| SQL condition | Application response |
|---|---|
| Condition is true | Database error (HTTP 500) |
| Condition is false | Normal response (HTTP 200) |

## 5. Prevention

- Use parameterized queries (prepared statements).
- Never concatenate user input directly into SQL queries.
- Validate user input on the server side.
- Handle database errors securely.
- Return generic error messages instead of database details.
- Apply least-privilege permissions to database accounts.
- Perform regular security testing to identify SQL injection vulnerabilities.

## 6. Key Takeaways

- Conditional error-based SQL injection is a type of Blind SQL Injection.
- It uses database errors to reveal information.
- `CASE WHEN` allows errors to occur only when a condition is true.
- `TO_CHAR(1/0)` can trigger a division-by-zero error in Oracle.
- `LENGTH()` helps determine the length of a value.
- `SUBSTR()` extracts individual characters from a string.
- Passwords can be extracted character by character by observing error responses.
- Even when query results are hidden, database errors can leak sensitive information.

# SQL Injection – Visible Error-Based SQL Injection

## 1. Why Does It Happen?

- Applications sometimes execute SQL queries using user-controlled input.
- Even if query results are not displayed, database error messages may reveal sensitive information.
- Some databases include the actual query value inside error messages.
- Attackers can intentionally trigger database errors to expose data.
- This technique is called **Visible Error-Based SQL Injection**.

## 2. Payload Used

The following examples use a `TrackingId` cookie and PostgreSQL-style SQL syntax.

**Step 1: Trigger a SQL error**

```sql
'
```

An unclosed quotation mark can generate a database error.

**Step 2: Close the query and comment out the remaining SQL**

```sql
'--
```

**Step 3: Test a basic SQL subquery**

```sql
' AND 1=CAST((SELECT 1) AS int)--
```

**Step 4: Retrieve a username through an error message**

```sql
' AND 1=CAST((SELECT username FROM users LIMIT 1) AS int)--
```

**Step 5: Retrieve the administrator's password**

```sql
' AND 1=CAST((SELECT password FROM users LIMIT 1) AS int)--
```

## 3. Payload Explanation

- `'` – Closes the existing string in the SQL query.
- `AND` – Adds another condition to the original query.
- `1=` – Creates a comparison that expects an integer value.
- `CAST()` – Converts a value from one data type to another.
- `SELECT 1` – Executes a simple subquery returning the value `1`.
- `SELECT username FROM users` – Retrieves usernames from the users table.
- `SELECT password FROM users` – Retrieves passwords from the users table.
- `LIMIT 1` – Restricts the result to one row.
- `AS int` – Converts the returned value into an integer.
- `--` – Comments out the remaining part of the SQL query.

**Important:** If the selected value is text, converting it to an integer can generate a database error. Some database error messages reveal the original text value.

## 4. How Does It Work?

1. First, inject a single quotation mark to check whether the application produces a SQL error.
2. Use `--` to comment out the remaining part of the original query.
3. Test a basic subquery using `CAST()`.
4. Retrieve a value from the `users` table.
5. Convert the returned text value into an integer using `CAST()`.
6. Because the text cannot be converted into an integer, the database generates an error.
7. The error message may reveal the original text value.
8. Use `LIMIT 1` to make sure the subquery returns only one row.
9. Retrieve the required information, such as the administrator's password, through the error message.

**Example Error:**

```text
ERROR: invalid input syntax for type integer: "administrator"
```

Here, the database error reveals the username `administrator`.

## 5. Prevention

- Use parameterized queries (prepared statements).
- Never concatenate user input directly into SQL queries.
- Validate user input on the server side.
- Never expose detailed database error messages to users.
- Log database errors securely on the server.
- Apply least-privilege permissions to database accounts.
- Perform regular security testing to identify SQL injection vulnerabilities.

## 6. Key Takeaways

- Visible Error-Based SQL Injection uses database error messages to expose information.
- `CAST()` converts a value from one data type to another.
- Invalid type conversion can reveal the original database value through an error.
- `LIMIT 1` restricts a query to one returned row in PostgreSQL.
- Sensitive information can be extracted even when normal query results are hidden.
- Detailed database error messages can create serious security risks.
# Blind SQL Injection – Time Delays

## 1. Why Does It Happen?

- Sometimes an application executes SQL queries but does not display query results.
- The application may also respond in the same way whether the query succeeds or fails.
- However, attackers can inject SQL commands that intentionally delay the database response.
- By measuring the response time, attackers can identify whether the injected SQL command was executed.
- This technique is called **Blind SQL Injection with Time Delays**.

## 2. Payload Used

The following example uses PostgreSQL syntax and a tracking cookie named `TrackingId`.

**Step 1: Inject a time-delay payload**

```sql
x'||pg_sleep(10)--
```

**Step 2: Observe the response time**

- If the application takes approximately 10 seconds to respond, the injected delay was executed.
- If there is no delay, the payload may not have worked or the SQL syntax may be incorrect.

## 3. Payload Explanation

- `x` – Represents the original tracking cookie value.
- `'` – Closes the existing string in the SQL query.
- `||` – Concatenates strings in PostgreSQL.
- `pg_sleep(10)` – PostgreSQL function that pauses query execution for 10 seconds.
- `--` – Comments out the remaining part of the SQL query.

## 4. How Does It Work?

1. First, identify an input field or cookie that may be vulnerable to SQL injection.
2. Inject a time-delay payload into the input.
3. Use `pg_sleep(10)` to make the database wait for 10 seconds.
4. Submit the request to the application.
5. Measure how long the server takes to respond.
6. If the response is delayed by approximately 10 seconds, it indicates that the database executed the injected sleep function.
7. This confirms a potential Blind SQL Injection vulnerability even when query results are not visible.

**Example:**

| Payload | Expected response |
|---|---|
| Normal input | Normal response time |
| `pg_sleep(10)` | Approximately 10-second delay |

## 5. Prevention

- Use parameterized queries (prepared statements).
- Never concatenate user input directly into SQL queries.
- Validate user input on the server side.
- Apply least-privilege permissions to database accounts.
- Avoid allowing user input to execute database functions.
- Monitor unusual database query execution times.
- Perform regular security testing to identify SQL injection vulnerabilities.

## 6. Key Takeaways

- Blind SQL Injection can exist even when query results are not displayed.
- Time-based SQL injection uses response delays to identify successful SQL execution.
- `pg_sleep(10)` pauses PostgreSQL query execution for 10 seconds.
- `||` is the string concatenation operator in PostgreSQL.
- Response time can reveal information about database execution.
- Time-based SQL injection is useful when normal responses and error messages do not reveal query results.
- Database-specific delay functions have different syntax.

# Blind SQL Injection – Time Delays and Information Retrieval

## 1. Why Does It Happen?

- Sometimes an application executes SQL queries but does not display query results or database errors.
- The application may return the same response whether a query succeeds or fails.
- However, attackers can inject SQL commands that create a conditional time delay.
- By measuring the response time, attackers can determine whether a specific SQL condition is true or false.
- This technique allows attackers to retrieve sensitive information character by character.
- It is called **Blind SQL Injection with Time Delays and Information Retrieval**.

## 2. Payload Used

The following examples use PostgreSQL syntax and a tracking cookie named `TrackingId`.

**Step 1: Test a true condition**

```sql
x';SELECT CASE WHEN (1=1) THEN pg_sleep(10) ELSE pg_sleep(0) END--
```

**Step 2: Test a false condition**

```sql
x';SELECT CASE WHEN (1=2) THEN pg_sleep(10) ELSE pg_sleep(0) END--
```

**Step 3: Check whether the administrator account exists**

```sql
x';SELECT CASE WHEN (username='administrator') THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users--
```

**Step 4: Determine the password length**

```sql
x';SELECT CASE WHEN (username='administrator' AND LENGTH(password)>1) THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users--
```

Change the number to test different password lengths.

**Step 5: Retrieve password characters one by one**

```sql
x';SELECT CASE WHEN (username='administrator' AND SUBSTRING(password,1,1)='a') THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users--
```

Change the character position and test different characters until the complete password is identified.

## 3. Payload Explanation

- `x` – Represents the original tracking cookie value.
- `'` – Closes the existing string in the SQL query.
- `;` – Separates SQL statements.
- `SELECT` – Executes a SQL query.
- `CASE WHEN` – Checks a condition and selects an action based on its result.
- `1=1` – Always true condition.
- `1=2` – Always false condition.
- `pg_sleep(10)` – Pauses PostgreSQL execution for 10 seconds.
- `pg_sleep(0)` – Does not introduce a delay.
- `FROM users` – Specifies the table containing user information.
- `username='administrator'` – Checks the administrator account.
- `LENGTH(password)` – Returns the number of characters in the password.
- `SUBSTRING(password,1,1)` – Extracts one character from the password, starting at position 1.
- `--` – Comments out the remaining part of the SQL query.

## 4. How Does It Work?

1. First, inject a condition that is always true.
2. If the response takes approximately 10 seconds, the time-delay function is working.
3. Test a false condition and confirm that the response is immediate.
4. Check whether the administrator account exists using the `users` table.
5. Use `LENGTH(password)` to determine the password length.
6. Change the tested length until the condition becomes false.
7. Once the password length is known, use `SUBSTRING()` to extract individual characters.
8. Test possible characters at each password position.
9. A correct character triggers a 10-second delay.
10. An incorrect character produces a normal response.
11. Repeat the process for every character position to reconstruct the password.

**Example Response:**

| Condition | Expected response |
|---|---|
| Condition is true | Approximately 10-second delay |
| Condition is false | Normal response |
| Password character matches | Approximately 10-second delay |
| Password character does not match | Normal response |

## 5. Prevention

- Use parameterized queries (prepared statements).
- Never concatenate user input directly into SQL queries.
- Validate user input on the server side.
- Apply least-privilege permissions to database accounts.
- Avoid allowing untrusted input to execute database functions.
- Monitor unusual database query execution times.
- Perform regular security testing to identify SQL injection vulnerabilities.

## 6. Key Takeaways

- Time-based Blind SQL Injection retrieves information through response-time differences.
- `CASE WHEN` executes different actions depending on a condition.
- `pg_sleep(10)` creates a 10-second delay in PostgreSQL.
- `LENGTH()` helps determine the length of a password.
- `SUBSTRING()` extracts individual password characters.
- Passwords can be retrieved character by character.
- Automated testing can help identify the correct character at each position.
- Even when query results and errors are hidden, response timing can expose sensitive information.

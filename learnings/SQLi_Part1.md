## SQL injection vulnerability in WHERE clause allowing retrieval of hidden data:

General query to retrieve items in an e-com:
SELECT * FROM Products WHERE category= 'Gifts' AND released = 1
- We can retreive hidden data from the database by commenting out the relased condition of the query in the search bar or any entry point.
- There are two ways to acheive this
- We can use a simple '-- (comment)
- Also, we can use 'OR1=1 Since 1=1 is true in all cases it retrieves all the information.

## SQLi vulnerability in login bypass

General query in login where username and password is admin:
SELECT * FROM users WHERE username='admin' AND password='admin'
- We can bypass the login by commenting out the password part of the query.
- Search bar is generally not the entry point for logins. Therefore, we use the command in the Username textbox.
Ex: Username : admin'--
- The limitation here is we must know the username

## SQLi querying DB type and version of Oracle

- We were told that there was a vulnerability in the Category section.
- I used mitmproxy to proxy the query since it was not available via the searchbar.
- First, I determined the number of colums in the DB.
- Later, in mitm we edited the query by referring the Cheatsheat since we already knew it was Oracle.
- Note: if the command does not solve the lab we should interchange the column names.
- mitmproxy alternatives: Burpsuite, Zap 


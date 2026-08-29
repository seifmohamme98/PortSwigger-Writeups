
## Vulnerability Summary

The product category filter is vulnerable to SQL injection. Using a UNION-based attack, it's possible to query the underlying Oracle database directly and retrieve its type and version information.

## Reconnaissance

- I intercepted a request in Burp while browsing the "Tech gifts" category, which showed the category parameter being sent in the URL: `GET /filter?category=Tech+gifts`

![Intercepted request showing category parameter](Img/Pasted%20image%2020260829045727.png)
- Before jumping into the version extraction, I needed to confirm the number of columns and their data types. I tried `Tech gifts' UNION SELECT 'abc','def' FROM dual--`

![Testing column count and data type with dual table](Img/Pasted%20image%2020260829051130.png)
- This returned the page normally with all the products listed, no errors — confirming the original query returns exactly 2 columns, and both accept text data.


![Products page returned normally confirming 2 text columns](Img/Pasted%20image%2020260829050043.png)
## Exploitation Steps

- Since the dual table trick worked for the column count test, I used the same approach but replaced one of the placeholder values with BANNER from Oracle's built-in `v$version` table, which holds version details about the database.
- Payload:  `'+UNION+SELECT+BANNER,+NULL+FROM+v$version--`

![Payload request extracting BANNER from v$version](Img/Pasted%20image%2020260829050214.png)
- The response returned multiple rows (since `v$version` contains several version-related entries), each showing up as a separate "product" on the page. One of them read: `Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production`, confirming the database type and version.

![Oracle database version revealed in response](Img/Pasted%20image%2020260829050459.png)
## Impact

- This lets an attacker fingerprint the exact database engine and version powering the application, which is valuable reconnaissance for planning further attacks — different Oracle versions have different known vulnerabilities, available built-in functions, and quirks that could be exploited next.
- Beyond just the version, the same UNION technique could be extended to query other Oracle system tables and views, potentially exposing far more sensitive information like usernames, schema names, or stored data.

## Root Cause

- The application builds the SQL query by directly concatenating the category parameter into the query string, instead of treating it as a bound parameter, allowing the injected UNION SELECT to run as part of the actual query.
- Database error messages and query results aren't being restricted either, which is what allowed the injected data (the version banner) to be reflected directly back in the page's normal output.

## What I Learned
- I learned that different databases have different quirks when it comes to UNION injection
- This lab showed me that UNION attacks aren't just for pulling application data like usernames or products  they can also be used to fingerprint the database itself, which is often the first real step in a more targeted attack.


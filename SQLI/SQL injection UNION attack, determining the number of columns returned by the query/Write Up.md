## Vulnerability Summary

The product category filter is vulnerable to SQL injection. Using a UNION-based attack, it's possible to determine the exact number of columns returned by the underlying query, which is the first step toward extracting data from other tables.


## Reconnaissance

1. I intercepted a request in Burp while browsing the "Tech gifts" category, which showed the category parameter being sent in the URL: `GET /filter?category=Tech+gifts`
2. Since the goal here was specifically to find the number of columns, I moved straight into testing with `UNION SELECT NULL` payloads rather than doing any separate probing step first.
![[Pasted image 20260830083442.png]]

## Exploitation Steps

1. I sent the request to Repeater
2. started testing with a single NULL

![[Pasted image 20260830083649.png]]

- This returned a 500 Internal Server Error, meaning the original query returns more than one column
3. I added a Tech+gifts'+UNION+SELECT+NULL,NULL

![[Pasted image 20260830083920.png]]

- Still got a 500 Internal Server Error, so I kept going.
4. I added a Tech+gifts'+UNION+SELECT+NULL,NULL,NULL
![[Pasted image 20260830084147.png]]

- This time the response came back as 200 OK with no errors, confirming the original query returns exactly 4 columns

before: 
![[Pasted image 20260830082723.png]]

After:

![[Pasted image 20260830082603.png]]

## Impact

- On its own, determining the column count doesn't expose any sensitive data, but it's a critical reconnaissance step for a full UNION-based attack. without it, any attempt to extract real data like usernames, passwords.  would fail with a database error.
- This confirms the application is genuinely vulnerable to SQL injection and that the query results are reflected directly in the response, both of which are necessary conditions for a successful UNION attack.

## What I Learned
- I learned the systematic approach for finding the column count in a UNION attack, starting with one NULL and incrementally adding more until the error disappears, rather than guessing randomly.
- This lab made it clear why NULL specifically is used for this test: it's compatible with virtually any column data type, so it won't cause a type-mismatch error even before the column count is correct.
- 
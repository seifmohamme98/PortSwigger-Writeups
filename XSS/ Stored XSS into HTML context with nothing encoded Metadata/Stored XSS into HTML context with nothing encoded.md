## Metadata

- **Difficulty:** APPRENTICE
- **Category:** XSS
- **Lab URL:** [Lab: Stored XSS into HTML context with nothing encoded](https://portswigger.net/web-security/cross-site-scripting/stored/lab-html-context-nothing-encoded)
- **Date Solved:** 25/8/2026
- **Tool useed:** comment filed in web

## Vulnerability Summary
The blog comment functionality is vulnerable to Stored XSS. User input submitted through the comment form is stored and later reflected without any encoding, allowing arbitrary JavaScript to execute for anyone who views the blog post.

## Exploitation Steps
1. Since this is a Stored XSS lab, I figured the key was to find an input that actually gets saved in the database, so the payload would run for anyone visiting the page, not just the person submitting it.
2. Looking at the page, the comment section stood out as the perfect spot for this — comments get stored and displayed to every visitor.
3. I went with a straightforward JavaScript payload: `<script>alert("Hacked")</script>`.
4. I filled the rest of the fields (name, email, website) with random values, just to get past the form validation, then submitted the comment.
![Comment form filled with script payload](Img/Pasted%20image%2020260831090532.png)

5. Sure enough, the payload got stored and executed successfully the alert popped up confirming the script ran.
![Alert popup confirming stored XSS execution](Img/Pasted%20image%2020260831090555.png)

## Impact
- Since the payload is stored server-side, it doesn't require sending a crafted link to a specific victim  it just needs someone to visit the blog post, making this far more dangerous than a reflected XSS.
- An attacker could use this to steal session cookies from any visitor (including admins or other users), hijack accounts, perform actions on their behalf, or redirect them to a malicious site — all without any interaction beyond just viewing the page.

## What I Learned
- I learned the key difference between reflected and stored XSS: stored payloads persist on the server and affect every visitor, not just the one who submitted the request, which makes them significantly more dangerous.
- This lab reinforced the idea that any input field which gets saved and later displayed back to other users like comments, reviews, or profile fields  is a prime target for this kind of attack.

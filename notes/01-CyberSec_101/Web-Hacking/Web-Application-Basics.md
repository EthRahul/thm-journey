# Web Application Basics
**Path:** CyberSec_101 - Web Hacking
**Date:** 2026-03-13
**Difficulty:** Easy

---

## Commands Used

- `curl -X GET https://site.com/api/users` → Make GET request
- `curl -X POST https://site.com/api/user/2 -d "country=US"` → POST with data
- `curl -X DELETE https://site.com/api/user/1` → DELETE request
- `curl -I https://site.com` → Show only response headers
- `curl -H "Cookie: session=abc123" https://site.com` → Send custom header

---

## How a Web Application Works
```
User (Browser)  →  Request  →  Web Server  →  Database
User (Browser)  ←  Response ←  Web Server  ←  Database
```

- Frontend → what user sees (HTML, CSS, JavaScript)
- Backend → server side logic (Python, PHP, Java)
- Database → stores data (MySQL, MongoDB)

Three tier architecture:
- Tier 1 → Presentation (browser/frontend)
- Tier 2 → Logic (web server/backend)
- Tier 3 → Data (database)

---

## URL Structure — Every Part Explained
```
https://www.example.com:443/path/page?name=rahul&age=20#section
  |          |            |      |           |              |
scheme     host          port  path        query         fragment
```

- scheme → protocol used (http or https)
- host → domain name or IP of the server
- port → 80 for HTTP, 443 for HTTPS (hidden if default)
- path → location of the resource on the server
- query → extra data sent to server (starts with ?)
- fragment → jumps to a section on the page (starts with #)

---

## HTTP Request — Full Breakdown
```
GET /api/users HTTP/1.1
Host: tryhackme.com
User-Agent: Mozilla/5.0 Firefox/87.0
Accept: text/html
Accept-Language: en-US
Accept-Encoding: gzip, deflate
Cookie: session=abc123
Content-Type: application/json
Content-Length: 27
Connection: keep-alive

{"username": "rahul"}
```

Every part explained:

- GET → HTTP method (what action you want)
- /api/users → path (where on the server)
- HTTP/1.1 → version of HTTP being used
- Host → which website you are talking to
- User-Agent → identifies your browser or tool
- Accept → what response formats you can handle
- Accept-Language → preferred language
- Accept-Encoding → compression formats supported
- Cookie → session data sent with every request
- Content-Type → format of the request body
- Content-Length → size of body in bytes
- Connection → keep-alive reuses connection for speed
- Body → actual data being sent (after blank line)

---

## HTTP Response — Full Breakdown
```
HTTP/1.1 200 OK
Server: nginx/1.15.8
Date: Fri, 13 Mar 2026 16:25 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 68
Set-Cookie: session=xyz789; HttpOnly; Secure
Last-Modified: Fri, 13 Mar 2026 16:25 GMT
Cache-Control: no-store
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
Strict-Transport-Security: max-age=31536000
Content-Security-Policy: default-src 'self'

<html>
  <body>Response content here</body>
</html>
```

Every part explained:

- HTTP/1.1 → version of HTTP
- 200 OK → status code and message
- Server → web server software running
- Date → when response was sent
- Content-Type → format of response body
- Content-Length → size of response body in bytes
- Set-Cookie → server sets cookie on your browser
- Last-Modified → when resource was last changed
- Cache-Control → how browser should cache response
- Empty line → separates headers from body
- HTML below → actual content (the response body)

---

## HTTP Methods

- GET → retrieve data, no body sent
- POST → send data to server (forms, logins, updates)
- PUT → replace entire resource with new data
- PATCH → partially update a resource
- DELETE → delete a resource
- HEAD → same as GET but returns only headers no body
- OPTIONS → ask server what methods are allowed
- TRACE → echoes request back, used for debugging

---

## Request Body Types (Content-Type)

Different ways to send data in a POST/PUT request:

1. application/x-www-form-urlencoded (default HTML forms)
```
username=rahul&password=secret123
```

2. multipart/form-data (file uploads)
```
------boundary
Content-Disposition: form-data; name="file"; filename="shell.php"
Content-Type: application/octet-stream
[file data here]
```

3. application/json (APIs and modern web apps)
```
{"username": "rahul", "password": "secret123"}
```

4. application/xml (older APIs)
```
<user><name>rahul</name><pass>secret123</pass></user>
```

5. text/plain (raw text)
```
just plain text data here
```

---

## HTTP Status Codes — Must Memorize

- 200 OK → request succeeded
- 201 Created → resource successfully created
- 204 No Content → success but nothing to return
- 301 Moved Permanently → page moved forever
- 302 Found → temporary redirect
- 304 Not Modified → use cached version
- 400 Bad Request → something wrong with your request
- 401 Unauthorized → not logged in
- 403 Forbidden → logged in but no permission
- 404 Not Found → resource does not exist
- 405 Method Not Allowed → wrong HTTP method used
- 429 Too Many Requests → rate limited
- 500 Internal Server Error → server crashed
- 502 Bad Gateway → server got bad response upstream
- 503 Service Unavailable → server overloaded or down
**Informational Responses (100-199)**  
These codes mean the server has received part of the request and is waiting for the rest. It’s a "keep going" signal.

**Successful Responses (200-299)**  
These codes mean everything worked as expected. The server processed the request and sent back the requested data.

**Redirection Messages (300-399)**  
These codes tell you that the resource you requested has moved to a different location, usually providing the new URL.

**Client Error Responses (400-499)**  
These codes indicate a problem with the request. Maybe the URL is wrong, or you’re missing some required info, like authentication.

**Server Error Responses (500-599)**  
These codes mean the server encountered an error while trying to fulfil the request. These are usually server-side issues and not the client’s fault.

---

## Request Headers — Full List

- Host → domain being requested
- User-Agent → browser or tool making request
- Accept → acceptable response formats
- Accept-Language → preferred language
- Accept-Encoding → compression support
- Authorization → credentials (Bearer token, Basic auth)
- Cookie → session tokens and stored data
- Referer → page you came from
- Origin → domain making the request (CORS)
- Content-Type → format of data being sent
- Content-Length → size of request body
- X-Forwarded-For → original client IP (proxy header)
- Connection → keep-alive or close

---

## Response Headers — Full List

General:
- Server → web server software (nginx, Apache)
- Date → timestamp of response
- Content-Type → format of response body
- Content-Length → size of response
- Cache-Control → caching instructions
- Set-Cookie → sets cookie on browser
- Location → redirect destination URL
- Last-Modified → when resource last changed
- ETag → unique identifier for resource version

---

## Security Headers — Very Important

These headers protect users from attacks:

1. Content-Security-Policy (CSP)
- Controls what resources browser can load
- Prevents XSS attacks
- Example: Content-Security-Policy: default-src 'self'
- This means only load resources from same domain

2. X-Frame-Options
- Prevents your site being loaded in an iframe
- Stops clickjacking attacks
- Values: DENY or SAMEORIGIN
- Example: X-Frame-Options: DENY

3. Strict-Transport-Security (HSTS)
- Forces browser to always use HTTPS
- Prevents SSL stripping attacks
- Example: Strict-Transport-Security: max-age=31536000
- max-age is in seconds (31536000 = 1 year)

4. X-Content-Type-Options
- Prevents browser from guessing content type
- Stops MIME sniffing attacks
- Example: X-Content-Type-Options: nosniff

5. Referrer-Policy
- Controls how much referrer info is shared
- Example: Referrer-Policy: no-referrer
- Protects sensitive URLs from leaking

6. Permissions-Policy
- Controls browser features like camera, mic, GPS
- Example: Permissions-Policy: geolocation=(), camera=()
- Empty () means feature is completely disabled

7. Set-Cookie Security Flags
- HttpOnly → JavaScript cannot access the cookie
  (prevents XSS stealing cookies)
- Secure → cookie only sent over HTTPS
- SameSite=Strict → cookie not sent cross-site
  (prevents CSRF attacks)

---

## Cookie Attributes — Full Breakdown
```
Set-Cookie: session=abc123; 
            HttpOnly; 
            Secure; 
            SameSite=Strict; 
            Path=/; 
            Domain=example.com; 
            Expires=Fri, 13 Mar 2027 00:00:00 GMT
```

- session=abc123 → cookie name and value
- HttpOnly → blocks JavaScript from reading it
- Secure → only sent over HTTPS connection
- SameSite=Strict → only sent from same site requests
- Path=/ → which paths the cookie applies to
- Domain → which domain the cookie belongs to
- Expires → when cookie is deleted automatically

---

## HTTP vs HTTPS

- HTTP → plain text, anyone on network can read it
- HTTPS → encrypted with TLS, safe from interception
- Port 80 → HTTP default
- Port 443 → HTTPS default

---

## APIs and REST

- API → way for apps to communicate with each other
- REST API → uses HTTP methods to interact with data
- Common REST pattern:
  - GET /api/users → get all users
  - GET /api/user/1 → get user with ID 1
  - POST /api/user → create new user
  - PUT /api/user/1 → update user with ID 1
  - DELETE /api/user/1 → delete user with ID 1
- JSON is the most common format for API data

---

## Gotchas / Surprises

- 403 and 401 are different:
  401 = not logged in
  403 = logged in but blocked
- 500 errors are interesting for hackers as they can
  reveal stack traces and sensitive server info
- Missing security headers = vulnerability in bug bounty
- HttpOnly cookie flag prevents most XSS cookie theft
- X-Forwarded-For can be spoofed to bypass IP restrictions
- Content-Type must match your request body format
  or server will reject the request
- multipart/form-data is used for file uploads which
  is often where file upload vulnerabilities exist

---

## My Understanding in Plain English

Every web interaction is an HTTP request and response.
The request has a method, path, headers and optional body.
The response has a status code, headers and body.
Security headers tell the browser how to protect users.
As a hacker you manipulate requests and look for missing
security headers to find and exploit vulnerabilities.

---

## Related Notes

- [[Burp Suite]]
- [[Web Hacking]]
- [[SQLi]]
- [[XSS]]
- [[CSRF]]
- [[File Upload Vulnerabilities]]

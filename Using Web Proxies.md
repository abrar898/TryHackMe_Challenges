<img width="915" height="413" alt="image" src="https://github.com/user-attachments/assets/c70acda9-c97a-459d-8478-600b85aeeee4" />


# Using Web Proxies — Structured Notes

## Table of Contents
- [Section 1: Intro to Web Proxies](#section-1-intro-to-web-proxies)
  - [1.1 What Are Web Proxies?](#11-what-are-web-proxies)
  - [1.2 Uses of Web Proxies](#12-uses-of-web-proxies)
  - [1.3 Burp Suite](#13-burp-suite)
  - [1.4 OWASP Zed Attack Proxy (ZAP)](#14-owasp-zed-attack-proxy-zap)
- [Section 2: Setting Up](#section-2-setting-up)
  - [2.1 Burp Suite (Install & Launch)](#21-burp-suite-install--launch)
  - [2.2 ZAP (Install & Launch)](#22-zap-install--launch)
- [Section 3: Proxy Setup](#section-3-proxy-setup)
  - [3.1 Pre-Configured Browser](#31-pre-configured-browser)
  - [3.2 Proxy Setup (Firefox / FoxyProxy)](#32-proxy-setup-firefox--foxyproxy)
  - [3.3 Installing CA Certificate](#33-installing-ca-certificate)
- [Section 4: Intercepting Web Requests](#section-4-intercepting-web-requests)
  - [4.1 Intercepting Requests — Burp](#41-intercepting-requests--burp)
  - [4.2 Intercepting Requests — ZAP](#42-intercepting-requests--zap)
  - [4.3 Manipulating Intercepted Requests](#43-manipulating-intercepted-requests)
- [Section 5: Intercepting Responses](#section-5-intercepting-responses)
  - [5.1 Burp Response Interception](#51-burp-response-interception)
  - [5.2 ZAP Response Interception](#52-zap-response-interception)
- [Section 6: Automatic Modification](#section-6-automatic-modification)
  - [6.1 Automatic Request Modification — Burp Match and Replace](#61-automatic-request-modification--burp-match-and-replace)
  - [6.2 Automatic Request Modification — ZAP Replacer](#62-automatic-request-modification--zap-replacer)
  - [6.3 Automatic Response Modification](#63-automatic-response-modification)
- [Section 7: Repeating Requests](#section-7-repeating-requests)
  - [7.1 Proxy History](#71-proxy-history)
  - [7.2 Repeating Requests — Burp](#72-repeating-requests--burp)
  - [7.3 Repeating Requests — ZAP](#73-repeating-requests--zap)
- [Section 8: Encoding/Decoding](#section-8-encodingdecoding)
  - [8.1 URL Encoding](#81-url-encoding)
  - [8.2 Decoding](#82-decoding)
  - [8.3 Encoding](#83-encoding)
- [Section 9: Proxying Tools](#section-9-proxying-tools)
  - [9.1 Proxychains](#91-proxychains)
  - [9.2 Metasploit](#92-metasploit)
- [Section 10: Burp Intruder](#section-10-burp-intruder)
  - [10.1 Target](#101-target)
  - [10.2 Positions](#102-positions)
  - [10.3 Payloads](#103-payloads)
  - [10.4 Settings](#104-settings)
  - [10.5 Attack](#105-attack)
- [Section 11: ZAP Fuzzer](#section-11-zap-fuzzer)
  - [11.1 Locations](#111-locations)
  - [11.2 Payloads](#112-payloads)
  - [11.3 Processors](#113-processors)
  - [11.4 Options](#114-options)
  - [11.5 Start](#115-start)
- [Section 12: Burp Scanner](#section-12-burp-scanner)
  - [12.1 Target Scope](#121-target-scope)
  - [12.2 Crawler](#122-crawler)
  - [12.3 Passive Scanner](#123-passive-scanner)
  - [12.4 Active Scanner](#124-active-scanner)
  - [12.5 Reporting](#125-reporting)
- [Section 13: ZAP Scanner](#section-13-zap-scanner)
  - [13.1 Spider](#131-spider)
  - [13.2 Passive Scanner](#132-passive-scanner)
  - [13.3 Active Scanner](#133-active-scanner)
  - [13.4 Reporting](#134-reporting)
- [Section 14: Extensions](#section-14-extensions)
  - [14.1 BApp Store](#141-bapp-store)
  - [14.2 ZAP Marketplace](#142-zap-marketplace)
  - [14.3 Closing Thoughts](#143-closing-thoughts)
- [Cheatsheet](#cheatsheet)

---

## Section 1: Intro to Web Proxies

Modern web and mobile apps constantly talk to back-end servers to send and receive data, which is then processed on the user's device. Because so much logic now lives on the back end, testing and securing these servers has become a core part of security work. Most of Web Application Penetration Testing revolves around examining these web requests, and the tool category used to capture, view, and manipulate this traffic is the **Web Proxy**. This section introduces why proxies matter and names the two tools (Burp Suite and ZAP) that the rest of the module covers in depth.

### 1.1 What Are Web Proxies?
Web proxies sit between a browser/mobile app and the back-end server, acting as a **man-in-the-middle (MITM)** tool that captures and displays every web request passing through. Unlike general network sniffers (e.g., Wireshark) that inspect all local traffic, web proxies focus specifically on web ports such as HTTP/80 and HTTPS/443. They are considered essential for any web pentester because they make capturing and replaying requests far easier than older CLI-based approaches. Once configured, a proxy shows every HTTP request an app sends and every response the server returns, and lets the tester intercept a request mid-flight to modify it and observe how the server reacts — a core technique in web penetration testing.

### 1.2 Uses of Web Proxies
While capturing and replaying HTTP requests is the primary function, web proxies support many additional testing activities. These extra capabilities make the proxy a central hub for most web-focused assessment work rather than a single-purpose sniffing tool. The module notes it will not cover specific attack techniques (those live in other HTB Academy modules) but will show which proxy feature supports which type of attack. The listed uses are:
- Web application vulnerability scanning
- Web fuzzing
- Web crawling
- Web application mapping
- Web request analysis
- Web configuration testing
- Code reviews

### 1.3 Burp Suite
Burp Suite ("Burp") is the most widely used web proxy for penetration testing, offering a polished UI and a built-in Chromium-based browser for testing targets directly. The free **Community Edition** is already powerful enough for most day-to-day testing, while certain advanced features are locked behind the paid **Burp Pro/Enterprise** tiers. The module primarily teaches features available in the free edition but also touches on select Pro features like the Active Web App Scanner. A free Burp Pro trial is available to those with an educational or business email.

Paid-only (Pro) features include:
- Active web app scanner
- Fast Burp Intruder
- Ability to load certain Burp Extensions

### 1.4 OWASP Zed Attack Proxy (ZAP)
ZAP is a free, open-source web proxy maintained by the OWASP community, meaning it has no paid tier or feature throttling. It has grown rapidly and is becoming the leading open-source alternative to Burp. Because it's community-driven, ZAP steadily gains many features that are Pro-only in Burp, at no cost. The module frames the choice between Burp and ZAP as situational: ZAP suits cases where a paid subscription isn't justified, while Burp Pro may be preferred for more mature, enterprise-grade engagements. Learning both tools gives flexibility to pick whichever fits a given pentest's needs.

---

## Section 2: Setting Up

Both Burp and ZAP run on Windows, macOS, and Linux, and come pre-installed on PwnBox and common pentesting distros like Parrot or Kali. This section walks through installing and launching both tools manually, which is useful if setting them up on a personal VM rather than using PwnBox.

### 2.1 Burp Suite (Install & Launch)
If not already installed, Burp can be downloaded from its official download page and installed using the OS-appropriate installer (Windows, Linux, macOS), following straightforward on-screen instructions. Burp can then be launched either from the terminal, from the application menu, or by running the cross-platform JAR file directly (which requires a Java Runtime Environment, usually bundled with the installer).

**Command explained:**
```shellsession
java -jar </path/to/burpsuite.jar>
```
- `java` — invokes the Java runtime.
- `-jar` — tells Java to execute the specified file as a runnable JAR archive.
- `</path/to/burpsuite.jar>` — the path to the downloaded Burp JAR file. This method works on any OS with JRE installed and is an alternative to double-clicking the file.

**Step-by-step setup:**
1. Download Burp from the official download page (or use the pre-installed PwnBox version).
2. Run the installer appropriate to your OS, or launch via `java -jar burpsuite.jar`.
3. On startup, choose a project type: **Temporary project** (Community Edition only option) or **New project on disk / Open existing project** (Pro/Enterprise only).
4. Select **Temporary project** and click Continue (saving progress is only needed for large/long-running engagements).
5. Choose configuration: **Use Burp Defaults** (recommended for starting out) or **Load a Configuration File**.
6. Click **Start Burp** to launch the tool with the chosen settings.
7. (Optional) Enable dark theme via `Burp > Settings > User interface > Display > Theme > Dark`.

### 2.2 ZAP (Install & Launch)
ZAP is downloaded from its official download page, choosing the installer matching your OS, or alternatively as a cross-platform JAR file launched the same way as Burp (`java -jar`). It can be started via the terminal command `zaproxy` or from the application menu.

**Step-by-step setup:**
1. Download ZAP from its official page, or use the PwnBox pre-installed version.
2. Launch via the `zaproxy` terminal command, the application menu, or `java -jar <zap.jar>`.
3. On startup, ZAP asks whether to persist the session: choose **No** to use a temporary (non-persistent) project, suitable for short engagements.
4. Click **Start** to launch ZAP with the chosen session option.
5. (Optional) Enable dark theme via `Tools > Options > Display > Look and Feel > Flat Dark`.

---

## Section 3: Proxy Setup

With both tools installed, this section covers their most-used feature: acting as a web proxy. Setting up a proxy means routing an application's (usually a browser's) traffic through Burp or ZAP so every request and response can be examined, intercepted, and modified, giving deep insight into what the application is doing behind the scenes.

### 3.1 Pre-Configured Browser
Both tools ship with a pre-configured browser that already has the correct proxy settings and CA certificate installed, making it the fastest way to start testing without manual browser configuration. In Burp, this is accessed via `Proxy > Intercept > Open Browser`. In ZAP, it's accessed by clicking the Firefox icon at the end of the top toolbar. For most of the module's exercises, using this pre-configured browser is sufficient.

### 3.2 Proxy Setup (Firefox / FoxyProxy)
To proxy a real browser like Firefox instead, its proxy settings must be manually configured to point at the proxy tool's listening port — **8080 by default** for both Burp and ZAP (any free port can be used instead). If the chosen port is already in use, the proxy will fail to start with an error. The listening port can be changed in Burp (`Proxy > Proxy settings > Proxy listeners`) or ZAP (`Tools > Options > Network > Local Servers/Proxies`), but the Firefox-side proxy setting must always match.

Rather than manually editing Firefox proxy settings each time, the **FoxyProxy** extension allows quick proxy switching. It comes pre-installed on PwnBox and can otherwise be added from the Firefox Extensions page.

**Step-by-step (FoxyProxy setup):**
1. Install the FoxyProxy extension (pre-installed on PwnBox).
2. Click the FoxyProxy icon in Firefox's toolbar, then select **Options**.
3. On the Options page, click **Add** in the left pane.
4. Enter IP `127.0.0.1`, Port `8080`, and a name (e.g., "Burp" or "ZAP"), then click **Save**.
5. Click the FoxyProxy icon again and select the saved **Burp/ZAP** profile to activate it.

*(Note: this configuration already exists in PwnBox, so steps 1–4 can be skipped there.)*

### 3.3 Installing CA Certificate
Installing the proxy tool's CA certificate in the browser is necessary so HTTPS traffic routes correctly without constant manual "accept" prompts. Without it, some HTTPS requests may fail to proxy properly.

**Step-by-step (Burp certificate):**
1. With Burp selected as the active proxy in FoxyProxy, browse to `http://burp`.
2. Click **CA Certificate** to download it.
3. In Firefox, go to `about:preferences#privacy`, scroll down, and click **View Certificates**.
4. Go to the **Authorities** tab, click **Import**, and select the downloaded certificate.
5. Check **Trust this CA to identify websites** and **Trust this CA to identify email users**, then click **OK**.

**Step-by-step (ZAP certificate):**
1. In ZAP, go to `Tools > Options > Network > Server Certificates`.
2. Click **Save** to export the certificate (or **Generate** first to create a new one).
3. Follow the same Firefox import steps as above (`about:preferences#privacy > View Certificates > Authorities > Import`).

Once the certificate is installed and the proxy configured, all Firefox traffic routes through the chosen web proxy.

---

## Section 4: Intercepting Web Requests

With the proxy configured, this section shows how to actually intercept, inspect, and manipulate HTTP requests before they reach the server — the core workflow of using a web proxy for testing.

### 4.1 Intercepting Requests — Burp
In Burp's **Proxy** tab, request interception is on by default and can be toggled via the **Intercept is on/off** button under the Intercept sub-tab. With interception on, visiting a target site in the pre-configured browser causes requests to pause in Burp awaiting action; clicking **Forward** sends the request onward. Since all Firefox traffic gets intercepted, unrelated requests may appear first — keep clicking Forward until the target request appears.

### 4.2 Intercepting Requests — ZAP
In ZAP, interception is **off by default**, shown by a green toggle button on the top bar. It can be toggled on/off by clicking that button or using the shortcut **CTRL+B**. Intercepted requests appear in the top-right pane; clicking the **step** button (next to the red break button) forwards the request. ZAP also offers the **Heads Up Display (HUD)**, enabled via a button at the end of the top menu, which surfaces most ZAP features directly inside the pre-configured browser. Within the HUD, the second button on the left pane toggles request interception. Captured requests appear in a popup with **Step** (forward one request and inspect its response) or **Continue** (forward all remaining requests) options.

### 4.3 Manipulating Intercepted Requests
Once intercepted, a request stays paused until forwarded, giving the tester a window to examine and edit it before it reaches the server. This is foundational to testing for vulnerability classes such as SQL injection, command injection, upload bypass, authentication bypass, XSS, XXE, error handling, and deserialization.

**Example command (intercepted raw request):**
```http
POST /ping HTTP/1.1
Host: 94.237.62.138:32306
Content-Type: application/x-www-form-urlencoded
...
ip=1
```
- `POST /ping HTTP/1.1` — the HTTP method, target path, and protocol version being sent to the server.
- `Host` — the server address and port the request is destined for.
- `Content-Type: application/x-www-form-urlencoded` — tells the server the body is form data.
- `ip=1` — the request body; a single parameter `ip` with value `1`, which is the editable injection point.

**Step-by-step (demonstrating command injection via interception):**
1. Turn on request interception in Burp or ZAP.
2. On the target page, enter an IP value and click the **Ping** button.
3. Observe the intercepted POST request to `/ping` with body `ip=1`.
4. Change the `ip` parameter's value from `1` to `;ls;` (front-end JS normally blocks non-numeric input, but intercepting bypasses this).
5. Click **Continue/Forward** to send the modified request.
6. Observe that the response now shows the output of the `ls` command instead of the normal ping output — confirming the back end does not validate input server-side.

---

## Section 5: Intercepting Responses

Beyond requests, it's sometimes necessary to intercept the **server's response** before it reaches the browser — for example, to enable disabled form fields or reveal hidden fields, which can assist penetration testing by removing client-side restrictions.

### 5.1 Burp Response Interception
Response interception in Burp is enabled under `Proxy > Proxy settings > Intercept Response` (under Response interception rules). Once enabled alongside request interception, refreshing the page with **CTRL+SHIFT+R** (hard refresh) and forwarding the request shows the intercepted HTML response for editing.

**Command/code explained:**
```html
<input type="text" id="ip" name="ip" min="1" max="255" maxlength="100"
    oninput="javascript: if (this.value.length > this.maxLength) this.value = this.value.slice(0, this.maxLength);"
    required>
```
- `type="text"` — changed from `type="number"`, removing the browser's numeric-only input restriction.
- `maxlength="100"` — changed from `3`, allowing much longer input strings (needed for payloads like `;ls;`).
- `oninput="..."` — JavaScript that trims input exceeding `maxLength`; left intact but now permits more characters due to the raised limit.

**Step-by-step:**
1. Enable **Intercept Response** under `Proxy > Proxy settings`.
2. Re-enable request interception and force-refresh the page (`CTRL+SHIFT+R`).
3. Forward the intercepted request; the intercepted **response** appears.
4. Edit `type="number"` to `type="text"` and `maxlength="3"` to `maxlength="100"`.
5. Forward the modified response and return to the browser to confirm the input field now accepts any text.

### 5.2 ZAP Response Interception
In ZAP, after stepping through an intercepted request, the response is automatically intercepted next, allowing the same HTML edits as in Burp before clicking **Continue**. ZAP's HUD additionally offers a one-click **Show/Enable** feature (the light-bulb icon, third button on the left pane) that automatically enables disabled fields or reveals hidden fields without needing to manually intercept and edit any response. Burp has an equivalent under `Proxy > Proxy settings > Response modification rules` (e.g., "Unhide hidden form fields"). ZAP's HUD also has a **Comments** indicator feature that reveals the content of HTML comments in the page (added via the `+` button on the left pane, selecting **Comments**), useful for spotting information unintentionally left in source code.

---

## Section 6: Automatic Modification

Rather than manually intercepting and editing every single request/response, both tools support **rule-based automatic modification**, applying a defined match-and-replace rule to all matching traffic automatically.

### 6.1 Automatic Request Modification — Burp Match and Replace
Configured under `Proxy > Proxy settings > HTTP match and replace rules`, using **Add**. The example replaces the `User-Agent` header with a custom value — useful for bypassing filters that block specific User-Agent strings.

**Rule settings explained:**
- **Type: Request header** — specifies the modification targets the request's headers, not its body.
- **Match: `^User-Agent.*$`** — a regex pattern matching the entire User-Agent header line.
- **Replace: `User-Agent: HackTheBox Agent 1.0`** — the full replacement line.
- **Regex match: True** — enables regex interpretation of the Match field since the exact original value is unknown.

**Step-by-step:**
1. Go to `Proxy > Proxy settings > HTTP match and replace rules` and click **Add**.
2. Set Type to **Request header**.
3. Set Match to `^User-Agent.*$` with Regex match enabled.
4. Set Replace to `User-Agent: HackTheBox Agent 1.0`.
5. Click **OK** — the rule activates immediately.
6. Verify by browsing any site in the pre-configured browser and checking the intercepted request's User-Agent header.

### 6.2 Automatic Request Modification — ZAP Replacer
Accessed via **CTRL+R** or `Replacer` in ZAP's options menu, offering equivalent functionality to Burp's Match and Replace.

**Rule settings explained:**
- **Description** — a human-readable label for the rule (e.g., "HTB User-Agent").
- **Match Type: Request Header** — targets request headers (and adds the header if not already present).
- **Match String: User-Agent** — the specific header to replace, selectable from a dropdown.
- **Replacement String: HackTheBox Agent 1.0** — the new header value.
- **Enable: True** — activates the rule.

ZAP also supports a regex-based **Request Header String** match type, and an **Initiators** tab to control where the rule applies (default: all HTTP(S) messages).

**Step-by-step:**
1. Open Replacer with **CTRL+R**, click **Add**.
2. Fill in Description, Match Type (Request Header), Match String (User-Agent), Replacement String, and set Enable to True.
3. Save the rule.
4. Enable request interception (**CTRL+B**) and visit any page in the pre-configured ZAP browser to confirm the header is replaced.

### 6.3 Automatic Response Modification
The same match-and-replace concept applies to **responses**, solving the problem from Section 5 where manual response edits don't persist across page refreshes.

**Burp rule settings explained:**
- **Type: Response body** — targets the HTML body of the response rather than headers.
- **Match: `type="number"`** — the literal string to find (no regex needed since the exact text is known).
- **Replace: `type="text"`** — the replacement string.
- **Regex match: False** — disabled since this is a literal string match.

**Step-by-step:**
1. Go to `Proxy > Options > Match and Replace` in Burp and click **Add**.
2. Set Type to **Response body**, Match to `type="number"`, Replace to `type="text"`, Regex off.
3. Add a second rule changing `maxlength="3"` to `maxlength="100"`.
4. Hard-refresh the page (**CTRL+SHIFT+R**) — the input field now permanently accepts any input across refreshes.
5. Click **Ping** to confirm the command injection payload now works without needing manual interception.

---

## Section 7: Repeating Requests

Manually re-intercepting and re-editing a request for every single test is slow (5–6 steps per command). **Request repeating** solves this by resending a previously captured request with quick edits, without needing to intercept live traffic each time.

### 7.1 Proxy History
Request history is viewed in Burp under `Proxy > HTTP History`, and in ZAP via the HUD's bottom History pane or the main UI's **History** tab. Both tools support filtering/sorting to locate specific requests among large volumes of traffic. Both also maintain separate **WebSockets history** for asynchronous connections, which is outside this module's scope. Clicking any history entry shows its full request/response details; Burp uniquely lets you view both the **Original Request** and the **Edited Request** if changes were made.

### 7.2 Repeating Requests — Burp
**Step-by-step:**
1. Locate the target request in Proxy History.
2. Press **CTRL+R** to send it to the Repeater tab.
3. Navigate to Repeater (or press **CTRL+SHIFT+R** directly).
4. Click **Send** to send the request and view the response.
5. (Optional) Right-click and select **Change Request Method** to switch between POST/GET without rewriting the request.
6. Edit any part of the request text directly and click **Send** again to resend with changes.

### 7.3 Repeating Requests — ZAP
**Step-by-step:**
1. Locate the request in history, right-click, and select **Open/Resend with Request Editor**.
2. Use the **Method** dropdown to change the HTTP method if needed.
3. Click **Send** to resend the request.
4. Alternatively, in the HUD, click the request in the bottom History pane to open the Request Editor.
5. Choose **Replay in Console** (response shown in HUD) or **Replay in Browser** (response rendered in-browser).
6. Edit the request text as needed before resending.

As noted at the end of this section, POST data is typically URL-encoded, which leads into the next section on encoding/decoding.

---

## Section 8: Encoding/Decoding

Modifying and resending custom HTTP requests often requires correct encoding/decoding so the server interprets the data properly. Both tools include built-in encoders/decoders for this purpose.

### 8.1 URL Encoding
Certain characters must be URL-encoded or the server may return an error:
- **Spaces** — may be misread as the end of the request data.
- **`&`** — interpreted as a parameter delimiter.
- **`#`** — interpreted as a fragment identifier.

**Step-by-step (Burp Repeater URL-encoding):**
1. Select the text to encode in Burp Repeater.
2. Right-click and choose `Convert Selection > URL > URL-encode key characters`, or press **CTRL+U**.
3. (Optional) Enable "URL-encode as you type" via right-click for automatic encoding while typing.

ZAP automatically URL-encodes request data in the background before sending, without requiring a manual step. Other encoding types mentioned include Full URL-Encoding and Unicode URL encoding for requests with many special characters.

### 8.2 Decoding
Web applications frequently encode data (e.g., cookies), and testers need to quickly decode it to read the original value, or encode data to match what the back end expects. Supported encoder types include HTML, Unicode, Base64, and ASCII hex.

**Step-by-step (Burp Decoder):**
1. Go to the **Decoder** tab in Burp.
2. Paste the encoded string, e.g.: `eyJ1c2VybmFtZSI6Imd1ZXN0IiwgImlzX2FkbWluIjpmYWxzZX0=`
3. Select **Decode as > Base64**.
4. View the decoded output: `{"username":"guest", "is_admin":false}`.

Recent Burp versions also offer the **Burp Inspector** tool (found in Proxy/Repeater panels) for quick inline encoding/decoding.

**Step-by-step (ZAP Encoder/Decoder/Hash):**
1. Press **CTRL+E** to open the Encoder/Decoder/Hash tool.
2. Paste the encoded text into the input field.
3. View automatic decoding results across multiple decoder tabs in the **Decode** tab.
4. (Optional) Click **Add New Tab** to create a custom tab with specific encoders/decoders.

### 8.3 Encoding
After decoding and modifying data (e.g., changing `"username":"guest"` to `"username":"admin"` and `"is_admin":false` to `true`), the value must be re-encoded back into its original format (e.g., Base64) before reuse in a request.

**Step-by-step:**
1. Decode the original value to inspect/edit it (as in 8.2).
2. Modify the plaintext as needed (e.g., `guest` → `admin`, `false` → `true`).
3. Re-encode using the same original method (e.g., Base64) — in Burp, select the new encoder in the output pane to chain encode/decode operations; in ZAP, copy the output and paste it back into the input field for re-processing.
4. Use the resulting encoded string in a request via Burp Repeater or ZAP Request Editor.

---

## Section 9: Proxying Tools

Beyond browsers, web proxies can also intercept traffic from **command-line tools and thick-client applications**, giving visibility into requests made outside a browser context. Each tool's proxy must be set to `http://127.0.0.1:8080` (or whatever port the proxy listens on), and the exact method varies per tool. Note: proxying typically slows down the proxied tool, so it should only be used when investigating requests, not for routine use.

### 9.1 Proxychains
**Proxychains** is a Linux tool that routes traffic from any CLI tool through a specified proxy, making it the simplest method for proxying command-line applications.

**Step-by-step:**
1. Edit `/etc/proxychains.conf`.
2. Comment out the final line (e.g., `#socks4 127.0.0.1 9050`).
3. Add the line: `http 127.0.0.1 8080`.
4. Run commands prefixed with `proxychains -q` (quiet mode suppresses connection info clutter).

**Command explained:**
```shellsession
proxychains -q curl http://SERVER_IP:PORT
```
- `proxychains` — routes the following command's traffic through the configured proxy.
- `-q` — quiet mode; suppresses proxychains' own connection logging output.
- `curl http://SERVER_IP:PORT` — the actual command being proxied, a simple HTTP GET request.

After running, the request appears in the web proxy's history (e.g., Burp Proxy tab), confirming traffic was routed correctly.

### 9.2 Metasploit
Metasploit modules can also be proxied to investigate and debug their HTTP traffic using the `PROXIES` flag.

**Command explained:**
```shellsession
msfconsole
msf6 > use auxiliary/scanner/http/robots_txt
msf6 auxiliary(scanner/http/robots_txt) > set PROXIES HTTP:127.0.0.1:8080
msf6 auxiliary(scanner/http/robots_txt) > set RHOST SERVER_IP
msf6 auxiliary(scanner/http/robots_txt) > set RPORT PORT
msf6 auxiliary(scanner/http/robots_txt) > run
```
- `msfconsole` — launches the Metasploit Framework console.
- `use auxiliary/scanner/http/robots_txt` — selects the robots.txt scanner module.
- `set PROXIES HTTP:127.0.0.1:8080` — routes this module's HTTP traffic through the proxy at the given address/port.
- `set RHOST SERVER_IP` / `set RPORT PORT` — sets the target host and port.
- `run` — executes the module against the configured target.

**Step-by-step:**
1. Launch `msfconsole`.
2. Select the desired module with `use`.
3. Set `PROXIES`, `RHOST`, and `RPORT` as needed.
4. Run the module with `run`.
5. Check the proxy tool's history to confirm the module's requests were captured.

This same proxying approach can be applied to any scanner, exploit, script, or thick-client application by configuring its proxy settings accordingly.

---

## Section 10: Burp Intruder

**Burp Intruder** is Burp's built-in web fuzzer, usable for fuzzing pages, directories, sub-domains, parameters, and parameter values — an alternative to CLI tools like ffuf, dirbuster, gobuster, or wfuzz. The free Community version is throttled to **1 request/second**, making it suitable only for short queries, while the Pro version has unlimited speed.

### 10.1 Target
The Target panel shows the host and port being fuzzed, auto-populated from the request sent to Intruder.

**Step-by-step:**
1. Browse to the target exercise in the pre-configured browser.
2. In Proxy History, right-click the relevant request and select **Send to Intruder** (or press **CTRL+I**).
3. Open the Intruder tab (or press **CTRL+SHIFT+I**) to view the Target details.

### 10.2 Positions
**Positions** defines where in the request the payload (wordlist words) will be inserted and iterated.

**Step-by-step:**
1. In the Positions sub-tab, locate the word to fuzz (e.g., `DIRECTORY` in `GET /DIRECTORY/`).
2. Select the word and click **Add §** (or manually wrap it with `§` markers) to mark it as the payload position.
3. Leave the two trailing blank lines at the end of the request intact to avoid server errors.

### 10.3 Payloads
This section configures what values are tried at the marked position, covering four sub-areas:

**Payload Position & Payload Type:**
- The **Payload set** corresponds to the marked position (only one here, since Attack type is **Sniper**).
- **Payload Type** options include:
  - **Simple List** — iterates line-by-line over a provided wordlist.
  - **Runtime file** — like Simple List but loads lines on-the-fly to save memory on very large wordlists.
  - **Character Substitution** — tries all permutations of specified character replacements.

**Payload Configuration:**
- For Simple List, load a wordlist file (e.g., `/opt/useful/seclists/Discovery/Web-Content/common.txt`) via **Load**, or add items manually via **Add**. Multiple sources can be combined into one list.

**Payload Processing:**
- Allows rule-based filtering/transformation of the wordlist. Example: adding a **Skip if matches regex** rule with pattern `^\..*$` to skip lines starting with a dot.

**Payload Encoding:**
- Toggles automatic URL-encoding of payload characters (left enabled by default, covering characters like `./^=<>&+?*:;'{}|^`).

### 10.4 Settings
Additional attack-wide options, including:
- **Number of retries on network failure** and **Pause before retry** — can be set to 0 for faster scanning.
- **Grep - Match** — flags responses containing a specific string (e.g., `200 OK`), useful for quickly spotting valid hits among many results. Configured by enabling it, clearing defaults, adding the match string, and disabling **Exclude HTTP Headers** if the match target is in the header.
- **Grep - Extract** — extracts and displays only a specific part of lengthy responses (not needed when simply filtering by status code).
- **Resource Pool** — controls network resource usage for large attacks (left at default for this example).

### 10.5 Attack
**Step-by-step:**
1. Click **Start Attack** to begin.
2. Observe that lines matching the skip rule (e.g., starting with `.`) are excluded from results.
3. Sort results by the **200 OK** column, or by **Status**/**Length**, to find successful hits.
4. Identify the hit (e.g., payload `admin` returning status 200) and manually verify by visiting the discovered path directly (e.g., `http://SERVER_IP:PORT/admin/`).

Intruder can similarly be used for password brute-forcing, parameter fuzzing, or AD-based password spraying (e.g., against OWA, SSL VPNs, RDS, Citrix), though its free-tier speed throttling limits practicality for large-scale attacks — which motivates the next section on ZAP's unthrottled fuzzer.

---

## Section 11: ZAP Fuzzer

**ZAP Fuzzer** is ZAP's equivalent to Burp Intruder. It lacks some of Intruder's advanced features but, importantly, has **no speed throttling**, making it more practical for larger fuzzing jobs on the free tier.

### 11.1 Locations
Equivalent to Intruder's Payload Position — marks where payloads will be inserted into the request.

**Step-by-step:**
1. Capture a sample request (e.g., visiting `http://SERVER_IP:PORT/test/`).
2. In Proxy History, right-click the request and select `Attack > Fuzz` to open the Fuzzer window.
3. Select the word to fuzz (e.g., `test`) and click **Add** on the right pane — this places a green marker and opens the Payloads window.

### 11.2 Payloads
Defines the wordlist(s) to use, selectable from 8 payload types, including:
- **File** — load a custom wordlist from disk.
- **File Fuzzers** — use ZAP's built-in wordlist databases (e.g., dirbuster lists), a key advantage over Intruder since no external wordlist is required.
- **Numberzz** — generates numeric sequences with custom increments.

**Step-by-step:**
1. Click **Add** in the Payloads window.
2. Select **File Fuzzers** as the Type.
3. Choose a built-in list (e.g., dirbuster's `directory-list-1.0.txt`).
4. Click **Add** to attach the wordlist; use **Modify** to review it.

### 11.3 Processors
Optional transformations applied to each payload word before sending, including Base64 Decode/Encode, MD5/SHA-1/256/512 Hash, Prefix/Postfix String, URL Decode/Encode, and custom Script processors.

**Step-by-step:**
1. Select **URL Encode** as the processor to ensure special characters in payloads don't break the request.
2. Click **Generate Preview** to confirm how payloads will appear once encoded.
3. Click **Add**, then **OK** to close the Payloads and Processors windows.

### 11.4 Options
Attack-wide settings similar to Intruder's Settings tab:
- **Concurrent Scanning Threads per Scan** — e.g., set to 20 for faster execution (limited by local CPU and server connection limits).
- **Depth First** — tries all wordlist values against one payload position before moving to the next position (e.g., all passwords for one user first).
- **Breadth First** — tries one wordlist value across all positions before moving to the next value (e.g., one password across all users first).

### 11.5 Start
**Step-by-step:**
1. Click **Start Fuzzer** to begin the attack.
2. Sort results by **Response code** to find entries with status 200.
3. Identify hits (e.g., payload `skills` returning 200 OK) and click the result to view full request/response details.
4. Use other indicators like **Size Resp. Body** (differing response size may indicate a distinct page) or **RTT** (response time delay, useful for time-based attacks like blind SQL injection) to spot less obvious hits.

---

## Section 12: Burp Scanner

Burp Scanner combines a **Crawler** (to map site structure) with **passive and active scanning** to detect web vulnerabilities. It is a **Pro-only feature**, unavailable in the free Community edition, reflecting its enterprise-level scope.

### 12.1 Target Scope
Before scanning, defining a **Target Scope** restricts which URLs are included, saving resources by ignoring out-of-scope items.

**Step-by-step:**
1. Go to `Target > Site map` to view all detected directories/files from proxied traffic.
2. Right-click an item and select **Add to scope**.
3. Optionally right-click an in-scope item and select **Remove from scope** to exclude risky items (e.g., a logout endpoint).
4. Go to `Target > Scope` to review/edit the scope list, including advanced regex-based include/exclude rules.

A scan can be started three ways: from a specific request in Proxy History, on a custom set of targets via **New Scan**, or on all in-scope items.

### 12.2 Crawler
The Crawler builds a site map by following links, forms, and requests found on target pages (unlike fuzzing tools like ffuf/dirbuster, it does not guess unreferenced pages).

**Step-by-step:**
1. Go to the Dashboard tab and click **New Scan**.
2. Choose **Crawl** (or **Crawl and Audit** to also run the scanner afterward).
3. In Scan configuration, click **Select from library** and choose a preset (e.g., "Crawl strategy - fastest").
4. In Application login, optionally provide credentials or record a manual login flow for authenticated scanning (left empty if no credentials available).
5. Click **OK** to start the scan; monitor progress under the Dashboard's **Tasks** pane.
6. Once finished ("Crawl Finished"), review the updated map at `Target > Site map`.

### 12.3 Passive Scanner
A **Passive Scan** analyzes already-captured page source without sending new requests, flagging potential issues (e.g., missing security headers, possible DOM-based XSS) along with a confidence rating, though it cannot verify findings without active testing.

**Step-by-step:**
1. Right-click a target in `Target > Site map` or Proxy History.
2. Select **Do passive scan** or **Passively scan this target**.
3. Once finished, click **View Details** and review the **Issue activity** tab for severity and confidence ratings.
4. Prioritize issues with **High severity** and **Certain/Firm confidence**, while still reviewing all findings for sensitive applications.

### 12.4 Active Scanner
The **Active Scanner** is the most comprehensive scan, performing (in order): a crawl + fuzz to find pages, a passive scan on all pages, verification requests for passive findings, JavaScript analysis, and fuzzing of identified parameters for vulnerabilities like XSS, Command Injection, and SQL Injection.

**Step-by-step:**
1. Right-click a request and select **Do active scan**, or use **New Scan** on the Dashboard and choose **Crawl and Audit**.
2. Configure Crawl settings as before, plus **Audit configurations** (vulnerability types to test, payload insertion points, etc.).
3. Select an Audit preset via **Select from library** (e.g., "Audit checks - critical issues only" for focusing on severe, server-compromising issues).
4. Optionally add login credentials, then click **OK** to start.
5. Monitor progress in the Dashboard's Tasks pane, or view live requests via the **Logger** tab.
6. Once complete, check **Issue activity**, filtering by **High** severity and **Certain** confidence to find the most actionable findings (e.g., an OS command injection on the `ip` parameter).

### 12.5 Reporting
**Step-by-step:**
1. Go to `Target > Site map`, right-click the target, and select `Issue > Report issues for this host`.
2. Choose the export format and select which information to include.
3. Open the exported report in a browser to review severity/confidence breakdowns and remediation guidance.

Note: scanner reports should supplement, not replace, a properly written client deliverable — they're best used as appendix data or for tracking-dashboard imports.

---

## Section 13: ZAP Scanner

ZAP bundles its own scanner, combining **ZAP Spider** (site mapping) with passive and active vulnerability scanning — ZAP's equivalent to Burp Scanner, but available fully for free.

### 13.1 Spider
**ZAP Spider** maps a site by following and validating links, similar to Burp's Crawler.

**Step-by-step:**
1. Locate a request in History, right-click, and select `Attack > Spider` — or use the HUD's **Spider Start** button (second button, right pane) after visiting the target page.
2. If prompted that the site isn't in scope, choose **Yes** to add it automatically.
3. Click **Start** on the popup to begin spidering.
4. Monitor progress via the HUD's Spider button or the main UI's Spider tab.
5. Once complete, review the discovered structure under the **Sites** tab or HUD's **Sites Tree** button (first button, right pane).

An additional **Ajax Spider** (third button, right pane) also detects links generated via JavaScript AJAX requests, useful as a follow-up to the standard Spider for more thorough coverage, though it takes longer.

### 13.2 Passive Scanner
As the Spider crawls, ZAP automatically runs a **passive scanner** on each response, flagging issues like missing security headers or DOM-based XSS even before an active scan starts. The left pane's alerts show issues on the current page; the right pane shows all alerts found across the application so far.

**Step-by-step:**
1. Let the Spider run (passive scanning happens automatically alongside it).
2. Check the **Alerts** tab in the main ZAP UI for the full findings list.
3. Click any alert to see its details and the specific pages it was found on.

### 13.3 Active Scanner
The **Active Scan** actively tests all mapped pages and parameters for vulnerabilities, running the Spider automatically first if it hasn't already been run.

**Step-by-step:**
1. Click the **Active Scan** button on the HUD's right pane (or trigger it from the main UI).
2. Monitor progress as ZAP sends test requests and the Alerts count grows.
3. Once finished, review alerts — click **High Alerts** to jump to the most severe findings (e.g., Remote OS Command Injection).
4. Click an alert for full details, including the attack example and evidence (e.g., `/etc/passwd` content leaked).
5. Click the alert's URL to view the exact request/response, and optionally replay it via ZAP HUD or Request Editor to confirm.

### 13.4 Reporting
**Step-by-step:**
1. Select `Report > Generate HTML Report` from the top bar.
2. Choose a save location (other formats like XML or Markdown are also available).
3. Open the generated report in a browser to review a categorized summary (e.g., counts of High/Medium/Low/Informational alerts) for use as a testing log or reference during the engagement.

---

## Section 14: Extensions

Both Burp and ZAP support community-built **extensions/add-ons** that extend their core functionality — Burp via its **BApp Store**, and ZAP via its **ZAP Marketplace**.

### 14.1 BApp Store
**Step-by-step:**
1. Click the **Extensions** tab in Burp, then the **BApp Store** sub-tab.
2. Sort extensions by **Popularity** to find the most widely used ones.
3. Select an extension (e.g., **Decoder Improved**) and click **Install**.
4. Once installed, access the extension via its new dedicated tab in Burp.

Note: some extensions are Pro-only, and some require additional dependencies not installed by default (e.g., **Jython**) before they can be used.

Notable extensions mentioned include: .NET Beautifier, J2EEScan, Software Vulnerability Scanner, Software Version Reporter, Active Scan++, Additional Scanner Checks, AWS Security Checks, Backslash Powered Scanner, Wsdler, Java Deserialization Scanner, C02, Cloud Storage Tester, CMS Scanner, Error Message Checks, Detect Dynamic JS, Headers Analyzer, HTML5 Auditor, PHP Object Injection Check, JavaScript Security, Retire.JS, CSP Auditor, Random IP Address Header, Autorize, CSRF Scanner, and JS Link Finder.

### 14.2 ZAP Marketplace
**Step-by-step:**
1. Click **Manage Add-ons** on the toolbar, then select the **Marketplace** tab.
2. Browse available add-ons, noting their build status (**Release** = stable; **Beta/Alpha** = may have issues).
3. Select and install desired add-ons (e.g., **FuzzDB Files** and **FuzzDB Offensive**, which add new wordlists for ZAP's fuzzer).
4. Use the newly available wordlists in fuzzing attacks (e.g., an OS Command Injection wordlist under `fuzzdb > attack > os-cmd-execution` for command injection fuzzing).

### 14.3 Closing Thoughts
The module closes by summarizing that both Burp Suite and ZAP are essential, complementary tools for web application penetration testing, each with free and paid/enterprise tiers offering different depth of features. Their value extends beyond offensive security testers to blue team practitioners and developers. The recommended next step is practicing on web-focused Hack The Box machines and other HTB Academy web modules, treating both tools as must-have staples alongside Nmap, Hashcat, Wireshark, tcpdump, sqlmap, Ffuf, and Gobuster.

---

## Cheatsheet

A quick command/shortcut reference for this module.

### Burp Shortcuts
| Shortcut | Description |
|---|---|
| `CTRL+R` | Send to repeater |
| `CTRL+SHIFT+R` | Go to repeater |
| `CTRL+I` | Send to intruder |
| `CTRL+SHIFT+I` | Go to intruder |
| `CTRL+U` | URL encode |
| `CTRL+SHIFT+U` | URL decode |

### ZAP Shortcuts
| Shortcut | Description |
|---|---|
| `CTRL+B` | Toggle intercept on/off |
| `CTRL+R` | Go to replacer |
| `CTRL+E` | Go to encode/decode/hash |

### Firefox Shortcuts
| Shortcut | Description |
|---|---|
| `CTRL+SHIFT+R` | Force refresh page |

### Key Commands Reference
| Command | Purpose |
|---|---|
| `java -jar burpsuite.jar` | Launch Burp from its JAR file |
| `zaproxy` | Launch ZAP from terminal |
| `burpsuite` | Launch Burp from terminal |
| `proxychains -q curl http://SERVER_IP:PORT` | Route a CLI tool's traffic through a proxy quietly |
| `set PROXIES HTTP:127.0.0.1:8080` (Metasploit) | Route a Metasploit module's traffic through a proxy |

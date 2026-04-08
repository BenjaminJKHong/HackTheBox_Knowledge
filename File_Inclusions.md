# 📁 HTB: INTRO TO FILE INCLUSIONS
### Module Summary & Attack Methodology

<p align="center">
  <img src="https://img.shields.io/badge/Platform-HackTheBox-green?style=for-the-badge&logo=hackthebox"/>
  <img src="https://img.shields.io/badge/Vuln-LFI%20%2F%20RFI-red?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Difficulty-Medium-orange?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Target-Web%20Apps-blue?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Cert-CPSA%20%2F%20CRT-yellow?style=for-the-badge"/>
</p>

---

## 📌 1.1 What are File Inclusions?

**File Inclusion** vulnerabilities occur when a web application allows users to control which file is loaded by the server-side script. 

Modern back-end languages use HTTP parameters to specify what is shown on the web page (e.g., loading headers, footers, or language files dynamically to reduce script size). If these functionalities are not securely coded without strict sanitization, an attacker can manipulate these parameters to force the server to display the content of unintended local files, leading to a **Local File Inclusion (LFI)**.

---

## ✨ Cause and Effect

- 🔴 **The Cause** : Insecure coding practices where user-controlled parameters (like `?page=about`) are passed directly into file-loading functions without explicit filtering.
  
- 🟢 **The Effect**: Attackers can traverse directories and read local files on the hosting server.
  
- 💥 **Impact Escalation** ; 
  - **Source Code Disclosure:** Reveals backend logic, potentially exposing previously unknown vulnerabilities.
  - **Sensitive Data Exposure:** Leaks credentials, database keys, and server configuration files.
  - **Remote Code Execution (RCE):** Gaining full system control via log poisoning, PHP wrappers, or malicious uploads.

---

## 💻 Vulnerable Code Examples

Different frameworks handle dynamic loading differently, but all share the common risk of executing or reading files from user-specified paths.

### PHP
Functions like `include()`, `require()`, and `file_get_contents()` are highly susceptible when input is not sanitized.

```php
if (isset($_GET['language'])) {
    include($_GET['language']);
}
```
### NodeJS
URL parameters (either GET parameters or URL paths) used directly in fs.readFile() or Express res.render().

```JavaScript
app.get("/about/:language", function(req, res) {
    res.render(`/${req.params.language}/about.html`);
});
```

### Java
Using include or import functions to render objects, local files, or URLs into front-end templates based on HTTP parameters.
```Java
<c:import url= "<%= request.getParameter('language') %>"/>
```

### .NET
Functions like Response.WriteFile or @Html.Partial() taking unvalidated file paths from the URL query.
```C#
@Html.Partial(HttpContext.Request.Query['language'])
```

## 🧱 Function Capabilities: Read vs Execute
Identifying the exact function in use determines the exploit path during a code audit. Executing files can lead to immediate code execution, while reading file content is limited to source code disclosure.

<img width="634" height="755" alt="image" src="https://github.com/user-attachments/assets/48dea19c-1906-4907-a7c7-6527b4bed734" />

---
---
## 📂 1.2 Exploiting LFI & Path Traversal

Once an LFI vulnerability is identified, the goal is to manipulate the input parameter to read sensitive local files on the server, such as `/etc/passwd` on Linux or `C:\Windows\boot.ini` on Windows.

### 🟢 Basic LFI

If the web application passes the input directly into the `include()` function without any modification, you can specify the absolute path to the file.

```php
// Backend PHP
include($_GET['language']);
```

## 🔀 Path Traversal
Developers often append or prepend strings to the user parameter.
```PHP
// Backend PHP
include("./languages/" . $_GET['language']);
```
To bypass this, we use Relative Paths to traverse directories by adding ../ (which refers to the parent directory). We go back several directories until we reach the root path (/), and then specify our target file.

- Payload: ?language=../../../../etc/passwd

  💡 Pro-Tip: Adding excessive ../ sequences will not break the path once you reach the root directory. However, for clean exploits and reports, calculate the exact number of directories you are away from the root.


## 🛡️ Bypassing String Manipulations
**1. Filename Prefix**

**Code     :** include("lang_" . $_GET['language']);

**Bypass   :** Prefix a / to trick the system into treating the prefix as a directory.

**Payload  :** ?language=/../../../etc/passwd (Note: The directory lang_/ must exist for this to work).


**2. Appended Extensions**

**Code  :** include($_GET['language'] . ".php");

**Bypass:** Requires advanced techniques (Null Byte Injection %00 or Path Truncation in older PHP versions).



## 🔄 Second-Order LFI Attacks
LFI isn't limited to direct URL parameters. A Second-Order Attack occurs when a malicious payload is stored in a database and later pulled by an insecure file-loading function.


**1) 💉 Inject :** A user registers an account with a malicious username: ../../../etc/passwd

**2) ⚙️ Execute:** The application attempts to load the user's avatar from /profile/$username/avatar.png.

**3) 💥 Impact :** The system dynamically constructs the path /profile/../../../etc/passwd/avatar.png, triggering the LFI when the avatar loads.


## Recap
---

## 🗺️ Module Syllabus & Attack Vectors

While basic LFI allows for reading files, this module focuses heavily on escalating those flaws into **Remote Code Execution (RCE)**. The upcoming sections cover the following advanced techniques:

- 📂 **Path Traversal:** Escaping intended web directories using `../` sequences to access the root filesystem.
- 🛡️ **Bypassing Restrictions:** Evading basic LFI filters, sanitization attempts, and WAFs.
- 💊 **PHP Wrappers:** Utilizing built-in filters (like `php://filter` or `data://`) to encode data or execute payloads.
- 🌐 **Remote File Inclusion (RFI):** Forcing the application to load and execute malicious scripts from an attacker-controlled server.
- 📤 **Malicious Uploads:** Combining arbitrary file upload vulnerabilities with LFI to trigger web shells.
- ☠️ **Log Poisoning:** Injecting malicious PHP code into server logs (like Apache or SSH logs) and then "including" the log file to execute the code.
- 🤖 **Automated Exploitation:** Utilizing tools to streamline the discovery and exploitation phases.
- 🔒 **Prevention:** Secure coding practices and configurations to mitigate inclusion vulnerabilities.

---

## 🎯 Prerequisites & Environment

To successfully navigate this module and its accompanying hands-on exercises, a foundational understanding of the following is assumed:

- 🐧 **Linux Fundamentals:** Comfortable navigating the Linux command line.
- 🕸️ **Web Requests:** Understanding how HTTP GET/POST requests function and how to intercept/manipulate them.
- 📡 **Introduction to Networking:** A firm grasp of basic networking concepts and protocols.

> **💡 Pro-Tip:** As you work through the module, it is highly recommended to reproduce all example commands in the provided **PwnBox** or your own local virtual machine to build practical muscle memory. There is no time limit, so take your time to thoroughly understand each tactic.

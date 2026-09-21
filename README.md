# CWE-78: OS Command Injection

This README explains CWE-78, a common and high-impact software vulnerability class, for educational purposes. It covers what the vulnerability is, why it happens, what it looks like in code, and how to prevent it.

## What is CWE-78?

CWE-78 is the Common Weakness Enumeration entry for **"Improper Neutralization of Special Elements used in an OS Command"**, more commonly known as **OS Command Injection**.

It occurs when an application builds a system command using untrusted input (such as data from a user, a web request, or a file) and passes that command to the underlying operating system shell for execution. If the input isn't properly validated or escaped, an attacker can insert extra shell syntax to run commands the developer never intended.

## Why it happens

Command injection typically arises when a program needs to call an external OS utility (like `ping`, `curl`, `convert`, `tar`, etc.) and takes a shortcut by concatenating user-supplied input directly into a shell command string, instead of invoking the program safely with separated arguments.

Shells interpret certain characters specially, including:

- `;`, `&&`, `||` — command chaining/separators
- `|` — piping output into another command
- `` ` `` and `$()` — command substitution
- `>`, `<`, `&` — redirection and backgrounding

If any of these reach the shell unescaped inside attacker-controlled input, they can be used to append or substitute additional commands.

## What vulnerable code looks like

Below is a simplified, illustrative example (not runnable as-is) showing the pattern to avoid:

```python
import os

def check_host(hostname):
    # VULNERABLE: hostname is concatenated directly into a shell command
    os.system(f"ping -c 1 {hostname}")
```

If `hostname` is meant to be something like `example.com`, but the application doesn't validate it, an attacker could supply a value containing shell metacharacters to chain on an additional command. The vulnerability is the *pattern* of building a shell string from untrusted input — not any specific payload.

## How to prevent it

- **Avoid the shell entirely when possible.** Use language APIs or libraries that perform the needed action natively instead of shelling out (e.g., a DNS/ICMP library instead of calling `ping`).
- **Use argument arrays, not string concatenation.** When you must call an external program, pass arguments as a list/array to an API that does not invoke a shell (e.g., Python's `subprocess.run([...], shell=False)`), so the OS never interprets shell metacharacters.
- **Validate and allow-list input.** Restrict input to a strict expected format (e.g., a valid IPv4/hostname regex) and reject anything else, rather than trying to blocklist "dangerous" characters.
- **Run with least privilege.** If a command must run, ensure the process has the minimum OS permissions needed, limiting the damage of any injection that slips through.
- **Sandbox and isolate.** Container or sandbox environments that execute external commands, so a successful injection has a smaller blast radius.
- **Use static analysis / SAST tools.** Many code-scanning tools can flag calls like `os.system`, `exec`, `popen`, or shell-invoking APIs fed by untrusted input.

## Further reading

- MITRE CWE-78: https://cwe.mitre.org/data/definitions/78.html
- OWASP Command Injection: https://owasp.org/www-community/attacks/Command_Injection
- OWASP WebGoat and PortSwigger Web Security Academy offer safe, guided hands-on labs for practicing identification and remediation of this vulnerability class.

## Disclaimer

This document is intended purely to explain the vulnerability class conceptually, for defensive and educational understanding. 

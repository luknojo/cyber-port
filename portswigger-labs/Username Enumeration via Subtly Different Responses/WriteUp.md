# Username Enumeration via Subtly Different Responses

## Overview

This lab demonstrates a more subtle form of username enumeration. Unlike traditional enumeration vulnerabilities that return clearly different error messages, this application returns nearly identical responses. However, a minor discrepancy in the error message allows attackers to identify valid usernames and subsequently perform a password brute-force attack.

## Objective

* Identify a valid username through subtle response differences.
* Brute-force the associated password.
* Authenticate successfully and access the user's account page.

---

## Reconnaissance

The login functionality was tested using invalid credentials while intercepting traffic with Burp Suite.

The intercepted request looked similar to:

```http
POST /login HTTP/1.1

username=invalid-user&password=invalid-password
```

The username parameter was selected and sent to Burp Intruder for further testing.

---

## Username Enumeration

### Intruder Configuration

**Attack Type:** Sniper

**Payload Position:**

```http
username=§invalid-user§&password=test
```

The supplied username wordlist was loaded as the payload source.

---

## Response Analysis

At first glance, every response appeared identical and returned the same error message:

```text
Invalid username or password.
```

To identify subtle differences, Burp Intruder's **Grep - Extract** feature was configured to extract the error message from each response.

### Discovery

After the attack completed, the extracted values were sorted and reviewed.

Most responses contained:

```text
Invalid username or password.
```

However, one response contained:

```text
Invalid username or password 
```

The difference was extremely subtle: the valid username response ended with a trailing space instead of a period.

This discrepancy revealed the existence of a valid username.

---

## Password Brute Force

Once a valid username was identified, the Intruder attack was reconfigured to target the password parameter.

### Updated Request

```http
username=valid-user&password=§password§
```

The candidate password list was loaded as the payload source.

### Results

Most requests returned:

```http
HTTP/1.1 200 OK
```

One request returned:

```http
HTTP/1.1 302 Found
```

The redirect indicated successful authentication.

The corresponding payload was identified as the correct password.

---

## Authentication

Using the valid username and discovered password, login was successful.

After authentication, the account page was accessible, completing the lab objectives.

---

## Vulnerability Analysis

### Root Cause

The application attempted to hide username validity by returning generic authentication errors.

However, a subtle implementation inconsistency introduced a detectable difference between:

* Invalid usernames
* Valid usernames with incorrect passwords

Although visually similar, the responses were not identical.

### Vulnerable Behavior

| Scenario                       | Response                        |
| ------------------------------ | ------------------------------- |
| Invalid username               | `Invalid username or password.` |
| Valid username, wrong password | `Invalid username or password ` |

This tiny formatting difference was enough to enable username enumeration.

---

## Security Impact

An attacker can:

1. Enumerate valid usernames.
2. Narrow the scope of password attacks.
3. Increase the efficiency of credential brute-forcing.
4. Gain unauthorized access if weak passwords are used.

Even minor differences in application responses can leak sensitive information.

---

## Remediation

### Ensure Identical Responses

Authentication failures should return exactly the same:

* Message content
* Length
* Formatting
* Status code
* Response timing

Example:

```text
Invalid username or password.
```

### Additional Defenses

* Implement rate limiting.
* Enforce account lockout policies.
* Use multi-factor authentication (MFA).
* Monitor authentication logs for abuse.
* Introduce CAPTCHA after repeated failures.

---

## Key Takeaway

This lab highlights how even a single extra character can introduce a username enumeration vulnerability. Security testing should include detailed analysis of response content, length, formatting, and timing, as attackers often exploit subtle differences that are easily overlooked during development.
<img width="682" height="403" alt="image" src="https://github.com/user-attachments/assets/bc1fd520-e932-428d-a6f0-4e1dadd5ac27" />
<img width="749" height="651" alt="image" src="https://github.com/user-attachments/assets/4075cf55-fd26-44eb-80d7-148cec777905" />
<img width="1300" height="618" alt="image" src="https://github.com/user-attachments/assets/d81f0280-6960-4d21-8350-9bb0ecb5b5b0" />


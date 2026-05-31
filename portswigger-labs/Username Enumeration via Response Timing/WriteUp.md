# Username Enumeration via Response Timing

## Overview

This lab demonstrates a username enumeration vulnerability based on response timing differences. Although the application returns identical error messages for failed authentication attempts, variations in processing time allow attackers to distinguish valid usernames from invalid ones.

The application also implements IP-based brute-force protection, but this can be bypassed by spoofing the client IP address using the `X-Forwarded-For` header.

## Objective

* Enumerate a valid username using response timing analysis.
* Bypass IP-based login protections.
* Brute-force the user's password.
* Successfully authenticate and access the account page.

---

## Initial Analysis

The login functionality was tested using invalid credentials while intercepting traffic with Burp Suite.

A typical authentication request looked similar to:

```http
POST /login HTTP/1.1

username=invalid-user&password=invalid-password
```

During testing, repeated login attempts resulted in temporary IP-based blocking, indicating the presence of brute-force protection.

---

## Identifying the Protection Bypass

Further testing revealed that the application trusted the `X-Forwarded-For` header.

By supplying different values in this header, requests appeared to originate from different IP addresses, effectively bypassing the rate-limiting mechanism.

Example:

```http
X-Forwarded-For: 1
```

```http
X-Forwarded-For: 2
```

```http
X-Forwarded-For: 3
```

This allowed large numbers of authentication attempts without triggering account lockouts or IP bans.

---

## Timing-Based Username Enumeration

### Observations

Testing revealed an important difference in response behavior:

* Invalid usernames returned responses in a consistent amount of time.
* Valid usernames caused slightly longer processing times.
* The delay became more noticeable when extremely long passwords were supplied.

This suggested that the application performed additional password validation steps only when a valid username existed.

---

## Username Enumeration Attack

The login request was sent to Burp Intruder.

### Attack Configuration

**Attack Type:** Pitchfork

The request was modified to include:

```http
X-Forwarded-For: §ip§
```

```http
username=§username§&password=AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
```

### Payloads

#### Position 1 – X-Forwarded-For

Configured as:

* Payload type: Numbers
* Range: 1–100
* Step: 1

This generated a unique spoofed IP address for every request.

#### Position 2 – Username

Loaded with the provided candidate username list.

---

## Identifying the Valid Username

After the attack completed, the following Intruder columns were enabled:

* Response received
* Response completed

The response times were reviewed and compared.

One username consistently produced significantly longer response times than the others.

Repeated testing confirmed that the timing difference was reproducible.

This username was identified as a valid account.

---

## Password Brute Force

After identifying a valid username, a second Intruder attack was configured.

### Updated Request

```http
X-Forwarded-For: §ip§
```

```http
username=valid-user&password=§password§
```

### Payloads

#### Position 1 – X-Forwarded-For

Numbers payload:

```text
1
2
3
...
100
```

#### Position 2 – Password

Loaded with the provided password list.

---

## Identifying the Correct Password

Most responses returned:

```http
HTTP/1.1 200 OK
```

One request returned:

```http
HTTP/1.1 302 Found
```

The redirect indicated successful authentication.

The corresponding password payload was identified as the valid credential.

---

## Authentication

Using the discovered username and password, authentication succeeded.

Accessing the user account page completed the lab objectives.

---

## Vulnerability Analysis

### Root Cause

The application leaked information through authentication timing differences.

The authentication process likely followed a workflow similar to:

```text
1. Check if username exists
2. If username exists:
      Validate password hash
3. Return authentication result
```

Because password verification only occurred for existing users, valid usernames required additional processing time.

This timing discrepancy enabled attackers to enumerate accounts.

---

## Security Impact

An attacker can:

* Identify valid usernames without relying on visible error messages.
* Bypass account discovery protections.
* Increase the efficiency of password attacks.
* Gain unauthorized access to user accounts.

Timing attacks are particularly dangerous because they can bypass applications that appear secure from a content-analysis perspective.

---

## Remediation

### Constant-Time Authentication Logic

Authentication should consume approximately the same amount of time regardless of whether a username exists.

For example:

* Always perform a password hash operation.
* Use dummy password hashes for non-existent users.
* Avoid branching logic that significantly alters execution time.

### Additional Protections

* Enforce rate limiting.
* Implement account lockout mechanisms.
* Require multi-factor authentication (MFA).
* Monitor for high-volume authentication attempts.
* Avoid trusting client-controlled headers such as `X-Forwarded-For` unless processed through trusted proxies.

---

## Key Takeaway

This lab demonstrates that authentication systems can leak sensitive information through timing differences even when response messages are identical. Combined with improper trust of the `X-Forwarded-For` header, these timing discrepancies allowed both username enumeration and password brute-forcing, ultimately leading to account compromise.

<img width="802" height="444" alt="image" src="https://github.com/user-attachments/assets/8ee4bf08-1bb4-4735-8d80-5c933a58e62e" />
<img width="608" height="622" alt="image" src="https://github.com/user-attachments/assets/fe71bb05-2f48-4457-b9fa-30cd2c1c80be" />
<img width="1355" height="616" alt="image" src="https://github.com/user-attachments/assets/ba8b412c-c5ec-4ea0-ac31-e164633af1e9" />





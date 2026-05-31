# Username Enumeration via Different Responses

## Overview

This lab demonstrates a common authentication vulnerability where the application reveals different error messages depending on whether a username exists. By analyzing these responses, it is possible to enumerate valid usernames and subsequently perform a password brute-force attack against the identified account.

## Objective

* Enumerate a valid username.
* Brute-force the associated password.
* Successfully authenticate and access the user account page.

---

## Reconnaissance

After accessing the login page, an authentication attempt was made using invalid credentials.

The application returned an error message indicating that the username was invalid.

Using Burp Suite, the authentication request was intercepted and examined:

```http
POST /login HTTP/1.1

username=invalid-user&password=invalid-password
```

---

## Username Enumeration

The username parameter was sent to Burp Intruder.

### Intruder Configuration

**Attack Type:** Sniper

**Payload Position:**

```http
username=§invalid-user§&password=test
```

The provided username wordlist was loaded as the payload source.

After launching the attack, the responses were analyzed by comparing:

* Response length
* Response content
* Status codes

### Discovery

Most requests returned a response similar to:

```text
Invalid username
```

However, one response produced a different message:

```text
Incorrect password
```

This indicated that the username existed in the application, but the supplied password was incorrect.

A valid username was therefore identified.

---

## Password Brute Force

With a valid username confirmed, the Intruder attack was reconfigured.

### Updated Request

```http
username=valid-user&password=§password§
```

The candidate password list was loaded as the payload source.

### Results

Most authentication attempts returned:

```http
HTTP/1.1 200 OK
```

One request produced a different response:

```http
HTTP/1.1 302 Found
```

The redirect indicated a successful login attempt.

The corresponding payload was identified as the correct password.

---

## Authentication

Using the enumerated username and discovered password, authentication was successful.

The account page became accessible, completing the lab objectives.

---

## Vulnerability Analysis

The root cause of the vulnerability is the application's inconsistent authentication responses.

### Vulnerable Behavior

| Scenario                       | Response             |
| ------------------------------ | -------------------- |
| Invalid username               | "Invalid username"   |
| Valid username, wrong password | "Incorrect password" |

These differences allow attackers to determine whether a username exists.

### Security Impact

An attacker can:

1. Enumerate valid usernames.
2. Reduce the attack surface for brute-force attacks.
3. Increase the likelihood of successful credential attacks.

---

## Remediation

To prevent username enumeration:

### Use Generic Error Messages

Instead of:

```text
Invalid username
```

and

```text
Incorrect password
```

Return:

```text
Invalid username or password
```

for all failed authentication attempts.

### Additional Protections

* Implement account lockout policies.
* Apply rate limiting.
* Use CAPTCHA after repeated failures.
* Monitor authentication logs for suspicious activity.
* Enforce strong password policies and MFA.
<img width="1323" height="592" alt="image" src="https://github.com/user-attachments/assets/1c0e21fe-ae53-4d34-bcf7-b16162189e4f" />
<img width="619" height="387" alt="image" src="https://github.com/user-attachments/assets/dd2e9bee-6dce-43e6-a440-666049825e77" />



---

## Conclusion

This lab demonstrated how subtle differences in authentication responses can lead to username enumeration. By identifying a valid username through response analysis and subsequently performing a password brute-force attack, it was possible to gain access to a user account. Consistent error handling and brute-force protections are essential to mitigate this class of vulnerability.

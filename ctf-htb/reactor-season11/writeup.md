# Reactor – Security Assessment Case Study

## Executive Summary

This assessment documents the identification of multiple security weaknesses within a web application environment that ultimately resulted in complete system compromise.

The attack path consisted of:

1. Discovery of a server-side vulnerability in a web application.
2. Access to application resources and internal data.
3. Exposure of credential material stored within a SQLite database.
4. Recovery of valid user credentials.
5. Authentication as a legitimate system user.
6. Identification of a privileged Node.js service exposing a debugging interface.
7. Abuse of the exposed debugging functionality to access sensitive root-level resources.

The assessment demonstrates how individually moderate weaknesses can be chained together to achieve full compromise of a host.

---

## Scope

Target Host: Reactor

Operating System:

* Ubuntu 24.04 LTS

Primary Technologies:

* Node.js
* Next.js
* SQLite
* OpenSSH

---

## Initial Findings

### Web Application Weakness

The exposed web application contained a server-side vulnerability that allowed unauthorized interaction with backend functionality.

This issue enabled execution of actions that should not have been available to untrusted users and provided access to internal application resources.

### Security Impact

An attacker could interact with backend components and gain visibility into resources not intended for external users.

---

## Application Data Exposure

During post-access analysis, a SQLite database used by the application was identified.

The database contained operational information as well as user account records.

### Observation

User-related records contained credential material that could be subjected to offline password recovery attempts.

### Security Impact

Storing password hashes that are susceptible to recovery significantly increases the likelihood of credential compromise.

---

## Credential Recovery

One of the recovered hashes corresponded to a weak password.

The recovered credentials provided access to a valid user account on the target system.

### Security Impact

Weak credentials can render otherwise secure authentication mechanisms ineffective.

Organizations should enforce:

* Strong password policies
* Password managers
* Multi-factor authentication
* Monitoring for credential reuse

---

## User-Level Access

Using recovered credentials, access was obtained to a legitimate user account.

At this stage, privileges remained limited and no administrative rights were available.

### Enumeration Highlights

The following areas were reviewed:

* Running processes
* Listening services
* Local groups
* Scheduled tasks
* Service configurations
* Sensitive files

---

## Discovery of a Privileged Service

A Node.js monitoring service was identified running with elevated privileges.

The service was configured with debugging functionality enabled.

### Key Observation

The process exposed a local debugging endpoint associated with the Node.js Inspector framework.

The service executed with root privileges.

### Security Impact

Debugging interfaces provide extensive visibility and control over running applications.

When exposed on privileged processes, they may allow unauthorized access to sensitive application functionality.

---

## Privilege Escalation

Further analysis demonstrated that the debugging interface could be leveraged to interact with the privileged runtime.

As a result, access to root-level resources became possible.

### Impact

Successful abuse of the debugging interface resulted in:

* Access to root-owned resources
* Complete host compromise
* Loss of confidentiality
* Loss of integrity
* Loss of trust in the affected system

---

## Root Cause Analysis

The compromise resulted from a combination of weaknesses:

1. Server-side application vulnerability
2. Exposure of sensitive database contents
3. Weak user credentials
4. Privileged service execution
5. Debugging interface enabled in production

No single issue alone represented the full compromise path; rather, the issues compounded one another.

---

## Mitigations

### Web Application Security

* Conduct secure code reviews
* Perform regular penetration testing
* Validate all untrusted input
* Apply least-privilege design principles

### Credential Security

* Enforce strong password policies
* Use modern password hashing algorithms
* Require multi-factor authentication
* Monitor for password reuse

### Database Protection

* Restrict access to application databases
* Encrypt sensitive data where appropriate
* Audit database permissions regularly

### Node.js Hardening

* Do not expose Inspector interfaces in production
* Disable debugging functionality when not required
* Restrict access to administrative services
* Separate privileged services from application-facing components

### Principle of Least Privilege

* Avoid running application services as root
* Create dedicated service accounts
* Limit filesystem permissions
* Restrict access to sensitive directories

---

## Lessons Learned

This assessment demonstrates the importance of defense in depth.

Even when individual vulnerabilities appear manageable, combining:

* Application weaknesses
* Credential exposure
* Weak passwords
* Misconfigured privileged services

can lead to full system compromise.

Reducing attack surface, enforcing least privilege, and disabling unnecessary debugging functionality are critical measures for preventing similar attack chains.
<img width="300" height="300" alt="image" src="https://github.com/user-attachments/assets/edd08480-0d3f-432a-93aa-13629b577297" />

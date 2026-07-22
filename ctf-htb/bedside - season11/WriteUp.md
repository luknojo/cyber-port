# Hack The Box - Bedside

## Information

- **Machine:** Bedside
- **Platform:** Hack The Box
- **Difficulty:** Medium
- **Operating System:** Linux

---

# Summary

Bedside is a medium-difficulty Linux machine that focuses on web application security, containerized environments, internal network pivoting, and Linux privilege escalation. The machine requires combining multiple findings across different attack surfaces to achieve full system compromise.

---

# Enumeration

I started by enumerating the target to identify exposed services and gather information about the available attack surface. During this phase, I identified the web application and continued with further reconnaissance.

---

# Web Enumeration

After analyzing the main application, I performed additional web enumeration which led to the discovery of another application. I continued exploring its functionality and identified a path that provided the initial foothold.

---

# Initial Access

Following the web assessment, I obtained an initial shell inside a containerized environment. From there, I stabilized my session and began post-exploitation enumeration.

---

# Container Enumeration

Inside the container, I enumerated the filesystem, running services, mounted volumes, and available resources. This process revealed an internal service that was inaccessible from the outside.

---

# Internal Pivoting

To continue the assessment, I established access to the internal service and performed further enumeration. This eventually led to the recovery of information that allowed me to move from the container to the host system.

---

# Host Enumeration

Once on the host, I carried out standard Linux enumeration, reviewing user permissions, available services, and privileged components in search of a privilege escalation path.

---

# Privilege Escalation

After analyzing the available privileged functionality, I identified an insecure configuration that allowed privilege escalation and ultimately obtained root access.

---

# Conclusion

This machine demonstrates the importance of proper isolation between containers and hosts, secure handling of internal services, and careful validation of privileged application workflows. Successfully compromising the machine required chaining together multiple findings across different stages of the assessment.


<img width="886" height="561" alt="image" src="https://github.com/user-attachments/assets/95922bb8-f8c2-4fd0-a8f3-8684437570cf" />

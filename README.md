# Web Application Security Assessment – Bus Ticket Booking Platform

## Overview
A security assessment performed on a bus ticket booking web application to identify vulnerabilities related to insufficient server-side validation and broken access control.

---

## Vulnerability Found
**Parameter Tampering – Ticket Price Manipulation**

| Detail | Info |
|---|---|
| **Vulnerability Type** | Parameter Tampering / Broken Access Control |
| **OWASP Category** | OWASP Top 10 – A01: Broken Access Control |
| **Severity** | Critical |
| **Tools Used** | Burp Suite, Kali Linux |

---

## Description
The application trusted client-supplied data without proper server-side validation. This allowed manipulation of ticket price parameters during the HTTP request lifecycle, enabling unauthorized price changes.

---

## Steps to Reproduce

**Step 1 – Intercept the Request**
- Configured Burp Suite as a proxy
- Navigated to the ticket booking page
- Initiated a ticket booking to capture the HTTP request in Burp Suite Intercept

**Step 2 – Identify the Vulnerable Parameter**
- Analyzed the intercepted HTTP POST request
- Identified the ticket price parameter being sent from the client side
- Example:
```
POST /book-ticket HTTP/1.1
Host: target-application.com

ticket_id=1023&price=500&seat=A1
```

**Step 3 – Manipulate the Parameter**
- Modified the price parameter to an unauthorized value
- Example:
```
ticket_id=1023&price=1&seat=A1
```

**Step 4 – Forward the Request**
- Forwarded the modified request to the server
- The server accepted the manipulated price without validation
- Ticket was booked at the unauthorized price

---

## Root Cause
The application relied on client-supplied data for critical business logic (ticket pricing) without implementing server-side validation. The server did not verify whether the submitted price matched the actual price stored in the database.

---

## Impact
- Unauthorized price manipulation by any user
- Financial loss to the business
- Broken trust in the booking system

---

## Recommendation
- Never trust client-supplied data for critical parameters like price, quantity, or discount
- Validate and verify all business-critical values on the server side against the database
- Implement proper access controls to prevent parameter tampering
- Use server-generated tokens for sensitive transaction values

---

## OWASP Reference
- [OWASP Top 10 – A01: Broken Access Control](https://owasp.org/Top10/A01_2021-Broken_Access_Control/)
- [OWASP Testing Guide – Testing for Parameter Tampering](https://owasp.org/www-project-web-security-testing-guide/)

---

## Tools Used
| Tool | Purpose |
|---|---|
| **Burp Suite** | Intercepting and modifying HTTP requests |
| **Kali Linux** | Testing environment |

---

## Author
**Rishivardhan V**
- B.E – Computer Science and Engineering (Cyber Security)
- S.A Engineering College, Chennai
- 📧 rishivardhan211006@gmail.com

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
- Analyzed the intercepted HTTP POST request to `/passenger-info`
- Identified the `fare` parameter being sent from the client side
- The request body contained the ticket fare value trusted directly from the client
- Example:
```
POST /passenger-info HTTP/1.1
Host: target-application.com

schedule_id=SCH3A881411K&seat=U12W&fare=705&seatName=U12W&serviceCode=BLN4CL
```

**Step 3 – Manipulate the Parameter**
- Modified the `fare` parameter from ₹705 to ₹1 using Burp Suite Intercept
- Example:
```
schedule_id=SCH3A881411K&seat=U12W&fare=1&seatName=U12W&serviceCode=BLN4CL
```

**Step 4 – Forward the Request**
- Forwarded the modified request to the server
- The server accepted the manipulated price without validation
- Ticket was booked at the unauthorized price

---

## Proof of Concept – Screenshots

> ⚠️ All identifying information has been redacted from screenshots to protect the target application.

**Screenshot 1 – Original Ticket Listing**

Shows the original ticket price displayed to the user before any manipulation.

![Original Ticket Listing](screenshots/Screenshot%202026-06-09%20224952.png)

---

**Screenshot 2 – Seat Selection Page**

Shows the seat selection page with the legitimate ticket price visible before booking.

![Seat Selection](screenshots/Screenshot%202026-06-09%20225020.png)

---

**Screenshot 3 – Burp Suite Intercept (Request Headers)**

Burp Suite intercepting the POST request to `/passenger-info` endpoint. Shows the HTTP headers of the captured request.

![Burp Suite Intercept Headers](screenshots/Screenshot%202026-06-09%20225143.png)

---

**Screenshot 4 – Burp Suite Intercept (Request Body)**

The full URL-encoded POST body intercepted by Burp Suite. The fare parameter containing the original price value (₹705) is visible and highlighted.

![Burp Suite Intercept Body](screenshots/Screenshot%202026-06-09%20225152.png)

---

**Screenshot 5 – Modified Request**

The fare parameter has been modified to an unauthorized value. The modified request is forwarded to the server.

![Modified Request](screenshots/Screenshot%202026-06-09%20225543.png)

---

**Screenshot 6 – Server Response (Price Manipulation Confirmed)**

The server accepted the manipulated request. Fare Details now show:
- Bus Fare: ₹35
- Discount: ₹35
- **Grand Total: ₹0.00**

This confirms the server did not validate the fare on the backend.

![Price Manipulation Result](screenshots/Screenshot%202026-06-09%20225621.png)

---

**Screenshot 7 – Final Confirmation (Grand Total = ₹0.00)**

Final booking page confirming the Grand Total is ₹0.00 — demonstrating successful parameter tampering.

![Final Confirmation](screenshots/Screenshot%202026-06-09%20225703.png)

---

## Root Cause
The application relied on client-supplied data for critical business logic (ticket pricing) without implementing server-side validation. The server did not verify whether the submitted `fare` value matched the authoritative price stored in the database.

The core architectural flaw is that the booking flow trusted the client to submit the correct price, rather than having the server calculate and enforce the price independently. In a secure design, the client should only transmit identifiers (such as `schedule_id` and `seat`), while the server looks up and enforces the authoritative fare from its own database before processing payment.

---

## Impact
- **Business Logic Bypass** — Core pricing logic completely bypassed by any user
- **Financial Fraud / Revenue Leakage** — Tickets can be booked at ₹0.00, causing direct financial loss
- Unauthorized price manipulation by any user with basic tools
- Broken trust in the booking and payment system
- Potential for large-scale automated exploitation at zero cost

---

## Recommendation
- Never trust client-supplied data for critical parameters like fare, quantity, or discount
- Validate and verify all business-critical values on the server side against the database
- Implement proper access controls to prevent parameter tampering
- Use server-generated tokens for sensitive transaction values

**Secure Architecture Pattern:**
The client request should only transmit the identifier (`schedule_id`) and the desired seat. The server must look up the authoritative price from the database using that identifier, calculate the total internally, and pass the server-calculated value directly to the payment gateway — never accepting a fare value from the client.

---

## Remediation Code Example

```python
# ❌ VULNERABLE IMPLEMENTATION
# Trusts the fare value submitted directly from the client
def checkout(request):
    user_provided_fare = request.POST.get('fare')
    return process_payment(user_provided_fare)


# ✅ SECURE IMPLEMENTATION (With error handling)
# Ignores client-supplied fare — fetches authoritative price from database
def checkout(request):
    schedule_id = request.POST.get('schedule_id')
    seat = request.POST.get('seat')

    # Server looks up the real price — client cannot influence this value
    authorized_fare = Database.get_fare_by_schedule(schedule_id, seat)

    # Validate the result before processing — prevents crashes and logic errors
    if not authorized_fare:
        return error_response("Invalid booking details", status=400)

    return process_payment(authorized_fare)
```

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

## Skills Demonstrated
- Web Application Security Testing
- Vulnerability Assessment
- Parameter Tampering Testing
- HTTP Request Analysis
- Burp Suite
- OWASP Top 10
- Kali Linux
- HTTP/HTTPS Protocol Understanding
- Security Documentation & Reporting

---

## Author
**Rishivardhan V**
- B.E – Computer Science and Engineering (Cyber Security)
- S.A Engineering College, Chennai
- 📧 rishivardhan211006@gmail.com
- 🔗 LinkedIn: linkedin.com/in/your-profile
- 🐙 GitHub: github.com/your-username

---

## Disclaimer
This project is intended for educational and learning purposes only.
All identifying information about the tested application has been removed.
No sensitive data, credentials, or proprietary information are disclosed.
This assessment was conducted in a controlled environment for learning purposes.
The techniques demonstrated should only be used on systems you have explicit
permission to test.

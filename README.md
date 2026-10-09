# OTP Bypass - CTF Write-up

## Overview

**Challenge:** OTP Bypass
**Platform:** CyLab Academy
**Category:** Web Security / Authentication
**Vulnerability:** Improper OTP Validation
**Operating System:** Windows
**Browser:** Brave
**Tools Used:** Burp Suite

**Objective:** Bypass the One-Time Password (OTP) verification process and retrieve the challenge flag.

---

## 1. Initial Reconnaissance

The challenge presented a web application with a user registration process.

After registering an account and submitting the registration form, the application redirected me to a page requesting an OTP (One-Time Password).

The OTP was intended to verify the user before granting access to the next stage of the application.

Since the challenge focused on OTP verification, I decided to investigate how the server processed OTP submission requests and how it responded to invalid input.

### Screenshot

<img width="532" height="184" alt="image" src="https://github.com/user-attachments/assets/6b2982dd-5e07-4133-87e5-73be0e964979" />

```markdown
![OTP verification page](images/otp-page.png)
```

---

## 2. Intercepting the OTP Request

I opened Burp Suite and configured the browser traffic to pass through its proxy.

I then entered a random OTP into the verification form and submitted it while Burp Suite intercepted the request.

The server responded that the OTP was invalid, which was the expected behavior for an incorrect code.

However, the main objective was not to guess the OTP. Instead, I wanted to understand how the application handled the verification request.

### Screenshot
<img width="1034" height="442" alt="image" src="https://github.com/user-attachments/assets/815382dc-8a09-456b-bab2-69737ec89044" />

```markdown
![Intercepted OTP request](images/otp-request.png)
```

---

## 3. Analyzing the HTTP Request

While inspecting the intercepted request in Burp Suite, I found a parameter containing the submitted OTP:

```http
OTP=1234
```

This parameter was included in the request sent to the server for verification.

The server processed the request and returned an invalid OTP response when the parameter was present with an incorrect value.

This observation suggested that modifying the request structure could reveal a weakness in the server's OTP validation logic.

---

## 4. Exploiting the OTP Validation Weakness

I modified the intercepted request by removing the OTP parameter:

```http
OTP=1234
```

I then forwarded the modified request to the server.

Instead of rejecting the request because the OTP was missing, the application marked the verification process as successful.

This was the critical finding: **the server treated a request without the OTP parameter as successfully verified rather than rejecting it**.

The application then granted access to the protected stage of the challenge.

### Screenshot

<img width="986" height="405" alt="image" src="https://github.com/user-attachments/assets/5e934fde-cd9b-4138-a8fa-74b202a252d7" />

```markdown
![Modified OTP request](images/otp-bypass.png)
```

---

## 5. Retrieving the Flag

After the server accepted the modified request, the application completed the verification process and revealed the challenge flag.

This confirmed that OTP verification could be bypassed without knowing the correct one-time password.

---

## 6. Attack Chain

The exploitation process can be summarized as follows:

```text
Register Account
       |
       v
Reach OTP Verification Page
       |
       v
Intercept Request Using Burp Suite
       |
       v
Submit Random OTP
       |
       v
Observe Invalid OTP Response
       |
       v
Inspect OTP Submission Parameters
       |
       v
Remove OTP Parameter
       |
       v
Forward Modified Request
       |
       v
Server Marks Verification Successful
       |
       v
Capture Flag
```

---

## 7. Vulnerability Analysis

The primary vulnerability was **improper server-side OTP validation**, resulting in an authentication bypass.

The application failed to reject an OTP verification request when the OTP parameter was missing. Instead, it treated the request as successfully verified.

This indicates a failure in the server-side verification logic.

### Why is this dangerous?

OTP verification is an additional authentication control intended to ensure that a user has supplied a valid, time-sensitive or one-time code.

If the server allows verification to succeed when the OTP is absent, an attacker may bypass that control by modifying the HTTP request.

Potential consequences include:

* Bypassing an additional authentication step
* Unauthorized access to protected application functionality
* Circumvention of account verification
* Weakening of multi-factor authentication, where OTP verification is part of the MFA process

The exact impact depends on what the OTP protects within the application.

---

## 8. Key Security Lessons

### 1. OTP validation must happen on the server

The server must verify that an OTP exists, is valid, belongs to the correct authentication transaction, and has not expired or already been used.

### 2. Missing parameters must fail securely

An absent OTP must never be interpreted as successful verification. The server should reject missing, malformed, expired, and incorrect OTP values.

### 3. HTTP requests can be modified

Burp Suite demonstrated that client-generated requests can be intercepted and changed before they reach the server. Security controls must remain effective even when clients submit unexpected requests.

### 4. Successful responses do not always mean secure validation

The application returned a successful result when the OTP parameter was removed. This demonstrated why it is important to test how an application handles missing and unexpected inputs, not just incorrect values.

### 5. Authentication workflows require negative testing

Developers and security testers should verify that authentication controls reject invalid input, missing parameters, expired codes, reused codes, and requests made out of sequence.

---

## 9. Methodology Summary

| Stage             | Action                         | Result                              |
| ----------------- | ------------------------------ | ----------------------------------- |
| Registration      | Created an account             | Reached OTP verification            |
| Interception      | Captured request in Burp Suite | Obtained the OTP submission request |
| Initial Testing   | Submitted a random OTP         | Server returned an invalid response |
| Request Analysis  | Inspected the OTP parameter    | Identified `OTP=1234`               |
| Exploitation      | Removed the OTP parameter      | Server accepted verification        |
| Post-Exploitation | Accessed the protected page    | Flag captured                       |

---

## 10. Tools and Environment

### Operating System

* Windows

### Browser

* Brave Browser

### Tool

* **Burp Suite:** Used to intercept, inspect, and modify the OTP verification request.

No automated brute-force tool was required. The vulnerability was identified by manually analyzing the HTTP request and observing how the server responded to a missing OTP parameter.

---

## Conclusion

The **OTP Bypass** challenge demonstrated how a flaw in server-side authentication logic can allow an attacker to circumvent OTP verification.

After registering an account, I intercepted the OTP submission request using Burp Suite and submitted an incorrect code to observe the server's response. Analysis of the request revealed the `OTP=1234` parameter.

I removed that parameter and forwarded the modified request. Rather than rejecting the request, the application marked the verification process as successful and revealed the flag.

### The main lesson is that **authentication systems must fail securely when required verification data is missing**. OTP verification must be explicitly enforced on the server, regardless of how the client structures or modifies its requests.

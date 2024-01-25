# CTD Security Guidelines

## SWAT Checklist

Online you can find the [SWAT Checklist](https://www.sans.org/cloud-security/securing-web-application-technologies/) which is a checklist of best practices for application security. 

<details><summary><code>SWAT Checklist</code></summary>
<br>

<pre>
Error Handling and Logging
  1. Display Generic Error Messages
  2. No Unhandled Exceptions
  3. Suppress Framework Generated Errors
  4. Log All Authentication Activities
  5. Log All Privilege Changes
  6. Log Administrative Activities
  7. Log Access to Sensitive Data
  8. Do Not Log Inappropriate Data
  9. Store Logs Securely

Data Protection
  10. Use HTTPS Everywhere
  11. Disable HTTP Access for All Protected Resources
  12. Use the Strict-Transport-Security Header
  13. Store User Passwords Using a Strong, Iterative, Salted Hash
  14. Securely Exchange Encryption Keys
  15. Set Up Secure Key Management Processes
  16. Weak TLS Configuration on Servers
  17. Use Valid HTTPS Certificates from a Reputable CA
  18. Disable Data Caching Using Cache Control Headers and Autocomplete
  19. Limit the Use and Storage of Sensitive Data

Configuration and Operations
  20. Automate Application Deployment
  21. Establish a Rigorous Change Management Process
  22. Define Security Requirements
  23. Conduct a Design Review
  24. Perform Code Reviews
  25. Perform Security Testing
  26. Harden the Infrastructure
  27. Define an Incident Handling Plan
  28. Educate the Team on Security

Authentication
  29. Don't Hardcode Credentials
  30. Develop a Strong Password Reset System
  31. Implement a Strong Password Policy and Move Towards Passwordless Authentication
  32. Implement Account Lockout against Brute Force Attacks
  33. Don't Disclose Too Much Information in Error Messages
  34. Store Database Credentials and API keys Securely
  35. Applications and Middleware Should Run with Minimal Privileges

Session Management
  36. Ensure That Session Identifiers Are Sufficiently Random
  37. Regenerate Session Tokens
  38. Implement an Idle Session Timeout
  39. Implement an Absolute Session Timeout
  40. Destroy Sessions at Any Sign of Tampering
  41. Invalidate the Session after Logout
  42. Place a Logout Button on Every Page
  43. Use Secure Cookie Attributes (HttpOnly, Secure and SameSite Flags)
  44. Set the Cookie Domain and Path Correctly
  45. Set the Cookie Expiration Time

Input and Output Handling
  46. Conduct Contextual Output Encoding
  47. Prefer Allowlists over Blocklists
  48. Use Parameterized SQL Queries
  49. Use Tokens to Prevent Forged Requests
  50. Set the Encoding for Your Application
  51. Validate Uploaded Files
  52. Use the Nosniff Header for Uploaded Content
  53. Validate the Source of Input
  54. Use the X-Frame-Options Header
  55. Use Secure HTTP Response Headers
  56. Deserialize Untrusted Data with Proper Controls

Access Control
  57. Apply Access Controls Checks Consistently
  58. Apply the Principle of Least Privilege
  59. Don't Use Direct Object References for Access Control Checks
  60. Don't Use Unvalidated Forwards or Redirects
</pre></details>
<br>

## OWASP Top 10

There is also an [OWASP Top 10](https://owasp.org/www-project-top-ten/) list. 

<details><summary><code>OWASP Top 10, 2021</code></summary>

<br>
<pre>
1. Broken Access Control
2. Cryptographic Failures
3. Injection
4. Insecure Design
5. Security Misconfiguration
6. Vulnerable and Outdated Components
7. Identification and Authentication Failures
8. Software and Data Integrity Failures
9. Security Logging and Monitoring Failures
10. Server-Side Request Forgery
</pre></details>
<br>

## CTD 30-Point Security Standard

Both lists reference [CWE](https://cwe.mitre.org/index.html).
If you use the CWE IDs to find the intersection of these security weaknesses,
the result is the CTD 30-Point Security Standard. 

<details open><summary><code>CTD 30-Point Security Standard</code></summary>
<br>

<pre>
Error Handling and Logging
  1. Display Generic Error Messages
  2. Suppress Framework Generated Errors
  3. Log All Authentication Activities
  4. Log All Privilege Changes
  5. Log Administrative Activities
  6. Log Access to Sensitive Data
  7. Do Not Log Inappropriate Data

Data Protection
  8. Use HTTPS Everywhere
  9. Disable HTTP Access for All Protected Resources
  10. Store User Passwords Using a Strong, Iterative, Salted Hash

Authentication
  11. Don't Hardcode Credentials
  12. Develop a Strong Password Reset System
  13. Implement a Strong Password Policy and Move Towards Passwordless Authentication
  14. Implement Account Lockout against Brute Force Attacks
  15. Store Database Credentials and API keys Securely

Session Management
  16. Regenerate Session Tokens
  17. Implement an Idle Session Timeout
  18. Implement an Absolute Session Timeout
  19. Invalidate the Session after Logout
  20. Use Secure Cookie Attributes (HttpOnly, Secure and SameSite Flags)

Input and Output Handling
  21. Conduct Contextual Output Encoding
  22. Use Parameterized SQL Queries
  23. Use Tokens to Prevent Forged Requests
  24. Validate Uploaded Files
  25. Use the Nosniff Header for Uploaded Content
  26. Validate the Source of Input
  27. Use Secure HTTP Response Headers
  28. Deserialize Untrusted Data with Proper Controls

Access Control
  29. Apply Access Controls Checks Consistently
  30. Don't Use Direct Object References for Access Control Checks
  31. Don't Use Unvalidated Forwards or Redirects
</pre></details>
<br>

## CTD 30-Point Security Standard ordered by OWASP Top 10

Below is the CTD 30-Point Security Standard checklist ordered by the [OWASP Top 10](https://owasp.org/www-project-top-ten/).

<details><summary><code>CTD 30-Point Security Standard ordered by OWASP Top 10</code></summary>
<br>

<pre>
Broken Access Control
  1. Use Tokens to Prevent Forged Requests
  2. Validate Uploaded Files (also Insecure Design)
  3. Apply Access Controls Checks Consistently
  4. Don't Use Direct Object References for Access Control Checks
  5. Don't Use Unvalidated Forwards or Redirects

Cryptographic Failures
  6. Use HTTPS Everywhere (also Insecure Design)
  7. Disable HTTP Access for All Protected Resources

Injection
  8. Use Secure Cookie Attributes (HttpOnly, Secure and SameSite Flags) (also Security Misconfiguration)
  9. Conduct Contextual Output Encoding
  10. Use Parameterized SQL Queries
  11. Validate the Source of Input (also Identification and Authentication Failures)
  12. Use Secure HTTP Response Headers

Insecure Design
  13. Display Generic Error Messages
  14. Suppress Framework Generated Errors
  15. Store User Passwords Using a Strong, Iterative, Salted Hash
  16. Store Database Credentials and API keys Securely
  17. Use the Nosniff Header for Uploaded Content

Identification and Authentication Failures
  18. Don't Hardcode Credentials
  19. Develop a Strong Password Reset System
  20. Implement a Strong Password Policy and Move Towards Passwordless Authentication
  21. Implement Account Lockout against Brute Force Attacks
  22. Regenerate Session Tokens
  23. Implement an Idle Session Timeout
  24. Implement an Absolute Session Timeout
  25. Invalidate the Session after Logout

Software and Data Integrity Failures
  26. Deserialize Untrusted Data with Proper Controls

Security Logging and Monitoring Failures
  27. Log All Authentication Activities
  28. Log All Privilege Changes
  29. Log Administrative Activities
  30. Log Access to Sensitive Data
  31. Do Not Log Inappropriate Data
</pre></details>
<br>

*Note: An alternate order for these guidelines—using the SWAT grouping—is Authentication, Session Management, Input/Output Handling and Logging. This is just a logical order, not from any list. Use it if you like.*

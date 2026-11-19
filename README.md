# LAB 4: Active Activity – Web Application Penetration Testing

**Name:** Ahmad Danial Syahir

---

## 1. Target Setup (Victim)

The target used for this penetration testing activity was Damn Vulnerable Web Application (DVWA), an intentionally vulnerable web application hosted locally on a Kali Linux virtual machine via Apache2, PHP 8.4, and MariaDB.

**Target Configuration:**

| Component | Configuration |
| :--- | :--- |
| Application | DVWA (`/var/www/html/dvwa`) |
| Hosting Stack | Apache2, PHP 8.4, MariaDB |
| Target URL | `http://127.0.0.1/dvwa/` |
| Database Config | `127.0.0.1` (`dvwa` DB, `dvwa` user) |
| Web Port | `80` (`127.0.0.1`) |
| Security Level | `Low` |

DVWA was initialized through `setup.php` and served locally over port `80` at `http://127.0.0.1/dvwa/`.

The DVWA security level was configured to **Low** to demonstrate common web application vulnerabilities in a controlled environment.

---

## 2. Own Hosts (Attacker)

- **Host OS & Kernel:** `Linux kali 6.16.8+kali-amd64 #1 SMP PREEMPT_DYNAMIC Kali 6.16.8-1kali1 (2025-09-24) x86_64 GNU/Linux` (VirtualBox VM)
- **Network Interfaces:** `lo` (`127.0.0.1/8`, `::1/128`); `eth0` `UP` (`10.0.2.15/24`, `fd17:625c:f037:2:561d:3cd7:7b6c:9fc9/64`, `fe80::e125:449a:aa90:8da5/64`)
- **Tools Used:** Firefox Browser, Kali Terminal

---

## 3. Vulnerabilities & Evidence

### 3.1 SQL Injection (`GET /vulnerabilities/sqli/`)
- **Payload:** `1' OR '1'='1`
- **Finding:** Unsanitized input in the `id` parameter allowed boolean logic injection, returning multiple user records (`admin`, `Gordon Brown`, `Hack Me`, `Pablo Picasso`, `Bob Smith`) instead of a single record.
- **Evidence:** ![DVWA SQL Injection](screenshots/Screenshot%202026-11-19%20210713.png)
- **Remediation:** Use prepared statements / parameterized queries and enforce least-privilege database accounts.

### 3.2 OS Command Injection (`POST /vulnerabilities/exec/`)
- **Payload:** `127.0.0.1 && id`
- **Finding:** Unsanitized input passed to the system ping utility allowed command chaining (`&&`), executing arbitrary OS commands under `uid=33(www-data)`.
- **Evidence:** ![DVWA Command Injection](screenshots/Screenshot%202026-11-19%20210949.png)
- **Remediation:** Avoid shell invocation with untrusted input; validate strict input formats and use safe native APIs.

### 3.3 Reflected Cross-Site Scripting (`GET /vulnerabilities/xss_r/`)
- **Payload:** `<script>alert(document.cookie)</script>`
- **Finding:** The `name` parameter was reflected directly into the HTML response without output encoding, executing arbitrary JavaScript in the browser and exposing session cookies (`security=low; PHPSESSID=8bc8aed41a929f389a10cfa4f80df0

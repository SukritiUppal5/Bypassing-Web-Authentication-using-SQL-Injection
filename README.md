# Bypassing-Web-Authentication-using-SQL-Injection

## Executive Summary
This project is a practical demonstration of **SQL Injection (SQLi)** vulnerabilities and their impact on web authentication. Using a local lab environment (**XAMPP** & **DVWA**), I successfully bypassed security controls to perform a full database user dump and gain unauthorized administrative access. 

The project highlights the critical importance of input sanitization and the dangers of using unsanitized data in database queries.

## Laboratory Setup
* **Host Environment:** Windows 10/11
* **Web Server Stack:** XAMPP (Apache & MariaDB)
* **Target Application:** Damn Vulnerable Web Application (DVWA)
* **Configuration:** * Security Level: Manually forced to `low`.
    * Source Code: Modified `login.php` to remove native PHP sanitization for educational demonstration.

## Vulnerability Discovery Phase

### 1. Database Interaction Confirmation
By injecting a single quote (`'`) into the input fields, I successfully triggered a MariaDB syntax error. This "error-based" discovery confirmed that the application was directly passing raw input into the SQL engine.

### 2. Full Data Exfiltration (Tautology Attack)
Using a logic-based payload, I bypassed the specific record lookup to list all users stored in the database.
* **Payload:** `' OR '1'='1`
* **Mechanism:** Since `1=1` is a mathematical truth, the `WHERE` clause evaluates to `TRUE` for every row, forcing the database to return the entire `users` table.

## The Authentication Bypass Exploit

The primary objective was to bypass the `login.php` authentication gate without a valid password.

### Technical Logic
The backend query was structured as follows:
`SELECT * FROM users WHERE user = '$username' AND password = '$password';`

### The "Comment" Attack
I utilized the following payload in the Username field:
**Payload:** `admin' #`

**Resulting SQL Query:**
`SELECT * FROM users WHERE user = 'admin' #' AND password = '...';`

**Breakdown:**
1.  `admin'` : Satisfies the username requirement and closes the SQL string.
2.  `#` : The MySQL/MariaDB comment character. It instructs the server to ignore the rest of the query (the password verification logic).
3.  **Result:** The application verified that the user `admin` exists and ignored the password check entirely, granting full administrative access.

---

## 🛠️ Source Code Analysis

### **Vulnerable Implementation (Demonstration Code)**
The following code snippet shows the lack of sanitization that makes the attack possible:
```php
$user = $_POST[ 'username' ]; // Direct POST data assignment
$pass = $_POST[ 'password' ]; // No hashing or escaping applied
$query = "SELECT * FROM `users` WHERE user='$user' AND password='$pass';";

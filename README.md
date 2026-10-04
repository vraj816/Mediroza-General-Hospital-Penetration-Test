# Project Overview
## Mediroza-General-Hospital-Penetration-Test
Black-box pentest documentation for Mediroza General Hospital (NetworkWalks ). Features SQLi authentication bypass, password recovery, PDF decryption, and SQL database exposure analysis.

# Milestone M1 — Initial Access

## Task 1 – Reconnaissance on the Patient Portal

### Command

```bash
curl https://medirozahospital.com/robots.txt
```
Purpose & Analysis Checked whether directory listing was enabled on the /patient/ path to map out the application's structure before testing. This revealed login.php, portal.php, download.php, and logout.php as exposed endpoints.

<img width="1918" height="918" alt="Screenshot_2026-10-03_07_38_50" src="https://github.com/user-attachments/assets/86a182dd-bb99-44d3-a6a1-e3680a1fe88f" />

## Task 2 – SQL Injection Authentication Bypass


<img width="1918" height="918" alt="Screenshot_2026-10-03_07_54_19" src="https://github.com/user-attachments/assets/d396c03d-5d14-45c1-a327-d17a7f7e7b17" />


<img width="1918" height="918" alt="Screenshot_2026-10-03_08_01_24" src="https://github.com/user-attachments/assets/56267ee4-f563-4939-9d1b-3fb7ee47490b" />


### Purpose & Analysis

Submitted admin'-- as the username to comment out the password check in the backend SQL query. The response confirmed an authenticated session was granted without any valid credentials.



## Task 3 – Download the Patient Reports


<img width="1918" height="918" alt="Screenshot_2026-10-03_08_01_32" src="https://github.com/user-attachments/assets/24c8a374-1983-4078-bce6-689004f10da8" />


# Milestone M2 — Offline Password Cracking

## Task 1 – Extract Crackable Hashes

### Purpose & Analysis

Extracted the `$pdf$...` formatted hash from each encrypted PDF for use in offline password-cracking analysis.

The extracted hashes were cross-checked against the Networkwalks Hash Calculator to verify that they were correctly formatted and suitable as input for offline cracking.

### Findings

- Identified the PDF encryption hash from each encrypted PDF.
- Confirmed the extracted hashes followed the expected `$pdf$...` format.
- Cross-checked the hashes using the Networkwalks Hash Calculator.
- Prepared the verified hashes for subsequent offline password-cracking activities.

### Result

**Status:** Successful

The encrypted PDF hashes were successfully extracted and validated as crackable hash inputs.


<img width="1918" height="918" alt="image" src="https://github.com/user-attachments/assets/3ae60957-d074-4ea8-ad28-25007ffdaa50" />


## Task 2 – Crack Each Password (Escalating Wordlist Strategy)


### Purpose & Analysis

An escalating wordlist strategy was used rather than assuming that a single wordlist would successfully crack all three PDF passwords.

The password-cracking process was performed progressively using:

Built-in 100-word list

JTR_default_password.txt

Result
Status: Successful

All three PDF password hashes were subjected to the escalating wordlist strategy, with the appropriate wordlist used to recover each password.


<img width="1919" height="918" alt="Screenshot 2026-10-03 180359" src="https://github.com/user-attachments/assets/19463235-0baf-4040-9b22-fbfce3f0c3bc" />


<img width="1919" height="918" alt="Screenshot 2026-10-04 151909" src="https://github.com/user-attachments/assets/bc9773f8-0b6f-4274-a965-b23139c926cf" />


<img width="1919" height="918" alt="Screenshot 2026-10-03 173857" src="https://github.com/user-attachments/assets/081ab20c-67e7-4bae-9ded-f5e3a875d06f" />


## Task 3 – Decrypt and View the Reports

### Commands

```bash
qpdf --password='!@#$%^&' --decrypt thompson.pdf thompson_decrypted.pdf
```


<img width="1919" height="918" alt="image" src="https://github.com/user-attachments/assets/0d0b825b-7a92-49b4-93dc-4b5dda531168" />


# Milestone M3 — Critical Data Exposure

## Task 1 – Metadata Discovery

### Command

```bash
exiftool report3_open.pdf | grep -i comment
```


<img width="1918" height="918" alt="Screenshot_2026-10-03_15_35_03" src="https://github.com/user-attachments/assets/76e543b4-eb39-41d8-aaef-5cb027e8a48b" />


## Task 2 – Locate the Exposed Backup

### Commands

```bash
curl -s https://medirozahospital.com/old/

curl -s -O https://medirozahospital.com/old/mediroza_db_backup_2019.sql
```


<img width="1918" height="918" alt="Screenshot_2026-10-03_15_37_53" src="https://github.com/user-attachments/assets/30b10e41-5c2f-4c9b-b632-c086a7654b14" />


## Task 3 – Restore and Query the Backup

### Commands

```bash
grep "INSERT INTO \`staff\`" mediroza_db_backup_2019.sql

grep "INSERT INTO \`shareholders\`" mediroza_db_backup_2019.sql

```


<img width="1918" height="918" alt="Screenshot_2026-10-04_00_43_19" src="https://github.com/user-attachments/assets/a9cb73bd-3225-4b21-9fe7-e721d17a80fe" />


<img width="1918" height="918" alt="Screenshot_2026-10-04_00_45_33" src="https://github.com/user-attachments/assets/e86fe61a-e407-492a-9fd3-f9a613251baa" />


### Purpose & Analysis

Restored the exposed database backup into an isolated local MySQL database to safely assess the scope of the information contained within the file.

The database was queried to identify its available tables and examine the staff and shareholders tables.

### Findings

The SQL backup was successfully restored into the mediroza_restore database.

The staff table contained 30 complete employee records.

Exposed employee information included:

Full name

Job title

Department

National ID

Monthly salary


# Mediroza General Hospital Penetration Test - Report

> **Security Assessment Report**  
> **Assessment Type:** Black-Box Penetration Test  
> **Target:** Mediroza General Hospital  
> **Program:** NetworkWalks Cybersecurity Internship  
> **Batch:** B083  
> **Author:** Vraj Patel



# 1. Executive Summary

This report documents a black-box penetration test conducted against the **Mediroza General Hospital** application as part of the **NetworkWalks Cybersecurity Internship**.

The assessment identified multiple vulnerabilities affecting authentication, patient-document protection, information disclosure, and backup management.

The most severe issue identified was a **SQL injection vulnerability in the patient portal login functionality**. The vulnerability allowed authentication to be bypassed without valid credentials. Before exploitation, the login page also allowed **username enumeration**, providing useful information that could assist an attacker during credential and authentication attacks.

After bypassing authentication, encrypted patient PDF reports were accessible through the patient-report functionality. The PDF passwords were subsequently subjected to controlled offline password-cracking analysis. The passwords were successfully recovered using wordlist-based techniques, demonstrating insufficient password strength.

Further analysis identified sensitive metadata within patient PDF files. The assessment also discovered a forgotten `old/` directory with directory listing enabled. This directory exposed a database backup that was accessible through the web server.

The exposed database backup contained confidential information, including staff salaries and shareholder data. The `staff` table contained **30 complete employee records**, including full names, job titles, departments, national identification information, and monthly salaries.

Overall, the assessment identified **two Critical, four High/Medium information-security weaknesses, and one additional sensitive-data exposure finding**, with the combined security posture assessed as **Critical**.

The highest-priority remediation actions are:

1. Eliminate SQL injection by implementing parameterized queries.
2. Remove database backups from publicly accessible directories.
3. Disable directory listing.
4. Enforce strong, unique passwords for encrypted patient documents.
5. Place patient PDFs outside the public web root or enforce strict access controls.
6. Prevent username enumeration through consistent authentication error messages.
7. Strip unnecessary metadata from patient documents.
8. Establish secure backup-storage and retention procedures.

---

# 2. Project Overview

## Mediroza-General-Hospital-Penetration-Test

The **Mediroza-General-Hospital-Penetration-Test** project documents a black-box penetration test performed against a simulated hospital environment provided by NetworkWalks.

The assessment focused on the following attack surface:

- Patient login functionality
- Authentication controls
- Patient report access
- Encrypted PDF documents
- PDF password strength
- PDF metadata
- Forgotten directories
- Directory listing
- Database backup exposure
- Confidential staff information
- Shareholder information

The testing demonstrated how multiple weaknesses can be chained together to increase the overall impact of a compromise.

---

# 3. Assessment Scope

The assessment was limited to the authorized Mediroza General Hospital environment provided as part of the NetworkWalks cybersecurity internship.

## In-Scope Components

| Component | Purpose |
|---|---|
| `patient/login.php` | Patient authentication |
| `patient/reports/` | Patient report storage/access |
| `patient_report_*.pdf` | Encrypted patient reports |
| `patient_report_3.pdf` | PDF metadata analysis |
| `old/` | Forgotten/legacy web directory |
| `old/mediroza_db_backup_2019.sql` | Exposed database backup |

Testing was performed within the agreed assessment scope.

---

# 4. Methodology

The assessment followed a progressive black-box penetration-testing methodology.

## Phase 1 — Reconnaissance

The publicly accessible application structure was reviewed to identify login functionality, patient-report functionality, and potentially exposed directories.

## Phase 2 — Authentication Testing

The login mechanism was tested for:

- Username enumeration
- Authentication weaknesses
- SQL injection
- Authentication bypass

## Phase 3 — Patient Report Access

Following authentication-bypass validation, the patient-report functionality was assessed to determine whether protected PDF documents could be accessed.

## Phase 4 — PDF Security Testing

Encrypted PDF files were analyzed through:

- Hash extraction
- Hash validation
- Wordlist-based password cracking
- PDF decryption
- Metadata inspection

## Phase 5 — Web Directory and Backup Assessment

The web application was examined for forgotten directories, directory listing, and accidentally exposed backup files.

## Phase 6 — Database Exposure Analysis

The exposed SQL backup was retrieved and restored into an isolated local MySQL/MariaDB environment.

The restored database was analyzed to determine the scope and sensitivity of the exposed information.

---

# 5. Findings Summary

| # | Vulnerability | Location | Risk |
|---|---|---|---|
| 1 | Username enumeration on login page | `patient/login.php` | **Medium** |
| 2 | SQL injection login bypass | `patient/login.php` | **Critical** |
| 3 | Encrypted PDFs accessible after login bypass | `patient/reports/` | **High** |
| 4 | Weak PDF passwords crackable with a wordlist | `patient_report_*.pdf` | **High** |
| 5 | Sensitive metadata left in patient PDF files | `patient_report_3.pdf` | **Medium** |
| 6 | Forgotten backup folder with directory listing enabled | `old/` | **Critical** |
| 7 | Confidential staff salaries and shareholder data in plain text | `old/mediroza_db_backup_2019.sql` | **Critical** |

## Risk Distribution

| Risk Level | Number of Findings |
|---|---:|
| 🔴 Critical | 3 |
| 🟠 High | 2 |
| 🟡 Medium | 2 |
| 🟢 Low | 0 |

> **Overall Risk Rating: 🔴 CRITICAL**

---

# 6. Detailed Findings

## 6.1 Username Enumeration on Login Page

| Attribute | Details |
|---|---|
| **Finding ID** | F-01 |
| **Severity** | 🟡 Medium |
| **Location** | `patient/login.php` |
| **Category** | Authentication / Information Disclosure |
| **Status** | Confirmed |

## Description

The patient login page was found to provide information that could allow an attacker to determine whether a supplied username exists.

Username enumeration occurs when an application responds differently depending on whether the supplied username is valid.

This can help an attacker build a list of valid accounts before attempting password attacks or exploiting other authentication weaknesses.

## Security Impact

Username enumeration reduces the uncertainty an attacker faces when targeting the authentication system.

When combined with the SQL injection vulnerability identified in the same login functionality, the issue contributes to the overall weakness of the authentication mechanism.

## Risk

**Medium**

The vulnerability does not independently provide complete authentication bypass, but it exposes useful information about valid accounts and can support further attacks.

## Remediation

The application should return the same generic error message for both invalid usernames and invalid passwords.

For example:

```text
Invalid username or password.
```

# 👤 Author

Name: Vraj Patel

LinkedIn : https://www.linkedin.com/in/vraj-patel-vp8816/

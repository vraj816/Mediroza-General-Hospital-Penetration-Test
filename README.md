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

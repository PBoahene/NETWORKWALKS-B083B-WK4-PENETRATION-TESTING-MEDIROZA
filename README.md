<div align="center">

# 🏥 Penetration Testing Capstone — Mediroza General Hospital (Simulated)

**A full black-box penetration test: recon → authentication bypass → data extraction → professional reporting**
</div>

<p align="center">
  <img src="https://img.shields.io/badge/Skill-Cybersecurity-404040?style=flat-square&labelColor=C00000" />
  <img src="https://img.shields.io/badge/Penetration%20Testing-C00000?style=flat-square&labelColor=000000" />
  <img src="https://img.shields.io/badge/SQL%20Injection-404040?style=flat-square&labelColor=000000" />
  <img src="https://img.shields.io/badge/Password%20Cracking-238F89?style=flat-square&labelColor=000000" />
  <img src="https://img.shields.io/badge/Metadata%20Analysis-404040?style=flat-square&labelColor=C00000" />
  <img src="https://img.shields.io/badge/GitHub-404040?style=flat-square&labelColor=0070C0&logo=github&logoColor=white" />
  <img src="https://img.shields.io/badge/NetworkWalks-404040?style=flat-square&labelColor=C00000" />
  <img src="https://img.shields.io/badge/Ethical%20Hacking-E87500?style=flat-square&labelColor=000000&logo=kalilinux&logoColor=white" />
</p>

---

## 📌 Project Overview

This repository documents **Week 4** of the Networkwalks Cybersecurity Internship — a full **black-box penetration test** against a simulated target, **Mediroza General Hospital** (`medirozahospital.com`), a deliberately vulnerable web application built by Networkwalks for training purposes.

Unlike previous weeks' single-tool exercises, this project chains together recon, web application exploitation, password cracking, metadata analysis, and professional report writing — mirroring a real-world client engagement from first contact to final deliverable.

---

## 🛡️ Authorization & Scope

This assessment was performed against `medirozahospital.com`, a target explicitly built and authorized by Networkwalks for this training exercise, as part of a simulated 5-day black-box engagement.

**In scope:** `medirozahospital.com` and its subdomains/pages only.
**Out of scope:** Social engineering, denial-of-service attacks, and any system or domain not explicitly listed above.

⚠️ **Important:** This project is strictly for education and research purposes. These techniques must never be applied to any system without explicit written permission from the owner. All data referenced (patient records, staff salaries, shareholder details) is entirely fictional, generated for this training scenario.

---

## 🎯 Milestones Covered

| Milestone | Goal |
|---|---|
| **M1** | Initial Access — recon, find and exploit an authentication weakness, retrieve 3 confidential files |
| **M2** | Data Extraction — crack the encryption on all 3 retrieved PDF files |
| **M3** | Deep Reconnaissance — use file metadata and recon clues to uncover a critical data exposure |
| **M4** | Penetration Testing Report — full professional write-up of the engagement |

---

# 🪜 Milestone 1 — Initial Access

## Step 1. Reconnaissance — robots.txt

```bash
$ curl https://medirozahospital.com/robots.txt
```

![](images/m1-robots.png)

**Finding:** The `robots.txt` file disallowed three folders: `/patient/`, `/staff/`, and `/old/`. These hidden folders are not meant to be publicly indexed by search engines, but nothing prevents a human visitor from browsing them directly — a classic recon win.

## Step 2. Find the Login Page

Opening `/patient/` in the browser revealed a login page:

```
https://medirozahospital.com/patient/login.php
```

![](images/m1-login-page.png)

## Step 3. Username Enumeration

Tested a fake username vs. a guessed-real username to see if the application's error messages differ:

| Attempt | Username | Password | Response |
|---|---|---|---|
| 1 | `bob` | `test123` | "Username not found" |
| 2 | `admin` | `test123` | "Incorrect password" |

![](images/m1-username-enum.png)

**Finding:** The two different error messages confirm `admin` is a valid account — a **username enumeration** vulnerability (Medium risk). A properly secured login should return the same generic message regardless of which field is wrong.

## Step 4. Test for SQL Injection

```
username: admin'   password: test123
```

![](images/m1-sqli-test.png)

**Finding:** The application returned a raw MySQL syntax error, confirming the username field is **not sanitized** and is vulnerable to SQL injection.

## Step 5. Bypass the Login

```
username: admin' --   password: anything
```

![](images/m1-login-bypass.png)

**Finding:** The `'  --` payload breaks out of the SQL query and comments out the password check entirely, logging in as `admin` with no valid password required — a **Critical** authentication bypass via SQL injection.

## Step 6. Download the Confidential Reports

Once inside the patient portal, three files were available and downloaded:

- `patient_report_1.pdf`
- `patient_report_2.pdf`
- `patient_report_3.pdf`

![](images/m1-portal-reports.png)

---

# 🪜 Milestone 2 — Crack the Encryption

## Step 1. Extract Each PDF's Hash

Each PDF was uploaded to the [Networkwalks Hash Calculator](https://networkwalks.com/hash-calculator/) to extract its `$pdf$...` hash.

![](images/m2-hash-calculator.png)

## Step 2. Crack Reports 1 & 2 (Built-in Wordlist)

Using the [Networkwalks Password Cracker](https://networkwalks.com/password-cracker/) with its built-in common-password list:

| File | Cracked Password |
|---|---|
| `patient_report_1.pdf` | *(redacted \u2014 see local report)* |
| `patient_report_2.pdf` | *(redacted \u2014 see local report)* |

![](images/m2-crack-1-2.png)

## Step 3. Crack Report 3 (Larger Wordlist Required)

The built-in list failed against `patient_report_3.pdf` ("Exhausted wordlist. No match."). Switching to the larger JTR wordlist succeeded.

![](images/m2-crack-3-fail.png)
![](images/m2-crack-3-success.png)

**Lesson:** When a small wordlist fails, it doesn't mean the password can't be cracked — it means a larger, more comprehensive list is needed. Real attackers keep multiple wordlists of varying size for exactly this reason.

## Step 4. Keep an Unlocked Copy

```bash
qpdf --password='<cracked-password>' --decrypt patient_report_3.pdf report3_open.pdf
```

An unlocked copy was needed so metadata tools could read all file properties in the next milestone (locked PDFs only expose an "Encryption" field).

---

# 🪜 Milestone 3 — Deep Reconnaissance

## Step 1. Read the Metadata

```bash
$ exiftool report3_open.pdf
```

![](images/m3-exiftool.png)

**Finding:** The metadata revealed:
- **Author:** `j.malik`
- **Comments:** *"DB backup moved to /old before site migration, do not delete"*

A careless note left inside a file by IT staff pointed directly to a forgotten backup location.

## Step 2. Connect Back to Recon

This matched the `/old/` folder already flagged in `robots.txt` during Milestone 1 — a good example of how a clue discovered later in an assessment can connect back to something spotted earlier.

## Step 3. Open the Old Folder

```
https://medirozahospital.com/old/
```

![](images/m3-old-folder.png)

**Finding:** Directory listing was enabled, exposing a database backup file: `mediroza_db_backup_2019.sql`.

```bash
$ wget https://medirozahospital.com/old/mediroza_db_backup_2019.sql
```

## Step 4. Analyze the Database Backup

The `.sql` file's `INSERT INTO staff` and `INSERT INTO shareholders` rows were extracted and converted into readable tables (AI-assisted) showing staff names, job titles, departments, monthly salaries, and shareholder equity details.

![](images/m3-sql-data.png)

## Step 5. Close the Loop

Cross-referencing the staff table against the PDF metadata confirmed `j.malik` is **Jameel Malik, IT Systems Administrator** — the person who moved the backup and left the note, completing the full attack chain from a careless metadata comment to a critical confidential data exposure.

---

# 📋 Milestone 4 — Findings Summary

| # | Vulnerability | Location | Risk |
|---|---|---|---|
| 1 | Username enumeration on login page | `patient/login.php` | Medium |
| 2 | SQL injection login bypass | `patient/login.php` | **Critical** |
| 3 | Encrypted PDFs accessible after login bypass | `patient/reports/` | High |
| 4 | Weak PDF passwords crackable with a wordlist | `patient_report_*.pdf` | High |
| 5 | Sensitive metadata left in patient PDF files | `patient_report_3.pdf` | Medium |
| 6 | Forgotten backup folder with directory listing enabled | `old/` | **Critical** |
| 7 | Confidential staff salaries & shareholder data in plain text | `old/mediroza_db_backup_2019.sql` | **Critical** |

## Recommendations

- **Username enumeration:** Return the same generic error message regardless of whether the username or password was wrong.
- **SQL injection:** Use parameterized queries / prepared statements. Never concatenate raw user input into SQL queries.
- **PDF access control:** Store PDFs outside the web root or behind proper access controls; use strong, unique passwords per file.
- **PDF metadata:** Strip metadata from patient files before distribution (`exiftool -all= filename.pdf`).
- **Directory listing & backup exposure:** Disable directory listing on all web folders. Remove or relocate old backup files — never store database backups inside a public web directory.

📄 The full professional penetration test report (Executive Summary, Methodology, Findings, Risk Ratings, Recommendations) is included in this repository: [`Mediroza-Pentest-Report.docx`](./Mediroza-Pentest-Report.docx)

---

# 🐞 Problems Encountered & Solutions

*(to be filled in with any real issues hit during testing)*

---

# 💡 What I Learned

- How individual, seemingly minor weaknesses (a verbose error message, an indexed `robots.txt`, a careless metadata comment) chain together into a critical full-scale breach.
- Hands-on experience with SQL injection authentication bypass, PDF password cracking under varying wordlist sizes, metadata forensics (`exiftool`), and PDF decryption (`qpdf`).
- How to trace an attack path end-to-end and present it clearly in a professional report a non-technical client could understand.
- Why defense-in-depth matters: no single fix would have stopped this chain — each layer (input validation, access control, metadata hygiene, backup management) needed to be addressed independently.
- How to translate technical findings into business risk ratings and actionable remediation steps.

---

# 🔐 Security & Ethical Use

This repository is intended strictly for education and research purposes, as part of the Networkwalks Cybersecurity Internship. The target (`medirozahospital.com`) was purpose-built and authorized by Networkwalks for this training exercise; all patient, staff, and shareholder data referenced is fictional. Do not use these techniques against any system without explicit written permission.

---

# 🔗 Tools & Resources

- **Networkwalks Hash Calculator:** [https://networkwalks.com/hash-calculator/](https://networkwalks.com/hash-calculator/)
- **Networkwalks Password Cracker:** [https://networkwalks.com/password-cracker/](https://networkwalks.com/password-cracker/)
- **exiftool:** [https://exiftool.org/](https://exiftool.org/)
- **qpdf:** [https://qpdf.sourceforge.io/](https://qpdf.sourceforge.io/)

---

# 👤 Author

**Boahene Prince**
Cybersecurity Intern, Batch B083B — Networkwalks

LinkedIn: [https://www.linkedin.com/in/boahene-prince-603b08372/](https://www.linkedin.com/in/boahene-prince-603b08372/)

---

## 📌 Project Information

**Program Name:** Cybersecurity at Networkwalks | **Week:** 04 | **Project:** Penetration Testing Capstone — Mediroza General Hospital (Simulated) | **Repository:** GitHub

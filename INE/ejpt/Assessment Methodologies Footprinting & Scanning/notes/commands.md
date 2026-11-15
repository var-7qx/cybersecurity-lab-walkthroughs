# Command Reference — Assessment Methodologies: Footprinting & Scanning CTF 1

A concise technical reference of all verified commands executed during the **INE Assessment Methodologies: Footprinting & Scanning CTF 1** laboratory engagement.

---

## 1. Web Service Footprinting & Banner Grabbing

### Inspect HTTP Response Headers
```bash
curl -I http://target.ine.local
```
- `-I` / `--head`: Fetches HTTP response headers only without downloading the message body.
- Target URL: `http://target.ine.local`.
- Disclosed: `Server: Werkzeug/3.0.6 Python/3.10.12` and custom header `Server: FLAG1_43584150cfbc48168f25c7057f953894`.

---

## 2. FTP Service Footprinting & Anonymous Access

### Connect to Target FTP Service
```bash
ftp target.ine.local
```
- Establishes a standard FTP control connection on port 21 to `target.ine.local`.
- Target Banner: `220 (vsFTPd 3.0.5)`.
- Authentication:
  - Username: `anonymous`
  - Password: `<blank / press Enter>`
  - Response: `230 Login successful.`

### List Remote Directory Contents
```text
ftp> ls
```
- Switches to Extended Passive Mode (`229 Entering Extended Passive Mode`).
- Lists directory files:
  - `creds.txt` (22 bytes)
  - `flag.txt` (39 bytes)

### Inspect Remote Text Files Directly within FTP Client
```text
ftp> less flag.txt
```
- Reads the content of `flag.txt` using the built-in pager, disclosing `FLAG3_0c37a842c6284cae96014debea36d9d7`.

```text
ftp> less creds.txt
```
- Reads the content of `creds.txt`, disclosing database credentials: `db_admin:[REDACTED]`.

---

## 3. Database Footprinting & Schema Enumeration

### Connect to Remote MySQL / MariaDB Server
```bash
mysql -h target.ine.local -u db_admin -p
```
- `-h target.ine.local`: Specifies the remote database host.
- `-u db_admin`: Authenticates with the harvested username.
- `-p`: Prompts securely for the password (`[REDACTED]`).
- Target Version: `8.0.39-0ubuntu0.22.04.1 (Ubuntu)`.

### Enumerate Databases on Target Server
```sql
Show databases;
```
- Queries the MySQL information schema to list all accessible databases.
- Discloses databases, including `FLAG4_dd50cd8b67cb43f6aa24d49461b64ff5`.


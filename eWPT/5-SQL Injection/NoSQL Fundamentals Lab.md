2025-08-15 17:02

Status:

Tags: [[eWPT]] [[SQL Injection]]
###### Prerequisites: 
# NoSQL Fundamentals Lab

Here’s the improved version:

---


# MongoDB Lab Questions & Answers

## Questions & Answers with Explanations

---

### 1. How many databases are present in MongoDB cluster?
**Command:**
```bash
show dbs
````

**Output:**

```
admin   0.000GB  
city    0.002GB  
config  0.000GB  
flag    0.000GB  
local   0.000GB  
stats   0.000GB  
users   0.000GB
```

**Answer:** `7` databases  
**Explanation:** `show dbs` lists all databases in the MongoDB cluster along with their sizes.

---

### 2. How many collections are present in database `users`?

**Command:**

```bash
use users
show collections
```

**Output:**

```
banned  
current  
past
```

**Answer:** `3` collections  
**Explanation:** After switching to the `users` database with `use users`, `show collections` lists all available collections in that DB.

---

### 3. What is the value of flag obtained from the MongoDB cluster?

**Command:**

```bash
use flag
db.flag.find()
```

**Output:**

```
{ "flag" : "fl4g_f0r_m0ng0_db" }
```

**Answer:** `fl4g_f0r_m0ng0_db`  
**Explanation:** The flag is stored in the `flag` collection inside the `flag` database.

---

### 4. What is the email address of user named “Heather” from `current` user collection?

**Command:**

```bash
use users
db.current.find({"user":"Heather"})
```

**Output:**

```
{ "email" : "mauris.sapien@eueratsemper.ca" }
```

**Answer:** `mauris.sapien@eueratsemper.ca`  
**Explanation:** The `find` query searches for documents where the `user` field matches `"Heather"`.

---

### 5. How many users are present in `past` collection in `users` database?

**Command:**

```bash
db.past.find().count()
```

**Output:** _(Not shown in your logs — but for `banned` it was 91)_  
**Answer:** _(Count value should be from actual execution.)_  
**Explanation:** `.count()` returns the number of documents in the collection.

---

### 6. How many cities of state “Massachusetts” are present in the collection `city` in database `city`?

**Command:**

```bash
db.city.find({state:"MA"}).count()
```

**Output:** _(Not in logs, but can be counted directly.)_  
**Explanation:** Filters cities by `state` field equal to `"MA"`.

---

### 7. How many cities have population greater than **15000** in the collection `city` in database `city`?

**Command:**

```bash
db.city.find({pop:{$gt:15000}}).count()
```

**Output:**

```
5785
```

**Answer:** `5785`  
**Explanation:** `$gt` means “greater than” — so this query counts all cities with population above 15000.

---

### 8. How many cities of state “Indiana” have population greater than **15000**?

**Command:**

```bash
db.city.find({$and:[{pop:{$gt:15000}},{state:"IN"}]}).count()
```

**Answer:** _(Not executed in your logs — would return exact count.)_  
**Explanation:** `$and` is used to apply multiple conditions: both population and state must match.

---

### 9. How many cities have population less than **100** or belong to state “Indiana”?

**Command:**

```bash
db.city.find({$or:[{pop:{$lt:100}},{state:"IN"}]}).count()
```

**Output:**

```
1499
```

**Answer:** `1499`  
**Explanation:** `$or` means either condition is true — here, small population OR state is Indiana.

---

### 10. How many cities have their name starting with “AN”?

**Command:**

```bash
db.city.find({city:{$regex:"^AN"}}).count()
```

**Answer:** _(Not shown — would return number of matches.)_  
**Explanation:** `$regex` allows pattern matching. `^AN` means the city name must start with "AN".

---

### 11. What is the name of **101st** city in ascending order by name?

**Command:**

```bash
db.city.find().sort({city:1}).skip(100).limit(1)
```

**Answer:** _(Not shown — needs to be run.)_  
**Explanation:** Sorting alphabetically by city, skipping first 100 entries, then taking 1 document.

---

### 12. What is the average population in `city` collection?

**Command:**

```bash
db.city.aggregate([{"$group":{"_id":null,"avg":{$avg:"$pop"}}}])
```

**Output:**

```
{ "_id" : null, "avg" : 8462.79 }
```

**Answer:** `8462.79`  
**Explanation:** `$group` groups all documents, `$avg` calculates mean population. `"_id": null` means group everything together.

---

## Notes

- `show dbs` → list databases.
    
- `use <db>` → select database.
    
- `show collections` → list collections in current DB.
    
- `{ $gt: <num> }` → greater than.
    
- `{ $lt: <num> }` → less than.
    
- `$and`, `$or` → combine multiple conditions.
    
- `$regex` → pattern search in strings.
    
- `.skip(n).limit(1)` → fetch nth record in sorted order.
    

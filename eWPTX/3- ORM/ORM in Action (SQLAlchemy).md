2026-02-04 06:53

Status:

Tags: [[eWPTX]] [[eWPTX/3- ORM/ORM]] [[SQL Injection]]
###### Prasyarat: [[ORM in Web Apps]]
# ORM dalam Praktik (SQLAlchemy)

## Skenario

Buat sebuah user record (bandingkan raw SQL vs ORM).

---

## Tanpa ORM (raw SQL)

```python
import sqlite3

conn = sqlite3.connect("example.db")
cursor = conn.cursor()

cursor.execute(
    "INSERT INTO users (username, email) VALUES (?, ?)",
    ("john_doe", "john@example.com"),
)
conn.commit()

cursor.close()
conn.close()
```

---

## Dengan ORM (SQLAlchemy)

```python
from sqlalchemy import Column, Integer, String, create_engine
from sqlalchemy.orm import declarative_base, sessionmaker

Base = declarative_base()


class User(Base):
    __tablename__ = "users"

    id = Column(Integer, primary_key=True)
    username = Column(String, nullable=False)
    email = Column(String, nullable=False)


engine = create_engine("sqlite:///example.db")
Session = sessionmaker(bind=engine)
session = Session()

# Create a new user (object -> row)
new_user = User(username="john_doe", email="john@example.com")
session.add(new_user)
session.commit()
```

## Apa yang terjadi
- `User` dipetakan ke tabel `users`
- atribut dipetakan ke kolom
- `session.add()` menjadwalkan operasi insert
- `session.commit()` mengirim SQL ke database
- `session.commit()` sends SQL to the DB

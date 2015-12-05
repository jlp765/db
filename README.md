# db
Unified db access module

The DB module provides a single library module to access the ``db_sqlite``, ``db_mysql`` and ``db_postgres`` modules.

It adds one extra proc, ``initDb()``

### Example
The following example is from the module documentation, and shows compile-time (hard-wired) selection of the database type.

The db module is also for multiple connections to multiple different database types simultaneously.

```
import db, math

var theDb: DbConnId             # this id uniquely identifies this database session

when defined(sqlite):           # configure the database type to connect to
  theDb = initDb(DbKind.Sqlite)
elif defined(mysql):
  theDb = initDb(DbKind.Mysql)
elif defined(postgres):
  theDb = initDb(DbKind.Postgres)
else:
  {.fatal: "DEFINE one of: sqlite mysql postgres  eg, -d:sqlite ".}

theDb.open("localhost", "nim", "nim", "test")   # actual db connection made here!

theDb.exec(sql"Drop table if exists myTestTbl")
let mquery = sql"""create table myTestTbl (
      Id    INT(11)     NOT NULL AUTO_INCREMENT PRIMARY KEY,
      Name  VARCHAR(50) NOT NULL,
      i     INT(11),
      f     DECIMAL(18,10))"""
let squery = sql"""create table myTestTbl (
      Id    INTEGER     PRIMARY KEY,
      Name  VARCHAR(50) NOT NULL,
      i     INT(11),
      f     DECIMAL(18,10))"""
let pquery = sql"""create table myTestTbl (
      Id    SERIAL PRIMARY KEY,
      Name  VARCHAR(50) NOT NULL,
      i     int,
      f     NUMERIC(18,10))"""

when defined(sqlite):           # create the table myTestTbl
  theDb.exec(squery)
elif defined(mysql):
  theDb.exec(mquery)
else:
  theDb.exec(pquery)

when not defined(sqlite):       # wrap multi inserts in a transaction to improved performance
theDb.exec(sql"START TRANSACTION")   # both mysql & postgres
else:
  theDb.exec(sql"BEGIN")   # sqlite only

for i in 1..1000:               # do 1000 inserts
  theDb.exec(sql"INSERT INTO myTestTbl (name,i,f) VALUES (?,?,?)",
        "Item#" & $i, i, sqrt(i.float))
theDb.exec(sql"COMMIT")

for x in theDb.instantRows(sql"select * from myTestTbl"):   # show what has been inserted
  echo x.row

let id = theDb.tryInsertId(sql"INSERT INTO myTestTbl (name,i,f) VALUES (?,?,?)",
             "Item#1001", 1001, sqrt(1001.0))
echo "Inserted item: ", theDb.getValue(sql"SELECT name FROM myTestTbl WHERE id=?", id)

theDb.close()   # bye  -> note theDb is now no longer valid
```

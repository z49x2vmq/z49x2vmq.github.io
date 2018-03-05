---
layout: post
title: 'Python으로 SQLite 접속하기'
date:   2018-03-01 06:10:00 +0900
tags: sqlite python
---

Python에서 Sqlite DB에 접속. 간단한 Statement 실행 예제.

```python
import sqlite3

# CREATE TABLE Statement
stmt_create_table_thp = """                                                 
CREATE TABLE IF NOT EXISTS THP_READINGS (
  MEASUREMENT_TIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
  DEVICE_ID varchar(20) NOT NULL,
  TEMPERATURE_CELSIUS smallint(5) DEFAULT NULL,
  HUMIDITY_RELATIVE smallint(5) DEFAULT NULL,
  PM1_0_CF smallint(5) DEFAULT NULL,
  PM2_5_CF smallint(5) DEFAULT NULL,
  PM10_0_CF smallint(5) DEFAULT NULL,
  PM1_0_ATMO smallint(5) DEFAULT NULL,
  PM2_5_ATMO smallint(5) DEFAULT NULL,
  PM10_0_ATMO smallint(5) DEFAULT NULL,
  PRIMARY KEY (MEASUREMENT_TIME,DEVICE_ID)
)
"""

# INSERT Statement
stmt_insert_thp = """
INSERT INTO 
THP_READINGS(DEVICE_ID , TEMPERATURE_CELSIUS, HUMIDITY_RELATIVE,
             PM1_0_CF  , PM2_5_CF           , PM10_0_CF,
             PM1_0_ATMO, PM2_5_ATMO         , PM10_0_ATMO)
VALUES(?, ?, ?,
       ?, ?, ?,
       ?, ?, ?)
"""

# Open Connection to sqlite
conn = sqlite3.connect('sample.db')

# Open Cursor
cursor = conn.cursor();
# Excute the create table statement
cursor.execute(stmt_create_table_thp)

# Set Insert values for prepared statement(Parameterized Query)
insert_values = ('aa-aa-aa-aa-aa-aa', 37, 37, 1, 2, 3, 4, 5, 6)

# Execute the insert statement and commit
cursor.execute(stmt_insert_thp, insert_values)
conn.commit();

# Close the cursor
cursor.close()

# Close the connection
conn.close()
```

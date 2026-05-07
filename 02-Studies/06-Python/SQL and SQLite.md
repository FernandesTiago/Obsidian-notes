
learned this to persist data properly instead of using JSON for everything. sqlite3 is built into python so no install needed

## connecting and setting up

import sqlite3

connection = sqlite3.connect("mydata.db") cursor = connection.cursor()

the .db file gets created automatically if it doesnt exist. cursor is what you use to actually run queries

## creating a table

cursor.execute(""" CREATE TABLE IF NOT EXISTS users ( id INTEGER PRIMARY KEY AUTOINCREMENT, name TEXT, age INTEGER, salary REAL ) """) connection.commit()

IF NOT EXISTS is important, without it it crashes if the table already exists AUTOINCREMENT means the id goes up by itself, you dont need to set it manually types are INTEGER, TEXT, REAL (float), BLOB

## INSERT

cursor.execute("INSERT INTO users (name, age) VALUES (?, ?)", (name, age)) connection.commit()

the ? placeholders are important. never put variables directly in the string like f"INSERT ... VALUES ({name})" because that opens you up to SQL injection. always use ? and pass the values as a tuple

## SELECT

cursor.execute("SELECT * FROM users") all_rows = cursor.fetchall() # list of tuples

cursor.execute("SELECT * FROM users WHERE id = ?", (user_id,)) one_row = cursor.fetchone() # one tuple or None if not found

cursor.execute("SELECT COUNT(*) FROM users") count = cursor.fetchone()[0] # the count is at index 0

cursor.execute("SELECT MAX(id) FROM users") max_id = cursor.fetchone()[0]

rows come back as tuples so row[0] is id, row[1] is name etc. always remember the id column shifts everything

## UPDATE

cursor.execute("UPDATE users SET name = ? WHERE id = ?", (new_name, user_id)) connection.commit()

ALWAYS use WHERE with UPDATE. without it you update every single row in the table

## DELETE

cursor.execute("DELETE FROM users WHERE id = ?", (user_id,)) connection.commit()

same rule, ALWAYS use WHERE with DELETE

## golden rules

- always commit() after INSERT, UPDATE, DELETE or changes dont save
- always use WHERE on UPDATE and DELETE
- always use ? placeholders, never string formatting for values
- rows come back as tuples, remember index 0 is id

## threading issue with FastAPI

learned this the hard way. sqlite connections cant be shared between threads. if youre using SQLite with FastAPI, create a new connection inside each method instead of storing it in **init**

def get_all(self): connection = sqlite3.connect("mydata.db") cursor = connection.cursor() cursor.execute("SELECT * FROM users") result = cursor.fetchall() connection.close() return result

## things I kept getting wrong

- forgetting commit() and wondering why nothing was saving
- forgetting WHERE on UPDATE and wiping the entire table once
- accessing row[1] when I wanted the name but the id was at row[0] shifting everything
- typo in INTEGER (wrote INTERGER) and sqlite just silently accepted it as a TEXT column
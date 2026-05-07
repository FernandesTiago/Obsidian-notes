
reading and writing files, and working with JSON for saving data

## opening files

with open("file.txt", "r") as file: content = file.read()

the with keyword handles closing the file automatically even if something crashes. before I knew this I was manually calling file.close() which is messier

### modes

- "r" read, crashes if file doesnt exist
- "w" write, creates if doesnt exist, OVERWRITES everything if it does
- "a" append, adds to the end without overwriting

### reading

file.read() # entire file as one string file.readlines() # list of lines, each line is a string with \n at the end

### writing

file.write("hello\n") # writes a string, you handle the newlines

## pathlib

one problem with file paths is that they break between operating systems. pathlib fixes this

from pathlib import Path

base = Path(**file**).parent # folder where the current script is file_path = base / "data.txt" # builds the path correctly for any OS

I always use this now so scripts can find their files no matter where you run them from

## handling FileNotFoundError

with open("data.txt", "r") as file: # crashes if file doesnt exist ...

the fix is try/except

try: with open("data.txt", "r") as file: content = file.read() except FileNotFoundError: content = "" # just start fresh

## JSON

JSON is the standard format for saving structured data. looks exactly like a python dict

### saving to JSON

import json

data = {"name": "Tiago", "scores": [10, 20, 30]}

with open("data.json", "w") as file: json.dump(data, file, indent=4)

always use "w" mode with json. JSON is one indivisible block, you cant append to it or you corrupt the structure

### loading from JSON

with open("data.json", "r") as file: data = json.load(file)

### handling errors when loading

the file might not exist yet, or it might be corrupted

try: with open("data.json", "r") as file: data = json.load(file) except (FileNotFoundError, json.JSONDecodeError): data = {} # start fresh

catching both in one except because either can happen

## things I kept getting wrong

- using "a" mode with JSON and corrupting the file
- forgetting that readlines() includes \n at the end of each line
- not using pathlib and having the script fail to find files when run from a different directory
- forgetting json.JSONDecodeError exists and only catching FileNotFoundError
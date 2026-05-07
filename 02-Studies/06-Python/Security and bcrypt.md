
learned this while building the login system in ex026. storing passwords as plain text is a really bad idea

## the problem with plain text passwords

if someone gets access to your database they can read every password directly. and since people reuse passwords, now they have access to everything else that person uses too

## hashing

a hash is a one way transformation. you put in a password, you get back a long string. there's no way to reverse it

"mypassword" -> hash function -> "3a7bd3e2360a..."

when someone logs in you don't decrypt the stored hash, you hash what they typed and compare the two hashes

## bcrypt

bcrypt is better than basic hash functions like sha256 for passwords because

- it's intentionally slow, so brute force attacks take way longer
- it automatically adds a salt (random data mixed in before hashing) so two users with the same password have different hashes

install it with pip install bcrypt

import bcrypt

### hashing a password (at signup)

password = "mypassword" hashed = bcrypt.hashpw(password.encode(), bcrypt.gensalt()) stored = hashed.decode() # convert bytes to string for storing in sqlite

gensalt() only runs at signup. the salt gets embedded inside the hash automatically

### verifying at login

stored_hash = "the string from the database" bcrypt.checkpw(input_password.encode(), stored_hash.encode()) # True or False

checkpw extracts the salt from the stored hash automatically and uses it to compare. you never touch the salt directly

### the full flow

signup:

1. user types password
2. bcrypt.hashpw(password.encode(), bcrypt.gensalt())
3. .decode() and store in db as TEXT

login:

1. user types password
2. fetch stored hash from db
3. bcrypt.checkpw(input.encode(), stored_hash.encode())
4. if True, login successful

## security principles I learned

never store plain text passwords, ever

dont tell the user whether the username or password was wrong specifically. just say "invalid username or password". if you say "username not found" an attacker knows which usernames exist

API keys should come from environment variables, not hardcoded in the code

os.getenv("API_KEY") # reads from environment, not in the code

## things I kept getting wrong

- storing hashed bytes directly without .decode() and getting encoding errors later
- forgetting to .encode() the stored hash when calling checkpw, it needs bytes on both sides
- old corrupted data in the db causing "invalid salt" errors even after fixing the code, needed to delete the db file
- row index being off by one because I forgot the id column comes first
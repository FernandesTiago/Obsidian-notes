
the main ways to store collections of data in python

## Lists

ordered, can have duplicates, can mix types (though usually dont)

my_list = [1, 2, 3, "four", True]

### useful list methods

my_list.append(item) # adds to the end my_list.pop() # removes and returns the last item my_list.pop(0) # removes and returns item at index 0 my_list.sort() # sorts in place, modifies the original sorted(my_list) # returns a new sorted list, original unchanged

min(my_list) # smallest value max(my_list) # largest value my_list.index(value) # returns the index of the value

### list comprehension

instead of building a list with a for loop you can do it in one line

squares = [x ** 2 for x in range(10)]

the expression comes first, then the for. looked weird at first but its really clean once you get used to it

you can also filter with an if at the end evens = [x for x in range(10) if x % 2 == 0]

## Dicts

key value pairs, like a real dictionary. keys are unique

person = { "name": "Tiago", "age": 23, "city": "São Paulo" }

accessing values person["name"] # "Tiago", crashes if key doesnt exist person.get("name") # "Tiago", returns None if key doesnt exist person.get("job", "unemployed") # returns "unemployed" if key doesnt exist

### modifying dicts

person["age"] = 24 # update existing key person["job"] = "developer" # add new key person.update({"age": 24, "job": "developer"}) # update multiple at once del person["city"] # delete a key

### nested dicts and lists of dicts

this comes up constantly

users = [ {"name": "Tiago", "age": 23}, {"name": "Ana", "age": 20} ]

for user in users: print(user["name"])

### sorting a list of dicts

sorted(users, key=lambda x: x["age"])

## things I kept getting wrong

- using [] instead of .get() and crashing when the key didnt exist
- confusing .sort() which modifies in place with sorted() which returns a new list
- trying to use a list as a dict key (lists cant be dict keys, they're not hashable)
- initializing a comparison variable to 0 instead of the first element of the list when looking for min/max manually
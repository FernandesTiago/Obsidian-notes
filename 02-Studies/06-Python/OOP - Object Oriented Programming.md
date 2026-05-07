
this was the biggest jump so far. went from writing functions to thinking in objects

## the basics

a class is a blueprint, an object is the thing you create from it

class Dog: def **init**(self, name, age): self.name = name self.age = age

```
def bark(self):
    print(f"{self.name} says woof!")
```

my_dog = Dog("Rex", 3) my_dog.bark()

**init** runs automatically when you create an object. self refers to the object itself, you need it as the first parameter in every method

## instance vs class attributes

instance attributes belong to each object individually class attributes are shared between all objects of that class

class Counter: total = 0 # class attribute, shared

```
def __init__(self, name):
    self.name = name    # instance attribute, unique per object
    Counter.total += 1
```

## dunder methods

special methods with double underscores that python calls automatically in certain situations

**init** - called when creating an object **str** - called when you print the object or use str() on it **del** - called when the object is destroyed (garbage collection)

def **str**(self): return f"Dog: {self.name}, Age: {self.age}"

without **str** printing an object gives you something ugly like <**main**.Dog object at 0x...>

## inheritance

one class can inherit from another, getting all its methods and attributes

class Animal: def **init**(self, name): self.name = name

```
def breathe(self):
    print("breathing")
```

class Dog(Animal): def **init**(self, name, breed): super().**init**(name) # calls Animal's **init** self.breed = breed

```
def bark(self):
    print("woof")
```

super() is how you call the parent class. important to not skip it or the parent's **init** doesnt run

## composition

instead of inheriting, one class can just use another class

class DataBase: def get_all(self): ...

class Menu: def **init**(self): self.db = DataBase() # Menu HAS a DataBase

```
def show(self):
    data = self.db.get_all()
    ...
```

this is what I used in ex025 and ex026. Menu doesnt inherit from DataBase, it just owns one and uses it

## polimorfism

same method name, different behavior depending on the class

class Cat: def speak(self): print("meow")

class Dog: def speak(self): print("woof")

animals = [Cat(), Dog()] for animal in animals: animal.speak() # each calls its own version

## if **name** == "**main**"

this prevents code from running when the file is imported by another file

if **name** == "**main**": app = Menu() app.run()

when you run the file directly, **name** is "**main**" so it runs when another file imports it, **name** is the filename so it doesnt run

## things I kept getting wrong

- forgetting self as the first parameter in methods
- forgetting to call super().**init**() in child classes
- confusing class attributes with instance attributes and accidentally sharing state between objects
- not knowing when to use inheritance vs composition. rule of thumb: use inheritance for "is a" relationships, composition for "has a" relationships
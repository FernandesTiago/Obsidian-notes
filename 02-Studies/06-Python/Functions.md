
functions are just reusable blocks of code. instead of writing the same thing 5 times you write it once and call it

## basic def

def greet(name): print(f"hello {name}")

greet("Tiago")

the def keyword defines the function, then you call it by name with ()

## return vs print

this took me a while to really get. print just shows something on screen, return sends the value back to whoever called the function

def add(a, b): return a + b

result = add(2, 3) # result is 5 print(result)

if you dont have a return, python returns None automatically. so if you do result = my_function() and the function has no return, result will be None. this caused a lot of bugs early on

rule I follow now: functions should return values, not print them. let the caller decide what to do with the result

## arguments

def greet(name, greeting="Hello"): print(f"{greeting}, {name}!")

greet("Tiago") # Hello, Tiago! greet("Tiago", "Hey") # Hey, Tiago!

the greeting="Hello" is a default argument. if you dont pass it, it uses the default. positional arguments come first, default ones after

## docstrings

def add(a, b): """adds two numbers and returns the result""" return a + b

the string right after the def is the docstring. its documentation inside the function. shows up when you hover over the function in pycharm which is pretty useful

## lambda

short anonymous functions, useful when you need a quick function for something like sorting

people = [{"name": "Tiago", "age": 23}, {"name": "Ana", "age": 20}] sorted_people = sorted(people, key=lambda x: x["age"])

the lambda x: x["age"] is a tiny function that takes x and returns x["age"]. sorted uses it to know what to sort by

## global

if you need to modify a variable from outside the function you need the global keyword

count = 0

def increment(): global count count += 1

honestly try to avoid this. its bad practice. better to pass things as parameters and return the result

## things I kept getting wrong

- forgetting return and using print inside functions instead
- not storing the return value so losing it
- modifying global variables without the global keyword and getting UnboundLocalError
- putting default arguments before positional ones, python doesnt allow that
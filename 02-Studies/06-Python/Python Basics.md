
starting to learn python

## Variables and Types

so python just figures out the type by itself, you don't need to say "this is an int" like in other languages apparently

name = "Tiago" age = 23 height = 1.80 is_cool = True

the types are str, int, float and bool. you can check what type something is with type() if you forget

### Casting

sometimes you need to convert between types, like when you get a number from input() it comes as a string so you need to convert it

number = int("42") text = str(123) decimal = float("3.14")

this tripped me up a lot at the start, input() ALWAYS returns a string even if the user types a number. classic bug

## Strings

just text in quotes, single or double doesnt matter

name = "Tiago" greeting = f"Hello, {name}!"

the f before the quotes makes it an f-string, useful to use variables in the string

useful things you can do with strings

- strip() removes whitespace from the sides
- upper() and lower() change the case
- title() capitalizes the first letter of each word
- split() breaks the string into a list by spaces
- replace() swaps one thing for another
- join() is the opposite of split, merges a list into a string

also isalpha() checks if its all letters, isnumeric() checks if its all numbers. used these a lot for input validation

## Input and Output

name = input("What is your name? ") print(f"Hello, {name}!")

input always returns string. always. burned by this more than once

if you need a number just wrap it age = int(input("How old are you? "))

## Arithmetic

pretty normal stuff

- - - - / work as expected
- / always returns a float even if the result is a whole number
- // is integer division, cuts the decimal
- % gives the remainder
- ** is exponent so 2 ** 3 = 8

square root needs the math module, math.sqrt(16) gives 4.0

## things I kept getting wrong

- forgot to cast input() to int when comparing to numbers
- used = instead of == in conditions
- forgot the f in f-strings and wondered why the variable wasnt showing up
- / giving 3.333 when I wanted 3, needed // instead
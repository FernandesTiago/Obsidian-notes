
how to deal with things that can go wrong without crashing the whole program

## try / except / else / finally

try: number = int(input("enter a number: ")) except ValueError: print("thats not a number") else: print(f"you entered {number}") finally: print("this always runs no matter what")

- try: the risky code goes here, keep it surgical, only wrap the line that can actually fail
- except: runs if something goes wrong
- else: runs only if try succeeded, no errors
- finally: always runs, even if there was an error. good for cleanup

the key thing I learned: try blocks should be as small as possible. dont wrap 20 lines in a try, only wrap the one line that can actually throw an error

## common exceptions

ValueError - wrong type, like int("abc") FileNotFoundError - file doesnt exist KeyError - dict key doesnt exist IndexError - list index out of range TypeError - wrong type for an operation ZeroDivisionError - dividing by zero json.JSONDecodeError - invalid json

## catching multiple exceptions

try: ... except (ValueError, TypeError): print("something went wrong with the types")

or handle them separately

try: ... except ValueError: print("value error") except TypeError: print("type error")

## with statement

for files, the with statement handles cleanup automatically so you dont need finally

with open("file.txt", "r") as file: content = file.read() file is automatically closed here even if something crashed

with basically does the finally for you

## raising exceptions

you can raise your own errors too

if age < 0: raise ValueError("age cant be negative")

learned this when building APIs, HTTPException in FastAPI works the same way. raise stops the function immediately just like return

## things I kept getting wrong

- wrapping too much code in try and not knowing which line actually failed
- forgetting else exists and putting success code inside try where it doesnt belong
- catching Exception (the base class) which catches everything including bugs I wanted to see
- not knowing what exception a specific thing throws and guessing wrong
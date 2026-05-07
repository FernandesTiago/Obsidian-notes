
this is basically how you make the program make decisions and repeat things

## if / elif / else

age = 18

if age >= 18: print("adult") elif age >= 13: print("teenager") else: print("kid")

pretty straightforward. the elif is just "else if", you can have as many as you want. else catches everything that didnt match above

### ternary

theres a shorter way to write a simple if/else in one line

result = "adult" if age >= 18 else "kid"

looked weird at first but its useful for simple cases. wouldnt use it for anything complex tho

## while

repeats while the condition is true. the classic use is keeping asking for input until the user gives something valid

while True: answer = input("yes or no? ") if answer == "yes" or answer == "no": break print("invalid, try again")

break exits the loop immediately. continue skips to the next iteration without running the rest of the loop body

the danger with while is infinite loops if you forget to break or update the condition

## for

for loops over a sequence, like a list or a range

for i in range(5): print(i) # 0 1 2 3 4

range(5) gives 0 to 4, not 5. always trips me up range(1, 6) gives 1 to 5 range(0, 10, 2) gives 0 2 4 6 8, the third argument is the step

looping over a list for item in my_list: print(item)

### enumerate

when you need the index AND the value at the same time

for index, value in enumerate(my_list): print(index, value)

way cleaner than doing my_list[i] manually

## things I kept getting wrong

- range(5) goes 0 to 4 not 1 to 5, kept getting off by one errors
- forgetting break in a while True loop and getting stuck
- using continue when I meant break and wondering why the loop kept going
- indentation errors, python is strict about this
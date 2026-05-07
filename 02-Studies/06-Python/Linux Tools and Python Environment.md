
notes on the terminal stuff and environment setup that I had to learn alongside python

## venv (virtual environments)

the problem: if you install packages globally they can conflict between projects. project A needs library version 1.0, project B needs version 2.0, they cant both win

the solution: each project gets its own isolated environment

creating one python -m venv .venv

activating it (linux/mac) source .venv/bin/activate

you know its active because you see (.venv) at the start of your terminal prompt

installing packages inside it pip install fastapi uvicorn bcrypt

these only install inside this venv, not globally. when you open a different project you activate its own venv

in pycharm you can set the interpreter to the venv's python in Settings > Python > Interpreter. once set pycharm uses it automatically

## pip

python's package manager

pip install package-name      # install 
pip install `package-name==1.2.3`     # install specific version 
pip list         # see installed packages 
pip freeze       # list packages in a format good for requirements.txt

## requirements.txt

a file that lists all your project's dependencies so someone else can install them all at once

pip freeze > requirements.txt # generates the file pip install -r requirements.txt # installs everything in it

good practice to have one in every project

## uvicorn

the server that runs FastAPI apps

uvicorn filename:app --reload

filename is the python file without .py app is the FastAPI instance variable name --reload restarts on file changes

## common linux commands I use

ls # list files in current directory ls -la # list including hidden files with details cd folder # change directory cd .. # go up one level pwd # print current directory path mkdir name # create folder rm file # delete file cat file # print file contents

## running python scripts

python script.py # run a script python -m venv .venv # run a module (venv is a module)

## things I kept getting wrong

- forgetting to activate the venv and installing packages globally by mistake
- pycharm using the global interpreter instead of the venv, had to set it manually in settings
- creating the venv with the wrong python version and having to redo it
- running uvicorn from the wrong directory and it not finding the file
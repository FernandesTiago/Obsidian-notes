
first time building an actual API. coming from terminal programs this felt like a big jump but its surprisingly approachable

## what is an API

a program that listens for HTTP requests and sends back data. instead of a human clicking buttons, code talks to code. the Claude API I used in my raspberry pi bot is the same concept, I was just on the client side then

## setup

pip install fastapi uvicorn

uvicorn is the server that keeps the app running and listening for requests

running it uvicorn filename:app --reload

--reload watches for file changes and restarts automatically. essential during development

## basic structure

from fastapi import FastAPI

app = FastAPI()

@app.get("/") def root(): return {"message": "hello"}

the @ is a decorator. @app.get("/") tells FastAPI to call this function when someone makes a GET request to /

FastAPI automatically converts the dict you return to JSON. you dont have to do anything

## HTTP methods

GET - read data, should never change anything POST - create something new, data comes in the request body DELETE - remove something PUT - replace something entirely

using GET to delete things technically works but its wrong because browsers and crawlers call GET freely assuming nothing will break

## path parameters

@app.get("/songs/{sid}") def get_song(sid: int): ...

the {sid} in the URL becomes a parameter in the function. type hints here matter, FastAPI validates automatically. if you pass a letter where an int is expected it returns a clean error without any code from you

## request body with Pydantic

POST requests send data in the body, not the URL. you define what that data looks like with a Pydantic model

from pydantic import BaseModel

class SongModel(BaseModel): song: str album: str artist: str

@app.post("/songs") def add_song(song: SongModel): db.add(song.song, song.album, song.artist) return {"message": "song added"}

FastAPI sees that SongModel is a Pydantic model and knows to look in the request body automatically. you access the fields like any object

## status codes

200 - ok 201 - created 400 - bad request, client did something wrong 401 - unauthorized 404 - not found 500 - internal server error, something crashed on the server

## HTTPException

from fastapi import HTTPException

@app.get("/songs/{sid}") def get_song(sid: int): song = db.find_by_id(sid) if not song: raise HTTPException(status_code=404, detail="song not found") return song

raise stops the function immediately like return. FastAPI catches it and sends the proper error response

## /docs

FastAPI generates an interactive documentation page at /docs automatically. you can test every endpoint there without writing any frontend code. massive time saver during development

## query parameters

@app.get("/songs") def songs(artist: str = None): ...

called like /songs?artist=Queen. different from path parameters which are in the URL itself

## things I kept getting wrong

- sqlite threading issue: connections created in **init** cant be used in FastAPI endpoints because they run in different threads. fix is to open a new connection inside each database method
- forgetting that POST body needs a Pydantic model, you cant just use str or int for body data
- returning raw tuples from sqlite instead of converting to dicts first, FastAPI cant serialize tuples
- not checking if a record exists before deleting and not returning 404 when it doesnt
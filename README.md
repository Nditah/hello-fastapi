## FastApi-App

[Completed App](https://github.com/Nditah/hello-fastapi) | 
[FastApi Official Website](https://fastapi.tiangolo.com//) | 


1. Make a Directory for the Project and navigate into it
     > mkdir fastapi-app && cd fastapi-app

2. Create a Python Virtual Environment and Activate it
     > python3 -m venv venv 
     > ls
     > source venv/bin/activate

3. Install Fastapi 
     > pip install fastapi

4. Install an ASGI server 
     > pip install "uvicorn[standard]"

5. Install package for the template 
     > pip install python-multipart jinja2

6. Install package for database support
     > pip install sqlalchemy


7. The Open the Project with VSCode and create three files: `app.py`, `database.py` and `models.py`

Note:
 Update Pip `python3 -m pip install --upgrade pip`

## app.py

```py
from fastapi import FastAPI, Depends, Request, Form, status

from starlette.responses import RedirectResponse
from starlette.templating import Jinja2Templates

from sqlalchemy.orm import Session
from database import SessionLocal, engine
import models

models.Base.metadata.create_all(bind=engine)

templates = Jinja2Templates(directory="templates")

app = FastAPI()

# Dependency
def get_db():
    db = SessionLocal()
    try: 
        yield db
    finally:
        db.close()

@app.get("/")
async def home(req: Request, db: Session = Depends(get_db)):
    todos = db.query(models.Todo).all()
    return templates.TemplateResponse("base.html", { "request": req, "todo_list": todos })

@app.post("/add")
def add(req: Request, title: str = Form(...), db: Session = Depends(get_db)):
    new_todo = models.Todo(title=title)
    db.add(new_todo)
    db.commit()
    url = app.url_path_for("home")
    return RedirectResponse(url=url, status_code=status.HTTP_303_SEE_OTHER)

@app.get("/update/{todo_id}")
def add(req: Request, todo_id: int, db: Session = Depends(get_db)):
    todo = db.query(models.Todo).filter(models.Todo.id == todo_id).first()
    todo.complete = not todo.complete
    db.commit()
    url = app.url_path_for("home")
    return RedirectResponse(url=url, status_code=status.HTTP_303_SEE_OTHER)


@app.get("/delete/{todo_id}")
def add(req: Request, todo_id: int, db: Session = Depends(get_db)):
    todo = db.query(models.Todo).filter(models.Todo.id == todo_id).first()
    db.delete(todo)
    db.commit()
    url = app.url_path_for("home")
    return RedirectResponse(url=url, status_code=status.HTTP_303_SEE_OTHER)


```

## models.py

```py
from sqlalchemy import Boolean, Column, Integer, String
from database import Base

class Todo(Base):
    __tablename__ = "todos"

    id = Column(Integer, primary_key=True)
    title = Column(String(100))
    complete = Column(Boolean, default=False) 

```

## database.py

```py
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
from sqlalchemy.ext.declarative import declarative_base

DB_URL = "sqlite:///./db.sqlite"

engine = create_engine(DB_URL, connect_args = { "check_same_thread": False })

SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

Base = declarative_base() 

```



## templates/base.html

```html
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Todo App - Fastapi</title>
        <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/semantic-ui@2.4.2/dist/semantic.min.css">
        <script src="https://cdn.jsdelivr.net/npm/semantic-ui@2.4.2/dist/semantic.min.js"></script>
    </head>
    <body>
        <div style="margin-top: 50px;" class="ui container">
            <h1 class="ui center aligned header">Fastapi ToDo App</h1>

            <form class="ui form" action="/add" method="post">
                <div class="field">
                    <label>Todo Title</label>
                    <input type="text" name="title" placeholder="Enter ToDo task...">
                    <br>
                </div>
                <button class="ui blue button" type="submit">Add</button>
            </form>

            <hr>

            {% for todo in todo_list %} 
            <div class="ui segment">
                <p class="ui big header">{{ todo.id }} | {{ todo.title }}</p>

                {% if todo.complete == False %}
                <span class="ui gray label">Not Complete</span>
                {% else %}
                <span class="ui green label">Complete</span>
                {% endif %}

                <a class="ui blue button" href="/update/{{ todo.id }}">Update</a>
                <a class="ui red button" href="/delete/{{ todo.id }}">Delete</a>

            </div>
            {% endfor %}

        </div>


    </body>

</html>

```

6. To run the app, with uvicorn using the file_name:app_instance

     > uvicorn app:app --reload

Preview the App at http://127.0.0.1:8000/ and the out-of-the-box [API Documentation](http://127.0.0.1:8000/docs)

7. Commit and push your code to github.com and Deactivate the Virtual Env.
   
    $ deactivate
    $ conda deactivate


###

- 📚 Sam's other blogs: [Dev.to](https://dev.to/nditah) | [Hashnode.dev](https://nditah.hashnode.dev/) | [Medium.com](https://nditah.medium.com/)
- [Github](https://github.com/Nditah)
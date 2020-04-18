## Plan

- High overview
- Create project
- Server
    - flask application
    - local deployment
    - database
    - docker-compose
    - heroku
- Client
    - react app
    - connect to the server
    - docker-compose
    - netlify
    
    

## Introduction

This series of articles is written to learn how to combine 
Flask and React in order to create the best blog in the world.

There are currently many tutorials about 
Flask, React, Docker, and Heroku. Many of them 
were helpful when I started with this stack. 
But the problem I found is that these tutorials usually concentrate 
only on a small part of the whole process I will talk about.

This tutorial will be about what I wanted to read 
a few months ago when I created my first project with Flask. 
The main goal of this tutorial is to:  

- to show how to develop a full-stack application with React and Flask locally; 
- to practice deploying these kinds of applications on cloud 
services like Heroku and Netlify;



## Architecture overview


First of all, we need to understand what services and technologies we will 
use to know the scope we want to cover. If you see some of these terms the 
first time in your life do not worry, later we will look at all of them in detail.

We will differentiate two configurations: local and production. 

### Local configuration

Requirements to the local configuration:

- it should be available to run locally of course;
- the development process should be comfortable;
- set up of the configuration should not take to much time on a 
new machine (or for new developer);

![13](https://oob-bucket-prod.s3.eu-central-1.amazonaws.com/1/6/00.02.high-overview-local.png)

To achieve these requirements we will use docker to run each 
component of the system (backend, frontend, database,...) in isolated containers. 
This approach will allow run and stop the whole system by a few commands in the 
terminal. Also, such an approach allows running different projects without 
conflicts (like the same ports, environment variables and so on).

### Production configuration

Requirements to the production configuration:

- it should work from the Internet of course;
- we should not spend much money on it (ideally at all);
- updating the system should not take lots of effort;

![12](https://oob-bucket-prod.s3.eu-central-1.amazonaws.com/1/6/high-overview.png)

At the current moment, there are plenty of cloud services that provide a 
possibility to deploy your application. The most famous of them are Amazon AWS, 
Microsoft Asure and Google Cloud. But for my pet projects, I choose Heroku 
as an alternative to these services because of  its nice free plan for small 
projects and simple configuration/deploy process.

Of course, there are limitations and I think main of them that instance 
on Heroku goes to sleep after about half-hour of inactive, but on the 
free plan, you do not have to provide bank card information at all. Anyway, 
I recommend look at comparing these clouds on Stackshare 
[[1]](https://stackshare.io/stackups/google-app-engine-vs-heroku)[[2]](https://stackshare.io/stackups/amazon-ec2-vs-heroku)[[3]](https://stackshare.io/stackups/heroku-vs-microsoft-azure), maybe you will 
decide that you want another cloud and this tutorial is not appropriate.




----------------------------------------------

## Links

* [[1] Google App Engine vs Heroku](https://stackshare.io/stackups/google-app-engine-vs-heroku)
* [[2] Amazon EC2 vs Heroku](https://stackshare.io/stackups/amazon-ec2-vs-heroku)
* [[3] Heroku vs Microsoft Azure](https://stackshare.io/stackups/heroku-vs-microsoft-azure)

-----------------------------------------------

That's an extension to the Miguels's articles and Flask mega tutorial about
hot to deploy an React Flask in the cloud to make it available in the Internet

And how to use docker to make the develoopment process more comfortable

https://blog.miguelgrinberg.com/post/how-to-deploy-a-react--flask-project

## Start server with flask

```shell script
mkdir my-awesome-blog
cd my-awesome-blog
mkdir server
mkdir client
```

```shell script
cd server
python3 -m venv venv
source venv/bin/activate
```

```shell script
pip install flask
```


to be able to export file we need to create module

https://docs.python.org/3/tutorial/modules.html



https://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-i-hello-world

create file server/app/__init__.py
```python
from flask import Flask

app = Flask(__name__)


@app.route('/')
def index():
    return "Welcome to My Awesome Blog"
```

```shell script
flask run
```

Now open http://localhost:5000 and see your working server

```shell script
my-awesome-blog
├── client
└── server
    └── app
        └── __init__.py
```

## Create data base

How to create DB. The easiest way in container!

https://hub.docker.com/_/postgres

```shell script
docker run --name some-postgres -e POSTGRES_PASSWORD=mysecretpassword -d postgres
```

check

```shell script
docker ps
```

And delete

```shell script
docker stop <id>
docker rm <id>
```

my-awesome-blog/docker-compose.yml
```yaml
version: '3.7'

services:
  db:
    image: postgres:12-alpine
    volumes:
      - postgres_data:/var/lib/postgresql/data/
    environment:
      - POSTGRES_USER=oob
      - POSTGRES_PASSWORD=oob
      - POSTGRES_DB=oob
    ports:
      - 5433:5432
```

And we can connect via ide or any other tool

https://blog.timescale.com/tutorials/how-to-install-psql-on-mac-ubuntu-debian-windows/

For example via psql:

```shell script
psql postgresql://mablog@localhost:5433/mablog
```

or

```shell script
docker-compose exec db psql postgresql://mablog@localhost:5432/mablog
```

enter the password `mablog`

and see welcome from database 
```shell script
mablog=# 
```
just type `\q` to exit


```shell script
pip install flask-sqlalchemy
#pip install Flask-Script
pip install flask-migrate
pip freeze > requirements.txt
```

models.py
```python
from app import db


class User(db.Model):
    __tablename__ = 'user'

    id = db.Column(db.BigInteger, primary_key=True)
    username = db.Column(db.String(64), index=True, unique=True)
    email = db.Column(db.String(120), index=True, unique=True)
    password_hash = db.Column(db.String(128))

    def __repr__(self):
        return '<User {}>'.format(self.username)

    @property
    def serialize(self):
        return {
            'id': self.id,
            'username': self.username,
            'email': self.email,
        }
```

\_\_init\_\_.py
```python
from flask import Flask, jsonify
from flask_migrate import Migrate
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'postgresql://mablog:mablog@localhost:5433/mablog'

db = SQLAlchemy(app)

migrate = Migrate(app, db)

from app.models import User


@app.route('/')
def index():
    return "Welcome to My Awesome Blog!!!!!!!!"


if __name__ == "__main__":
    app.run(host="0.0.0.0", debug=True)
```

```shell script
flask db init

pip install psycopg2-binary

flask db migrate -m "first migration"

flask db upgrade
```

Now we cat try to create entity of the user

let run python console 

```shell scirpt
python
```

```shell script
>>> from app import db
>>> from app.models import User
>>> user = User(username="username", email="email", password_hash="password")
>>> db.session.add(user)
>>> db.session.commit()
```

And to check that the user was save in db:

```python
>>> User.query.all()
```


## Run in docker-compose

https://medium.com/@trstringer/debugging-a-python-flask-application-in-a-container-with-docker-compose-fa5be981ec9a

So, we can try to run the app inside container.

To do that first of all we need Dockerfile


```dockerfile
FROM python:3.8.1-slim-buster

RUN apt-get update -y
RUN apt-get install -y python-pip python-dev build-essential

ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

WORKDIR /usr/src/app

RUN pip install --upgrade pip
COPY ./requirements.txt /usr/src/app/requirements.txt
RUN pip install -r requirements.txt

ADD . /usr/src/app/
```

Then we can add new service in docker-compose.yml

```yaml
version: '3.7'

services:
  db:
    image: postgres:12-alpine
    environment:
      - POSTGRES_USER=mablog
      - POSTGRES_PASSWORD=mablog
      - POSTGRES_DB=mablog
    ports:
      - 5433:5432
  web:
    build:
      context: server
      dockerfile: Dockerfile
    volumes:
      - ./server/app:/usr/src/app/app
    command: python app/__init__.py
    ports:
      - 5000:5000
    environment:
      - APP_SETTINGS=config.DevelopmentConfig
      - DATABASE_URL=postgresql://oob:oob@db:5432/oob
      - FLASK_APP=app.py
      - SECRET_KEY=this-is-a-magic-string-and-i-dont-know-why-i-need-it
      - FLASK_DEBUG=true
```

But we have a problem, we have one configuration on the local machine and another
in the docker container.


about entrypoint https://levelup.gitconnected.com/dockerizing-a-flask-application-with-a-postgres-database-b5e5bfc24848

### Add configs
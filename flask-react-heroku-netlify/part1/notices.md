

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
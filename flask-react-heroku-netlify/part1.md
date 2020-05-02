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


First off, we need to understand what services and technologies we will 
use to know the scope we want to cover. If you've never seen these terms in 
your life do not worry, we will look at all of them in detail later.

We will differentiate two configurations: local and production. 

### Local configuration

Requirements to the local configuration:

- It should be available to run locally of course;
- The development process should be comfortable;
- Set up of the configuration should not take too much time on a 
new machine (or for new developer);

![13](https://oob-bucket-prod.s3.eu-central-1.amazonaws.com/1/6/00.02.high-overview-local.png)

To achieve these requirements we will use docker to run each 
component of the system (backend, frontend, database,...) in isolated containers. 
This approach will allow the whole system to run and stop with just a few commands in the 
terminal. Also, this approach allows running different projects without 
conflicts (like the same ports, environment variables and so on).

### Production configuration

Requirements to the production configuration:

- It should work from the Internet;
- We should not spend much money on it (ideally none at all);
- Updating the system should not take lots of effort;

![12](https://oob-bucket-prod.s3.eu-central-1.amazonaws.com/1/6/high-overview.png)

There are plenty of cloud services that provide a 
possibility to deploy your application at the moment. The most popular ones 
are Amazon AWS, Microsoft Asure and Google Cloud. But for my pet projects, I choose Heroku 
as an alternative to these services because of  its nice free plan for small 
projects and simple configuration/deploy processes.

Of course, there are limitations and the biggest problems are the instances 
on Heroku that go to sleep after about half an hour of inactivity. On the 
free plan, you do not have to provide your banking information at all.  
I recommend looking at these cloud comparisons on Stackshare 
[[1]](https://stackshare.io/stackups/google-app-engine-vs-heroku)[[2]](https://stackshare.io/stackups/amazon-ec2-vs-heroku)[[3]](https://stackshare.io/stackups/heroku-vs-microsoft-azure), 
and decide if you want another cloud and if this tutorial is appropriate or not.

We will use the service Netlify where we will publish the built 
React app. It's useful because we can publish any static files there. 
Netlify will serve it inside it's own CDN and it will proxy API requests to the backend.

## Create Flask application

It was quite a long introduction, so let's get started. 
In this section, we will create a simple backend app to check that 
everything works well. 

### Project structure

Let's start from the opening terminal and create a new 
directory for the app:

```shell script
mkdir my-awesome-blog
cd my-awesome-blog
```

In this directory, we will create git repository to store 
the project on GitHub and deploy it in the clouds later:

```shell script
git init
```

You should see something like this:

> Initialized empty Git repository in /Users/obabichev/development/my-awesome-blog/.git/

Directly in the root directory, we will store files related to 
the whole project (like `docker-compose.yml`). 
For the client-side part with React and server-side part with 
Flask, we will create folders `client` and `server` respectively.


```shell script
mkdir server
mkdir client
```

### Python installation

For the next step, you will need to install python 3 on your computer (if it's not installed 
yet). You can check the version of python using the command:

```shell script
python --version
```

If the result looks like this:

> Python 2.7.10

It means that you have python 2. 
There is a chance that you have installed python 3, 
but it is available using the `python3` command. Use the following 
command to check it:

```shell script
python3 --version
```

I got the following result on my computer:

> Python 3.7.7

If you already have installed python 3 you can continue to the next 
step. Otherwise, you will have to install it [[4]](https://www.python.org/downloads/).

### Python environment

We can now open the server directory and create a python virtual 
environment there. I suggest creating a separate environment for 
each python project [[5]](https://docs.python.org/3/library/venv.html). 
All installed packages will be stored in the directory of the local environment 
(`venv` in our case). If you add environment variables 
(like `FLASK_APP` or `DATABASE_URL`) they will not conflict with other 
applications on the computer:

```shell script
cd server
python3 -m venv venv
source venv/bin/activate
```

After executing `source` command `(venv)` should be added to your 
terminal prompt. That means that your terminal session is
connected to this virtual environment and some magic will 
happen (like storing python packages in the `venv` directory)

> (venv) Babichevs-MacBook-Pro:server obabichev$

You can always close the environment using the following command:

```shell script
deactivate
```

### Simple Flask app

It's now time to implement a simple server application. I will not cover all the details 
of Python/Flask development, only the details of running and deploying. 
There is already a Flask Mega Tutorial by Miguel Grinberg [[6]](https://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-i-hello-world) 
that covers many topics related to Flask.

In the Flask Mega Tutorial there is a part about Heroku [[7]](https://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-xviii-deployment-on-heroku), 
but I would like to provide more details. 

From many perspectives, you can think about this article like 
a React extension for Flask Mega Tutorial.

First of all, we need to install Flask:

```shell script
pip install flask
pip freeze > requirements.txt
```

[](WE STOPPED HERE ON THE LAST LESSSON!!)

You can create a `.py` file with any name 
and write a code inside that will create an instance of the Flask server. 
But I used to make it as a separate module to be able 
to import it from anywhere. The main difference now, 
file with the code should be named as `__init__.py`.

Let's create a file `server/app/__init__.py` with the next content:

```python
from flask import Flask

app = Flask(__name__)

    
@app.route('/')
def index():
    return "Welcome to My Awesome Blog"
```

Now you can run this server by the command from `server` directory:

```shell script
flask run
```

And get a similar result:

> * Environment: production
>   WARNING: This is a development server. Do not use it in a production deployment.
>   Use a production WSGI server instead.
> * Debug mode: off
> * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)

Looks like server started without problems and if we open 
URL like this:

```
http://localhost:5000/?name=Oleg
```

We should get result:

> Hallo, Oleg

Your files structure at the moment should look like this:

```shell script
my-awesome-blog
├── client
└── server
    ├── requirements.txt        
    ├── venv
    └── app
        └── __init__.py
```

And the architecture of the app is the simplest at the current moment. We have only a Flask app that we run locally on the port 5000:

![20](https://oob-bucket-prod.s3.eu-central-1.amazonaws.com/1/6/00.03.simple-flask-structure2.png)

#### Add environment in PyCharm

If you use PyCharm and open the project with it you can get a 
small problem that IDE say you `Unresolved reference 'flask'`. 
Here you can say that `I installed it 3 minutes ago` and you 
will be right but IDE knows nothing about it.

![14](https://oob-bucket-prod.s3.eu-central-1.amazonaws.com/1/6/Screenshot_2020-04-24_at_20.33.25.png)

The problem that IDE may use another environment (global one for example) 
and you need to choose the local environment for this project implicitly.

To solve this problem you need to open PyCharm Preferences and find the `Project interpreter` tab:

![15](https://oob-bucket-prod.s3.eu-central-1.amazonaws.com/1/6/Screenshot_2020-04-24_at_20.36.51.png)

On this tab, you need to press the `Add` button (on the right side from the current interpreter) 
and select `bin/python` file from your project interpreter:

![17](https://oob-bucket-prod.s3.eu-central-1.amazonaws.com/1/6/Screenshot_2020-04-24_at_20.40.54.png)

After that IDE will use dependencies from the local environment.


## Adding database

It is a good moment to start to talk about the adding database. 
One of the simplest methods to get a database is just to install 
it on the machine. But actually I already don't remember when I 
was doing it last time. It is not too comfortable because you 
need to install certain DB on you machine, set up users, passwords, 
and so on. What if you need different databases for different 
projects, or share the project with other people. 

And here we come to the using of Docker. If you have never 
tried Docker (or containerization in general) I suggest spending 
time investigating this question cause it will save much time in 
the future (what will make you more productive as a developer).

To make the mental model simpler you can think that on the basic 
level docker operates with images and containers. Where each container 
is a small operation system with its own file system, environment, 
and run applications. And the image is the description of what should 
be inside the container. Docker takes an image as an input and runs 
a container with a certain operating system and content.

You can create an image on your own (and we will do that later) 
or find it on Docker Hub [[8]](https://hub.docker.com/). I will create a container with the 
Postgres database and in order to do that, I will use the official 
image that you can find [here [9]](https://hub.docker.com/_/postgres). There you can find all the details 
about how to use it.

### Run Postgres in docker

If you have no docker on your machine you can download it from the official web site [[10]](https://www.docker.com/get-started). To check 
that it works correctly we can run the command:

```shell script
docker --version
```

And get a result similar to this:

> Docker version 19.03.5, build 633a0ea

I will simply copy the command to create a Postgres container 
from Postgres` Docker Hub page:

```shell script
docker run --name some-postgres -e POSTGRES_PASSWORD=secret -d postgres
 ```

After executing this command docker returned me the next result:

> 590eaa4cfc3ab5e970bb7583f55a1ea287e8e14a2a974ea328c97c51e004dd16

At first glance, it might be looking weird but actually that is 
the id of created container what means that it was created successfully. 
Congratulations, you have your own container with Postgres database 
and if you rerun the last command you will get more, and more, and more...

But now we have a question of how to interact with it. The problem that 
container works inside docker and we have not direct access to it:

![23](https://oob-bucket-prod.s3.eu-central-1.amazonaws.com/1/6/00.04.container-with-postgres2.png)

Of course, docker provides instruments to work with containers. The simple thing we can do 
is to check what containers are working at the current moment:

```shell script
docker ps
```

As you can see on the screenshot I have only one container created some minutes ago:

![24](https://oob-bucket-prod.s3.eu-central-1.amazonaws.com/1/6/Screenshot_2020-04-25_at_11.31.24.png)

In this info, you can see id (short version), name of source image, what command run inside, 
when it was created, used ports, and the name.

```shell script
docker exec -ti 590eaa4cfc3a psql -U postgres
```

Let's look at this command in detail. The command `docker exec` allows 
running any command inside the container (do you remember 
that each container is like a small self-sufficient operation system?).

Flag `-t` says that docker should add `pseudo-TTY`, in other words, 
pseudo-terminal instead of row input (try to run it without this 
flag and you will feel the difference). Flag `-i` means interactive 
and without it, your interaction will be finished after executing 
the initial command (for example you will not be able to 
execute `select 1;` inside psql run in docker) [[11]](https://docs.docker.com/engine/reference/commandline/exec/). 

After then we should put the id of the container and then command we want 
to run. In this case, it is psql command [[12]](https://www.postgresql.org/docs/9.0/app-psql.html).

After that, you should see the prompt of the Postgres database. 
To check that it works we can simply run `select 1;` and get the result:

```shell script
 ?column?
----------
        1
(1 row)
```

It works. You can close psql with the command:

```shell script
\q
```

We know that database is working, but we still have no direct access to it 
from our machine. If you look at the result of `docker ps` one more 
time you will see that the port 5432 (actually DB is working on 
this port) exposed from this container. And you can forward this 
port into your machine. You just need to add the 
flag `-p <port on your machine>:<port on container>` 
to `docker run` command:

```shell script
docker run --name forwarded-postgres -e POSTGRES_PASSWORD
=secret -d -p 5433:5432 postgres
```

If you run `docker ps` you will see that there are already two containers 
(two databases) and port 5432 on one of them forwarded into port 5433 
on your machine (I specially chose different ports to make it clearer, 
but you can use 5432 on both sides):

![25](https://oob-bucket-prod.s3.eu-central-1.amazonaws.com/1/6/Screenshot_2020-04-25_at_12.48.29.png)

Now you can use any Postgres client to connect to the 
DB (psql, pgAdmin, ...). I prefer to use PyCharm possibilities, 
so for me, it looks like this:

![26](https://oob-bucket-prod.s3.eu-central-1.amazonaws.com/1/6/Screenshot_2020-04-25_at_13.38.44.png)


### Remove databases

Now we may have two or more databases and I think we don't want 
to keep them forever. Docker allows without problem to remove 
existing containers. To do that you need firstly stop the container 
and then remove. Bot commands look like:

```shell script
docker stop <container id>
docker rm <container id>
```

You can stop and remove all the containers one by one, but in the 
case when you want to remove all container there is a faster way. 
You can run the command `docker ps -q` which will return only ids 
of containers (but only of running containers) and 
command `docker ps -q -a` (flag `-a` means all) which will return ids 
of all containers (even stopped).

With this knowledge we can stop all the containers with the command:

```shell script
# be sure that you use ` but not ' or "
docker stop `docker ps -q` 
```

And then remove all stopped containers:

```shell script
docker rm `docker ps -a -q`
```

## Connect server to database

Here I suggest remove all containers if you were playing with 
its creating in the previous part and create new one by the next command:

```shell script
docker run --name forwarded-postgres -e POSTGRES_USER=mablog -e POSTGRES_PASSWORD=mablog -e POSTGRES_DB=mablog -d -p 5433:5432 postgres
```

The main difference that here I use the word `mablog` as DB name, 
user name, and user password for the new database.

In this part, we will connect the server that runs on the host 
machine to the database in docker:

![27](https://oob-bucket-prod.s3.eu-central-1.amazonaws.com/1/6/00.05.flask-with-postgres.png)

I suggest looking at how to work with the database in Flask mega tutorial [[13]](https://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-iv-database) for more details 
but the main steps will be covered here also.

First of all, we need to define our model and create an instance of SQLAlchemy. 
Let's start from installing `flask-sqlalchemy`:

```shell script
pip install flask-sqlalchemy
pip freeze > requirements.txt
```

And creating of file `server/app/models.py` with the next content:

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

As you can see here is a simple model for `user` table with 
columns `id`, `username`, `email`, and `password`, but it will be 
enough for testing and implementing authorization. If you see 
errors about import `db` don't worry, we will fix it.

To know how to connect to the database SQLAlchemy requires path 
and credentials to this database. To do that we can add 
parameter `SQLALCHEMY_DATABASE_URI` with the full address to the DB 
we created in docker.  Let's open file `server/__init__.py` and right 
after creating `app` (`app = Flask(__name__)`) add:

```python
app.config['SQLALCHEMY_DATABASE_URI'] = 'postgresql://mablog:mablog@localhost:5434/mablog'
```

In this url you can see that we use login and password `mablog` (`mablog:mablog`), 
port 5433 and dbname `mablog`. After that we can create instance of SQLAlchemy
and import `User` model:

```python
db = SQLAlchemy(app)

from app.models import User
```

You can run the app but actually nothing will happen because firstly 
we don't use this model and secondly this model is not applied to 
the database. We have only python code that describes the model 
we want, but Postgres DB knows nothing about it. Let's about the second 
problem in detail.

### About imports in PyCharm

if you open in PyCharm not the directory server, but the whole 
project (directory `my-awesome-blog`) like I, you can get an error 
with import like this:

![28](https://oob-bucket-prod.s3.eu-central-1.amazonaws.com/1/6/Screenshot_2020-04-25_at_17.39.55.png)

It happens because IDE thinks that content root in the folder 
you initially opened, but we actually write the code with the 
assumption that content root in the `server` folder.

So everything we need is to change the content root in the project structure. 
Open `Project Structure` tab in preferences, remove the current 
content root and press `Add Content Root`:

![29](https://oob-bucket-prod.s3.eu-central-1.amazonaws.com/1/6/Screenshot_2020-04-25_at_19.04.33.png)

And in the appeared window choose `server` directory:

![30](https://oob-bucket-prod.s3.eu-central-1.amazonaws.com/1/6/Screenshot_2020-04-25_at_19.06.28.png)

After that IDE should work with imports respectively our expectations.

### Migrations

Why do we need migrations? Actually it is not a big deal to create a 
database with certain structure, but the problem is to change the database 
structure when your app already launched and there are user's 
data you don't want to lose.

For example, imagine that you have a table `posts` in the database, 
users already started to generate content and everything works well. 
Then you want to add a new column to this table (for example `language`), 
but you can not drop the whole database and create new one, you have 
to write something we will call migration that will change the existing 
database for the new functionality.

In the Flask there is a special library to generate and run migrations. 
Let's install the required packages:

```shell script
pip install Flask-Script
pip install flask-migrate
pip freeze > requirements.txt
```

Also, I prefer to write code for migrations separately from the 
project code and fortunately, in the Flask there is a quite standard 
way by creating `manage.py` file. Let's create the 
file `server/manage.py` with the next content:

```python
from flask_script import Manager
from flask_migrate import Migrate, MigrateCommand

from app import app, db

migrate = Migrate(app, db)
manager = Manager(app)

manager.add_command('db', MigrateCommand)

if __name__ == '__main__':
    manager.run()
```

It is a useful thing because this code is not a part of the 
launched server, but here you can import an app instance and use it. 
In other words, the suggestion is to move in this file everything 
you need to run independently from the normal server life cycle. 
It might be migrations, any scripts to generate code, create seed 
data, and other tasks.

You can see that in this case, we create a migrate instance that 
allows us to generate and run migrations and register it as a 'db' command.

Now we can start creating migrations. And the first command 
we need to run is `db init`:

```shell script
python manage.py db init
```

Here you can see that we simply run the file manage.py in python, 
add `db` argument in order to say to the manager that we want 
to run command registered as `db` and then arguments of this 
command (here only `init` word). You should get the result like this:

>  Creating directory /Users/obabichev/development/my-awesome-blog/server/migrations ...  done
>  Creating directory /Users/obabichev/development/my-awesome-blog/server/migrations/versions ...  done
>  Generating /Users/obabichev/development/my-awesome-blog/server/migrations/script.py.mako ...  done
>  Generating /Users/obabichev/development/my-awesome-blog/server/migrations/env.py ...  done
>  Generating /Users/obabichev/development/my-awesome-blog/server/migrations/README ...  done
>  Generating /Users/obabichev/development/my-awesome-blog/server/migrations/alembic.ini ...  done
>  Please edit configuration/connection/logging settings in '/Users/obabichev/development/my-awesome-
>  blog/server/migrations/alembic.ini' before proceeding.


And find a new directory in the project called `migrations`. You need 
to run the command db init` only once for the project to set up 
configurations and helping scripts for migrations.

The next command we need is `db migrate`. The main thing this 
command does is the comparing of the model described in python files 
and the actual state of the database. In the case of differences, it 
will create a new file with the code that will apply changes to the 
database to make data model and database structure equivalent.
So this command already requires a working database to make comparisons.
We need to run the next command:

```shell script
python manage.py db migrate -m "Users table"
```

If you got an error with the text `ModuleNotFoundError: No module named 'psycopg2'` 
I suggest to install package `psycopg2-binary` instead of `psycopg2` because in this 
case you will get already built package (and avoid the building of it on your machine what 
requires additional prerequisites [[14]](https://pypi.org/project/psycopg2-binary/)). 


```shell script
pip install psycopg2-binary
pip freeze > requirements.txt
```

Restart `db migrate` command if you had to install psycopg2. 
Finally, you should get a message like this:

> INFO  [alembic.runtime.migration] Context impl PostgresqlImpl.
> INFO  [alembic.runtime.migration] Will assume transactional DDL.
> INFO  [alembic.autogenerate.compare] Detected added table 'user'
> INFO  [alembic.autogenerate.compare] Detected added index 'ix_user_email' on '['email']'
> INFO  [alembic.autogenerate.compare] Detected added index 'ix_user_username' on '['username']'
>   Generating /Users/obabichev/development/my-awesome-blog/server/migrations/versions/f62579cae873_users_table.py
>   ...  done

Now you should see a new file in the directory `migrations/versions`. 
In my case, it is `f62579cae873_users_table.py`. If you open this 
file you will find some constants and two methods there.

The constant `revision` is the unique identification of 
the migration. The constant `down_revision` references to the 
migration that should be applied before this migration (and it 
is `null` for the first migration). These two constants are needed 
to know the order of the migrations. Also `alembic` (the library 
that actually applies migrations under the hood) keeps id of the 
last applied migration in the database what allows it to know 
should be any migrations applied or not.

And the main part of the migrations is the methods `upgrade` 
and `downgrade`. The first one will be called when alembic will 
apply this migration and usually, this method adds changes to the 
database structure. The second one will be applied when you 
will try to rollback this migration.

Now we have migration, so we can apply it to the database:

```shell script
python manage.py db upgrade
```

I got the next result that is saying that new migration was applied:

> INFO  [alembic.runtime.migration] Context impl PostgresqlImpl.
> INFO  [alembic.runtime.migration] Will assume transactional DDL.
> INFO  [alembic.runtime.migration] Running upgrade  -> f62579cae873, Users table

You can check the database and find there the table for users. 
You also should find the table `alembic_version` where the id of 
last applied migration will be stored:

![32](https://oob-bucket-prod.s3.eu-central-1.amazonaws.com/1/6/Screenshot_2020-04-26_at_09.24.07.png)

If you want rollback the changes you just need to run the command:

```shell script
python manage.py db downgrade
```

### About deprecated functionality

If you saw during this part error in the console like this:

```shell script
SQLALCHEMY_TRACK_MODIFICATIONS adds significant overhead and will be disabled by default in the future.  
Set it to True or False to suppress this warning. 
```

You can read it and notice that SQLAlchemy suggest you disable the 
feature that will be disabled by default in the future.

You can read it and notice that SQLAlchemy suggests you 
implicitly disable or enable this feature. So, all we need is 
just to put the right config property in the app. Let's add 
the next row in `__init.py__.py` right after creating an app instance:

```python
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
```

Now your `__init__.py` should looks like this:

```python
from flask import Flask, request
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'postgresql://mablog:mablog@localhost:5433/mablog'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False

db = SQLAlchemy(app)
from app.models import User


@app.route('/')
def index():
    name = request.args.get('name')
    return "Hallo, {}".format(name)
```

And the files structure like this:

```shell script
.
├── client
└── server
    ├── app
    │   ├── __init__.py
    │   └── models.py
    ├── manage.py
    ├── migrations
    │   ├── README
    │   ├── alembic.ini
    │   ├── env.py
    │   ├── script.py.mako
    │   └── versions
    │       └── f62579cae873_users_table.py
    └── requirements.txt
```

## Authorization

We already have a working database and configured SQLAlchemy 
to work with it, but actually we don't use the database 
for something useful. So before we move to the 
dockerizing Flask app I will create some authorization 
services. Usually, it is the first thing I implement in my 
applications. If you don't need authorization or don't 
want to spend time for it now you can move to the next 
part, but I expect that I will use these services from the 
React app.

In general, the authorization will be pretty simple. 
We need 3 services: register, login, and get the user. 
When the register service is called we need to create new 
instance of the User model and save it in the database after 
that set the cookies (to keep information about this user 
among the next request) and return the user object to 
the callee. Similar thing for the login service, but 
we need to check the password instead of creating a new 
user. And one more service to get the current user 
information (based on cookies), that will allow 
the frontend to check is the user authorized:

![34](https://oob-bucket-prod.s3.eu-central-1.amazonaws.com/1/6/00.06.auth-services2.png)

In this part, we will write a bit more code than in previous 
but in the end, we will have working authorization. 
Of course, it will not be implemented from scratch, 
we will use the library `flask_login`, 
I recommend to look at the related article in the mega 
tutorial [[15]](https://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-v-user-logins). 
Let's start with installing this library:

```shell script
pip install Flask-Login
pip freeze > requirements.txt
```

Firstly we will update our `User` model. 
The main thing Flask Login will do for us is to keep 
the information about a logged user in the cookies, 
but by default, Flask Login knows nothing about our 
data model. In the documentation [] written that the 
User model should implement methods `is_authenticated`, 
`is_active`, `is_anonymous`, `get_id`, and until we need 
a custom implementation of these methods we can 
use `UserMixin` with simple implementation.

Let's open `models.py` and add `UserMixin` to the `User` class:

```python
class User(UserMixin, db.Model):
```

Also, we know that Flask Login stores the only 
id of the user in the cookies, and it is necessary to 
provide the library a method to get the user entity by id. 
We can do that in a next way:

```python
@login.user_loader
def load_user(user_id):
    return User.query.get(int(user_id))
```

Also, I will add two helper methods into the `User` class 
to work with passwords. One of these methods will convert 
a password into hash and store in the user entity, and the 
next one will take a string as argument and check is it the 
right password or not. After that the whole content of 
`models.py` should be like this:

```python
from flask_login import UserMixin
from werkzeug.security import generate_password_hash, check_password_hash

from app import db
from app import login


class User(UserMixin, db.Model):
    __tablename__ = 'user'

    id = db.Column(db.BigInteger, primary_key=True)
    username = db.Column(db.String(64), index=True, unique=True)
    email = db.Column(db.String(120), index=True, unique=True)
    password_hash = db.Column(db.String(128))

    def __repr__(self):
        return '<User {}>'.format(self.username)

    def set_password(self, password):
        self.password_hash = generate_password_hash(password)

    def check_password(self, password):
        return check_password_hash(self.password_hash, password)

    @property
    def serialize(self):
        return {
            'id': self.id,
            'username': self.username,
            'email': self.email,
        }


@login.user_loader
def load_user(user_id):
    return User.query.get(int(user_id))

```

For the services, I will create the separate file `app/routes.py`. As I already told above we need three services: 
- `/register` should take `{username, email, password}` JSON, 
create a new user, and save it. After that, it should 
call `login_user(user)` (in the code I will also 
set `remember` to `True`, but you can take value 
for it from UI) to say Flask Login that this user 
should be saved in the cookies, and then return a JSON 
with the user as a result;
- `/login` should take `{email, password}` JSON, 
try to find the user with this email in the DB, check 
the password, and if everything is ok to call 
`login_user(user)` and return user in JSON;
- `/user` should return JSON with the authorized user. 
In the request, the information about the user is in 
cookies. Flask Login knows how to take this information 
from cookies (first of all because this library saved 
it there), we already create methods for Flask Login that 
helps it to find the user in DB, and the library provides 
found user with `current_user` variable:

```python
from flask import request, jsonify
from flask_login import login_user, current_user

from app import app, db
from app.models import User


@app.route('/api/register', methods=['POST'])
def register():
    content = request.get_json()
    user = User(username=content['username'],
                email=content['email'])
    user.set_password(content['password'])
    db.session.add(user)
    db.session.commit()
    login_user(user, remember=True)
    return jsonify(user.serialize)


@app.route('/api/login', methods=['POST'])
def login():
    content = request.get_json()

    user = User.query.filter_by(email=content['email']).first()
    if user is None or not user.check_password(content['password']):
        return jsonify({'message': 'Wrong credentials'}), 400

    login_user(user, remember=True)
    return jsonify(user.serialize)


@app.route('/api/user')
def get_current_user():
    if current_user.is_authenticated:
        return jsonify(current_user.serialize)
    return jsonify(None)
```

After creating the file with routes I can update `__init.py__` file to make the 
app working. I removed the `index` route I added some time ago and added imports 
of new files. This file has the following content now: 

```python
from flask import Flask
from flask_login import LoginManager
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'postgresql://mablog:mablog@localhost:5434/mablog'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
app.config['SECRET_KEY'] = 'this secret should not be in the code and we will remove it from here'

db = SQLAlchemy(app)
login = LoginManager(app)

from app import models
from app import routes
```

Now we can run the app and check that everything works.

### Test services with Postman

When you have JSON API it is getting more complicated 
to test services. For simple requests, the terminal application `curl` might 
be used, but when you have lots of services (and many of them have POST 
or PUT methods) it might be not too useful. I prefer to use Postman 
application [[17]](https://www.postman.com/) where you can create and call 
requests, and organize them with folders. For example, I can check 
the `register` service as follows:

![36](https://oob-bucket-prod.s3.eu-central-1.amazonaws.com/1/6/Screenshot_2020-05-02_at_11.02.43.png)

If you don't want to install Postman and just need to 
check that everything works well you can use the `curl` command:

```shell script
curl --header "Content-Type: application/json" --request POST --data '{"username": "test user 2","email": "test2@test.test","password": "test"}' http://localhost:5000/api/register
```

## Dockerize Flask app


### Dockerfile

## Links

* [[1] Google App Engine vs Heroku](https://stackshare.io/stackups/google-app-engine-vs-heroku)
* [[2] Amazon EC2 vs Heroku](https://stackshare.io/stackups/amazon-ec2-vs-heroku)
* [[3] Heroku vs Microsoft Azure](https://stackshare.io/stackups/heroku-vs-microsoft-azure)
* [[4] Install Python 3](https://www.python.org/downloads/)
* [[5] Creation of virtual environments](https://docs.python.org/3/library/venv.html)
* [[6] Flask Mega Tutorial](https://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-i-hello-world)
* [[7] The Flask Mega-Tutorial Part XVIII: Deployment on Heroku](https://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-xviii-deployment-on-heroku)
* [[8] Docker Hub](https://hub.docker.com/)
* [[9] Postgres image on Docker Hub](https://hub.docker.com/_/postgres)
* [[10] Get Started with Docker](https://www.docker.com/get-started)
* [[11] Docker exec](https://docs.docker.com/engine/reference/commandline/exec/)
* [[12] psql documentation](https://www.postgresql.org/docs/9.0/app-psql.html)
* [[13] The Flask Mega-Tutorial Part IV: Database](https://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-iv-database)
* [[14] PyPi: psycopg2-binary](https://pypi.org/project/psycopg2-binary/)
* [[15] The Flask Mega-Tutorial Part V: User Login](https://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-v-user-logins)
* [[16] Flask-Login](https://flask-login.readthedocs.io/en/latest/)
* [[17] Postman](https://www.postman.com/)
* [[] Installing psycopg2-binary with Python:3.6.4-alpine doesn't work #684](https://github.com/psycopg/psycopg2/issues/684)
* [[] Build and run your image](https://docs.docker.com/get-started/part2/)
* [[] docker run (command line reference)](https://docs.docker.com/engine/reference/commandline/run/)
* [[] About Alpine Linux](https://alpinelinux.org/about/)



   

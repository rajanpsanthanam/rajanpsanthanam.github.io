---
layout: post
title:  "load testing with locust"
date:   2016-12-18
categories: python
---
Before I start, let me give a brief on what this post will cover. In this post, we are going to write a small `flask` application
which will expose a post endpoint to create users and get endpoint to list users. We will use `locust` to do load testing on the application. By load testing I mean we can stimulate users to hit those endpoint simultaneously and see how the server responds.

The scope of this post ends in setting up the stage, once we are ready with the files we should deploy it in the actuall server and start testing it against the server configuration. Offcourse this makes sense to do with real world application. This post will give you an idea on where to start.

Before we start writing the code, we have some pre requisites to satisfy.

We need `mongodb` to persist the data. You can install mongodb referring the official [docs][mongo-docs].

Once mongodb is installed successfully, we can start the server by

{% highlight python %}

mongod

{% endhighlight %}

We need a virtual environment to install the application dependencies. You can do the same by trying the below commands
on your terminal.

{% highlight python %}

# create requirements file
touch requirements.txt

# copy the below contents into requirements.txt
flask
pymongo
flask-pymongo
locustio
faker

# create virtualenv
virtualenv venv

# activate virtualenv
source venv/bin/activate

# install the requirements
pip install -r requirements.txt


{% endhighlight %}

Once the requirements are installed, you can try `pip list` to see what packages are installed.

Lets write the application now, create `application.py` file and copy the below contents.

{% highlight python %}
# application.py

from flask import Flask
from flask import jsonify
from flask import request
from flask_pymongo import PyMongo

app = Flask(__name__)

app.config['MONGO_DBNAME'] = 'userdb'
app.config['MONGO_URI'] = 'mongodb://localhost:27017/userdb'

mongo = PyMongo(app)

@app.route('/users', methods=['GET'])
def list():
  output = []
  for user in mongo.db.users.find():
    output.append({'name' : user['name'], 'email' : user['email']})
  return jsonify({'result' : output})

@app.route('/users/<name>', methods=['GET'])
def retrieve(name):
  user = mongo.db.users.find_one({'name' : name})
  output = {'name' : user['name'], 'email' : user['email']} if user else {}
  return jsonify({'result' : output})

@app.route('/users/', methods=['POST'])
def create():
  users = mongo.db.users
  name = request.json['name']
  email = request.json['email']
  user_id = users.insert({'name': name, 'email': email})
  user = users.find_one({'_id': user_id })
  output = {'name' : user['name'], 'email' : user['email']}
  return jsonify({'result' : output})

app.run(debug=True)

{% endhighlight %}

You can now run the flask application. Make sure, the virtualenv is active while running the application.

{% highlight python %}

python application.py

{% endhighlight %}

By default flask application will run in port 5000. We can test the application by creating and listing users. You can use applications like `postman` to do this or simply use curl for the purpose.

{% highlight python %}

# create users
curl -XPOST -H "Content-Type: application/json" http://localhost:5000/users/ -d '{"name": "test", "email": "test@test.com"}'

# get list of users
curl -XGET http://localhost:5000/users

{% endhighlight %}

Now that we have a working application, we can write locustfile to test the application.

You can read about locust from the official [docs][locust-docs]

{% highlight python %}
# locustfile.py
import json
from locust import HttpLocust, TaskSet, task
from faker import Faker
fake = Faker()

class UserTaskSet(TaskSet):

    @task(1)
    def get_users(self):
        response = self.client.get("/users")
        print('Response status code: {code}'.format(code=response.status_code))

    @task(3)
    def create_user(self):
        data = {
            "name": fake.name(),
            "email": fake.email()
        }
        headers = {'content-type': 'application/json'}
        response = self.client.post("/users/", data=json.dumps(data), headers=headers)
        print('Response status code: {code}'.format(code=response.status_code))

class UserLocust(HttpLocust):
    task_set = UserTaskSet
    host = 'http://localhost:5000'
    min_wait = 5000
    max_wait = 15000

{% endhighlight %}

Each method in the TaskSet subclass is a task which we want to perform. We can provide relative ratio's on which task should be triggered. Here I have given 1:3 ratio, hence 3 create task will be triggered per list task.

Now run the locust server using the below command. If the file is not named as locustfile.py which is the default filename, use -f flag to point the file.

{% highlight python %}

locust

{% endhighlight %}

Locust server will run in 8089 port by default. We can now stimulate users and frequency of requests using locust. We can gradually increase the load and see where the application breaks.

[mongo-docs]: https://docs.mongodb.com/manual/installation/
[locust-docs]: http://docs.locust.io/en/latest/

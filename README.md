<!-- {% raw %} -->
<img src="https://i.imgur.com/iohZqCp.png">

# Django Setup, URLs, and Views

## Road Map

1. Learning Django Game Plan
2. Review the Request/Response Cycle
3. Start the Cat Collector Project
4. Defining Routes (URLs)
5. Defining View Functions
6. Summary
7. Labs for Cat Collector Lessons

## 1. Learning Django Game Plan

We've got a great set of Django lessons coming your way! The remaining lectures this week are 100% Django.

The lectures will add features, piece-by-piece, to a modern full-stack reference app named **CatCollector**.

Let me show you the [**final version**](https://github.com/RaulJiminian/django-catcollector-final) we’re going to build this week.

Then, after the lessons, you will use lab time to repeat what you saw in the lesson by building your own app named anything you want, say - **FinchCollector**.

Here’s an overview of the high-level topics we’ll be covering this week, in order:

1. Django Setup, URLs, and Views
2. Data Models, Serializers, and Migrations
3. Django Class-based Views
4. One-to-Many Models
5. Details and a Rep
6. Many-to-Many Models
7. Django Authentication
8. Uploading Images to the Cloud via AWS S3 Buckets (Bonus content)

Let’s begin with part 1!

## 2. Review the Request/Response Cycle in Django

In Unit 2, we learned that in a full-stack web application:

- Clicking links and submitting forms on the front-end (browser) sends HTTP requests to the web app running on a web server.
- The web server has a routing mechanism that matches HTTP requests to code.
- That code typically performs CRUD, then either:
    - Renders dynamic templates for Read data operations.
    - Redirects the browser in the case of Create, Update or Delete data operations.

Once again, let’s review this diagram that shows how a request flows through a Django project:

![MVT Framework](./mvt-pattern.png)

### Minimalist vs. Full-featured Frameworks

#### Review The Philosophy of Express

Express was a minimalist framework that didn’t provide much functionality out of the box.

It gave us a way to define routes, map controller actions to those routes, and render dynamic views.

Express didn’t have many rules, for example, we could name files anything and put them anywhere we wanted.

If we did need additional capability, it usually meant installing and configuring additional middleware.

#### The Philosophy of Django

Unlike Express, Django, is a full-featured web framework that provides a lot of built-in functionality.

However, Django has many *conventions*, and it expects us to follow its rules.

You will find that Django has all sorts of *helper* classes, methods, etc.

What Express has to offer can be grasped in a matter of days, whereas Django could take months to feel comfortable with what it has to offer.

Luckily for us though, the basics aren’t too bad though, as long as you don’t try to learn every little detail about each helper, etc.

## 3. Start the **Cat Collector** Project

Let's begin with an initial setup. First create a directory inside of your lessons folder called **django-catcollector**.

CD into that directory and set up your virtual environment

```
pipenv shell
```

Next, install the necessary dependencies

```
pipenv install django psycopg2-binary djangorestframework
```

Open the project folder in VS Code:

```
code .
```

### Create the database

Databases are not automatically created by Django, so let's create one.

Create (touch) a file called `create-database.sql` and fill it with the following:

```sql
CREATE DATABASE catcollector;

CREATE USER cat_admin WITH PASSWORD 'password';

GRANT ALL PRIVILEGES ON DATABASE catcollector TO cat_admin;

```

We can execute that script by running `psql -f create-database.sql` from the
command line.

### Start the Project

Let's put in a 'rep' creating a Django project/app!

Run the following command to create the Django project (**don't forget the dot**):

```
django-admin startproject catcollector .
```

The above command generated and configured a Django project named **catcollector**.

### Create the App

A Django _project_ contains Django _apps_. Apps represent major functionality in a project.

For Cat Collector, you will need an app to implement the main functionality, in this case collecting cats 😸

It makes sense to name the main app generically, so let's do it. Please make sure you are in your virtual environment before running the following command:

```
python manage.py startapp main_app
```

You'll now find a **main_app** folder within the top-level project folder. That folder has been configured to be a Python package - which is a way to organize modules.

Let's include it as part of the Cat Collector project by adding it to the `INSTALLED_APPS` in **settings.py** (let's take advantage and add the rest_framework to installed apps as well):

```python
INSTALLED_APPS = [
	'django.contrib.admin',
	'django.contrib.auth',
	'django.contrib.contenttypes',
	'django.contrib.sessions',
	'django.contrib.messages',
	'django.contrib.staticfiles',
  	'rest_framework',
  	'main_app',
]
```

Let's check to make sure the project starts up:

```
python manage.py runserver
```

Ignore the red message about unapplied migrations, we'll take care of those in a bit.

Browse to `localhost:8000` and make sure you see the rocket on the page:

<img src="https://i.imgur.com/RozMgJ0.png">

### Connecting to the Database

Earlier, we created a dedicated `catcollector` PostgreSQL database. A Django project's configuration lives in **settings.py**. Let's update it to use our `catcollector` database:

**Note:** Some students may have to include the following fields(HOST, USER, PASSWORD), it depends how your Postgres is setup locally. For example, the most common setup for Windows and Linux users may require you to include these three fields to properly connect to your local Postgres instance. 

> 👀 You can also add the 'PORT' property (see below) to change the port to something other than :8000

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'catcollector',
        # 'HOST': 'localhost',  <-- (optional) some computers might need this line
        # 'USER': 'cat_admin', <-- (optional) postgres user name, if you have to sign into an account to open psql, you will want to add that user name here.
        # 'PASSWORD': 'password', <-- (optional) postgres user password, if you have to sign into an account to open psql, you will want to add that user password here.
        # 'PORT': 3000 <-- if you desire to use a port other than 8000, you can change that here to any valid port id, some number between 1 and 65535 that isn't in use by some other process on your machine. The reason for this port number range is because of how TCP/IP works, a TCP/IP protocol network(the most widely used protocol used on the web) allocated 16 bits for port numbers. This means that number must be greater than 0 and less than 2^15 -1. 
    }
}
```

> 👀 By default, Django uses SQLite, a lightweight database that is not recommended for deployment. 

You may be seeing some red text about unapplied migrations in your terminal if your project successfully connected to the `catcollector` database. Let's explain a little more about that and get that cleared up. 

### Migrating the Pending Migrations

We use migrations to update the database's schema over time to meet the project's needs.

There are several migrations **pending** (i.e., *waiting to be applied to the database*) - so let's apply them:

```
python manage.py migrate
```

You should see something similar to this:

![migrations](./migrations.png)

😄 Nice - no more pending migrations! We will cover migrations in more detail in the next lesson.

## 4. Defining Routes (URLs)

Just like with Express, there needs to be a route defined that matches each HTTP request coming from the browser.

<details>
<summary>
❓ What kind of error will we receive when there is no route defined that matches a given HTTP request? 
</summary>
<hr>

**404 Not Found**

<hr>
</details>

But, let's not forget that Django's routing system only cares about the URL ("path") of the request and ignores the HTTP method.

Let's work on our initial url setup next.

### One-time URL Setup

In Django, routes are defined within modules named ***urls.py***.

There's an existing project ***catcollector/urls.py*** that we could add additional routes to, however it is *best practice* for each Django app to define it's own routes and to ***include*** those URLs in the project.

> 👀 There are some helpful comments at the top of ***catcollector/urls.py***.

Start by setting up ***main_app***'s ***urls.py*** file. But wait...there's not one:

1. Create the **urls.py** module:

	```
	touch main_app/urls.py
	```

2. Let's *include* it in the project's urls file - **catcollector/urls.py**

	```python
	from django.contrib import admin
	# Add the include function to the import
	from django.urls import path, include
	
	urlpatterns = [
	    path('admin/', admin.site.urls),
	    # '' represents the "starts with" path
	    path('', include('main_app.urls')),
	]
	```


	> 👀 Be sure to import the `include` function near the top.
	
	Each item in the `urlpatterns` list defines a URL-based route or, as in the case above, mounts the routes contained in another urls.py module.
	
	Similar to how Express appends paths defined in a router module to the path in `app.use`, the paths defined in `'main_app.urls'` will be appended to the path specified in the `include` function.
	
	You can now close ***catcollector/urls.py***, since all routes we define from this point forward will be defined within ***main_app/urls.py*** - until we get to authentication that is.

3. Now for the boilerplate needed in ***main_app/urls.py***:

	```python
	from django.urls import path
	
	urlpatterns = [
	
	]
	```

	Notice that we've imported the `path` function that will be used to define each route. 
	
	We've also created the required `urlpatterns` list which will hold each route we define for `main_app`.

### Define `main_app`'s Home Page URL

With the setup done, we're ready to define the route to return our first response when a user hits our home route.

In **main_app/urls.py**:

```python
# import Home view from the views file
from .views import Home

urlpatterns = [
  path('', Home.as_view(), name='home'),
]
```

The above code defines a *root* route using an **empty string** and maps it to the `views.home` view function that does not exist yet - making the server unhappy.

The `name='home'` kwarg gives the route a name.  Naming a route is optional, but is considered a best practice.

The Home route has been defined!  On to the view...

## 5. Defining View Functions

<details>
<summary>
❓ What is the equivalent to a Django View Function in Express?
</summary>
<hr>

**Controller Function**

<hr>
</details>

In the Home route we referenced a *view* function named `home`.

We will define all of the app's views in **main_app/views.py**.

Let's define the `home` view function and respond with JSON (since we are building a RESTful API):

```python
from rest_framework.views import APIView
from rest_framework.response import Response

# Define the home view
class Home(APIView):
  def get(self, request):
    content = {'message': 'Welcome to the cat-collector api home route!'}
    return Response(content)
```

**A few things to keep in mind** - We have removed the render import, which traditionally only serves HTML templates, from the original code in the views file with imports from django rest framework:

**APIView:**

- APIView is a class provided by Django Rest Framework that is used as a base for all views in DRF.
- It extends Django's basic View class and provides additional functionality that is specific to working with APIs.
- Using APIView allows you to handle different HTTP methods (GET, POST, etc.) with class methods and provides features like authentication, permissions, and throttling.

**Response:**

- Response is a class in DRF that is used to return responses from your API views.
- It extends the basic HttpResponse from Django but is specifically tailored for returning data in various formats (like JSON, XML, etc.).
- DRF's Response class also handles content negotiation based on the client's request, meaning it can automatically adjust the response format (JSON, HTML, etc.) based on what the client can accept.

## 6. Summary

You now have a minimal (but functional) application setup that responds with JSON.

You now know pretty much *all there is to know* about the structure of a Django app!

In the next lesson, we'll learn about Django Models where we'll code a `Cat` Model and use it to "CRUD" 😸😸😸 inside of the database!

## 7. Labs for Cat Collector Lessons

In this unit, the goal of each Django lab will be to repeat everything we've done in the lesson -- except you'll "collect" something new (like Finches) and call the project something like ***finchcollector***, or whatever you are collecting 😀

The final version of your "______ Collector" **will be** a deliverable.

Because your completed *Collector* project will be fairly comprehensive, makes it a great candidate for an addition to your portfolio.

Be sure to create your project within your **~/projects** folder and push it to a repo in your personal GitHub account.

<!-- {% endraw %} -->

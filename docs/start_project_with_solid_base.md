
### Starting a DRF API

- **Create a virtual environment**:
  ```
  python -m venv your_environment_name
  ```

- **Activate your virtual environment**:
  ```
  source your_environment_name/bin/activate
  ```

- **Install Django and Django REST Framework**:
  ```
  pip install django djangorestframework
  ```

- **Start your project**:
  ```
  django-admin startproject your_project_name
  ```

- **Navigate to your project directory**:
  ```
  cd your_project_name
  ```

You will notice a directory with the name you chose. Inside, you will find several important files for your application:

```
your_project_name/
    __init__.py
    settings.py
    asgi.py
    urls.py
    wsgi.py
```

Here's a brief overview of each file:
1. **settings.py**: Configures your Django project (database, apps, middleware, etc.).
2. **asgi.py**: Enables asynchronous communication with web servers.
3. **urls.py**: Maps URLs to views, directing web traffic in your project.
4. **wsgi.py**: Facilitates synchronous communication with web servers for deployment.

### Continuing the Project

To start an app within your project, ensure you're in the project directory and run:

```
django-admin startapp your_app_name
```

Your directory structure should now include:

```
your_project_name/
    __init__.py
    settings.py
    asgi.py
    urls.py
    wsgi.py
your_app_name/
    __pycache__/
    migrations/
    __init__.py
    admin.py
    apps.py
    models.py
    tests.py
    urls.py
    views.py
```

> PS: It's good practice to create multiple apps as necessary to ensure better readability and code management.

### Linking the App to Your Project

Open the `settings.py` file in your project directory and look for the `INSTALLED_APPS` list. Here, add `'rest_framework'` and your app's name (`'your_app_name'`) to register them with your project:

```
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'rest_framework',
    'your_app_name',
]
```

### Enhancing Your App

You may need to create additional files within your app (if they aren't already present):

- **serializers.py**
- **urls.py**
- **filters.py**

```
your_app_name/
    __pycache__/
    migrations/
    __init__.py
    admin.py
    apps.py
    models.py
    tests.py
    urls.py
    views.py
    serializers.py
```

Brief file overviews:

1. **admin.py**: Registers models with the Django admin interface.
2. **apps.py**: Configures application-specific settings.
3. **models.py**: Defines data models (database schema).
4. **tests.py**: Contains application tests.
5. **urls.py**: Maps URLs to views.
6. **views.py**: Handles requests and returns responses.
7. **serializers.py**: Converts data for API handling.

### Setting Up Routers

Use the `DefaultRouter` from DRF for benefits like:
- **Auto-generating CRUD URLs**.
- **Out-of-the-box CRUD functionality**.
- **Browsable API interface**.

Configure the app's `urls.py` and the project's `urls.py` as follows:

> **App `urls.py`**:
```python
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from .views import ViewSetTest

app_name = 'your_app_name'

router = DefaultRouter()
router.register(r'url-path-name', ViewSetTest)

urlpatterns = router.urls
```

> **Project `urls.py`**:
```python
from django.urls import path, include

urlpatterns = [
    ...
    path("api/v1/your_app_name", include("your_app_name.urls")),
    ...
]
```

Now, if you run your app, you'll see the routes you've created.

```
python manage.py runserver
```

### Further Steps

- Transform `settings.py` into a directory to separate development and production settings.
- Use `.env` files and `django-environ` for environment variables.
- Install the `django-filters` and create the filters.py file on your app.
- Provide a `.env.dist` as a template.
- Create a `.gitignore` file to exclude `.env`, `db.sqlite3`, virtual environments, etc., from version control.
- Generate a `requirements.txt` file for your application dependencies in production and development enviroment.

#### Transform `settings.py` into a Directory:

Inside your project directory `your_project_name` create a directory with the name _settings_ inside the _settings directory_ create three files `base.py`, `development.py` and `production.py`.

Inside the `base.py` will be the majority of the settings, but not DEBUG, DATABASES, ALLOWED_HOSTS or others that will change on development to production. It varies with the project.

> **E.g of `development.py:`**

```python
from .base import *

DEBUG = True

# Development-specific apps
INSTALLED_APPS += ['debug_toolbar', ]

# Development-specific middleware
MIDDLEWARE += ['debug_toolbar.middleware.DebugToolbarMiddleware', ]

# Local SQLite database for development

DATABASES = {
	'default': {
		'ENGINE': 'django.db.backends.sqlite3',
		'NAME': BASE_DIR / 'db.sqlite3',
	}
}
```

> **E.g of `production.py`:**
```python
from .base import *

DEBUG = False

ALLOWED_HOSTS = []

DATABASES = {

'default': {
	# In production we will use MySQL
	}
}

# Security settings
SECURE_SSL_REDIRECT = True
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True
```

PS: But now that we separated the settings between production and development it'll change a little the dynamic of running the project, because now we need to specify which settings we want to use:

```
python manage.py runserver --settings=auto_control.settings.development
```

or 

```
python manage.py runserver --settings=auto_control.settings.production
```

### Utilizing `.env` Files and `django-environ` for Environment Variables

Environment variables are essential for keeping your application's sensitive information, such as database credentials and secret keys, out of version control. To manage them efficiently:

- Install `django-environ`:

  ```
  pip install django-environ
  ```

- Create a `.env` file in the root of your project directory.
- Create a `.env.dist` too, this way other users will know  the attributes they need to have on their `.env` file.

> **Your `.env` file shoul be created inside your project DIR:**

```
your_project_name/
	.env
	.env.dist
your_app_name/
```

> **E.g of a `.env.dist` file:**

```
DEBUG=Boolean
SECRET_KEY=EnterYourSecretKeyHere
```

> **E.g `.env` file:**

```
DEBUG=True
SECRET_KEY=django-insecure-124343253656456
```

- In your settings (e.g., `base.py`), use `django-environ` to load the environment variables:

  ```python
  import environ

  env = environ.Env()
  environ.Env.read_env()  # reads the .env file

  SECRET_KEY = env('SECRET_KEY')
  DEBUG = env.bool('DEBUG', default=False)
  ```


### Installing `django-filters`

```
pip install django-filter
```

- In your settings (e.g., `base.py`) add `django-filters` to your INSTALLED_APPS and REST_FRAMEWORK  attributes:
	- If the REST_FRAMEWORK  doesn't exit, create it.

```
INSTALLED_APPS = [
    ...
    'django_filters',
]
```

```
REST_FRAMEWORK = {
    ...
    'DEFAULT_FILTER_BACKENDS':['django_filters.rest_framework.DjangoFilterBackend'],
}

```

Now create the `filters.py` file on your app if it not exist yet.

If you want to know more about the use of the `filters.py` on your API, below on the **append 1** I'll explain more how marveoulous it is and how we can use it.

### Creating a `.gitignore` File

To avoid committing unnecessary or sensitive files to your Git repository, create a `.gitignore` file in your project root. Add the following entries:

> **Your `.gitignore` file should be created on the ROOT:**
```
your_project_name/
your_app_name/
.gitignore
```

> **This are examples of things that should go on your `.gitignore` file, I recommend you ask to the GPT to create a `.gitignore` file to a DRF API application, it'll be more complete**
```
.env
*.pyc
__pycache__/
db.sqlite3
/static
/media
.DS_Store
venv/
```

### Generating a `requirements.txt` File

To ensure that other developers or your deployment environment can install the exact versions of your project's dependencies, generate a `requirements.txt` file:

```
pip freeze > requirements.txt
```

PS: If you made a clone of a project that have the requirements.txt file and want to install run the command (Create and activate your venv enviroment before running the comand).

```
pip install -r requirements.txt
```

### Summary

This guide has walked you through setting up a Django Rest Framework project, emphasizing best practices for project structure, environment management, and settings configuration. By following these steps, you're setting a solid foundation for building scalable and maintainable web applications with Django and DRF.

Remember to update and refine your documentation as your project evolves. Good documentation is key to maintaining a project's health and facilitating collaboration among developers.



### Append 1 `django-filters`:

#### What is Django-filters?

`Django-filters` is a reusable Django application allowing users to filter querysets dynamically. When integrated with Django Rest Framework (DRF), it provides a powerful mechanism for filtering resources in web APIs, making it easier to refine the results returned by endpoints based on query parameters provided by the client.

#### Installment and Setup

As we saw above, is necessary install with **pip** and make adjustments on the `settings.py` file and create `filters.py` file on your app

#### Implementing

On you `filters.py` file you need to make some imports, from django.filters and the model you need to filter. As an example let's use a model of a library.

> **E.g `models.py`:**
```python
from django.db import models

class Library(models.Model):
	book_name = models.Charfield(max_length=150)
	author = models.Charfield(max_length=150)
	...
```

> **E.g `filters.py`:**
```python
from django.filters import rest_framework as filters
from .models import Library

class LibraryFilter(filters.FilterSet):
	class Meta:
		model: Library
		fields = ['book_name', 'author', 'published_year']

```

> **E.g `views.py`:**
```python
from rest_framework import viewsets
from .models import Library
from .serializers import BookSerializer
from .filters import LibraryFilter


class LibraryViewSet(viewsets.ModelViewSet):
	queryset = Library.objects.all()
	serializer_class = LibrarySerializer
	filterset_class = LibraryFilter

```

### Using the Filtered API

Once the filter is set up, you can use it by making HTTP GET requests with query parameters corresponding to the fields you've defined in your `FilterSet`. For example:

```
GET /api/books/?book_name=Sapiens&author=Yuval Noah
```
### Custom Filtering

`django-filters` also allows for more complex filtering needs, such as filtering by a range of dates, checking for substring occurrences, or even using custom filter methods.

For instance, to filter books published within a certain date range:

pythonCopy code

```python
class LibraryFilter(filters.FilterSet):
	published_year__range = filters.DateFromToRangeFilter()
	
	class Meta:
		 model = Library
		 fields = ['book_name', 'author', 'published_year']
```

>**Using it:**
```
GET /api/books/?published_year__range_start=1990&published_year__range_end=2000
````

Integrating `django-filters` with DRF enhances your API with dynamic filtering, making it more flexible and user-friendly. Defining `FilterSets` and using them in viewsets allows users to efficiently search and find the data they need.
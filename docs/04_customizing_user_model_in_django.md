## Customizing User Model in Django Using `AbstractUser`

Django's flexibility allows for easy customization of the user authentication model. This guide will show you how to extend the default `User` model using `AbstractUser` to add custom fields and use an email for authentication instead of a username.

### Why Customize the User Model?

Customizing the user model is essential when you need to store additional information about users or change the authentication mechanism. Django's `AbstractUser` class provides a clean slate for adding new fields and methods to the user model.

### Step 1: Extend `AbstractUser`

#### 1. **Create a Custom User Model**

First, create a new model that extends `AbstractUser`. In your app's `models.py`, add:

```python
from django.contrib.auth.models import AbstractUser

class CustomUser(AbstractUser):
    # Add additional fields here
    # Example: Adding a birth date field
    birth_date = models.DateField(null=True, blank=True)
```

#### 2. **Set the Custom User Model**

Before applying migrations, specify your custom user model in the project's `settings.py`:

```python
AUTH_USER_MODEL = 'your_app.CustomUser'
```

Replace `'your_app'` with the name of the app where your custom user model is defined.

### Step 2: Use Email as the Primary Identifier

To authenticate users using their email instead of a username, you'll need to modify your custom user model and the user manager.

#### 1. **Modify the Custom User Model**

Update your `CustomUser` model to make the `email` field unique and remove the username field:

```python
class CustomUser(AbstractUser):
    username = None
    email = models.EmailField(unique=True)
    
    USERNAME_FIELD = 'email'
    REQUIRED_FIELDS = []

    objects = CustomUserManager()  # This will be defined next
```

#### 2. **Create a Custom User Manager**

A custom user manager is required to handle creating and managing users without a username. In the same `models.py` file, add:

```python
from django.contrib.auth.models import BaseUserManager

class CustomUserManager(BaseUserManager):
    use_in_migrations = True

    def _create_user(self, email, password, **extra_fields):
        if not email:
            raise ValueError('The Email field must be set')
        email = self.normalize_email(email)
        user = self.model(email=email, **extra_fields)
        user.set_password(password)
        user.save(using=self._db)
        return user

    def create_user(self, email, password=None, **extra_fields):
        extra_fields.setdefault('is_staff', False)
        extra_fields.setdefault('is_superuser', False)
        return self._create_user(email, password, **extra_fields)

    def create_superuser(self, email, password=None, **extra_fields):
        extra_fields.setdefault('is_staff', True)
        extra_fields.setdefault('is_superuser', True)

        if extra_fields.get('is_staff') is not True:
            raise ValueError('Superuser must have is_staff=True.')
        if extra_fields.get('is_superuser') is not True:
            raise ValueError('Superuser must have is_superuser=True.')

        return self._create_user(email, password, **extra_fields)
```

### Step 3: Update the Admin Interface

To manage `CustomUser` instances in the Django admin, you'll need to define a custom `ModelAdmin`:

```python
from django.contrib import admin
from django.contrib.auth.admin import UserAdmin
from .models import CustomUser

class CustomUserAdmin(UserAdmin):
    model = CustomUser
    fieldsets = UserAdmin.fieldsets + (
            (None, {'fields': ('birth_date',)}),
    )
    add_fieldsets = UserAdmin.add_fieldsets + (
            (None, {'fields': ('birth_date',)}),
    )
    list_display = ['email', 'first_name', 'last_name', 'is_staff', 'birth_date']
    search_fields = ('email', 'first_name', 'last_name')
    ordering = ('email',)

admin.site.register(CustomUser, CustomUserAdmin)
```

### Step 4: Apply Migrations

After making these changes, run the following commands to create and apply migrations for your custom user model:

```bash
python manage.py makemigrations
python manage.py migrate
```

### Summary

By extending `AbstractUser`, you can customize the Django user model to fit your application’s needs, including adding additional fields and using email as the primary identifier for authentication. This approach keeps your user model flexible and tailored to your project's requirements.
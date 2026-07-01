# Django Complete Lecture (Class 1 + Class 2 Combined)

## 1. What is Django?

Django is a **high-level Python web framework** that helps developers build secure, maintainable, and scalable web applications quickly.

It handles most of the repetitive parts of web development — like authentication, database connection, and routing — so you can focus on your app logic.

### Why Use Django?
- Fast Development
- Secure (protects from common attacks)
- Built-in Admin Panel
- ORM (Object Relational Mapper) to work with databases easily
- Scalable — suitable for small or large apps
- Free and open source

### Other Python Web Frameworks
1. **Flask** — Micro Web Framework
2. **FastAPI** — Modern & Fast
3. **Pyramid** — Flexible & Scalable
4. **Tornado**
5. **Bottle**
6. **Web2py**

---

## 2. Django Multi-App System

One of the biggest advantages of Django is that a single project can contain **multiple apps**.

**Multi-App Architecture** means:
- One Project → Multiple Apps
- Each app handles a separate functionality

### Example: Facebook
Inside Facebook there are many modules, which would be treated as separate **Apps** in Django:
- Login
- Messenger
- Notifications
- Marketplace
- User

---

## 3. Django Architecture — MVT Pattern

Django follows the **MVT** architecture:

**MVT = Model – View – Template**

| Component | Description |
|---|---|
| **Model** | Handles the data (database layer). Defines the structure of your database tables. |
| **View** | Handles the logic. Connects the Model and the Template. |
| **Template** | Handles the presentation layer (HTML files shown to users). |

### MVT Flow (Step by Step)
1. User requests a page using the browser.
2. Django receives the request and sends it to the View.
3. The View interacts with the Model (if data is needed).
4. Data is sent to the Template to be displayed.
5. Template sends the final HTML back to the user.

---

## 4. Django Project Structure

| File | Purpose |
|---|---|
| `manage.py` | Command-line utility for Django |
| `__init__.py` | Marks the directory as a Python package |
| `settings.py` | The heart of your project — contains all configuration settings for your website |
| `urls.py` | Controls URL routing — decides which view function runs for a given URL |
| `asgi.py` | Required for modern, real-time features (WebSockets, async tasks, background updates) |
| `wsgi.py` | The WSGI (Web Server Gateway Interface) entry point |

---

## 5. Important Commands (Quick Reference)

```bash
# Install Django
pip install Django

# Create a new project
django-admin startproject myproject

# Create a new app
python manage.py startapp myapp

# Run the development server
python manage.py runserver

# Create migration files (based on model changes)
python manage.py makemigrations

# Apply migrations to database
python manage.py migrate
```

---

## 6. Full Workflow — From Project to Admin Panel

### Step 1 — Create Django Project
```bash
django-admin startproject myproject
```

### Step 2 — Create App
```bash
python manage.py startapp myapp
```

### Step 3 — Register App in `settings.py`
```python
INSTALLED_APPS = [
    # ... default apps stay the same
    'myapp',
]
```

### Step 4 — Create Model
Open `myapp/models.py`:

```python
from django.db import models

class Student(models.Model):
    name = models.CharField(max_length=100)
    age = models.IntegerField()
    email = models.EmailField()

    def __str__(self):
        return self.name
```

### Step 5 — Make Migrations
```bash
python manage.py makemigrations
python manage.py migrate
```

### Step 6 — Create Superuser
```bash
python manage.py createsuperuser
```
Fields:
- Username
- Email *(optional — can be left blank)*
- Password

### Step 7 — Register Model in Admin Panel
Open `myapp/admin.py`:

```python
from django.contrib import admin
from .models import Student

admin.site.register(Student)
```



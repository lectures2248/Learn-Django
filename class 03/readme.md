# Django Class 3 — Views, URLs & Templates (Completing the MVT Cycle)

In Class 2 we created the **Model** (Student) and saw it in the Admin Panel. Now we complete the MVT cycle by building the **View** and the **Template**, so we can actually display data on a real webpage in the browser.

---

## 1. Recap — MVT Flow

| Component | Description |
|---|---|
| **Model** | Handles the data (database layer) — *already done in Class 2* |
| **View** | Handles the logic. Connects Model and Template — *today* |
| **Template** | Handles the presentation layer (HTML) — *today* |

Flow: **Browser → urls.py → View → Model (if needed) → Template → Browser**

---

## 2. URL Routing

Django has two levels of `urls.py`:

- **Project-level** `urls.py` (inside `myproject/`) — main entry point
- **App-level** `urls.py` (inside `myapp/`) — created manually per app

### Step 1 — Create `urls.py` inside your app
`myapp/urls.py`:
```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.home, name='home'),
    path('students/', views.student_list, name='student_list'),
]
```

### Step 2 — Connect app urls to the project
`myproject/urls.py`:
```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('myapp.urls')),
]
```

`include()` tells Django: "any URL that isn't `admin/` should be checked inside `myapp/urls.py`."

---

## 3. Views — The Logic Layer

Open `myapp/views.py`.

### A Simple View (no template yet)
```python
from django.http import HttpResponse

def home(request):
    return HttpResponse("Hello, welcome to my Django site!")
```

### A View that Uses a Template
```python
from django.shortcuts import render

def home(request):
    return render(request, 'home.html')
```

- `request` — the incoming browser request (always the first parameter)
- `render()` — combines a template with data and returns an HTTP response

---

## 4. Templates — The Presentation Layer

### Step 1 — Create the templates folder
```
myapp/
    templates/
        home.html
```

### Step 2 — Tell Django where to find templates
In `settings.py`, check `TEMPLATES`:
```python
TEMPLATES = [
    {
        ...
        'DIRS': [],   # Django auto-detects 'templates/' folder inside each app
        'APP_DIRS': True,
        ...
    },
]
```
`APP_DIRS: True` means Django automatically looks inside every app's `templates/` folder — no extra setup needed for a basic project.

### Step 3 — Write basic HTML
`myapp/templates/home.html`:
```html
<h1>Welcome to My Django Site</h1>
<p>This page is rendered from a template.</p>
```

---

## 5. Passing Data from View to Template

This is where Model + View + Template finally connect.

### View — Fetch data from the Model
```python
from django.shortcuts import render
from .models import Student

def student_list(request):
    students = Student.objects.all()
    return render(request, 'students.html', {'students': students})
```

- `Student.objects.all()` — fetches all rows from the Student table (ORM in action)
- The dictionary `{'students': students}` — sends this data to the template as **context**

### Template — Display the data
`myapp/templates/students.html`:
```html
<h1>Student List</h1>

<ul>
    {% for student in students %}
        <li>{{ student.name }} - {{ student.age }} - {{ student.email }}</li>
    {% endfor %}
</ul>
```

### Django Template Syntax
| Syntax | Purpose |
|---|---|
| `{{ variable }}` | Print a variable's value |
| `{% tag %}` | Logic — loops, conditions, etc. |
| `{% for x in list %} ... {% endfor %}` | Loop through data |
| `{% if condition %} ... {% endif %}` | Conditional display |

---

## 6. Template Inheritance (Basics)

Instead of repeating the same HTML (header, navbar, footer) on every page, Django lets you create a **base template**.

`myapp/templates/base.html`:
```html
<html>
<head><title>My Django Site</title></head>
<body>
    <h2>My Website Header</h2>
    {% block content %}
    {% endblock %}
</body>
</html>
```

`myapp/templates/students.html`:
```html
{% extends 'base.html' %}

{% block content %}
    <h1>Student List</h1>
    <ul>
        {% for student in students %}
            <li>{{ student.name }} - {{ student.age }}</li>
        {% endfor %}
    </ul>
{% endblock %}
```

- `{% extends %}` — inherits from the base template
- `{% block content %}` — the section that each child page fills in differently

---

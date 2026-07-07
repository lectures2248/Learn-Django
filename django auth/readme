# Lecture: Building a Django Login/Register System

## Step 1: Create the Django project

```bash
django-admin startproject myproject
cd myproject
```

## Step 2: Create the app

```bash
python manage.py startapp myapp
```

**Why an "app"?** Django projects are made of one or more apps, each handling one concern. Here, `myapp` will handle authentication (register/login/dashboard/logout).

---

## Step 3: Register the app in settings

Open `myproject/settings.py` and add `myapp` to `INSTALLED_APPS`:

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',        # already here by default — gives us User, login(), authenticate(), logout()
    'django.contrib.contenttypes',
    'django.contrib.sessions',    # already here by default — enables session cookies
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'myapp',                      # <-- add this line
]
```

Until you add `'myapp'` here, Django doesn't know your app exists — its templates, models, etc. won't be picked up.

**Why nothing extra for auth?** `django.contrib.auth` and `django.contrib.sessions` ship with every new Django project by default. That means the `User` model and session-based login already exist before you write a single line of your own code.

---

## Step 4: Tell Django where to send unauthenticated users

Still in `settings.py`, add:

```python
LOGIN_URL = "/login/"
```

We haven't built `/login/` yet, but we're setting this now because later we'll protect the dashboard with `@login_required`, and Django needs to know where to redirect a non-logged-in visitor. Without this, Django defaults to `/accounts/login/`, which doesn't exist in our app.

---

## Step 5: Decide the app's models (or lack thereof)

Open `myapp/models.py`. We leave it **empty**:

```python
from django.db import models
# Create your models here.
```

**Why no custom model?** Django's built-in `User` model (from `django.contrib.auth.models`) already has `username`, `email`, and a hashed `password` field — everything this app needs. Building a custom user model is a bigger topic (`AUTH_USER_MODEL`) reserved for when you need extra fields. For now, we reuse the built-in one.

Since we're not adding models, there's no need to run `makemigrations` for `myapp` — but the built-in `auth` app's tables (including `User`) still need to exist in the database:

```bash
python manage.py migrate
```

This creates `db.sqlite3` with all the default Django tables (including `auth_user`).

---

## Step 6: Plan the URLs

Before writing views, decide what pages we need:

| URL | Purpose |
|---|---|
| `/register/` | show + handle sign-up form |
| `/login/` | show + handle login form |
| `/dashboard/` | protected page, logged-in users only |
| `/logout/` | log the user out |

Create `myapp/urls.py` (this file doesn't exist by default — you create it):

```python
from django.urls import path
from . import views

urlpatterns = [
    path("register/", views.register, name="register"),
    path("login/", views.login_user, name="login"),
    path("dashboard/", views.dashboard, name="dashboard"),
    path("logout/", views.logout_user, name="logout"),
]
```

Note: the view is named `login_user`, not `login` — that's intentional, because `login` is already the name of the function we'll import from `django.contrib.auth`. Naming our view the same would shadow it.

Each `path()` also gets a `name=` — this lets us refer to the URL by name later (`{% url 'login' %}`, `redirect("dashboard")`) instead of hardcoding `"/login/"` everywhere.

---

## Step 7: Wire the app's URLs into the project

Open `myproject/urls.py` (this one already exists) and include our app's routes:

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('myapp.urls')),   # <-- add this
]
```

`include('myapp.urls')` means: "for anything not matched above, hand it off to `myapp/urls.py`." So `/register/` in the browser resolves project urls → falls through to app urls → matches `register/` → calls `views.register`.

---

## Step 8: Write the `register` view

Open `myapp/views.py` and start with imports:

```python
from django.shortcuts import render, redirect
from django.contrib.auth import authenticate, login, logout
from django.contrib.auth.decorators import login_required
from django.contrib.auth.models import User
from django.views.decorators.cache import never_cache
```

Now write the view:

```python
@never_cache
def register(request):

    if request.method == "POST":

        username = request.POST.get("username")
        email = request.POST.get("email")
        password = request.POST.get("password")

        # Check if username already exists
        if User.objects.filter(username=username).exists():
            return render(request, "register.html", {
                "error": "Username already exists."
            })

        # Create user
        User.objects.create_user(
            username=username,
            email=email,
            password=password
        )

        return redirect("login")

    return render(request, "register.html")
```

Walk through the logic in order:

1. A browser visiting `/register/` normally sends a **GET** request first. `request.method == "POST"` is `False`, so we skip straight to the last line and just render the empty form.
2. When the user fills the form and clicks submit, the browser sends a **POST** request instead, carrying the form fields in `request.POST`.
3. We pull out `username`, `email`, `password` using `.get()` (safe — returns `None` instead of crashing if a field is missing).
4. `User.objects.filter(username=username).exists()` — a database query — checks if that username is already taken. If so, re-render the form with an error message instead of creating a duplicate account.
5. `User.objects.create_user(...)` — this is the critical line. Unlike `User(username=..., password=...); User.save()`, `create_user()` **hashes the password** before storing it. Never bypass this method for setting passwords.
6. Once the account is created, `redirect("login")` sends the browser to the `login` named URL (`/login/`) with an HTTP redirect response.

`@never_cache` sits on top so browsers don't cache this page (relevant once we also protect pages after login — we don't want the back button showing a stale cached page).

---

## Step 9: Write the `login_user` view

```python
@never_cache
def login_user(request):

    if request.user.is_authenticated:
        return redirect("dashboard")

    if request.method == "POST":

        username = request.POST.get("username")
        password = request.POST.get("password")

        user = authenticate(
            request,
            username=username,
            password=password
        )

        if user is not None:
            login(request, user)
            return redirect("dashboard")

        else:
            return render(request, "login.html", {
                "error": "Invalid username or password."
            })

    return render(request, "login.html")
```

Step by step:

1. `request.user` is available on **every** request because of `AuthenticationMiddleware` (already enabled by default in `settings.py MIDDLEWARE`). If a session cookie identifies this browser as already logged in, `request.user.is_authenticated` is `True` — in that case we skip the form entirely and go straight to `/dashboard/`.
2. On **POST**, read `username`/`password` from the form.
3. `authenticate(request, username=..., password=...)` looks up the user by username, hashes the submitted password the same way `create_user()` did, and compares the hashes. It returns a `User` object on success or `None` on failure — it does **not** log anyone in by itself.
4. If authentication succeeded, `login(request, user)` is what actually creates the session — Django writes the user's ID into the session store and sends a session cookie to the browser. From this point on, every future request from that browser includes the cookie, and `AuthenticationMiddleware` uses it to populate `request.user`.
5. If authentication failed, re-render the login form with an error — note we don't reveal *which* field was wrong, just "Invalid username or password" (a basic security practice).
6. On plain **GET**, show the empty login form.

---

## Step 10: Write the `dashboard` view (the protected page)

```python
@login_required
@never_cache
def dashboard(request):
    return render(request, "dashboard.html")
```

1. `@login_required` wraps the view: **before** `dashboard()`'s body ever runs, Django checks `request.user.is_authenticated`. If `False`, it redirects to `LOGIN_URL` (the `/login/` we configured in Step 4) — the function body never executes.
2. If the user *is* authenticated, the view just renders `dashboard.html`. We don't manually pass `{"user": ...}` in the context — the template can access `request.user` directly, because the `auth` context processor (already registered in `TEMPLATES` in `settings.py`) exposes `request` to every template automatically.
3. **Decorator order matters.** `@login_required` is listed first (outermost), `@never_cache` second (innermost). Read decorators top-to-bottom as "outer wraps inner" — so the authentication check happens first, and only if it passes does the response get the "don't cache" headers applied.

---

## Step 11: Write the `logout_user` view

```python
@never_cache
def logout_user(request):
    logout(request)
    return redirect("login")
```

`logout(request)` deletes the session data tied to this browser (the reverse of what `login()` did in Step 9). After this call, `request.user` becomes anonymous again on the next request, so a subsequent visit to `/dashboard/` bounces back to `/login/`.

---

## Step 12: Build the templates

Django looks for templates inside each app's `templates/` folder (because `APP_DIRS: True` in `settings.py TEMPLATES`). Create `myapp/templates/register.html`, `login.html`, `dashboard.html`.

### `register.html`
```html
<h1>Register</h1>

{% if error %}
<p style="color:red">{{ error }}</p>
{% endif %}

<form method="POST">
    {% csrf_token %}
    <input type="text" name="username" placeholder="Username"><br><br>
    <input type="email" name="email" placeholder="Email"><br><br>
    <input type="password" name="password" placeholder="Password"><br><br>
    <button type="submit">Register</button>
</form>

<a href="{% url 'login' %}">Already have an account? Login</a>
```

Two details that must match the view exactly:
- Each `<input name="...">` must match the key the view reads via `request.POST.get("...")` in Step 8 (`username`, `email`, `password`).
- `{% csrf_token %}` is **mandatory** on every `<form method="POST">`. Django's `CsrfViewMiddleware` (enabled by default) rejects any POST request that doesn't include this token, as protection against cross-site request forgery. Forget it, and submitting the form returns a 403 error.

### `login.html`
Same shape — form posts to itself, shows `{{ error }}` if the view passed one, and links to `{% url 'register' %}`.

### `dashboard.html`
```html
<h1>Dashboard</h1>

<h2>Welcome {{ request.user.username }}</h2>

<p>Email: {{ request.user.email }}</p>

<a href="{% url 'logout' %}">Logout</a>
```

`{{ request.user.username }}` works because of the same `request` context processor mentioned in Step 10 — we never explicitly pass `user` into this template's context.

---

## Step 13: Run the server and test the full flow

```bash
python manage.py runserver
```



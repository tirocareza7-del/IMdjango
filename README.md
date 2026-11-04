# CampusConnect - Django Setup & Lab Activity Notes

This repository contains the complete setup notes, custom user model configuration, and user authentication implementation for the **CampusConnect** Django application.

---

## 🛠️ Phase 1: Environment & Prerequisites Setup

### 1. Verify Python Installation
Ensure Python is added to the system PATH during installation if not recognized.
```bash
python --version
# or
py --version
```

### 2. Create Project Directory
```bash
cd C:\Users\L13X08W37
mkdir campusconnect
cd campusconnect
code .
```

### 3. Set Up & Activate Virtual Environment
```bash
python -m venv .venv
.venv\Scripts\activate
```

### 4. Install Django & Lock Dependencies
```bash
pip install Django
python -m django --version
pip freeze > requirements.txt
```

---

## 🏗️ Phase 2: Project Initialization

### 1. Start Django Project Structure
```bash
django-admin startproject campusconnect .
```

### 2. Create Application Modules
```bash
python manage.py startapp accounts
python manage.py startapp dashboard
```

---

## ⚙️ Phase 3: Configuration (`campusconnect/settings.py`)

### 1. Register Installed Apps
Add the newly created applications to the `INSTALLED_APPS` list:
```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'accounts',
    'dashboard',
]
```

### 2. Configure Authentication & Custom User Model
Specify redirect targets and point to the custom user model at the bottom of `settings.py`:
```python
LOGIN_REDIRECT_URL = 'dashboard'
LOGOUT_REDIRECT_URL = 'login'
AUTH_USER_MODEL = 'accounts.CustomUser'
```

### 3. Configure Template Directories
Update the `TEMPLATES` setting to include the root templates directory:
```python
'DIRS': [BASE_DIR / 'templates'],
```

---

## 🗄️ Phase 4: Custom User Model & Database Migrations

### 1. Define Model (`accounts/models.py`)
```python
from django.contrib.auth.models import AbstractUser
from django.db import models

class CustomUser(AbstractUser):
    class Role(models.TextChoices):
        STUDENT = 'STUDENT', 'Student'
        ADMIN = 'ADMIN', 'Administrator'
    
    role = models.CharField(max_length=10, choices=Role.choices, default=Role.STUDENT)
    course = models.CharField(max_length=50, blank=True)
    
    def __str__(self):
        return self.username
```

### 2. Run Migrations & Create Superuser
Execute the following commands in your active virtual environment terminal:
```bash
python manage.py makemigrations accounts
python manage.py migrate
python manage.py createsuperuser
```

---

## 📝 Phase 5: Forms, Views & Templates

### 1. Create User Registration Form (`accounts/forms.py`)
Create a new file named `forms.py` inside the `accounts` folder:
```python
from django.contrib.auth.forms import UserCreationForm
from accounts.models import CustomUser

class CustomUserCreationForm(UserCreationForm):
    class Meta(UserCreationForm.Meta):
        model = CustomUser
        fields = ('username', 'email', 'first_name', 'last_name', 'role', 'course')
```

### 2. Create Registration View (`accounts/views.py`)
```python
from django.shortcuts import render
from django.urls import reverse_lazy
from django.views.generic import CreateView
from accounts.forms import CustomUserCreationForm

class SignUpView(CreateView):
    form_class = CustomUserCreationForm
    template_name = 'registration/signup.html'
    success_url = reverse_lazy('login')
```

### 3. Create Template Files

#### `templates/base.html`
```html
<!DOCTYPE html>
<html>
<head><title>CampusConnect</title></head>
<body>
<nav>
    {% if user.is_authenticated %}
        Hi, {{ user.username }}!
        <form method="post" action="{% url 'logout' %}">
            {% csrf_token %}
            <button type="submit">Log out</button>
        </form>
    {% else %}
        <a href="{% url 'login' %}">Log in</a>
        <a href="{% url 'signup' %}">Sign up</a>
    {% endif %}
</nav>
<main>{% block content %}{% endblock %}</main>
</body>
</html>
```

#### `templates/registration/signup.html`
```html
{% extends "base.html" %}

{% block content %}
<h2>Create your account</h2>
<form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Sign up</button>
</form>
{% endblock %}
```

#### `templates/registration/login.html`
```html
{% extends "base.html" %}

{% block title %}CampusConnect - Log In{% endblock %}

{% block content %}
<h2>Log in to your account</h2>
<form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Log in</button>
</form>
{% endblock %}
```

---

## 🔗 Phase 6: URL Routing Configuration

### 1. App-level URLs (`accounts/urls.py`)
```python
from django.urls import path
from accounts import views

urlpatterns = [
    path('signup/', views.SignUpView.as_view(), name='signup'),
]
```

### 2. Project-level URLs (`campusconnect/urls.py`)
```python
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('accounts.urls')),
    path('accounts/', include('django.contrib.auth.urls')),
]
```

---

## ⚡ Running & Testing the Project

To start the local development server:
```bash
python manage.py runserver
```

Open your web browser and navigate to:
* **Signup Page:** `http://127.0.0.1:8000/signup/`
* **Login Page:** `http://127.0.0.1:8000/accounts/login/`
* **Admin Dashboard:** `http://127.0.0.1:8000/admin/`

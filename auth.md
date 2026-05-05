## Django Built-in Authentication and Authorisation

1. [Setup](#setup)
2. [The User Model](#the-user-model)
3. [Creating Users](#creating-users)
4. [Authentication Functions](#authentication-functions)
5. [Restricting Access](#restricting-access)
6. [Permissions and Groups](#permissions-and-groups)
7. [Built-in Auth Views](#built-in-auth-views)
8. [Built-in Auth Forms](#built-in-auth-forms)
9. [Auth Data in Templates](#auth-data-in-templates)
10. [Password Management](#password-management)
11. [Custom User Model](#custom-user-model)
12. [Custom Authentication Backend](#custom-authentication-backend)
13. [Complete Walkthrough](#complete-walkthrough)

### Setup

Django's authentication system is provided by `django.contrib.auth` and is enabled by default in new projects. Verify these entries in `settings.py`:

```python
# settings.py
INSTALLED_APPS = [
    'django.contrib.auth',
    'django.contrib.contenttypes',
    # ...
]

MIDDLEWARE = [
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    # ...
]
```

_Listing: Required settings for Django's authentication system._

Run `python manage.py migrate` to create the auth tables (`auth_user`, `auth_group`, `auth_permission`, etc.).

---

### The User Model

The default `User` model (`django.contrib.auth.models.User`) has these fields:

_Table: Default User model fields._

| Field          | Type           | Description                                    |
| :------------- | :------------- | :--------------------------------------------- |
| `username`     | CharField(150) | Required. Unique.                              |
| `password`     | CharField      | Hashed password. Never stored in plain text.   |
| `email`        | EmailField     | Optional.                                      |
| `first_name`   | CharField(150) | Optional.                                      |
| `last_name`    | CharField(150) | Optional.                                      |
| `is_active`    | BooleanField   | Whether the account is active. Default `True`. |
| `is_staff`     | BooleanField   | Whether the user can access the admin site.    |
| `is_superuser` | BooleanField   | Whether the user has all permissions.          |
| `date_joined`  | DateTimeField  | When the account was created.                  |
| `last_login`   | DateTimeField  | Last login timestamp.                          |

Superusers and staff users are not separate classes—they are regular `User` objects with special flags set.

---

### Creating Users

**From the shell:**

```python
from django.contrib.auth.models import User

# Regular user
user = User.objects.create_user('john', 'john@example.com', 'secretpass')
user.first_name = 'John'
user.save()

# Superuser
admin = User.objects.create_superuser('admin', 'admin@example.com', 'adminpass')
```

_Listing: Creating users programmatically._

> **Note:** Always use `create_user()` or `create_superuser()`. Never use `User.objects.create()` directly—it will not hash the password.

**From the command line:**

```bash
python manage.py createsuperuser --username=admin --email=admin@example.com
```

_Listing: Creating a superuser via the management command._

**Changing passwords:**

```python
user = User.objects.get(username='john')
user.set_password('newpassword')
user.save()
```

_Listing: Changing a password programmatically._

```bash
python manage.py changepassword john
```

_Listing: Changing a password via the management command._

---

### Authentication Functions

Django provides three core functions in `django.contrib.auth`:

#### authenticate()

Verifies credentials and returns a `User` object or `None`:

```python
from django.contrib.auth import authenticate

user = authenticate(request, username='john', password='secretpass')
if user is not None:
    # Credentials are valid
    pass
else:
    # Invalid credentials
    pass
```

_Listing: Verifying user credentials._

#### login()

Attaches an authenticated user to the current session:

```python
from django.contrib.auth import authenticate, login

def login_view(request):
    if request.method == 'POST':
        username = request.POST['username']
        password = request.POST['password']
        user = authenticate(request, username=username, password=password)
        if user is not None:
            login(request, user)
            return redirect('dashboard')
        else:
            return render(request, 'login.html', {'error': 'Invalid credentials'})
    return render(request, 'login.html')
```

_Listing: A complete login view._

#### logout()

Removes the user from the session:

```python
from django.contrib.auth import logout

def logout_view(request):
    logout(request)
    return redirect('login')
```

_Listing: A logout view._

> **Note:** `logout()` flushes the entire session. Any data stored in the session is lost.

#### request.user

Every request has a `user` attribute. For authenticated users it is a `User` instance; for anonymous visitors it is an `AnonymousUser` instance:

```python
def my_view(request):
    if request.user.is_authenticated:
        return HttpResponse(f"Hello, {request.user.username}")
    else:
        return HttpResponse("Hello, guest")
```

_Listing: Checking authentication status via request.user._

---

### Restricting Access

#### Function-Based Views — @login_required

```python
from django.contrib.auth.decorators import login_required

@login_required
def dashboard(request):
    return render(request, 'dashboard.html')
```

_Listing: Restricting a view to authenticated users._

If the user is not logged in, they are redirected to `settings.LOGIN_URL` (default: `/accounts/login/`) with a `?next=` parameter.

**Optional parameters:**

```python
@login_required(login_url='/my-login/', redirect_field_name='return_to')
def dashboard(request):
    ...
```

_Listing: Customising the login URL and redirect field name._

#### Class-Based Views — LoginRequiredMixin

```python
from django.contrib.auth.mixins import LoginRequiredMixin
from django.views.generic import TemplateView

class DashboardView(LoginRequiredMixin, TemplateView):
    template_name = 'dashboard.html'
    login_url = '/login/'           # optional
    redirect_field_name = 'next'    # optional
```

_Listing: Restricting a class-based view to authenticated users._

> **Important:** `LoginRequiredMixin` must be the **leftmost** parent in the inheritance list.

#### @user_passes_test

Restricts access based on a custom condition:

```python
from django.contrib.auth.decorators import user_passes_test

def is_admin(user):
    return user.is_staff

@user_passes_test(is_admin, login_url='/login/')
def admin_panel(request):
    return render(request, 'admin_panel.html')
```

_Listing: Restricting access with a custom test._

**Class-based equivalent — UserPassesTestMixin:**

```python
from django.contrib.auth.mixins import UserPassesTestMixin
from django.views.generic import TemplateView

class AdminPanel(UserPassesTestMixin, TemplateView):
    template_name = 'admin_panel.html'

    def test_func(self):
        return self.request.user.is_staff
```

_Listing: UserPassesTestMixin for class-based views._

#### @permission_required

Checks whether a user has a specific permission:

```python
from django.contrib.auth.decorators import permission_required

@permission_required('myapp.add_post', login_url='/login/')
def create_post(request):
    ...

# Raise 403 instead of redirecting
@login_required
@permission_required('myapp.add_post', raise_exception=True)
def create_post(request):
    ...
```

_Listing: Permission-based access control._

**Class-based equivalent — PermissionRequiredMixin:**

```python
from django.contrib.auth.mixins import PermissionRequiredMixin
from django.views.generic import CreateView

class PostCreateView(PermissionRequiredMixin, CreateView):
    permission_required = 'myapp.add_post'
    # or multiple: permission_required = ['myapp.add_post', 'myapp.change_post']
```

_Listing: PermissionRequiredMixin for class-based views._

#### LoginRequiredMiddleware (Django 5.1+)

Makes all views require authentication by default. Individual views can opt out:

```python
# settings.py
MIDDLEWARE = [
    # ...
    'django.contrib.auth.middleware.LoginRequiredMiddleware',
]
```

```python
from django.contrib.auth.decorators import login_not_required

@login_not_required
def public_page(request):
    return render(request, 'public.html')
```

_Listing: Global login requirement with per-view opt-out._

---

### Permissions and Groups

#### Default Permissions

Django automatically creates four permissions for every model: `add`, `change`, `delete`, and `view`. The format is `<app_label>.<action>_<model_name>`:

```python
user.has_perm('myapp.add_post')       # Can add?
user.has_perm('myapp.change_post')    # Can change?
user.has_perm('myapp.delete_post')    # Can delete?
user.has_perm('myapp.view_post')      # Can view?
```

_Listing: Checking default permissions._

#### Custom Permissions

Define custom permissions in the model's `Meta` class:

```python
class Post(models.Model):
    title = models.CharField(max_length=200)
    is_published = models.BooleanField(default=False)

    class Meta:
        permissions = [
            ('can_publish', 'Can publish posts'),
            ('can_archive', 'Can archive posts'),
        ]
```

_Listing: Defining custom permissions._

Run `python manage.py migrate` to create them.

#### Creating Permissions Programmatically

```python
from django.contrib.auth.models import Permission
from django.contrib.contenttypes.models import ContentType
from myapp.models import Post

content_type = ContentType.objects.get_for_model(Post)
permission = Permission.objects.create(
    codename='can_feature',
    name='Can feature posts',
    content_type=content_type,
)
```

_Listing: Creating a permission programmatically._

#### Assigning Permissions to Users

```python
from django.contrib.auth.models import User, Permission

user = User.objects.get(username='john')

# Add a permission
perm = Permission.objects.get(codename='can_publish')
user.user_permissions.add(perm)

# Check (re-fetch user to clear cache)
user = User.objects.get(username='john')
user.has_perm('myapp.can_publish')  # True
```

_Listing: Assigning and checking permissions on a user._

> **Note:** Permissions are cached on the `User` object. After adding or removing permissions, re-fetch the user from the database.

#### Groups

A `Group` is a collection of permissions that can be assigned to multiple users at once:

```python
from django.contrib.auth.models import Group, Permission

# Create a group
editors = Group.objects.create(name='Editors')

# Add permissions
perm = Permission.objects.get(codename='can_publish')
editors.permissions.add(perm)

# Add a user to the group
user.groups.add(editors)

# User now inherits all group permissions
user.has_perm('myapp.can_publish')  # True
```

_Listing: Working with groups._

_Table: Group and permission management methods._

| Operation             | Code                                 |
| :-------------------- | :----------------------------------- |
| Add to group          | `user.groups.add(group)`             |
| Remove from group     | `user.groups.remove(group)`          |
| Clear all groups      | `user.groups.clear()`                |
| Set groups            | `user.groups.set([group1, group2])`  |
| Add permission        | `user.user_permissions.add(perm)`    |
| Remove permission     | `user.user_permissions.remove(perm)` |
| Clear all permissions | `user.user_permissions.clear()`      |

---

### Built-in Auth Views

Django provides class-based views for login, logout, and password management. Include them all at once:

```python
# project urls.py
from django.urls import path, include

urlpatterns = [
    path('accounts/', include('django.contrib.auth.urls')),
]
```

_Listing: Including all built-in auth URLs._

This registers the following URL patterns:

_Table: Built-in auth URL patterns._

| URL Pattern                        | Name                      | View                        | Default Template                            |
| :--------------------------------- | :------------------------ | :-------------------------- | :------------------------------------------ |
| `accounts/login/`                  | `login`                   | `LoginView`                 | `registration/login.html`                   |
| `accounts/logout/`                 | `logout`                  | `LogoutView`                | `registration/logged_out.html`              |
| `accounts/password_change/`        | `password_change`         | `PasswordChangeView`        | `registration/password_change_form.html`    |
| `accounts/password_change/done/`   | `password_change_done`    | `PasswordChangeDoneView`    | `registration/password_change_done.html`    |
| `accounts/password_reset/`         | `password_reset`          | `PasswordResetView`         | `registration/password_reset_form.html`     |
| `accounts/password_reset/done/`    | `password_reset_done`     | `PasswordResetDoneView`     | `registration/password_reset_done.html`     |
| `accounts/reset/<uidb64>/<token>/` | `password_reset_confirm`  | `PasswordResetConfirmView`  | `registration/password_reset_confirm.html`  |
| `accounts/reset/done/`             | `password_reset_complete` | `PasswordResetCompleteView` | `registration/password_reset_complete.html` |

> **Important:** Django does **not** provide default templates. You must create them yourself in a `registration/` directory inside your templates folder.

#### Key Settings

```python
# settings.py
LOGIN_URL = '/accounts/login/'            # Where @login_required redirects
LOGIN_REDIRECT_URL = '/accounts/profile/'  # Where LoginView redirects after success
LOGOUT_REDIRECT_URL = '/'                  # Where LogoutView redirects after logout
```

_Listing: Authentication-related settings._

#### Customising Individual Views

```python
from django.contrib.auth import views as auth_views

urlpatterns = [
    path('login/', auth_views.LoginView.as_view(
        template_name='myapp/login.html',
        redirect_authenticated_user=True,
    ), name='login'),

    path('logout/', auth_views.LogoutView.as_view(
        next_page='/',
    ), name='logout'),
]
```

_Listing: Customising auth views with keyword arguments._

#### Login Template Example

Create `templates/registration/login.html`:

```html
<pre>
{% extends "base.html" %}
{% block content %}
<h2>Login</h2>

{% if form.errors %}
<p style="color: red;">Invalid username or password.</p>
{% endif %}

<form method="post">
  {% csrf_token %}
  {{ form.as_p }}
  <button type="submit">Login</button>
</form>

<p><a href="{% url 'password_reset' %}">Forgot password?</a></p>
{% endblock %}
</pre>
```

_Listing: A minimal login template._

---

### Built-in Auth Forms

All forms are in `django.contrib.auth.forms`:

_Table: Built-in authentication forms._

| Form                      | Purpose                                                    |
| :------------------------ | :--------------------------------------------------------- |
| `AuthenticationForm`      | Login form. Validates username and password.               |
| `UserCreationForm`        | Registration form. Fields: username, password1, password2. |
| `UserChangeForm`          | Used in admin to edit user details.                        |
| `PasswordChangeForm`      | Change password (requires old password).                   |
| `SetPasswordForm`         | Set new password (no old password required).               |
| `PasswordResetForm`       | Enter email to receive a reset link.                       |
| `AdminPasswordChangeForm` | Used in admin to change a user's password.                 |

#### Using UserCreationForm for Registration

```python
# views.py
from django.contrib.auth.forms import UserCreationForm
from django.shortcuts import render, redirect

def register(request):
    if request.method == 'POST':
        form = UserCreationForm(request.POST)
        if form.is_valid():
            form.save()
            return redirect('login')
    else:
        form = UserCreationForm()
    return render(request, 'registration/register.html', {'form': form})
```

_Listing: Registration view using UserCreationForm._

```html
<pre>
<!-- templates/registration/register.html -->
{% extends "base.html" %}
{% block content %}
<h2>Register</h2>
<form method="post">
  {% csrf_token %}
  {{ form.as_p }}
  <button type="submit">Register</button>
</form>
<p>Already have an account? <a href="{% url 'login' %}">Login</a></p>
{% endblock %}
</pre>
```

_Listing: Registration template._

#### Extending UserCreationForm

```python
# forms.py
from django.contrib.auth.forms import UserCreationForm
from django.contrib.auth.models import User

class CustomUserCreationForm(UserCreationForm):
    class Meta:
        model = User
        fields = ('username', 'email', 'first_name', 'last_name')
```

_Listing: Adding extra fields to the registration form._

---

### Auth Data in Templates

The `django.contrib.auth.context_processors.auth` context processor (enabled by default) provides two variables in every template:

#### {{ user }}

```html
<pre>
{% if user.is_authenticated %}
  <p>Welcome, {{ user.username }}!</p>
  <a href="{% url 'logout' %}">Logout</a>
{% else %}
  <a href="{% url 'login' %}">Login</a>
{% endif %}
</pre>
```

_Listing: Using the user variable in templates._

#### {{ perms }}

```html
<pre>
{% if perms.myapp.can_publish %}
  <button>Publish</button>
{% endif %}

{% if perms.myapp %}
  <p>You have some permissions in myapp.</p>
{% endif %}
</pre>
```

_Listing: Checking permissions in templates._

---

### Password Management

#### Session Invalidation on Password Change

When a user changes their password, all their other sessions are invalidated. Django's built-in `PasswordChangeView` handles this automatically. For custom password change views, use `update_session_auth_hash()`:

```python
from django.contrib.auth import update_session_auth_hash
from django.contrib.auth.forms import PasswordChangeForm

def change_password(request):
    if request.method == 'POST':
        form = PasswordChangeForm(user=request.user, data=request.POST)
        if form.is_valid():
            form.save()
            update_session_auth_hash(request, form.user)  # Keeps user logged in
            return redirect('dashboard')
    else:
        form = PasswordChangeForm(user=request.user)
    return render(request, 'change_password.html', {'form': form})
```

_Listing: Custom password change view with session preservation._

#### Password Reset via Email

Requires email configuration in `settings.py`:

```python
# settings.py
EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'
EMAIL_HOST = 'smtp.gmail.com'
EMAIL_PORT = 587
EMAIL_USE_TLS = True
EMAIL_HOST_USER = 'your@email.com'
EMAIL_HOST_PASSWORD = 'your-app-password'
```

_Listing: Email configuration for password reset._

The password reset email template (`registration/password_reset_email.html`):

```txt
Someone requested a password reset for {{ user.get_username }}.

Follow this link to reset your password:
{{ protocol }}://{{ domain }}{% url 'password_reset_confirm' uidb64=uid token=token %}

If you didn't request this, ignore this email.
```

_Listing: Password reset email template._

---

### Custom User Model

#### Using AbstractUser (Recommended for New Projects)

Extends the default user model with additional fields while keeping all built-in functionality:

```python
# models.py
from django.contrib.auth.models import AbstractUser

class CustomUser(AbstractUser):
    phone = models.CharField(max_length=20, blank=True)
    bio = models.TextField(blank=True)
```

```python
# settings.py
AUTH_USER_MODEL = 'myapp.CustomUser'
```

```python
# admin.py
from django.contrib import admin
from django.contrib.auth.admin import UserAdmin
from .models import CustomUser

admin.site.register(CustomUser, UserAdmin)
```

_Listing: Custom user model using AbstractUser._

> **Warning:** Set `AUTH_USER_MODEL` **before** running `migrate` for the first time. Changing it mid-project requires manual schema changes.

#### Using AbstractBaseUser (Full Control)

For completely custom user models (e.g., email-based login instead of username):

```python
# models.py
from django.contrib.auth.models import AbstractBaseUser, BaseUserManager, PermissionsMixin

class CustomUserManager(BaseUserManager):
    def create_user(self, email, password=None, **extra_fields):
        if not email:
            raise ValueError('Email is required')
        email = self.normalize_email(email)
        user = self.model(email=email, **extra_fields)
        user.set_password(password)
        user.save()
        return user

    def create_superuser(self, email, password=None, **extra_fields):
        extra_fields.setdefault('is_staff', True)
        extra_fields.setdefault('is_superuser', True)
        return self.create_user(email, password, **extra_fields)

class CustomUser(AbstractBaseUser, PermissionsMixin):
    email = models.EmailField(unique=True)
    first_name = models.CharField(max_length=150)
    is_active = models.BooleanField(default=True)
    is_staff = models.BooleanField(default=False)

    objects = CustomUserManager()

    USERNAME_FIELD = 'email'
    REQUIRED_FIELDS = ['first_name']

    def __str__(self):
        return self.email
```

_Listing: Custom user model using AbstractBaseUser for email-based authentication._

#### Extending with a Profile Model (No Custom User Model)

If you cannot change the user model, use a one-to-one relationship:

```python
from django.contrib.auth.models import User

class Profile(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE)
    phone = models.CharField(max_length=20, blank=True)
    avatar = models.ImageField(upload_to='avatars/', blank=True)

    def __str__(self):
        return self.user.username
```

_Listing: Profile model linked to the default User._

#### Referencing the User Model

Always use `get_user_model()` or `settings.AUTH_USER_MODEL` instead of importing `User` directly:

```python
# In models.py (ForeignKey)
from django.conf import settings

class Post(models.Model):
    author = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)

# In views.py or other code
from django.contrib.auth import get_user_model

User = get_user_model()
users = User.objects.all()
```

_Listing: Properly referencing the user model._

---

### Custom Authentication Backend

To authenticate against a source other than the database (e.g., email instead of username, LDAP, external API):

```python
# backends.py
from django.contrib.auth.backends import BaseBackend
from django.contrib.auth import get_user_model

User = get_user_model()

class EmailBackend(BaseBackend):
    def authenticate(self, request, username=None, password=None):
        try:
            user = User.objects.get(email=username)
            if user.check_password(password):
                return user
        except User.DoesNotExist:
            return None

    def get_user(self, user_id):
        try:
            return User.objects.get(pk=user_id)
        except User.DoesNotExist:
            return None
```

```python
# settings.py
AUTHENTICATION_BACKENDS = [
    'myapp.backends.EmailBackend',
    'django.contrib.auth.backends.ModelBackend',  # Keep as fallback
]
```

_Listing: Custom authentication backend for email-based login._

---

### Complete Walkthrough

A complete example combining built-in views, registration, and a protected dashboard.

**Step 1: URL Configuration.**

```python
# project urls.py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('accounts/', include('django.contrib.auth.urls')),
    path('accounts/', include('myapp.urls')),
]
```

```python
# myapp/urls.py
from django.urls import path
from . import views

urlpatterns = [
    path('register/', views.register, name='register'),
    path('dashboard/', views.dashboard, name='dashboard'),
]
```

_Listing: URL configuration for the complete auth system._

**Step 2: Settings.**

```python
# settings.py
LOGIN_URL = '/accounts/login/'
LOGIN_REDIRECT_URL = '/accounts/dashboard/'
LOGOUT_REDIRECT_URL = '/accounts/login/'
```

_Listing: Auth redirect settings._

**Step 3: Views.**

```python
# views.py
from django.shortcuts import render, redirect
from django.contrib.auth.decorators import login_required
from django.contrib.auth.forms import UserCreationForm
from django.contrib.auth import login

def register(request):
    if request.method == 'POST':
        form = UserCreationForm(request.POST)
        if form.is_valid():
            user = form.save()
            login(request, user)  # Auto-login after registration
            return redirect('dashboard')
    else:
        form = UserCreationForm()
    return render(request, 'registration/register.html', {'form': form})

@login_required
def dashboard(request):
    return render(request, 'dashboard.html')
```

_Listing: Registration view with auto-login and protected dashboard._

**Step 4: Templates.**

Create `templates/base.html`:

```html
<pre>
<!DOCTYPE html>
<html>
<head>
  <title>{% block title %}My Site{% endblock %}</title>
</head>
<body>
  <nav>
    {% if user.is_authenticated %}
      <span>Hello, {{ user.username }}</span>
      <form method="post" action="{% url 'logout' %}" style="display:inline;">
        {% csrf_token %}
        <button type="submit">Logout</button>
      </form>
    {% else %}
      <a href="{% url 'login' %}">Login</a>
      <a href="{% url 'register' %}">Register</a>
    {% endif %}
  </nav>
  {% block content %}{% endblock %}
</body>
</html>
</pre>
```

_Listing: Base template with conditional auth navigation._

Create `templates/registration/login.html`:

```html
<pre>
{% extends "base.html" %}
{% block title %}Login{% endblock %}
{% block content %}
<h2>Login</h2>
{% if form.errors %}
<p style="color: red;">Invalid username or password.</p>
{% endif %}
<form method="post">
  {% csrf_token %}
  {{ form.as_p }}
  <button type="submit">Login</button>
</form>
<p><a href="{% url 'password_reset' %}">Forgot password?</a></p>
<p>Don't have an account? <a href="{% url 'register' %}">Register</a></p>
{% endblock %}
</pre>
```

_Listing: Login template._

Create `templates/registration/register.html`:

```html
<pre>
{% extends "base.html" %}
{% block title %}Register{% endblock %}
{% block content %}
<h2>Create Account</h2>
<form method="post">
  {% csrf_token %}
  {{ form.as_p }}
  <button type="submit">Register</button>
</form>
<p>Already have an account? <a href="{% url 'login' %}">Login</a></p>
{% endblock %}
</pre>
```

_Listing: Registration template._

Create `templates/dashboard.html`:

```html
<pre>
{% extends "base.html" %}
{% block title %}Dashboard{% endblock %}
{% block content %}
<h2>Dashboard</h2>
<p>Welcome, {{ user.get_full_name|default:user.username }}!</p>
<ul>
  <li>Username: {{ user.username }}</li>
  <li>Email: {{ user.email }}</li>
  <li>Joined: {{ user.date_joined }}</li>
</ul>
<p><a href="{% url 'password_change' %}">Change password</a></p>
{% endblock %}
</pre>
```

_Listing: Protected dashboard template._

Create `templates/registration/password_change_form.html`:

```html
<pre>
{% extends "base.html" %}
{% block title %}Change Password{% endblock %}
{% block content %}
<h2>Change Password</h2>
<form method="post">
  {% csrf_token %}
  {{ form.as_p }}
  <button type="submit">Change Password</button>
</form>
{% endblock %}
</pre>
```

_Listing: Password change template._

Create `templates/registration/password_change_done.html`:

```html
<pre>
{% extends "base.html" %}
{% block title %}Password Changed{% endblock %}
{% block content %}
<h2>Password Changed Successfully</h2>
<p>Your password has been updated. <a href="{% url 'dashboard' %}">Back to dashboard</a></p>
{% endblock %}
</pre>
```

_Listing: Password change confirmation template._

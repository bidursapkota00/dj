# Django Complete Guide

## Table of Content

1. [The MVC Architectural Pattern](#the-mvc-architectural-pattern)
2. [The Role of the Backend](#the-role-of-the-backend)
3. [Introduction to Django](#introduction-to-django)
4. [URLs and Views](#urls-and-views)
5. [Templates and Static Files](#templates-and-static-files)
6. [Data and Models](#data-and-models)
7. [The Django Admin Interface](#the-django-admin-interface)
8. [Forms](#forms)
9. [Sessions and Cookies](#sessions-and-cookies)
10. [Authentication and Authorisation](#authentication-and-authorisation)
11. [Practical Exercises](#practical-exercises)
12. [Middleware](#middleware)
13. [Comparison of Backend Frameworks](#comparison-of-backend-frameworks)
14. [Database Types](#database-types)
15. [Building a Complete CRUD Application](#building-a-complete-crud-application)
16. [Testing](#testing)
17. [Deployment](#deployment)
18. [UI Testing with Selenium](#ui-testing-with-selenium)
19. [Common Web Security Vulnerabilities](#common-web-security-vulnerabilities)

## The MVC Architectural Pattern

### Introduction to MVC

MVC (Model-View-Controller) is a software architectural pattern that separates an application into three interconnected components. Originally developed for desktop graphical user interfaces, MVC has become the foundational pattern for most modern web frameworks. Its central principle is the separation of concerns: each component has a single, well-defined responsibility and should not interfere with the responsibilities of the others.

This separation yields several practical benefits: the codebase becomes easier to develop, test, and maintain; frontend and backend developers can work in parallel; individual components can be modified or replaced without affecting the rest of the application; and each component can be tested independently.

### Model — The Data Layer

The Model represents the application's data and business logic. It is responsible for defining data structures (tables, fields, relationships), validating data before it is persisted, performing database queries and updates, implementing business rules (calculations, constraints, workflows), and notifying other components when data changes.

The Model is entirely independent of the user interface. It does not know or care how data will be displayed—it focuses solely on data management.

### View — The Presentation Layer

The View is responsible for rendering data for the user. It handles the visual presentation: generating HTML, applying CSS, formatting data for display, and producing responses in various formats (HTML, JSON, XML). The View receives data from the Controller and renders it. Critically, the View should contain no business logic—it should only display, never process data.

### Controller — The Logic Layer

The Controller acts as an intermediary between the Model and the View. It receives incoming HTTP requests from users, parses and validates user input, calls appropriate Model methods to retrieve or modify data, selects the appropriate View for the response, and returns the final HTTP response.

The Controller is the "traffic controller" of the application, directing the flow of data between the Model and the View.

![MVC Diagram](/images/mvc.webp)

_Figure 3.1: The MVC architectural pattern showing the interaction between Model, View, and Controller._

### The Request-Response Cycle in MVC

The following sequence describes how a typical request is processed in an MVC application:

1. The user performs an action (clicks a link, submits a form).
2. The browser sends an HTTP request to the server.
3. The server's routing system directs the request to the appropriate Controller.
4. The Controller receives the request and validates the input.
5. The Controller calls the relevant Model methods to retrieve or modify data.
6. The Model performs database operations and applies business rules.
7. The Model returns the processed data to the Controller.
8. The Controller selects the appropriate View.
9. The View receives the data and generates the response (typically HTML).
10. The response is sent back to the user's browser.
11. The browser renders the page for the user.

### Benefits of MVC

_Table 3.1: Benefits of the MVC architectural pattern._

| Benefit                    | Description                                                              |
| :------------------------- | :----------------------------------------------------------------------- |
| **Separation of Concerns** | Each component has a single responsibility, producing cleaner code       |
| **Maintainability**        | Changes in one component do not affect the others                        |
| **Testability**            | Each component can be tested independently                               |
| **Parallel Development**   | Frontend and backend developers can work simultaneously                  |
| **Reusability**            | Models and Views can be reused across different parts of the application |
| **Scalability**            | Individual components can be scaled independently                        |
| **Code Organisation**      | A clear structure makes the codebase easier to navigate                  |

### MVC Implementations Across Frameworks

Different web frameworks implement the MVC pattern with varying terminology, but the core principle of separation of concerns remains consistent.

_Table 3.2: MVC implementations in popular web frameworks._

| Framework     | Language   | MVC Implementation           |
| :------------ | :--------- | :--------------------------- |
| Django        | Python     | MTV (Model-Template-View)    |
| Ruby on Rails | Ruby       | Traditional MVC              |
| ASP.NET MVC   | C#         | Traditional MVC              |
| Spring MVC    | Java       | Traditional MVC              |
| Laravel       | PHP        | Traditional MVC              |
| Express.js    | JavaScript | Flexible (can implement MVC) |

---

## The Role of the Backend

### What Is the Backend?

The backend (also called the server-side) is the part of a web application that operates on the server and is invisible to the end user. It is responsible for processing user requests, managing databases, executing business logic, authenticating and authorising users, and sending responses to the frontend.

While the frontend is what users interact with—buttons, forms, visual layout—the backend is what makes everything function behind the scenes.

### Frontend versus Backend

_Table 3.3: Comparison of frontend and backend responsibilities._

| Frontend                                           | Backend                                                     |
| :------------------------------------------------- | :---------------------------------------------------------- |
| Runs in the user's browser                         | Runs on the web server                                      |
| Uses languages such as HTML, CSS, and JavaScript   | Uses languages such as Python, Java, PHP, Ruby, and Node.js |
| Is visible to the user and enables interaction     | Is hidden from the user                                     |
| Focuses on building the user interface             | Handles data processing and application logic               |
| Includes elements like buttons, forms, and layouts | Includes databases, APIs, and authentication systems        |

### Core Responsibilities of the Backend

**Server Logic:** A server is a computer (or process) that listens continuously for incoming requests on a designated port (e.g., port 80 for HTTP, port 443 for HTTPS). When a request arrives, the server creates a thread or process to handle it, processes the request, generates a response, and sends the response back to the client.

![HTTP](/images/http.webp)

_Figure 3.2: HTTP request-response communication between client and server._

**Routing:** Routing is the mechanism by which the server determines which code should execute based on the requested URL. It maps URL patterns to specific handler functions:

- `/home` → display the homepage
- `/products` → display the product list
- `/products/123` → display the product with ID 123
- `/api/users` → return user data as JSON

Routes can be static (exact URL match, such as `/about`), dynamic (URLs containing variables, such as `/users/<id>`), or wildcard (catch-all patterns, such as `/blog/*`). URL parameters may appear as path parameters (`/users/123`) or query parameters (`/search?q=python&page=2`).

**Business Logic:** Business logic is the code that implements the rules and operations specific to the application. Examples include calculating discounts and totals, checking user permissions, validating uniqueness constraints, processing transactions, and generating reports. Business logic should be separated from the presentation layer, testable in isolation, reusable across interfaces, and robust in handling edge cases.

### Security Layers

Security is critical for backend applications. Multiple layers of defence protect the application and its users:

- **Authentication** (Who are you?) — Verifying user identity through login systems, multi-factor authentication, or OAuth/social login.
- **Authorisation** (What can you do?) — Checking user permissions through role-based access control (e.g., Admin, User, Guest) and resource ownership verification.
- **Input validation** — Sanitising user input to prevent SQL injection, Cross-Site Scripting (XSS), and other injection attacks.
- **Data protection** — Encrypting sensitive data, hashing passwords (never storing plain text), using HTTPS for data in transit, and implementing secure session management.
- **CSRF protection** — Preventing Cross-Site Request Forgery by requiring CSRF tokens in forms.
- **Rate limiting** — Preventing brute-force attacks and abuse by limiting API requests per user.

---

## Introduction to Django

### What Is Django?

Django is a high-level Python web framework designed for the rapid development of secure and maintainable web applications. It follows four core principles:

- **"Batteries included"** — Django ships with a comprehensive set of built-in features (authentication, ORM, admin panel, form handling, templating) so that developers do not need to assemble disparate libraries.
- **DRY (Don't Repeat Yourself)** — The framework encourages code reuse and discourages redundancy.
- **Rapid development** — Django's conventions and built-in tools dramatically accelerate the development cycle.
- **Security first** — Django provides built-in protection against common web vulnerabilities including SQL injection, XSS, CSRF, and clickjacking.

_Table 3.4: Key features of Django and their benefits._

| Feature                             | Benefit                                      |
| :---------------------------------- | :------------------------------------------- |
| **Built-in Admin Panel**            | Automatic admin interface for managing data  |
| **ORM (Object-Relational Mapping)** | Work with databases using Python, not SQL    |
| **Security**                        | Protection against SQL injection, XSS, CSRF  |
| **Scalability**                     | Powers large sites like Instagram, Pinterest |
| **Large Community**                 | Extensive documentation and packages         |
| **Authentication System**           | Built-in user management                     |

### The MTV Pattern — Django's Interpretation of MVC

Django implements the MVC pattern using its own terminology: MTV (Model-Template-View). The correspondence between the two naming conventions is as follows:

_Table 3.5: Mapping between MVC and Django's MTV pattern._

| MVC        | Django MTV   | Responsibility                      |
| :--------- | :----------- | :---------------------------------- |
| Model      | **Model**    | Data structure and database access  |
| View       | **Template** | HTML presentation and rendering     |
| Controller | **View**     | Business logic and request handling |

In Django, a "View" is a Python function or class that receives an HTTP request and returns an HTTP response (analogous to a Controller in traditional MVC). A "Template" is the HTML file that renders data for the user (analogous to a View in traditional MVC).

![MTV Diagram](/images/mtv.webp)

_Figure 3.3: Django's MTV (Model-Template-View) pattern._

### Setting Up a Django Development Environment

The recommended workflow for setting up a Django project involves creating an isolated Python virtual environment, installing Django within it, and initialising the project in the current directory using the `.` (dot) syntax. This approach keeps the project directory structure clean and avoids creating a redundant nested folder.

**Step 1: Install Python**

```bash
# Verify Python installation
python --version
# or
python3 --version
```

_Listing 3.1: Verifying the Python installation._

**Step 2: Create a Project Directory and Virtual Environment**

A virtual environment isolates the project's Python dependencies from the system-wide Python installation, preventing version conflicts between different projects.

```bash
# Create and enter the project folder
mkdir challenges_project
cd challenges_project

# Create a virtual environment named 'venv'
python -m venv venv

# Activate the virtual environment
# Windows (Command Prompt):
venv\Scripts\activate
# macOS/Linux:
source venv/bin/activate
```

_Listing 3.2: Creating and activating a Python virtual environment._

When the virtual environment is active, the terminal prompt is prefixed with `(venv)`, indicating that all subsequent Python and pip commands operate within the isolated environment.

**Step 3: Install Django**

```bash
pip install django
# Verify the Django version
django-admin --version
```

_Listing 3.3: Installing Django within the virtual environment._

**Step 4: Create a Django Project in the Current Directory**

```bash
# Create the project in the current directory (note the trailing dot)
django-admin startproject challenges_project .
```

_Listing 3.4: Creating a Django project in the current directory._

The trailing dot (`.`) instructs Django to place the project configuration package and `manage.py` directly in the current directory, rather than creating a redundant parent folder. The resulting directory structure is as follows:

```text
challenges_project/          # Project root (created manually)
│
├── venv/                    # Virtual environment (not part of Django)
├── manage.py                # Command-line utility for Django
│
└── challenges_project/      # Project configuration package
    ├── __init__.py          # Makes this directory a Python package
    ├── settings.py          # Project configuration
    ├── urls.py              # Root URL routing
    ├── asgi.py              # ASGI configuration (async deployment)
    └── wsgi.py              # WSGI configuration (standard deployment)
```

_Listing 3.5: Directory structure of a newly created Django project with a virtual environment._

### Key Project Files

**`manage.py`** is the command-line utility for interacting with the Django project. Common commands include:

```bash
python manage.py runserver        # Start the development server
python manage.py makemigrations   # Generate migration files from model changes
python manage.py migrate          # Apply migrations to the database
python manage.py createsuperuser  # Create an admin superuser
```

_Listing 3.6: Frequently used manage.py commands._

**`settings.py`** contains all project configuration, including database settings, the list of installed applications, middleware configuration, template engine settings, static file paths, and security settings.

**`urls.py`** (at the project level) defines the root URL routing table, mapping URL patterns to view functions or to application-level URL configurations.

### Setting Up the IDE

The following Visual Studio Code extensions are recommended for Django development:

1. **Python** — Core Python language support (IntelliSense, debugging, linting).
2. **Pylance** — Fast, feature-rich Python language server providing type checking and auto-completion.
3. **autopep8** — Automatic Python code formatting according to PEP 8 style guidelines.
4. **Django** — Syntax highlighting for Django template files.
5. **SQLite Viewer** — Visual browser for SQLite database files (Django's default database).
6. **djlint** — Formatting and linting for Django HTML templates.

After installing the extensions, configure the Python interpreter to use the project's virtual environment:

```json
// .vscode/settings.json
{
  "python.pythonPath": "venv/Scripts/python",
  "python.languageServer": "Pylance"
}
```

_Listing 3.7: VS Code workspace configuration for Django development._

**Installing djlint.** The `djlint` tool provides linting and formatting for Django HTML templates:

```bash
pip install djlint
```

_Listing 3.8: Installing djlint for Django template formatting._

Configure djlint as the default formatter for Django HTML files in the VS Code workspace:

```json
// .vscode/settings.json
{
  "emmet.includeLanguages": {
    "django-html": "html"
  },
  "[html][django-html]": {
    "editor.defaultFormatter": "monosans.djlint"
  }
}
```

_Listing 3.9: Configuring djlint as the Django HTML formatter in VS Code._

### Starting the Development Server

```bash
# Start the development server (default port 8000)
python manage.py runserver

# The server is accessible at http://127.0.0.1:8000/
# Press Ctrl+C to stop the server

# Start on a custom port
python manage.py runserver 8080
```

_Listing 3.10: Starting the Django development server._

When the server starts successfully, the terminal displays output similar to:

```text
Watching for file changes with StatReloader
Performing system checks...

System check identified no issues (0 silenced).

You have 18 unapplied migration(s). Your project may not work properly until
you apply the migrations for app(s): admin, auth, contenttypes, sessions.
Run 'python manage.py migrate' to apply them.
March 09, 2026 - 12:00:00
Django version 5.1, using settings 'challenges_project.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CTRL-BREAK.
```

_Listing 3.11: Terminal output when starting the Django development server for the first time._

Navigating to `http://127.0.0.1:8000/` in a browser displays Django's default welcome page, confirming that the project is configured correctly. The migration warning can be resolved by running `python manage.py migrate`.

### Django Apps

A Django project is composed of one or more apps. An app is a self-contained module that implements a specific piece of functionality (e.g., a blog, user authentication, an e-commerce catalogue). This modular design promotes code reuse—an app developed for one project can be installed in another.

**The distinction between a project and an app:**

- A project represents the entire web application and its configuration (settings, root URLs, deployment configuration).
- An app represents a specific feature or functional module within the project.

**Creating an app:**

```bash
python manage.py startapp challenges
```

_Listing 3.12: Creating a Django app._

This command generates the following directory structure:

```text
challenges/
├── __init__.py       # Makes this directory a Python package
├── admin.py          # Admin panel configuration
├── apps.py           # App configuration metadata
├── models.py         # Database models
├── tests.py          # Unit tests
├── views.py          # View functions/classes
└── migrations/       # Database migration files
    └── __init__.py
```

_Listing 3.13: Directory structure of a newly created Django app._

**Registering the app.** Every app must be registered in the project's `settings.py` file by adding it to the `INSTALLED_APPS` list:

```python
# challenges_project/settings.py
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'challenges',  # Register the custom app
]
```

_Listing 3.14: Registering a custom app in settings.py._

**Complete project structure with an app:**

```text
challenges_project/              # Project root
│
├── venv/                        # Virtual environment
├── manage.py                    # Command-line utility
│
├── challenges_project/          # Project configuration package
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py                  # Root URL routing
│   ├── asgi.py
│   └── wsgi.py
│
└── challenges/                  # Application package
    ├── __init__.py
    ├── admin.py
    ├── apps.py
    ├── models.py
    ├── views.py
    ├── urls.py                  # App-level URL routing (created manually)
    ├── tests.py
    └── migrations/
        └── __init__.py
```

_Listing 3.15: Complete project structure with one registered app._

---

## URLs and Views

### Routing in Django

Routing is the mechanism that maps incoming URLs to specific Python functions (views) that handle the request and produce a response. Django uses a URL dispatcher defined in `urls.py` files. The matching process works as follows:

1. A request arrives with a URL (e.g., `/challenges/january/`).
2. Django checks URL patterns in order, from top to bottom.
3. The first matching pattern wins.
4. The associated view function is called with the request object.
5. The view returns an HTTP response.

URLs map web addresses to views. Views are Python functions (or classes) that receive a web request and return a web response. Views contain the logic that processes requests.

### The Request Object

Every view function receives a `request` object as its first argument. This object encapsulates all information about the incoming HTTP request:

- `request.method` — The HTTP method (GET, POST, PUT, DELETE, etc.).
- `request.GET` — A dictionary-like object containing query string parameters.
- `request.POST` — A dictionary-like object containing POST data.
- `request.body` — The raw request body as bytes.
- `request.headers` — A dictionary of HTTP headers.
- `request.user` — The currently authenticated user.
- `request.session` — The session data dictionary.

### Handling HTTP Methods

A single view function can handle different HTTP methods by inspecting `request.method`:

```python
from django.http import HttpResponse

def article_view(request):
    if request.method == 'GET':
        return HttpResponse("Showing articles")
    elif request.method == 'POST':
        return HttpResponse("Creating article")
    elif request.method == 'DELETE':
        return HttpResponse("Deleting article")
```

_Listing 3.16: Handling multiple HTTP methods in a single view._

### Query Parameters

Query parameters from the URL's query string are accessed through `request.GET`:

```python
# URL: /search/?q=python&page=2

def search(request):
    query = request.GET.get('q', '')        # 'python'
    page = request.GET.get('page', '1')     # '2'
    return HttpResponse(f"Searching: {query}, Page: {page}")
```

_Listing 3.17: Accessing query parameters from the request._

### Creating Views and URL Configurations

The following walkthrough demonstrates the creation of views and URL patterns for a "Monthly Challenges" application.

**Step 1: Create the view function** in the app's `views.py`:

```python
# challenges/views.py
from django.http import HttpResponse

def index(request):
    return HttpResponse("Welcome to the Challenges App!")
```

_Listing 3.18: A simple view function returning an HTTP response._

**Step 2: Create app-level URL configuration.** Create a new file `challenges/urls.py` (this file is not generated automatically):

```python
# challenges/urls.py
from django.urls import path
from . import views

urlpatterns = [
    path('', views.index, name='index'),
]
```

_Listing 3.19: App-level URL configuration._

**Step 3: Include the app URLs in the project's root URL configuration:**

```python
# challenges_project/urls.py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('challenges/', include('challenges.urls')),
]
```

_Listing 3.20: Including app-level URLs in the root URL configuration._

With this configuration, navigating to `http://127.0.0.1:8000/challenges/` invokes the `index` view.

### Adding Multiple Views

Additional views are defined in `views.py` and mapped to URL patterns in the app's `urls.py`:

```python
# challenges/views.py
def january(request):
    return HttpResponse("January Challenge: Exercise daily!")

def february(request):
    return HttpResponse("February Challenge: Read a book!")
```

```python
# challenges/urls.py
urlpatterns = [
    path('', views.index, name='index'),
    path('january/', views.january, name='january'),
    path('february/', views.february, name='february'),
]
```

_Listing 3.21: Multiple views and their corresponding URL patterns._

### Dynamic Path Segments and Path Converters

Django supports dynamic URL segments using angle-bracket syntax. The captured value is passed as a keyword argument to the view function:

```python
# challenges/urls.py
urlpatterns = [
    path('<month>/', views.monthly_challenge, name='monthly-challenge'),
]

# challenges/views.py
def monthly_challenge(request, month):
    return HttpResponse(f"Challenge for {month}")
```

_Listing 3.22: Dynamic path segment capturing a string parameter._

**Path converters** constrain the type of value that a dynamic segment will match:

_Table 3.6: Django path converters._

| Converter | Matches                                               | Example           |
| :-------- | :---------------------------------------------------- | :---------------- |
| `str`     | Any non-empty string (default)                        | `<str:month>`     |
| `int`     | Positive integers                                     | `<int:month>`     |
| `slug`    | Slug strings (letters, numbers, hyphens, underscores) | `<slug:title>`    |
| `uuid`    | UUID strings                                          | `<uuid:pk>`       |
| `path`    | Any string, including forward slashes                 | `<path:filepath>` |

```python
path('<str:month>/', views.monthly_challenge),
path('<int:month>/', views.monthly_challenge_by_number),
path('<slug:title>/', views.post_detail),
```

_Listing 3.23: URL patterns using path converters._

### Dynamic View Logic with Data Dictionaries

The following example demonstrates a view that serves different content based on the captured URL parameter:

```python
# challenges/views.py
from django.http import HttpResponse, HttpResponseNotFound

monthly_challenges = {
    'january': 'Exercise daily for 30 minutes',
    'february': 'Read one book',
    'march': 'Learn something new each day',
    'april': 'Drink at least 2 liters of water daily',
    'may': 'Wake up early every day',
    'june': 'Practice a new skill for 20 minutes daily',
    'july': 'Avoid junk food for the entire month',
    'august': 'Write a daily journal entry',
    'september': 'Learn and revise one topic each day',
    'october': 'Limit social media usage to 30 minutes per day',
    'november': 'Express gratitude by writing one thankful note daily',
    'december': 'Reflect on the year and plan goals for next year',
}

def monthly_challenge(request, month):
    try:
        challenge_text = monthly_challenges[month.lower()]
        return HttpResponse(challenge_text)
    except KeyError:
        return HttpResponseNotFound("This month is not supported!")
```

_Listing 3.24: A view using a dictionary to serve dynamic content based on the URL._

### Redirects

Django provides two mechanisms for redirecting users to a different URL:

```python
from django.http import HttpResponseRedirect
from django.shortcuts import redirect

# Method 1: HttpResponseRedirect (explicit)
def old_url(request):
    return HttpResponseRedirect('/challenges/january/')

# Method 2: redirect shortcut (preferred)
def monthly_challenge_by_number(request, month):
    months = list(monthly_challenges.keys())
    if month > len(months):
        return HttpResponse("Invalid month", status=404)
    return redirect(f'/challenges/{months[month - 1]}/')
```

_Listing 3.25: Redirecting using HttpResponseRedirect and the redirect shortcut._

### The reverse() Function and Named URLs

Hardcoding URLs in view functions is fragile—if a URL pattern changes, every reference must be updated manually. The `reverse()` function resolves a URL dynamically from its name, ensuring consistency:

```python
from django.urls import reverse

def monthly_challenge_by_number(request, month):
    months = list(monthly_challenges.keys())
    redirect_month = months[month - 1]
    redirect_url = reverse('monthly-challenge', args=[redirect_month])
    return redirect(redirect_url)
```

_Listing 3.26: Using reverse() with named URLs for dynamic URL resolution._

### Returning HTML from Views

Views can return HTML content directly as strings, although this approach is suitable only for simple cases. For anything beyond trivial HTML, templates should be used (covered in Section 3.5):

```python
def index(request):
    list_items = ""
    months = list(monthly_challenges.keys())

    for month in months:
        capitalized_month = month.capitalize()
        month_path = reverse("monthly-challenge", args=[month])
        list_items += f'<li><a href="{month_path}">{capitalized_month}</a></li>'

    response_data = f"<ul>{list_items}</ul>"
    return HttpResponse(response_data)
```

_Listing 3.27: Dynamically generating HTML with a list of links in a view function._

While functional, embedding HTML strings within Python code mixes presentation with logic. Django's template system, discussed in the next section, separates these concerns.

---

## Templates and Static Files

### Introduction to Templating

Templating is the process of generating dynamic HTML by combining a template (the HTML structure) with data from the view. Templates separate presentation from logic, adhering to Django's MTV architecture.

A Django template is a text file—typically HTML—that contains static content mixed with **Django Template Language (DTL)** constructs. DTL provides three primary mechanisms:

- **Variable interpolation** with `{{ variable }}` — Outputs the value of a variable.
- **Template tags** with `{% tag %}` — Provides control flow logic (conditionals, loops).
- **Filters** with `{{ variable|filter }}` — Transforms data for display (e.g., converting to uppercase, truncating text).

### JSON Responses

For API endpoints, views return JSON data instead of HTML. Django provides `JsonResponse` for this purpose:

```python
from django.http import JsonResponse

def api_articles(request):
    data = {
        'articles': [
            {'id': 1, 'title': 'First'},
            {'id': 2, 'title': 'Second'},
        ],
        'count': 2
    }
    return JsonResponse(data)
```

_Listing 3.28: Returning a JSON response from a Django view._

The resulting HTTP response contains:

```json
{
  "articles": [
    { "id": 1, "title": "First" },
    { "id": 2, "title": "Second" }
  ],
  "count": 2
}
```

### Adding and Registering Templates

Templates are stored within a conventional directory structure inside each app. Django's template loader requires a specific nested folder pattern to avoid naming conflicts between apps:

```text
challenges/
└── templates/
    └── challenges/
        └── challenge.html
```

_Listing 3.29: Conventional template directory structure within a Django app._

The inner `challenges/` folder prevents naming collisions when multiple apps contain templates with the same filename. For example, `challenges/challenge.html` and `blog/challenge.html` can coexist without ambiguity.

The app must be registered in `settings.py` (as shown in Listing 3.10) for Django's template engine to discover its templates.

### Rendering Templates

Django provides two approaches for rendering templates.

**Using `render_to_string`** — Renders the template to a string, which is then wrapped in an `HttpResponse`:

```python
# challenges/views.py
from django.template.loader import render_to_string
from django.http import HttpResponse, HttpResponseNotFound

def monthly_challenge(request, month):
    try:
        challenge_text = monthly_challenges[month.lower()]
        response_data = render_to_string("challenges/challenge.html")
        return HttpResponse(response_data)
    except KeyError:
        return HttpResponseNotFound("<h1>This month is not supported!</h1>")
```

_Listing 3.30: Rendering a template using render_to_string._

**Using the `render` shortcut** (preferred) — Combines template rendering and response creation in a single call:

```python
# challenges/views.py
from django.shortcuts import render
from django.http import HttpResponseNotFound

def monthly_challenge(request, month):
    try:
        challenge_text = monthly_challenges[month.lower()]
        return render(request, "challenges/challenge.html")
    except KeyError:
        return HttpResponseNotFound("<h1>This month is not supported!</h1>")
```

_Listing 3.31: Rendering a template using the render shortcut._

### Template Variable Interpolation

Data is passed from the view to the template as a dictionary (the template context). Within the template, variables are output using double curly braces `{{ }}`:

```html
<pre>
<!-- challenges/templates/challenges/challenge.html -->
<h1>{{ month_name }} Challenge</h1>
<h2>{{ text }}</h2>
</pre>
```

```python
# challenges/views.py
def monthly_challenge(request, month):
    try:
        challenge_text = monthly_challenges[month.lower()]
        return render(request, "challenges/challenge.html", {
            "text": challenge_text,
            "month_name": month.capitalize()
        })
    except KeyError:
        return HttpResponseNotFound("<h1>This month is not supported!</h1>")
```

_Listing 3.32: Passing context variables to a template._

The dictionary `{"text": challenge_text, "month_name": month.capitalize()}` constitutes the template context. The keys become variable names accessible within the template.

### Template Filters

Filters transform the value of a variable for display purposes. They are applied using the pipe (`|`) syntax:

```html
<pre>
{{ name|upper }}              <!-- JOHN -->
{{ name|lower }}              <!-- john -->
{{ name|title }}              <!-- John Doe -->
{{ text|truncatewords:20 }}   <!-- First 20 words... -->
{{ date|date:"Y-m-d" }}      <!-- 2026-01-17 -->
{{ price|floatformat:2 }}     <!-- 19.99 -->
{{ list|length }}             <!-- 5 -->
{{ value|default:"N/A" }}    <!-- Shows "N/A" if the value is empty -->
{{ html_content|safe }}       <!-- Renders raw HTML (use with caution) -->
</pre>
```

_Listing 3.33: Commonly used Django template filters._

The `title` filter capitalises the first letter of each word, `truncatewords` limits the output to a specified number of words, and `safe` marks content as safe for rendering without HTML escaping (this should be used only with trusted content).

```html
<pre>
<head>
  <title>{{ month_name|title }} Challenge</title>
</head>
<body>
  <h1>{{ month_name|title }} Challenge</h1>
  <h2>{{ text }}</h2>
</body>
</pre>
```

_Listing 3.34: Applying the title filter in a template._

### The for Tag — Looping in Templates

The `{% for %}` tag iterates over a sequence, rendering its body once for each item. The `forloop` context variable provides iteration metadata such as `forloop.counter` (1-indexed iteration count).

```python
# challenges/views.py
def index(request):
    months = list(monthly_challenges.keys())
    return render(request, "challenges/index.html", {
        "months": months
    })
```

```html
<pre>
<!-- challenges/templates/challenges/index.html -->
<ul>
  {% for month in months %}
  <li>
    <a href="/challenges/{{ month }}">
      {{ forloop.counter }} - {{ month|title }}
    </a>
  </li>
  {% endfor %}
</ul>
</pre>
```

_Listing 3.35: Using the for tag to iterate over a list in a template._

### The url Tag — Dynamic URL Generation in Templates

Hardcoding URLs in templates is fragile. The `{% url %}` tag generates URLs dynamically from their registered names, ensuring that templates remain correct even if URL patterns change:

```html
<pre>
<!-- challenges/templates/challenges/index.html -->
<ul>
  {% for month in months %}
  <li>
    <a href="{% url 'monthly-challenge' month %}">
      {{ forloop.counter }} - {{ month|title }}
    </a>
  </li>
  {% endfor %}
</ul>
</pre>
```

_Listing 3.36: Using the url tag for dynamic URL generation._

The `{% url 'monthly-challenge' month %}` expression resolves to the URL pattern named `monthly-challenge`, passing `month` as the dynamic path argument.

### The if Tag — Conditional Rendering

The `{% if %}` tag conditionally includes or excludes content based on a Boolean expression:

```python
# challenges/views.py
monthly_challenges = {
    'january': 'Exercise daily for 30 minutes',
    # ...
    'december': None  # No challenge defined
}
```

```html
<pre>
<!-- challenges/templates/challenges/challenge.html -->
<h1>{{ month_name|title }} Challenge</h1>
{% if text is not None %}
  <h2>{{ text }}</h2>
{% else %}
  <p>There is no challenge for this month yet!</p>
{% endif %}
</pre>
```

_Listing 3.37: Conditional rendering with the if tag._

### Template Inheritance

Template inheritance is Django's mechanism for creating a consistent page layout across an application. A **base template** defines the overall structure and declares named blocks that child templates can override.

**Step 1: Configure a global templates directory** by updating `settings.py`:

```python
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [
            BASE_DIR / "templates"
        ],
        'APP_DIRS': True,
        # ...
    }
]
```

_Listing 3.38: Adding a global templates directory in settings.py._

**Step 2: Create the base template** (`templates/base.html`):

```html
<pre>
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>{% block page_title %}My Challenges{% endblock page_title %}</title>
    {% block css_files %}{% endblock css_files %}
  </head>
  <body>
    {% block content %}{% endblock content %}
  </body>
</html>
</pre>
```

_Listing 3.39: Base template with named blocks._

**Step 3: Create child templates** that extend the base:

```html
<pre>
<!-- challenges/templates/challenges/index.html -->
{% extends "base.html" %}

{% block page_title %}
  All Challenges
{% endblock page_title %}

{% block content %}
  <ul>
    {% for month in months %}
      <li><a href="{% url 'monthly-challenge' month %}">{{ month|title }}</a></li>
    {% endfor %}
  </ul>
{% endblock content %}
</pre>
```

_Listing 3.40: Child template extending the base template._

```html
<pre>
<!-- challenges/templates/challenges/challenge.html -->
{% extends "base.html" %}

{% block page_title %}
  {{ month_name|title }} Challenge
{% endblock page_title %}

{% block content %}
  <h1>{{ month_name|title }} Challenge</h1>
  {% if text is not None %}
    <h2>{{ text }}</h2>
  {% else %}
    <p>There is no challenge for this month yet!</p>
  {% endif %}
{% endblock content %}
</pre>
```

_Listing 3.41: Another child template overriding base blocks._

The `{% extends "base.html" %}` directive must be the first tag in the child template. Each `{% block %}` in the child replaces the corresponding block in the base template.

### Including Partial Template Snippets

The `{% include %}` tag inserts the content of another template file at the point of inclusion. This is useful for reusable UI components such as navigation headers and footers:

**Create a partial template** (`challenges/templates/challenges/includes/header.html`):

```html
<pre>
<header>
  <nav>
    <a href="{% url 'index' %}">All Challenges</a>
  </nav>
</header>
</pre>
```

_Listing 3.42: A reusable header partial template._

**Include the partial in other templates:**

```html
<pre>
{% block content %}
  {% include "challenges/includes/header.html" %}
  <!-- Remaining content -->
{% endblock content %}
</pre>
```

_Listing 3.43: Including a partial template._

Included templates have access to all context variables available to the parent template. Additional variables can be passed using the `with` keyword:

```html
<pre>
{% include "challenges/includes/header.html" with title="Challenges" subtitle="Monthly" %}
</pre>
```

_Listing 3.44: Passing extra variables to an included template._

### Custom 404 Templates

Django allows custom error pages. To create a custom 404 page, place a `404.html` template in the global templates directory:

```html
<pre>
<!-- templates/404.html -->
{% extends "base.html" %}

{% block page_title %}
  Page Not Found
{% endblock page_title %}

{% block content %}
  <h1>We could not find that page!</h1>
  <p>Sorry, but we could not find a matching page.</p>
{% endblock content %}
</pre>
```

_Listing 3.45: Custom 404 error template._

To trigger a 404 response from a view, raise the `Http404` exception:

```python
from django.http import Http404

def monthly_challenge(request, month):
    try:
        challenge_text = monthly_challenges[month.lower()]
        return render(request, "challenges/challenge.html", {
            "text": challenge_text,
            "month_name": month
        })
    except KeyError:
        raise Http404()
```

_Listing 3.46: Raising Http404 to trigger the custom 404 template._

> **Note:** Custom error pages are displayed only when `DEBUG = False` in `settings.py`. During development (when `DEBUG = True`), Django shows its default debug error page. When setting `DEBUG = False`, you must also set `ALLOWED_HOSTS` (e.g., `ALLOWED_HOSTS = ['*']` for testing purposes).

### Static Files

Static files are assets that do not change dynamically—CSS stylesheets, JavaScript files, images, and fonts. Django provides a built-in static file serving system for development.

**App-level static files** follow a directory convention similar to templates:

```text
challenges/
└── static/
    └── challenges/
        ├── css/
        │   └── challenges.css
        └── images/
            └── logo.png
```

_Listing 3.47: App-level static file directory structure._

**Loading static files in templates** requires two steps: loading the `static` template tag library with `{% load static %}` and referencing files using the `{% static %}` tag:

```css
/* challenges/static/challenges/css/challenges.css */
ul {
  list-style: none;
}
```

_Listing 3.48: A simple CSS stylesheet for the challenges app._

First, add a block for CSS files in the base template:

```html
<pre>
<!-- templates/base.html -->
<head>
  {% block css_files %}{% endblock css_files %}
</head>
</pre>
```

Then load the stylesheet in the child template:

```html
<pre>
<!-- challenges/templates/challenges/index.html -->
{% load static %}

{% block css_files %}
  <link rel="stylesheet" href="{% static 'challenges/css/challenges.css' %}" />
{% endblock css_files %}
</pre>
```

_Listing 3.49: Loading a static CSS file in a template._

### Global Static Files

Static files shared across multiple apps (such as a global stylesheet or a common JavaScript library) are placed in a project-level `static/` directory. This directory must be registered in `settings.py`:

```python
# settings.py
STATIC_URL = '/static/'

STATICFILES_DIRS = [
    BASE_DIR / 'static',  # Project-level static files
]
```

_Listing 3.50: Configuring the global static files directory._

```css
/* static/styles.css */
@import url("https://fonts.googleapis.com/css2?family=Roboto+Condensed:wght@400;700&display=swap");

html {
  font-family: "Roboto Condensed", sans-serif;
}

body {
  margin: 0;
  background-color: #1a1a1a;
}
```

_Listing 3.51: A global stylesheet loaded from the project-level static directory._

The global stylesheet is loaded in the base template so that it applies to all pages:

```html
<pre>
<!-- templates/base.html -->
{% load static %}

<head>
  <link rel="stylesheet" href="{% static 'styles.css' %}" />
  {% block css_files %}{% endblock css_files %}
</head>
</pre>
```

_Listing 3.52: Loading the global stylesheet in the base template._

Individual app templates can then load their own additional stylesheets in the `css_files` block:

```html
<pre>
<!-- challenges/templates/challenges/challenge.html -->
{% extends "base.html" %}
{% load static %}

{% block css_files %}
  <link rel="stylesheet" href="{% static 'challenges/css/challenge.css' %}" />
  <link rel="stylesheet" href="{% static 'challenges/includes/header.css' %}" />
{% endblock css_files %}
</pre>
```

_Listing 3.53: Loading app-specific stylesheets alongside the global stylesheet._

```html
<pre>
<!-- challenges/templates/challenges/index.html -->
{% extends "base.html" %}
{% load static %}

{% block css_files %}
  <link rel="stylesheet" href="{% static 'challenges/css/challenges.css' %}" />
  <link rel="stylesheet" href="{% static 'challenges/includes/header.css' %}" />
{% endblock css_files %}
</pre>
```

_Listing 3.54: Loading stylesheets in the index template._

This layered approach—a global stylesheet in the base template and app-specific stylesheets in child templates—ensures visual consistency across the application while allowing per-page customisation.

---

## Data and Models

### Introduction to Object-Relational Mapping (ORM)

Object-Relational Mapping (ORM) is a programming technique that provides a bridge between a relational database and an object-oriented programming language. Instead of writing raw SQL queries, developers interact with the database through the language's native objects and methods. The ORM translates these high-level operations into the appropriate SQL statements automatically.

The advantages of using an ORM are significant. First, developers write Python (or another host language) instead of SQL, which keeps the codebase uniform and easier to maintain. Second, ORM-generated queries are database-agnostic—the same Python code can target SQLite during development and PostgreSQL in production without modification. Third, the ORM automatically parameterises queries, providing built-in protection against SQL injection attacks. Finally, an object-oriented interface to the database is inherently more readable and more consistent with the rest of the application code.

![Class to Table Mapping](/images/class-to-table-mapping.webp)

_Figure 3.4: Mapping between Python classes and relational database tables in an ORM._

Django supports several database backends:

- **SQLite** — The default, file-based database. Suitable for development and small-scale applications.
- **PostgreSQL** — The recommended database for production deployments.
- **MySQL** — Widely used in the web hosting industry.
- **Oracle** — Used in enterprise environments.

### Django's Built-In ORM

Django includes a powerful ORM as part of its "batteries included" philosophy. Models are defined as Python classes that inherit from `django.db.models.Model`. Each class maps to a database table, and each class attribute maps to a table column.

To demonstrate the ORM, consider a fresh project for a book catalogue application:

```bash
# Create a new project and app
django-admin startproject book_store .
python manage.py startapp book_outlet
```

_Listing 3.55: Creating the book_store project and book_outlet app._

After creating the app, register it in `settings.py`:

```python
# book_store/settings.py
INSTALLED_APPS = [
    # ... default apps ...
    'book_outlet',
]
```

_Listing 3.56: Registering the book_outlet app._

**Defining a model.** Models are defined in the app's `models.py` file. The following model defines a `Book` table with `title` and `rating` columns:

```python
# book_outlet/models.py
from django.db import models

class Book(models.Model):
    title = models.CharField(max_length=50)
    rating = models.IntegerField()
```

_Listing 3.57: A simple Book model._

Django automatically adds an `id` field as the primary key. The model above is equivalent to the following SQL:

```sql
CREATE TABLE book_outlet_book (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    title VARCHAR(50) NOT NULL,
    rating INTEGER NOT NULL
);
```

_Listing 3.58: SQL equivalent of the Book model._

### Common Model Field Types

Django provides a comprehensive set of field types, each mapping to an appropriate database column type.

_Table 3.7: Common Django model field types._

| Field Type        | Description                      | Example                               |
| :---------------- | :------------------------------- | :------------------------------------ |
| `CharField`       | Short text (requires max_length) | `name = CharField(max_length=100)`    |
| `TextField`       | Long text                        | `bio = TextField()`                   |
| `IntegerField`    | Integer numbers                  | `age = IntegerField()`                |
| `FloatField`      | Decimal numbers                  | `price = FloatField()`                |
| `BooleanField`    | True/False values                | `active = BooleanField(default=True)` |
| `DateField`       | Date only                        | `birth_date = DateField()`            |
| `DateTimeField`   | Date and time                    | `created_at = DateTimeField()`        |
| `EmailField`      | Email with validation            | `email = EmailField()`                |
| `URLField`        | URL with validation              | `website = URLField()`                |
| `ForeignKey`      | Many-to-one relationship         | `author = ForeignKey(Author)`         |
| `ManyToManyField` | Many-to-many relationship        | `tags = ManyToManyField(Tag)`         |

### Field Options

Field options are keyword arguments passed to field constructors to control their behaviour at both the database and form levels.

_Table 3.8: Common field options._

| Option         | Description                                                 |
| :------------- | :---------------------------------------------------------- |
| `max_length`   | Maximum character length (required for CharField)           |
| `default`      | Default value when none is provided                         |
| `null=True`    | Allow NULL values in the database                           |
| `blank=True`   | Allow empty values in forms                                 |
| `unique=True`  | Enforce unique values across all rows                       |
| `choices`      | Restrict the field to a predefined set of values            |
| `auto_now_add` | Automatically set to the current timestamp on creation      |
| `auto_now`     | Automatically update to the current timestamp on every save |

> **Note:** `null=True` and `blank=True` serve different purposes. `null` controls the database constraint (whether a NULL value is permitted in the column), while `blank` controls form validation (whether the field may be left empty when submitting a form). For string-based fields, Django convention is to use `blank=True` with `default=""` rather than `null=True`, because Django stores empty strings rather than NULL for text fields.

### Migrations

After defining or modifying models, the changes must be propagated to the database through Django's migration system. Migrations are version-controlled files that describe incremental changes to the database schema.

```bash
# Step 1: Generate migration files based on model changes
python manage.py makemigrations

# Step 2: Apply the migrations to the database
python manage.py migrate
```

_Listing 3.59: Creating and applying migrations._

The migration workflow performs the following operations:

1. **Detection** — Django compares the current model definitions against the previous migration files and identifies the changes.
2. **Generation** — A new migration file is created in the app's `migrations/` directory, containing the SQL operations needed to update the schema.
3. **Versioning** — Each migration is numbered sequentially, enabling Django to track which migrations have been applied.
4. **Application** — Running `migrate` executes the pending migration files against the database, altering its schema accordingly.
5. **Reversal** — Migrations can be reversed to undo schema changes, which is useful during development.

### CRUD Operations via the Django Shell

The Django interactive shell provides a convenient environment for performing database operations directly. It is invoked with:

```bash
python manage.py shell
```

_Listing 3.60: Opening the Django interactive shell._

**Creating records.** New records can be created by instantiating a model object and calling its `save()` method, or by using the `create()` shortcut on the model's manager:

```python
from book_outlet.models import Book

# Method 1: Instantiate and save
harry_potter = Book(title="Harry Potter 1 – The Philosopher's Stone", rating=5)
harry_potter.save()

# Method 2: Use the create() shortcut (saves automatically)
Book.objects.create(title="Lord of the Rings", rating=4, author="J.R.R. Tolkien", is_bestselling=True)
```

_Listing 3.61: Creating records using the Django ORM._

The `save()` method generates and executes an `INSERT` statement:

```sql
INSERT INTO book_outlet_book (title, rating)
VALUES ('Harry Potter 1 – The Philosopher''s Stone', 5);
```

_Listing 3.62: SQL equivalent of the ORM create operation._

**Reading records.** The `all()` method retrieves all records from the table as a `QuerySet`:

```python
Book.objects.all()
# <QuerySet [<Book: Harry Potter 1 (5)>, <Book: Lord of the Rings (4)>]>
```

_Listing 3.63: Retrieving all records._

**Updating records.** To update a record, modify its attributes and call `save()`:

```python
harry_potter = Book.objects.all()[0]
harry_potter.author = "J.K. Rowling"
harry_potter.is_bestselling = True
harry_potter.save()
```

_Listing 3.64: Updating a record._

**Deleting records.** The `delete()` method removes a record from the database:

```python
book = Book.objects.all()[0]
book.delete()
# (1, {'book_outlet.Book': 1})
```

_Listing 3.65: Deleting a record._

**Adding a `__str__` method.** By default, model instances are displayed as `Book object (1)`. Defining a `__str__` method provides a human-readable representation:

```python
# book_outlet/models.py
from django.db import models
from django.core.validators import MinValueValidator, MaxValueValidator

class Book(models.Model):
    title = models.CharField(max_length=50)
    rating = models.IntegerField(
        validators=[MinValueValidator(1), MaxValueValidator(5)])
    author = models.CharField(null=True, max_length=100)
    is_bestselling = models.BooleanField(default=False)

    def __str__(self):
        return f"{self.title} ({self.rating})"
```

_Listing 3.66: Enhanced Book model with validators and a string representation._

After modifying the model, migrations must be regenerated and applied:

```bash
python manage.py makemigrations
python manage.py migrate
```

_Listing 3.67: Regenerating migrations after model changes._

### Retrieving Data — get, filter, and QuerySet Methods

The Django ORM provides several methods for querying data with varying levels of specificity.

**The `get()` method** retrieves a single record matching the given conditions. It raises `DoesNotExist` if no match is found and `MultipleObjectsReturned` if more than one record matches:

```python
Book.objects.get(title="My Story")
# <Book: My Story (2)>

Book.objects.get(is_bestselling=True)
# Raises MultipleObjectsReturned if more than one bestseller exists
```

_Listing 3.68: Using the get() method._

**The `filter()` method** returns a QuerySet containing all records that match the specified conditions. Multiple conditions are combined with AND logic:

```python
Book.objects.filter(is_bestselling=True)
# <QuerySet [<Book: Lord of the Rings (4)>, <Book: Harry Potter 1 (5)>]>

Book.objects.filter(is_bestselling=False, rating=2)
# <QuerySet [<Book: My Story (2)>]>
```

_Listing 3.69: Using the filter() method with multiple conditions._

_Table 3.9: Summary of common QuerySet methods._

| Method       | Description                                                                 |
| :----------- | :-------------------------------------------------------------------------- |
| `all()`      | Returns all records                                                         |
| `get()`      | Returns a single record (raises an error if not found or if multiple exist) |
| `filter()`   | Returns records matching the specified conditions                           |
| `exclude()`  | Returns records NOT matching the specified conditions                       |
| `order_by()` | Sorts records by the specified field(s)                                     |
| `first()`    | Returns the first record in the QuerySet                                    |
| `last()`     | Returns the last record in the QuerySet                                     |
| `count()`    | Returns the number of records                                               |
| `exists()`   | Returns `True` if records exist, `False` otherwise                          |
| `values()`   | Returns dictionaries instead of model instances                             |
| `distinct()` | Removes duplicate rows from the result                                      |

### Lookup Expressions (Field Lookups)

Django's field lookups enable comparison operators within queries. Lookups are appended to field names using double underscores (`__`):

```python
# Standard Python comparison operators do not work in filter()
Book.objects.filter(rating < 3)    # SyntaxError!

# Use Django's lookup syntax instead
Book.objects.filter(rating__lt=3)
# <QuerySet [<Book: My Story (2)>, <Book: Some random book (1)>]>

Book.objects.filter(rating__lt=3, title__contains="Story")
# <QuerySet [<Book: My Story (2)>]>
```

_Listing 3.70: Using field lookups in filter queries._

_Table 3.10: Django field lookup expressions._

| Lookup       | Description               | Example                             |
| :----------- | :------------------------ | :---------------------------------- |
| `exact`      | Exact match               | `filter(name__exact='John')`        |
| `iexact`     | Case-insensitive exact    | `filter(name__iexact='john')`       |
| `contains`   | Contains substring        | `filter(title__contains='Python')`  |
| `icontains`  | Case-insensitive contains | `filter(title__icontains='python')` |
| `startswith` | Starts with               | `filter(name__startswith='J')`      |
| `endswith`   | Ends with                 | `filter(email__endswith='.com')`    |
| `gt`         | Greater than              | `filter(age__gt=18)`                |
| `gte`        | Greater than or equal     | `filter(age__gte=18)`               |
| `lt`         | Less than                 | `filter(price__lt=100)`             |
| `lte`        | Less than or equal        | `filter(price__lte=100)`            |
| `in`         | In a list of values       | `filter(id__in=[1, 2, 3])`          |
| `isnull`     | Is NULL                   | `filter(bio__isnull=True)`          |

### Complex Queries with Q Objects

By default, multiple keyword arguments passed to `filter()` are combined with AND logic. To construct OR queries or negate conditions, Django provides the `Q` object from `django.db.models`:

```python
from django.db.models import Q

# OR query: rating < 3 OR is_bestselling = True
Book.objects.filter(Q(rating__lt=3) | Q(is_bestselling=True))
# <QuerySet [<Book: Lord of the Rings (4)>, <Book: Harry Potter 1 (5)>,
#            <Book: My Story (2)>, <Book: Some random book (1)>]>

# Combining Q objects with keyword arguments (AND logic with OR inside)
Book.objects.filter(Q(rating__lt=3) | Q(is_bestselling=True), author="J.K. Rowling")
# <QuerySet [<Book: Harry Potter 1 (5)>]>
```

_Listing 3.71: Constructing OR queries using Q objects._

> **Important:** When mixing `Q` objects with keyword arguments in a `filter()` call, all `Q` objects must appear before any keyword arguments. Placing keyword arguments before `Q` objects results in a `SyntaxError`, because Python requires positional arguments to precede keyword arguments.

**Case-insensitive lookups.** The `icontains` lookup performs case-insensitive substring matching, which is reliable across all database backends (unlike `contains`, whose case sensitivity depends on the database collation):

```python
Book.objects.filter(rating__lt=3, title__icontains="story")
# <QuerySet [<Book: My Story (2)>]>
```

_Listing 3.72: Case-insensitive filtering with icontains._

### QuerySet Chaining, Lazy Evaluation, Ordering, and Aggregation

**Lazy evaluation.** QuerySets are lazily evaluated—Django does not execute a database query until the QuerySet is actually consumed (e.g., iterated over, sliced, printed, or converted to a list). This allows queries to be chained and refined without incurring additional database hits:

```python
bestsellers = Book.objects.filter(is_bestselling=True)        # No DB query yet
amazing_bestsellers = bestsellers.filter(rating__gt=4)         # Still no DB query
print(amazing_bestsellers)  # DB query executes HERE
# <QuerySet [<Book: Harry Potter 1 (5)>]>
```

_Listing 3.73: Demonstrating lazy evaluation by chaining QuerySet filters._

The original `bestsellers` QuerySet remains unmodified. Chaining produces a new QuerySet at each step.

**Ordering.** The `order_by()` method sorts records. A field name prefixed with `-` (hyphen) specifies descending order:

```python
Book.objects.order_by('title')   # Ascending by title
Book.objects.order_by('-title')  # Descending by title
```

_Listing 3.74: Ordering QuerySets._

**Aggregation.** Aggregate functions compute a single summary value across a QuerySet:

```python
from django.db.models import Count, Avg

Book.objects.aggregate(Avg('rating'))
# {'rating__avg': 3.0}

Book.objects.aggregate(total=Count('id'))
# {'total': 4}
```

_Listing 3.75: Aggregate queries using Avg and Count._

### Integrating Models with Views and Templates

With models and CRUD operations understood, the next step is to connect the data layer to the presentation layer. The following walkthrough demonstrates how to build a book catalogue that lists all books and provides a detail page for each.

**Step 1: Create the base template** (`book_outlet/templates/book_outlet/base.html`):

```html
<pre>
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>{% block title %}{% endblock title %}</title>
  </head>
  <body>
    {% block content %} {% endblock content %}
  </body>
</html>
</pre>
```

_Listing 3.76: Base template for the book outlet app._

**Step 2: Create the index view and template.** The view retrieves all books from the database and passes them to the template:

```python
# book_outlet/views.py
from django.shortcuts import render
from .models import Book

def index(request):
    books = Book.objects.all()
    return render(request, "book_outlet/index.html", {
        "books": books
    })
```

_Listing 3.77: Index view passing all books to the template._

```html
<pre>
{% extends "book_outlet/base.html" %}

{% block title %}
  All Books
{% endblock title %}

{% block content %}
  <ul>
    {% for book in books %}
      <li>
        <a href="{% url 'book-detail' book.id %}">
          {{ book.title }}
        </a>
        (Rating: {{ book.rating }})
      </li>
    {% endfor %}
  </ul>
{% endblock content %}
</pre>
```

_Listing 3.78: Index template listing all books with links to detail pages._

**Step 3: Create the detail view and template.** The `get_object_or_404` shortcut retrieves a single record or raises a 404 error if it does not exist:

```python
# book_outlet/views.py
from django.shortcuts import get_object_or_404, render
# from django.http import Http404

def book_detail(request, id):
    # try:
    #     book = Book.objects.get(pk=id)
    # except:
    #     raise Http404()
    book = get_object_or_404(Book, pk=id)
    return render(request, "book_outlet/book_detail.html", {
        "title": book.title,
        "author": book.author,
        "rating": book.rating,
        "is_bestseller": book.is_bestselling
    })
```

_Listing 3.79: Detail view using get_object_or_404._

```html
<pre>
{% extends "book_outlet/base.html" %}

{% block title %}
  {{ title }}
{% endblock %}

{% block content %}
  <h1>{{ title }}</h1>
  <h2>{{ author }}</h2>
  <p>The book has a rating of {{ rating }}
  {% if is_bestseller %}
    and is a bestseller.
  {% else %}
    but is not a bestseller.
  {% endif %}
  </p>
{% endblock %}
</pre>
```

_Listing 3.80: Book detail template with conditional rendering._

**Step 4: Configure URLs.** Register the app's URL patterns at both the project and app levels:

```python
# book_store/urls.py (project level)
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path("", include("book_outlet.urls"))
]
```

```python
# book_outlet/urls.py (app level)
from django.urls import path
from . import views

urlpatterns = [
    path("", views.index),
    path("<int:id>", views.book_detail, name="book-detail")
]
```

_Listing 3.81: URL configurations for the book outlet app._

This example demonstrates the complete data flow in Django's MTV architecture: the Model defines the data structure, the View retrieves data from the database and prepares it for display, and the Template renders the HTML presentation. The `{% url %}` tag generates links to named URL patterns, and the `{% if %}` tag provides conditional rendering based on model attributes.

### Using Slugs Instead of IDs

Integer primary keys (e.g., `/books/3`) are functional but expose internal database identifiers in the URL. Slugs provide human-readable, SEO-friendly URLs (e.g., `/books/harry-potter-1`) that are both more descriptive and more stable.

**Updating the model.** A `SlugField` is added to the `Book` model. The `save()` method is overridden to auto-generate the slug from the title using Django's `slugify` utility, and `get_absolute_url()` is defined to return the canonical URL for each book:

```python
# book_outlet/models.py
from django.db import models
from django.core.validators import MinValueValidator, MaxValueValidator
from django.urls import reverse
from django.utils.text import slugify


class Book(models.Model):
    title = models.CharField(max_length=50)
    rating = models.IntegerField(
        validators=[MinValueValidator(1), MaxValueValidator(5)])
    author = models.CharField(null=True, max_length=100)
    is_bestselling = models.BooleanField(default=False)
    # Harry Potter 1 => harry-potter-1
    slug = models.SlugField(default="", null=False, db_index=True, unique=True)

    def get_absolute_url(self):
        return reverse("book-detail", args=[self.slug])

    def save(self, *args, **kwargs):
        self.slug = slugify(self.title)
        super().save(*args, **kwargs)

    def __str__(self):
        return f"{self.title} ({self.rating})"
```

_Listing: Updated Book model with SlugField, get_absolute_url, and overridden save._

After adding the `slug` field, migrations must be generated and applied:

```bash
python manage.py makemigrations
python manage.py migrate
```

_Listing: Creating and applying the slug migration._

**Populating slugs for existing records.** Because the `slug` field is derived from the title during `save()`, existing records must be re-saved to generate their slugs:

```python
Book.objects.get(title="Harry Potter 1").save()
Book.objects.get(title="Harry Potter 1").slug
# 'harry-potter-1'

Book.objects.get(title="Lord of the Rings").save()
Book.objects.get(title="Lord of the Rings").slug
# 'lord-of-the-rings'

Book.objects.get(title="My Story").save()
Book.objects.get(title="Some random book").save()
```

_Listing: Re-saving existing records to populate slugs._

**Updating the URL configuration.** The URL pattern is changed from `<int:id>` to `<slug:slug>`, which matches URL-friendly strings containing letters, numbers, hyphens, and underscores:

```python
# book_outlet/urls.py
from django.urls import path
from . import views

urlpatterns = [
    path("", views.index),
    path("<slug:slug>", views.book_detail, name="book-detail")
]
```

_Listing: URL configuration updated to use slug-based routing._

**Updating the view.** The view parameter and lookup field are changed from `id` to `slug`:

```python
# book_outlet/views.py
def book_detail(request, slug):
    book = get_object_or_404(Book, slug=slug)
    return render(request, "book_outlet/book_detail.html", {
        "title": book.title,
        "author": book.author,
        "rating": book.rating,
        "is_bestseller": book.is_bestselling
    })
```

_Listing: Detail view updated to accept and look up by slug._

**Updating the index template.** Instead of manually constructing URLs with `{% url %}`, the template now calls `get_absolute_url` on each book instance, which returns the slug-based URL:

```html
<pre>
{% extends "book_outlet/base.html" %}

{% block title %}
  All Books
{% endblock %}

{% block content %}
  <ul>
    {% for book in books %}
      <li><a href="{{ book.get_absolute_url }}">{{ book.title }}</a> (Rating: {{ book.rating }})</li>
    {% endfor %}
  </ul>
{% endblock %}
</pre>
```

_Listing: Index template using get_absolute_url for slug-based links._

The `get_absolute_url` method is the recommended Django convention for obtaining a model instance's canonical URL. Using it in templates decouples the URL structure from the template logic—if the URL pattern changes, only the model method and URL configuration need to be updated.

### Adding a Summary with Aggregation

The index page can be enhanced with summary statistics by using Django's aggregation functions alongside the existing QuerySet.

**Updating the view.** The `Avg` aggregation function computes the average rating across all books. The `count()` method returns the total number of records:

```python
# book_outlet/views.py
from django.db.models import Avg

def index(request):
    books = Book.objects.all().order_by("-rating")
    num_books = books.count()
    avg_rating = books.aggregate(Avg("rating"))  # {'rating__avg': 3.0}

    return render(request, "book_outlet/index.html", {
        "books": books,
        "total_number_of_books": num_books,
        "average_rating": avg_rating
    })
```

_Listing: Index view with aggregation for book count and average rating._

The `aggregate()` method returns a dictionary whose keys follow the pattern `<field>__<function>` (e.g., `rating__avg`). Unlike QuerySet methods such as `filter()` and `order_by()`, `aggregate()` evaluates immediately and returns a plain dictionary rather than a QuerySet.

**Updating the index template.** The summary statistics are rendered below the book list:

```html
<pre>
{% extends "book_outlet/base.html" %}

{% block title %}
  All Books
{% endblock %}

{% block content %}
  <ul>
    {% for book in books %}
      <li><a href="{{ book.get_absolute_url }}">{{ book.title }}</a> (Rating: {{ book.rating }})</li>
    {% endfor %}
  </ul>

  <hr>

  <p>Total Number Of Books: {{ total_number_of_books }}</p>
  <p>Average Rating: {{ average_rating.rating__avg }}</p>
{% endblock %}
</pre>
```

_Listing: Index template displaying summary statistics._

The `average_rating.rating__avg` syntax accesses the `rating__avg` key from the dictionary returned by `aggregate()`. The dot notation in Django templates functions as both attribute access and dictionary key lookup.

> **Reference:** The official Django documentation provides exhaustive references for [Model Fields](https://docs.djangoproject.com/en/5.0/ref/models/fields/), [Validators](https://docs.djangoproject.com/en/5.0/ref/validators/), [QuerySets](https://docs.djangoproject.com/en/5.0/ref/models/querysets/), and [Field Lookups](https://docs.djangoproject.com/en/5.0/ref/models/lookups/).

---

## The Django Admin Interface

### Creating a Superuser

Django includes a built-in administration interface that provides a web-based GUI for managing application data. Before accessing the admin panel, a superuser account must be created:

```bash
python manage.py createsuperuser
```

_Listing 3.82: Creating an admin superuser._

The command prompts for a username, email address, and password. Once the superuser is created, start the development server and navigate to `http://127.0.0.1:8000/admin/` in a browser to access the admin panel.

### Registering Models

By default, the admin panel only displays Django's built-in models (Users, Groups). To manage custom models through the admin interface, they must be explicitly registered in the app's `admin.py` file:

```python
# book_outlet/admin.py
from django.contrib import admin
from .models import Book

admin.site.register(Book)
```

_Listing 3.83: Registering the Book model with the admin site._

After refreshing the admin panel, the `Book` model appears under the app's section, providing a full CRUD interface for managing book records.

### Customising the Admin Panel

The default admin interface can be customised by creating a `ModelAdmin` subclass. Common customisations include filtering options, column display, and search functionality:

```python
# book_outlet/admin.py
from django.contrib import admin
from .models import Book

class BookAdmin(admin.ModelAdmin):
    list_filter = ("author", "rating",)
    list_display = ("title", "author",)
    search_fields = ['title',]

admin.site.register(Book, BookAdmin)
```

_Listing 3.84: Customising the admin panel with filters, column display, and search._

The `list_filter` attribute adds filtering widgets to the sidebar, `list_display` controls which columns are shown in the list view, and `search_fields` adds a search bar that queries the specified fields.

### Handling Slug Fields in the Admin

When a model includes a `SlugField` that is auto-generated from another field (such as the title), there are two common approaches to managing it in the admin panel.

**Approach 1: Hide the field entirely.** Setting `editable=False` on the model field removes it from all forms, including the admin. The slug is generated exclusively by the overridden `save()` method:

```python
# book_outlet/models.py
slug = models.SlugField(default="", null=False, db_index=True, unique=True, editable=False)
```

_Listing: Making the slug field non-editable._

With `editable=False`, the field does not appear in the admin add/edit forms at all, preventing accidental manual edits.

**Approach 2: Auto-populate in the admin.** The `prepopulated_fields` option in `ModelAdmin` provides a live preview of the slug as the user types the source field. The slug is generated client-side using JavaScript and can still be manually adjusted before saving:

```python
# book_outlet/admin.py
from django.contrib import admin
from .models import Book

class BookAdmin(admin.ModelAdmin):
    prepopulated_fields = {"slug": ("title",)}
    list_filter = ("author", "rating",)
    list_display = ("title", "author",)
    search_fields = ['title',]

admin.site.register(Book, BookAdmin)
```

_Listing: Using prepopulated_fields to auto-fill the slug from the title._

> **Note:** When using `prepopulated_fields`, the overridden `save()` method that calls `slugify()` on the model should be removed (or made conditional), because the slug is already being set by the admin form. If both mechanisms are active, the `save()` override will always overwrite any manual slug edits made in the admin.

---

## Relationships

Relational databases model real-world associations between entities through foreign keys and join tables. Django's ORM provides three field types that map directly to the three fundamental relationship cardinalities: one-to-many, one-to-one, and many-to-many.

_Table: Django relationship field types._

| Cardinality      | Django Field      | Database Implementation                       | Example                               |
| :--------------- | :---------------- | :-------------------------------------------- | :------------------------------------ |
| **One-to-Many**  | `ForeignKey`      | Foreign key column on the "many" side         | An author has many books              |
| **One-to-One**   | `OneToOneField`   | Foreign key column with a unique constraint   | An author has one address             |
| **Many-to-Many** | `ManyToManyField` | Intermediate join table created automatically | A book is published in many countries |

### One-to-Many Relationships (ForeignKey)

A one-to-many relationship exists when a single record in one table is associated with multiple records in another. In Django, this is represented using `ForeignKey` on the "many" side of the relationship.

**Defining the models.** The `Author` model is created as a separate entity, and the `Book` model references it through a `ForeignKey`:

```python
# book_outlet/models.py
class Author(models.Model):
    first_name = models.CharField(max_length=100)
    last_name = models.CharField(max_length=100)

    def full_name(self):
        return f"{self.first_name} {self.last_name}"

    def __str__(self):
        return self.full_name()


class Book(models.Model):
    # ... other fields ...
    author = models.ForeignKey(
        Author, on_delete=models.CASCADE, null=True, related_name="books")
```

_Listing: Author model and ForeignKey relationship on Book._

The `ForeignKey` field accepts several important arguments:

- **`on_delete`** — Defines the behaviour when the referenced record is deleted. `models.CASCADE` deletes all related books when an author is deleted. Other options include `models.SET_NULL` (sets the field to NULL), `models.PROTECT` (prevents deletion), and `models.SET_DEFAULT` (sets to the default value).
- **`null=True`** — Allows a book to exist without an author.
- **`related_name="books"`** — Customises the reverse accessor name on the `Author` model (defaults to `book_set`).

After defining the models, migrations must be generated and applied:

```bash
python manage.py makemigrations
python manage.py migrate
```

_Listing: Creating and applying the relationship migration._

**Working with ForeignKey in the shell.** The following session demonstrates creating related records and querying across relationships:

```python
>>> from book_outlet.models import Book, Author

# Create an author
>>> jkrowling = Author(first_name="J.K.", last_name="Rowling")
>>> jkrowling.save()

>>> Author.objects.all()
<QuerySet [<Author: Author object (1)>]>
>>> Author.objects.all()[0].first_name
'J.K.'

# Create a book linked to the author
>>> hp1 = Book(title="Harry Potter 1", rating=5, is_bestselling=True,
...            slug="harry-potter-1", author=jkrowling)
>>> hp1.save()
```

_Listing: Creating related Author and Book records._

**Forward access (Book → Author).** Accessing the `ForeignKey` field on a book instance returns the related `Author` object:

```python
>>> harrypotter = Book.objects.get(title="Harry Potter 1")
>>> harrypotter.author
<Author: Author object (1)>
>>> harrypotter.author.first_name
'J.K.'
>>> harrypotter.author.last_name
'Rowling'
```

_Listing: Accessing the related author from a book instance._

**Cross-relationship filtering.** Django's double-underscore syntax allows filtering across relationships, including chained lookups:

```python
>>> books_by_rowling = Book.objects.filter(author__last_name="Rowling")
>>> books_by_rowling
<QuerySet [<Book: Harry Potter 1 (5)>]>

>>> books_by_rowling = Book.objects.filter(author__last_name__contains="wling")
>>> books_by_rowling
<QuerySet [<Book: Harry Potter 1 (5)>]>
```

_Listing: Filtering books by author attributes using double-underscore syntax._

**Reverse access (Author → Books).** By default, Django creates a reverse manager named `<model>_set` on the related model:

```python
>>> jkr = Author.objects.get(first_name="J.K.")
>>> jkr.book_set.all()
<QuerySet [<Book: Harry Potter 1 (5)>]>
```

_Listing: Reverse access using the default book_set manager._

When `related_name="books"` is specified on the `ForeignKey`, the reverse manager uses that name instead:

```python
>>> jkr.books.all()
<QuerySet [<Book: Harry Potter 1 (5)>]>

>>> jkr.books.get(title="Harry Potter 1")
<Book: Harry Potter 1 (5)>

>>> jkr.books.filter(rating__gt=3)
<QuerySet [<Book: Harry Potter 1 (5)>]>
```

_Listing: Reverse access using the custom related_name._

The reverse manager supports the same QuerySet methods as any other manager—`all()`, `filter()`, `get()`, `count()`, and so on.

**Registering in the admin.** To manage authors through the admin panel:

```python
# book_outlet/admin.py
admin.site.register(Author)
```

_Listing: Registering the Author model with the admin site._

### One-to-One Relationships (OneToOneField)

A one-to-one relationship exists when each record in one table corresponds to exactly one record in another table. In Django, this is represented using `OneToOneField`, which is functionally a `ForeignKey` with `unique=True`.

**Defining the models.** An `Address` model is created, and the `Author` model is updated with a `OneToOneField` referencing it:

```python
# book_outlet/models.py
class Address(models.Model):
    street = models.CharField(max_length=80)
    postal_code = models.CharField(max_length=5)
    city = models.CharField(max_length=50)

    def __str__(self):
        return f"{self.street}, {self.postal_code}, {self.city}"

    class Meta:
        verbose_name_plural = "Address Entries"


class Author(models.Model):
    first_name = models.CharField(max_length=100)
    last_name = models.CharField(max_length=100)
    address = models.OneToOneField(
        Address, on_delete=models.CASCADE, null=True)

    def full_name(self):
        return f"{self.first_name} {self.last_name}"

    def __str__(self):
        return self.full_name()
```

_Listing: Address model with OneToOneField on Author._

The `class Meta` inner class configures model-level metadata. The `verbose_name_plural` attribute overrides the default pluralisation (which would produce "Addresss") to display "Address Entries" in the admin panel.

After defining the models, migrations must be generated and applied:

```bash
python manage.py makemigrations
python manage.py migrate
```

_Listing: Creating and applying the one-to-one relationship migration._

**Working with OneToOneField in the shell.** The following session demonstrates assigning addresses to authors and accessing the relationship in both directions:

```python
>>> from book_outlet.models import Author, Address, Book

# Create address records
>>> addr1 = Address(street="Some Street", postal_code="12345", city="London")
>>> addr2 = Address(street="Another Street", postal_code="67890", city="New York")
>>> addr1.save()
>>> addr2.save()

# Assign an address to an author
>>> jkr = Author.objects.get(first_name="J.K.")
>>> jkr.address = addr1
>>> jkr.save()
```

_Listing: Creating addresses and assigning one to an author._

**Forward access (Author → Address).** Accessing the `OneToOneField` returns the related object directly (not a manager):

```python
>>> jkr.address
<Address: Address object (1)>
>>> jkr.address.street
'Some Street'
```

_Listing: Accessing the related address from an author._

**Reverse access (Address → Author).** Unlike `ForeignKey`, the reverse side of a `OneToOneField` returns a single object rather than a manager, accessed using the lowercase model name:

```python
>>> Address.objects.all()[0].author
<Author: J.K. Rowling>
>>> Address.objects.all()[0].author.first_name
'J.K.'
```

_Listing: Reverse access from an address to its author._

**Registering in the admin.** To manage addresses through the admin panel:

```python
# book_outlet/admin.py
admin.site.register(Address)
```

_Listing: Registering the Address model with the admin site._

### Many-to-Many Relationships (ManyToManyField)

A many-to-many relationship exists when each record in one table can be associated with multiple records in another table, and vice versa. In Django, this is represented using `ManyToManyField`. Django automatically creates an intermediate join table to manage the associations.

**Defining the models.** A `Country` model is created, and the `Book` model is updated with a `ManyToManyField`:

```python
# book_outlet/models.py
class Country(models.Model):
    name = models.CharField(max_length=80)
    code = models.CharField(max_length=2)

    def __str__(self):
        return self.name

    class Meta:
        verbose_name_plural = "Countries"


class Book(models.Model):
    # ... other fields ...
    published_countries = models.ManyToManyField(Country, null=False)
```

_Listing: Country model with ManyToManyField on Book._

After defining the models, migrations must be generated and applied:

```bash
python manage.py makemigrations
python manage.py migrate
```

_Listing: Creating and applying the many-to-many relationship migration._

**Working with ManyToManyField in the shell.** Unlike `ForeignKey` and `OneToOneField`, many-to-many relationships cannot be set through the constructor. Instead, the `add()` method is used after both records have been saved:

```python
>>> from book_outlet.models import Country, Book

# Create a country
>>> germany = Country(name="Germany", code="DE")
>>> germany.save()

# Add the country to a book's published_countries
>>> mys = Book.objects.all()[1]
>>> mys.published_countries.add(germany)
```

_Listing: Adding a many-to-many relationship between a book and a country._

> **Important:** Both records must be saved to the database before calling `add()`. Attempting to add an unsaved object raises a `ValueError`.

**Forward access (Book → Countries).** The `ManyToManyField` provides a manager for querying related records:

```python
>>> mys.published_countries.all()
<QuerySet [<Country: Country object (1)>]>

>>> mys.published_countries.filter(code="DE")
<QuerySet [<Country: Country object (1)>]>

>>> mys.published_countries.filter(code="UK")
<QuerySet []>
```

_Listing: Querying a book's published countries._

**Reverse access (Country → Books).** The reverse manager uses the default `<model>_set` naming convention. A custom `related_name` can be specified on the `ManyToManyField` to override this:

```python
>>> ger = Country.objects.all()[0]
>>> ger.book_set.all()
<QuerySet [<Book: My Story (3)>]>
>>> ger.books.all()
AttributeError: 'Country' object has no attribute 'books'
```

_Listing: Reverse access from a country to its books._

**Registering in the admin.** To manage countries through the admin panel:

```python
# book_outlet/admin.py
admin.site.register(Country)
```

_Listing: Registering the Country model with the admin site._

---

## Forms

### HTML Forms and HTTP Methods

HTML forms are the primary mechanism through which users submit data to a web server. A form collects user input—text, selections, file uploads—and transmits it to the server for processing.

Forms use one of two HTTP methods to transmit data:

_Table 3.11: HTTP methods used with HTML forms._

| Method   | Usage                               | Data Location                      |
| :------- | :---------------------------------- | :--------------------------------- |
| **GET**  | Retrieving data, search forms       | URL query string (`?name=value`)   |
| **POST** | Submitting data, creating resources | Request body (hidden from the URL) |

**GET** is appropriate for search forms, filters, and any operation where the resulting URL should be bookmarkable. **POST** is appropriate for login forms, registration, file uploads, and any operation that modifies server-side state or transmits sensitive data.

A basic HTML form in Django includes the CSRF token (discussed in Section 3.8.3):

```html
<form method="POST" action="/submit/">
  {% csrf_token %}

  <label for="username">Username:</label>
  <input type="text" id="username" name="username" />

  <label for="email">Email:</label>
  <input type="email" id="email" name="email" />

  <button type="submit">Submit</button>
</form>
```

_Listing 3.85: A basic HTML form with CSRF protection._

The key attributes of the `<form>` element are `method` (the HTTP method), `action` (the URL to which the data is sent), and `name` on each input field (the key used to access the submitted data on the server).

### Processing Form Data in Django

**Accessing POST data.** When a form is submitted via POST, the submitted values are available through the `request.POST` dictionary-like object:

```python
from django.http import HttpResponse
from django.shortcuts import render

def register(request):
    if request.method == 'POST':
        username = request.POST.get('username')
        email = request.POST.get('email')
        # Process the data (e.g., save to database)
        return HttpResponse(f"Welcome, {username}!")

    # Show empty form for GET request
    return render(request, 'register.html')
```

_Listing 3.86: Processing POST data in a view._

**Accessing GET data.** Query parameters from the URL are accessed through `request.GET`:

```python
def search(request):
    query = request.GET.get('q', '')       # Default to empty string
    page = request.GET.get('page', '1')
    return HttpResponse(f"Searching for: {query}")
```

_Listing 3.87: Accessing GET query parameters._

_Table 3.12: Methods for accessing form data._

| Method                                 | Description                                              |
| :------------------------------------- | :------------------------------------------------------- |
| `request.POST.get('field')`            | Returns the value, or `None` if the field is missing     |
| `request.POST.get('field', 'default')` | Returns the value, or the specified default              |
| `request.POST['field']`                | Returns the value; raises `KeyError` if missing          |
| `request.POST.getlist('field')`        | Returns a list of values (for checkboxes, multi-selects) |

### CSRF Protection

Cross-Site Request Forgery (CSRF) is a class of attack in which a malicious website tricks a user's browser into submitting a request to another site where the user is authenticated. Consider the following attack scenario:

1. A user logs into their bank's website.
2. Without logging out, the user visits a malicious website in another tab.
3. The malicious website contains a hidden form that submits a transfer request to the bank.
4. Because the user's browser automatically includes the bank's session cookie, the bank processes the request as if it were legitimate.
5. Funds are transferred without the user's knowledge.

**Django's CSRF protection mechanism.** Django mitigates CSRF attacks by generating a unique, unpredictable token for each user session. The protection flow is as follows:

1. The user requests a page containing a form.
2. Django generates a unique CSRF token and embeds it in the response.
3. The token is included in the form as a hidden input field.
4. When the user submits the form, the token is sent along with the form data.
5. Django validates that the submitted token matches the one stored in the user's session.
6. If the token is valid, the request is processed; if it is invalid or missing, Django returns a `403 Forbidden` response.

**Using the CSRF token in templates.** The `{% csrf_token %}` template tag must be placed inside every `<form>` element that uses the POST method:

```html
<form method="POST" action="/submit/">
  {% csrf_token %}
  <input type="text" name="username" />
  <button type="submit">Submit</button>
</form>
```

_Listing 3.88: Including the CSRF token in a form._

The `{% csrf_token %}` tag renders a hidden input field containing the token:

```html
<input type="hidden" name="csrfmiddlewaretoken" value="abc123xyz..." />
```

_Listing 3.89: HTML output of the csrf_token template tag._

### The Django Forms API

While it is possible to build forms using raw HTML and process the submitted data manually, Django provides a Forms API that automates form rendering, validation, and error handling.

_Table 3.13: Benefits of using the Django Forms API._

| Benefit                 | Description                                    |
| :---------------------- | :--------------------------------------------- |
| **Automatic Rendering** | Forms can render themselves as HTML            |
| **Validation**          | Built-in and custom validation support         |
| **Security**            | CSRF protection and XSS prevention             |
| **Data Binding**        | Easy binding of form data to Python objects    |
| **Error Handling**      | Automatic error message generation and display |

Django provides two types of form classes:

1. **`forms.Form`** — A standard form where fields are defined manually. Used for forms not directly tied to a database model (e.g., contact forms, search forms).
2. **`forms.ModelForm`** — A form automatically generated from a database model. Fields, labels, and validation rules are derived from the model definition.

**Defining a form.** A form class inherits from `forms.Form` and declares fields as class attributes:

```python
# forms.py
from django import forms

class ContactForm(forms.Form):
    name = forms.CharField(max_length=100)
    email = forms.EmailField()
```

_Listing 3.90: Defining a Django form._

**Processing the form in a view.** The view creates a form instance—either empty (for GET requests) or populated with submitted data (for POST requests). The `is_valid()` method triggers validation and populates `cleaned_data` with the sanitised values:

```python
# views.py
from django.http import HttpResponse
from django.shortcuts import render
from .forms import ContactForm

def contact(request):
    if request.method == 'POST':
        form = ContactForm(request.POST)
        if form.is_valid():
            name = form.cleaned_data['name']
            email = form.cleaned_data['email']
            return HttpResponse("Thank you!")
    else:
        form = ContactForm()

    return render(request, 'contact.html', {'form': form})
```

_Listing 3.91: Processing a Django form in a view._

**Rendering the form in a template.** Django provides several methods for rendering forms:

```html
<pre>
<form method="POST" action="{% url 'contact' %}">
  {% csrf_token %}

  {# Option 1: Render each field as a paragraph #}
  {{ form.as_p }}

  {# Option 2: Render as table rows #}
  {{ form.as_table }}

  {# Option 3: Render as list items #}
  {{ form.as_ul }}

  {# Option 4: Manual rendering (provides full control over layout) #}
  <div class="form-group">
    <label for="{{ form.name.id_for_label }}">Name:</label>
    {{ form.name }} {{ form.name.errors }}
  </div>

  <button type="submit">Send</button>
</form>
</pre>
```

_Listing 3.92: Form rendering options in templates._

### Form Fields, Field Arguments, and Widgets

**Common form field types.** Django provides field types that map to corresponding HTML input elements:

_Table 3.14: Common Django form field types._

| Field Type            | Description                  | HTML Element                    |
| :-------------------- | :--------------------------- | :------------------------------ |
| `CharField`           | Text input                   | `<input type="text">`           |
| `EmailField`          | Email input with validation  | `<input type="email">`          |
| `IntegerField`        | Integer input                | `<input type="number">`         |
| `FloatField`          | Decimal number input         | `<input type="number">`         |
| `DateField`           | Date input                   | `<input type="date">`           |
| `DateTimeField`       | Date and time input          | `<input type="datetime-local">` |
| `BooleanField`        | Checkbox                     | `<input type="checkbox">`       |
| `ChoiceField`         | Dropdown select              | `<select>`                      |
| `MultipleChoiceField` | Multiple selection           | `<select multiple>`             |
| `FileField`           | File upload                  | `<input type="file">`           |
| `ImageField`          | Image upload with validation | `<input type="file">`           |

**Common field arguments.** Field constructors accept keyword arguments that control validation and rendering behaviour:

```python
from django import forms

class ExampleForm(forms.Form):
    # required — Whether the field is mandatory (default: True)
    name = forms.CharField(required=True)

    # max_length — Maximum permitted character length
    username = forms.CharField(max_length=50)

    # min_length — Minimum required character length
    password = forms.CharField(min_length=8)

    # initial — Pre-populated default value
    country = forms.CharField(initial='Nepal')

    # help_text — Descriptive text displayed alongside the field
    email = forms.EmailField(help_text='Enter a valid email address')

    # label — Custom label text (overrides the default field name)
    dob = forms.DateField(label='Date of Birth')

    # error_messages — Custom error messages for specific validation errors
    phone = forms.CharField(
        error_messages={
            'required': 'Phone number is required',
            'max_length': 'Phone number too long'
        }
    )

    # disabled — Renders the field as read-only
    id_number = forms.CharField(disabled=True)
```

_Listing 3.93: Common form field arguments._

**Widgets.** Widgets control how a form field is rendered as HTML. Each field type has a default widget (e.g., `CharField` uses `TextInput`), but this can be overridden:

```python
name = forms.CharField(
    widget=forms.TextInput(attrs={'class': 'form-control', 'placeholder': 'Enter Name'})
)

password = forms.CharField(
    widget=forms.PasswordInput()     # Renders as <input type="password">
)

birth_date = forms.DateField(
    widget=forms.DateInput(attrs={'type': 'date'})   # HTML5 date picker
)
```

_Listing 3.94: Customising field rendering with widgets._

The `attrs` dictionary adds HTML attributes (such as `class`, `placeholder`, and `type`) to the rendered input element.

### Form Validation

Validation occurs when `form.is_valid()` is called. Django performs validation in three stages:

**A. Built-in validation** — Enforced automatically based on field type and arguments:

- `required=True` ensures the field is not empty.
- `max_length` and `min_length` constrain string length.
- `EmailField` validates the email format; `URLField` validates URL format.

**B. Custom field validation (`clean_<fieldname>`)** — A method named `clean_<fieldname>` validates a single field and can raise `ValidationError` if the value is invalid:

```python
def clean_email(self):
    email = self.cleaned_data['email']
    if not email.endswith('@example.com'):
        raise forms.ValidationError("Only example.com emails are allowed!")
    return email
```

_Listing 3.95: Custom single-field validation._

**C. Cross-field validation (`clean`)** — The `clean()` method validates relationships between multiple fields:

```python
def clean(self):
    cleaned_data = super().clean()
    password = cleaned_data.get("password")
    confirm_password = cleaned_data.get("confirm_password")

    if password != confirm_password:
        raise forms.ValidationError("Passwords do not match")
```

_Listing 3.96: Cross-field validation in the clean method._

**Displaying non-field errors** (errors raised in the `clean()` method) in the template:

```html
<pre>
{% if form.non_field_errors %}
<div class="errors">
  {% for error in form.non_field_errors %}
  <p>{{ error }}</p>
  {% endfor %}
</div>
{% endif %}
</pre>
```

_Listing 3.97: Displaying non-field validation errors in a template._

### Comprehensive Form Validation — A Complete Walkthrough

This section presents a complete registration form application that demonstrates server-side validation for a variety of input types. The form collects a user's name, gender, hobbies, appointment date and time, country, résumé file, email, phone number, password, and password confirmation. The validation requirements are as follows:

- All fields are required.
- The appointment date and time must not be in the past.
- The résumé file must be in PDF, Microsoft Word, or image format, with a maximum size of 2 MB.
- The email must be valid.
- The phone number must match the pattern `9*********` (10 digits starting with 9) or `01*******` (8 digits starting with 01).
- The password must be at least 8 characters long and contain at least one lowercase letter, one uppercase letter, one digit, and one symbol.
- The password and confirmation password must match.

**Project setup:**

```bash
mkdir django-form-validation
cd django-form-validation
python -m venv venv
venv\Scripts\activate          # Windows
# source venv/bin/activate     # macOS/Linux
pip install django
django-admin startproject config .
python manage.py startapp registration
```

_Listing 3.98: Setting up the form validation project._

**Project structure:**

```text
django-form-validation/
├── config/
│   ├── __init__.py
│   ├── asgi.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
├── registration/
│   ├── migrations/
│   ├── static/
│   │   └── registration/
│   │       └── css/
│   │           └── styles.css
│   ├── templates/
│   │   └── registration/
│   │       ├── form.html
│   │       └── success.html
│   ├── __init__.py
│   ├── admin.py
│   ├── apps.py
│   ├── forms.py
│   ├── models.py
│   ├── urls.py
│   └── views.py
├── media/
│   └── resumes/
├── manage.py
└── venv/
```

_Listing 3.99: Directory structure of the form validation project._

**Step 1: Register the app and configure media settings** in `config/settings.py`:

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'registration',
]
```

```python
# At the bottom of config/settings.py
import os

MEDIA_URL = '/media/'
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
```

_Listing 3.100: App registration and media configuration._

**Step 2: Define the model** in `registration/models.py`:

```python
from django.db import models


class Registration(models.Model):
    GENDER_CHOICES = [
        ('M', 'Male'),
        ('F', 'Female'),
        ('O', 'Other'),
    ]

    HOBBY_CHOICES = [
        ('football', 'Football'),
        ('tableTennis', 'Table Tennis'),
        ('basketball', 'Basketball'),
    ]

    COUNTRY_CHOICES = [
        ('', '-- Select --'),
        ('Nepal', 'Nepal'),
        ('India', 'India'),
        ('USA', 'USA'),
    ]

    name = models.CharField(max_length=100)
    gender = models.CharField(max_length=1, choices=GENDER_CHOICES)
    hobbies = models.JSONField(default=list)
    appointment = models.DateTimeField()
    country = models.CharField(max_length=50, choices=COUNTRY_CHOICES)
    email = models.EmailField(unique=True)
    phone = models.CharField(max_length=15)
    resume = models.FileField(upload_to='resumes/')
    password = models.CharField(max_length=128)
    created_at = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return self.name

    class Meta:
        ordering = ['-created_at']
```

_Listing 3.101: Registration model with choice fields and file upload._

Run the migrations after defining the model:

```bash
python manage.py makemigrations
python manage.py migrate
```

_Listing 3.102: Applying migrations for the Registration model._

**Step 3: Create the form with validation** in `registration/forms.py`:

```python
from django import forms
from django.utils import timezone
from .models import Registration
import re


class RegistrationForm(forms.Form):
    """Registration form with comprehensive validation."""

    GENDER_CHOICES = [
        ('M', 'Male'),
        ('F', 'Female'),
        ('O', 'Other'),
    ]

    HOBBY_CHOICES = [
        ('football', 'Football'),
        ('tableTennis', 'Table Tennis'),
        ('basketball', 'Basketball'),
    ]

    COUNTRY_CHOICES = [
        ('', '-- Select --'),
        ('Nepal', 'Nepal'),
        ('India', 'India'),
        ('USA', 'USA'),
    ]

    name = forms.CharField(max_length=100)

    gender = forms.ChoiceField(
        choices=GENDER_CHOICES,
        widget=forms.RadioSelect,
        error_messages={'required': 'Please select gender'}
    )

    hobbies = forms.MultipleChoiceField(
        choices=HOBBY_CHOICES,
        widget=forms.CheckboxSelectMultiple,
        error_messages={'required': 'Please select at least one hobby'}
    )

    appointment = forms.DateTimeField(
        widget=forms.DateTimeInput(attrs={'type': 'datetime-local'}),
        error_messages={'required': 'Please select an appointment date and time'}
    )

    country = forms.ChoiceField(
        choices=COUNTRY_CHOICES,
        error_messages={'required': 'Please select a country'}
    )

    email = forms.EmailField(
        error_messages={
            'required': 'Email is required',
            'invalid': 'Please enter a valid email address'
        }
    )

    phone = forms.CharField(
        max_length=15,
        widget=forms.NumberInput,
        error_messages={'required': 'Phone number is required'}
    )

    resume = forms.FileField(
        widget=forms.FileInput(attrs={
            'accept': '.pdf,.jpg,.jpeg,.png,.doc,.docx',
        }),
        error_messages={'required': 'Please upload a résumé'}
    )

    password = forms.CharField(
        widget=forms.PasswordInput,
        error_messages={'required': 'Password is required'}
    )

    confirm_password = forms.CharField(
        widget=forms.PasswordInput,
        error_messages={'required': 'Password confirmation is required'}
    )

    def clean_appointment(self):
        """Validate that the appointment is not in the past."""
        appointment = self.cleaned_data.get('appointment')
        if appointment < timezone.now():
            raise forms.ValidationError(
                'Appointment date and time cannot be in the past'
            )
        return appointment

    def clean_phone(self):
        """Validate phone number format."""
        phone = self.cleaned_data.get('phone')
        phone_regex = r'^(?:9\d{9}|01\d{7})$'
        if not re.match(phone_regex, phone):
            raise forms.ValidationError(
                'Please enter a valid phone number (9_________ or 01_______)'
            )
        return phone

    def clean_resume(self):
        """Validate résumé file type and size."""
        resume = self.cleaned_data.get('resume')
        allowed_extensions = ['pdf', 'jpg', 'jpeg', 'png', 'doc', 'docx']
        extension = resume.name.split('.')[-1].lower()
        if extension not in allowed_extensions:
            raise forms.ValidationError('Unsupported file format')

        max_size = 2 * 1024 * 1024  # 2 MB in bytes
        if resume.size > max_size:
            raise forms.ValidationError('File size must be less than 2 MB')
        return resume

    def clean_password(self):
        """Validate password strength."""
        password = self.cleaned_data.get('password')
        password_regex = r'^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[^a-zA-Z\d\s]).{8,}$'
        if not re.match(password_regex, password):
            raise forms.ValidationError(
                'Password must be at least 8 characters long and include '
                'one uppercase letter, one lowercase letter, one digit, '
                'and one symbol'
            )
        return password

    def clean(self):
        """Validate that the confirmation password matches the password."""
        cleaned_data = super().clean()
        password = cleaned_data.get('password')
        confirm_password = cleaned_data.get('confirm_password')

        if password != confirm_password:
            self.add_error(
                'confirm_password',
                'Confirmation password does not match the password'
            )
        return cleaned_data
```

_Listing 3.103: Complete registration form with field-level and cross-field validation._

**Step 4: Create the view** in `registration/views.py`:

```python
from django.shortcuts import render, redirect
from django.contrib import messages
from django.contrib.auth.hashers import make_password
from .forms import RegistrationForm
from .models import Registration


def registration_form(request):
    """Handle registration form display and submission."""
    if request.method == 'POST':
        form = RegistrationForm(request.POST, request.FILES)

        if form.is_valid():
            data = form.cleaned_data
            registration = Registration(
                name=data['name'],
                gender=data['gender'],
                hobbies=data['hobbies'],
                appointment=data['appointment'],
                country=data['country'],
                email=data['email'],
                phone=data['phone'],
                resume=data['resume'],
                password=make_password(data['password']),
            )
            registration.save()

            messages.success(request, 'Registration submitted successfully!')
            return redirect('registration:form')
    else:
        form = RegistrationForm()

    return render(request, 'registration/form.html', {'form': form})
```

_Listing 3.104: Registration view handling both GET and POST requests._

Note the use of `request.FILES` as the second argument to the form constructor. This is required for forms that include file upload fields (`enctype="multipart/form-data"`). The `make_password()` function from `django.contrib.auth.hashers` securely hashes the password before storing it in the database.

**Step 5: Configure URLs.** Create `registration/urls.py` and update `config/urls.py`:

```python
# registration/urls.py
from django.urls import path
from . import views

app_name = 'registration'

urlpatterns = [
    path('', views.registration_form, name='form'),
]
```

```python
# config/urls.py
from django.contrib import admin
from django.urls import path, include
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
    path('admin/', admin.site.urls),
    path('registration/', include('registration.urls')),
]

# Serve media files during development
if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

_Listing 3.105: URL configurations for the registration app._

The `app_name` variable enables URL namespacing, allowing URLs to be referenced as `'registration:form'` rather than just `'form'`. The `static()` helper at the bottom of the project URLs serves uploaded media files during development (in production, media files are served by the web server directly).

**Step 6: Create the template** (`registration/templates/registration/form.html`):

```html
<pre>
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1.0" />
        <title>Form Validation (Django)</title>
        <style>
            .errorlist {
                color: red;
            }
            .alert {
                padding: 1rem;
                border-radius: 4px;
                margin-bottom: 1rem;
                text-align: center;
            }
            .alert-success {
                background: #d1fae5;
                color: #065f46;
            }
            .alert-error {
                background: #fee2e2;
                color: #991b1b;
            }
        </style>
    </head>
    <body>
        <div class="container">
            <h1>Registration Form</h1>
            {% if messages %}
                {% for message in messages %}<div class="alert alert-{{ message.tags }}">{{ message }}</div>{% endfor %}
            {% endif %}
            <form id="registrationForm"
                  method="POST"
                  action="{% url 'registration:form' %}"
                  enctype="multipart/form-data"
                  onsubmit="return handleSubmit();">
                {% csrf_token %}
                {{ form.as_p }}
                <button type="submit" class="btn">Submit</button>
            </form>
        </div>
        <script>
            function handleSubmit() {
                const form = document.getElementById("registrationForm");
                const formData = new FormData(form);
                const name = formData.get('name');

                if (!name) {
                    alert("Name is required");
                    return false;
                }
                return true;
            }
        </script>
    </body>
</html>
</pre>
```

_Listing 3.106: Registration form template with inline styles and client-side validation._

The template combines Django's server-side form rendering (`{{ form.as_p }}`) with a simple client-side validation function using JavaScript. The `enctype="multipart/form-data"` attribute on the `<form>` element is essential for file uploads. Django's messages framework displays success or error notifications after form submission.

**Running the project:**

```bash
python manage.py runserver
```

_Listing 3.107: Starting the development server._

Navigate to `http://127.0.0.1:8000/registration/` to test the registration form. Submitting the form triggers both client-side (JavaScript) and server-side (Django form validation) checks. If validation fails, the form is re-rendered with error messages next to the invalid fields. If validation succeeds, the data is saved to the database and a success message is displayed.

---

## Sessions and Cookies

### Cookies

HTTP is a stateless protocol—each request is independent and carries no memory of previous interactions. Cookies provide one mechanism for maintaining state across requests. A cookie is a small piece of data (up to approximately 4 KB) that the server sends to the client's browser. The browser stores the cookie and automatically includes it in every subsequent request to the same domain.

Key characteristics of cookies:

- Stored on the client (in the browser).
- Automatically attached to every HTTP request to the originating domain.
- Accessible by JavaScript unless the `HttpOnly` flag is set.
- Can be configured with an expiration date or a maximum age.
- Limited to approximately 4 KB per cookie.

### Cookie Attributes

_Table 3.15: Standard cookie attributes._

| Attribute    | Description                                                        |
| :----------- | :----------------------------------------------------------------- |
| `name=value` | The key-value data stored in the cookie                            |
| `Expires`    | The date and time at which the cookie expires                      |
| `Max-Age`    | The cookie's lifetime in seconds                                   |
| `Domain`     | The domain(s) that may access the cookie                           |
| `Path`       | The URL path(s) for which the cookie is valid                      |
| `Secure`     | Restricts transmission to HTTPS connections only                   |
| `HttpOnly`   | Prevents JavaScript from accessing the cookie (mitigates XSS)      |
| `SameSite`   | Controls cross-site request behaviour (`Strict`, `Lax`, or `None`) |

### Working with Cookies in Django

**Setting cookies.** Cookies are set on the response object using the `set_cookie()` method. The `max_age` parameter specifies the cookie's lifetime in seconds:

```python
# views.py
from django.http import HttpResponse


def set_cookie(request):
    response = HttpResponse("Cookie Set!")

    # Set a cookie that expires in 30 days
    response.set_cookie('user', 'John Doe', max_age=86400 * 30)

    # Set additional cookies
    response.set_cookie('theme', 'dark', max_age=86400 * 30)
    response.set_cookie('language', 'en', max_age=86400 * 30)

    # Secure cookie with all security flags enabled
    # response.set_cookie(
    #     'session_token',
    #     token_value,
    #     max_age=86400 * 30,
    #     secure=True,       # Transmit only over HTTPS
    #     httponly=True,      # Inaccessible to JavaScript
    #     samesite='Strict'   # Prevents cross-site sending
    # )

    return response
```

_Listing 3.108: Setting cookies in a Django view._

**Accessing cookies.** Cookies sent by the client are available through the `request.COOKIES` dictionary:

```python
# views.py
from django.http import HttpResponse


def get_cookie(request):
    user = request.COOKIES.get('user')

    if user:
        return HttpResponse(f"Welcome {user}")
    else:
        return HttpResponse("Cookie not set.")
```

_Listing 3.109: Accessing cookies from the request._

**Deleting cookies.** A cookie is deleted by calling `delete_cookie()` on the response object, which instructs the browser to remove it:

```python
# views.py
from django.http import HttpResponse


def delete_cookie(request):
    response = HttpResponse("Cookie Deleted!")
    response.delete_cookie('user')
    return response
```

_Listing 3.110: Deleting a cookie._

### Sessions

While cookies store data on the client, sessions store data on the server. A session associates a unique identifier (the session ID) with server-side data. The session ID itself is stored in a cookie on the client's browser and is sent with every request, allowing the server to retrieve the corresponding data.

Common use cases for sessions include:

- Keeping users logged in across requests.
- Storing shopping cart contents.
- Remembering user preferences and settings.
- Tracking progress through multi-step forms (wizards).

### How Sessions Work

The session workflow proceeds as follows:

1. The user makes an initial request to the server.
2. The server creates a unique session ID and stores it (along with an empty data dictionary) in its session backend (database, cache, or file system).
3. The server sends the session ID to the browser in a cookie (typically named `sessionid`).
4. The browser stores the cookie and includes it in every subsequent request.
5. On each request, the server reads the session ID from the cookie and retrieves the associated data from the session backend.
6. The server can read, modify, or delete the session data as needed.

Django enables sessions by default. The `django.contrib.sessions` app and `SessionMiddleware` are included in new projects automatically.

### Session Configuration

Session behaviour can be customised in `settings.py`:

```python
# settings.py

# Session lifetime in seconds (default: 2 weeks)
SESSION_COOKIE_AGE = 86400 * 30    # 30 days

# Whether the session expires when the browser closes
SESSION_EXPIRE_AT_BROWSER_CLOSE = False
```

_Listing 3.111: Configuring session settings._

### Using Sessions in Django

**Storing data in a session:**

```python
request.session['username'] = username
```

**Retrieving data from a session:**

```python
username = request.session.get('username', 'Guest')
```

**Clearing all session data:**

```python
request.session.flush()    # Removes all session data and deletes the session
```

_Listing 3.112: Basic session operations — store, retrieve, and clear._

_Table 3.16: Django session methods._

| Method                                | Description                                 |
| :------------------------------------ | :------------------------------------------ |
| `request.session['key'] = value`      | Store a value in the session                |
| `request.session.get('key')`          | Retrieve a value (returns `None` if absent) |
| `request.session.get('key', default)` | Retrieve a value with a default             |
| `del request.session['key']`          | Delete a specific key from the session      |
| `request.session.flush()`             | Clear all session data                      |
| `request.session.set_expiry(seconds)` | Set the session expiration time             |

### Sessions versus Cookies

_Table 3.17: Comparison of cookies and sessions._

| Aspect          | Cookies                      | Sessions                   |
| :-------------- | :--------------------------- | :------------------------- |
| **Storage**     | Client (browser)             | Server                     |
| **Size Limit**  | ~4 KB per cookie             | No practical limit         |
| **Security**    | Visible and editable by user | Hidden from user           |
| **Data Access** | Accessible by JavaScript     | Server-side access only    |
| **Use Case**    | Preferences, theme, language | User identity, cart, state |

> **Note:** In practice, sessions and cookies are used together. The session ID is stored in a cookie, while the actual session data resides on the server. This provides the convenience of cookies (automatic transmission with each request) with the security of server-side storage (the user cannot read or tamper with the data).

---

## Authentication and Authorisation

### Authentication versus Authorisation

Authentication and authorisation are related but distinct security concepts:

_Table 3.18: Authentication versus authorisation._

| Concept            | Question           | Purpose                          |
| :----------------- | :----------------- | :------------------------------- |
| **Authentication** | "Who are you?"     | Verify the user's identity       |
| **Authorisation**  | "What can you do?" | Determine the user's permissions |

Authentication must occur before authorisation—a user's identity must be established before the system can determine what actions they are permitted to perform.

### The Authentication Flow

A typical authentication flow proceeds as follows:

1. The user submits credentials (username and password) via a login form.
2. The server verifies the credentials against the database.
3. If the credentials are valid, the server creates a session and stores the user's identity. If the credentials are invalid, the server returns an error message.
4. The session ID is sent to the client as a cookie.
5. The client includes the session cookie in all subsequent requests.
6. The server verifies the session on each request to confirm the user's identity.

### Implementing a Login System — A Complete Walkthrough

The following walkthrough demonstrates a custom login system using Django sessions and hashed passwords. The application includes login, dashboard, logout, and registration functionality.

**Step 1: Define the Student model.**

```python
# models.py
from django.db import models


class Student(models.Model):
    username = models.CharField(max_length=100, unique=True)
    password = models.CharField(max_length=100)
    name = models.CharField(max_length=200)
    email = models.EmailField()

    def __str__(self):
        return self.username
```

_Listing 3.113: Student model for the authentication system._

```bash
python manage.py makemigrations
python manage.py migrate
```

_Listing 3.114: Running migrations for the Student model._

**Step 2: Create the login, dashboard, and logout views.** The login view uses `check_password()` from `django.contrib.auth.hashers` to verify the submitted password against the stored hash. Upon successful authentication, the student's ID and name are stored in the session:

```python
# views.py
from django.shortcuts import render, redirect
from django.contrib.auth.hashers import check_password
from .models import Student


def student_login(request):
    if request.method == 'POST':
        username = request.POST.get('username')
        password = request.POST.get('password')

        try:
            student = Student.objects.get(username=username)
            if check_password(password, student.password):
                request.session['student_id'] = student.id
                request.session['student_name'] = student.name
                return redirect('dashboard')
            else:
                return render(request, 'myauthapp/login.html', {
                    'error': 'Invalid username/password'
                })
        except Student.DoesNotExist:
            return render(request, 'myauthapp/login.html', {
                'error': 'Invalid username/password'
            })

    return render(request, 'myauthapp/login.html')


def dashboard(request):
    if 'student_id' not in request.session:
        return redirect('login')

    student_name = request.session.get('student_name')
    return render(request, 'myauthapp/dashboard.html', {'name': student_name})


def logout(request):
    request.session.flush()
    return redirect('login')
```

_Listing 3.115: Login, dashboard, and logout views using sessions._

> **Security note:** The `check_password()` function compares a plain-text password against a hashed value. Passwords must never be stored in plain text; they should always be hashed using `make_password()` before being saved to the database.

**Step 3: Create the login template.**

```html
<!-- templates/myauthapp/login.html -->
<!DOCTYPE html>
<html>
  <head>
    <title>Student Login</title>
  </head>
  <body>
    <h1>Student Login</h1>

    {% if error %}
    <p style="color: red;">{{ error }}</p>
    {% endif %}

    <form method="POST">
      {% csrf_token %}
      <label>Username:</label>
      <input type="text" name="username" required /><br /><br />

      <label>Password:</label>
      <input type="password" name="password" required /><br /><br />

      <button type="submit">Login</button>
    </form>
  </body>
</html>
```

_Listing 3.116: Login template._

**Step 4: Create the dashboard template.**

```html
<pre>
<!-- templates/myauthapp/dashboard.html -->
<!DOCTYPE html>
<html>
  <head>
    <title>Dashboard</title>
  </head>
  <body>
    <h1>Welcome {{ name }}</h1>
    <p>You have successfully logged in.</p>
    <a href="{% url 'logout' %}">Logout</a>
  </body>
</html>
</pre>
```

_Listing 3.117: Dashboard template with logout link._

**Step 5: Configure URLs.**

```python
# app-level urls.py
from django.urls import path
from . import views

urlpatterns = [
    path('login/', views.student_login, name='login'),
    path('dashboard/', views.dashboard, name='dashboard'),
    path('logout/', views.logout, name='logout'),
]
```

```python
# project-level urls.py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('auth/', include('myauthapp.urls')),
]
```

_Listing 3.118: URL configurations for the authentication app._

**Step 6: Create a test user.** Use the Django shell to create a student with a hashed password:

```python
# In the Django shell (python manage.py shell)
from django.contrib.auth.hashers import make_password
from myauthapp.models import Student

Student.objects.create(
    username="b2rsp",
    password=make_password("password123"),
    name="Bidur",
    email="bidur@example.com"
)
```

_Listing 3.119: Creating a test user with a hashed password via the Django shell._

Navigate to `http://127.0.0.1:8000/auth/login/` to test the login functionality.

### Adding a Registration View

A registration view allows new users to create accounts through the web interface rather than through the Django shell:

```python
# views.py
from django.contrib.auth.hashers import check_password, make_password
from .models import Student


def student_register(request):
    if request.method == 'POST':
        username = request.POST.get('username')
        password = request.POST.get('password')
        confirm_password = request.POST.get('confirm_password')
        name = request.POST.get('name')
        email = request.POST.get('email')

        # Validate required fields
        if not username or not password or not confirm_password or not name or not email:
            return render(request, 'myauthapp/register.html', {
                'error': 'All fields are required'
            })

        # Validate password confirmation
        if password != confirm_password:
            return render(request, 'myauthapp/register.html', {
                'error': 'Passwords do not match'
            })

        # Check for duplicate username
        if Student.objects.filter(username=username).exists():
            return render(request, 'myauthapp/register.html', {
                'error': 'Username already exists'
            })

        # Check for duplicate email
        if Student.objects.filter(email=email).exists():
            return render(request, 'myauthapp/register.html', {
                'error': 'Email already registered'
            })

        # Create the student with a hashed password
        Student.objects.create(
            username=username,
            password=make_password(password),
            name=name,
            email=email
        )

        return render(request, 'myauthapp/register.html', {
            'success': 'Account created successfully! You can now login.'
        })

    return render(request, 'myauthapp/register.html')
```

_Listing 3.120: Student registration view with validation._

**Registration template:**

```html
<pre>
<!-- templates/myauthapp/register.html -->
<!DOCTYPE html>
<html>
    <head>
        <title>Student Registration</title>
    </head>
    <body>
        <h1>Create Account</h1>
        {% if error %}<p style="color: red;">{{ error }}</p>{% endif %}
        {% if success %}
            <p style="color: green;">{{ success }}</p>
            <a href="{% url 'login' %}">Go to Login</a>
        {% else %}
            <form method="POST">
                {% csrf_token %}
                <label>Username:</label>
                <input type="text" name="username" required />
                <br /><br />
                <label>Full Name:</label>
                <input type="text" name="name" required />
                <br /><br />
                <label>Email:</label>
                <input type="email" name="email" required />
                <br /><br />
                <label>Password:</label>
                <input type="password" name="password" required />
                <br /><br />
                <label>Confirm Password:</label>
                <input type="password" name="confirm_password" required />
                <br /><br />
                <button type="submit">Register</button>
            </form>
            <p>
                Already have an account? <a href="{% url 'login' %}">Login here</a>
            </p>
        {% endif %}
    </body>
</html>
</pre>
```

_Listing 3.121: Registration template with conditional rendering._

**Update the URL configuration** to include the registration endpoint:

```python
# app-level urls.py
urlpatterns = [
    path('login/', views.student_login, name='login'),
    path('register/', views.student_register, name='register'),
    path('dashboard/', views.dashboard, name='dashboard'),
    path('logout/', views.logout, name='logout'),
]
```

_Listing 3.122: Updated URL configuration with registration._

---

## Practical Exercises

This section presents a collection of practical exercises that consolidate the concepts covered in Sections 3.6 through 3.10. Each exercise specifies a problem statement, validation requirements, and a complete solution comprising the model, form, view, template, and URL configuration.

### Exercise: Patient Registration Form

**Problem statement.** Create a server-side script to validate and store patient data into a `patients` table with the following fields: name, patient ID, mobile number, gender, address, date of birth, and doctor name. The validation rules are:

- Name, mobile, doctor name, gender, and date of birth are required.
- Mobile number must be 10 digits starting with 98, 97, or 96.
- Date of birth must be in `YYYY-MM-DD` format.

**Model:**

```python
# models.py
from django.db import models


class Patient(models.Model):
    GENDER_CHOICES = [
        ('M', 'Male'),
        ('F', 'Female'),
        ('O', 'Other'),
    ]

    name = models.CharField(max_length=200)
    patient_id = models.CharField(max_length=50, unique=True)
    mobile = models.CharField(max_length=10)
    gender = models.CharField(max_length=1, choices=GENDER_CHOICES)
    address = models.TextField(blank=True, null=True)
    dob = models.DateField()
    doctor_name = models.CharField(max_length=200)
    created_at = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return f"{self.name} ({self.patient_id})"
```

_Listing 3.123: Patient model._

**Form with validation:**

```python
# forms.py
from django import forms
import re


class PatientForm(forms.Form):
    GENDER_CHOICES = [
        ('', '-- Select Gender --'),
        ('M', 'Male'),
        ('F', 'Female'),
        ('O', 'Other'),
    ]

    name = forms.CharField(
        max_length=200,
        error_messages={'required': 'Name is required'}
    )

    patient_id = forms.CharField(max_length=50, required=False)

    mobile = forms.CharField(
        max_length=10,
        error_messages={'required': 'Mobile is required'}
    )

    gender = forms.ChoiceField(
        choices=GENDER_CHOICES,
        error_messages={'required': 'Gender is required'}
    )

    address = forms.CharField(widget=forms.Textarea, required=False)

    dob = forms.CharField(
        error_messages={'required': 'Date of Birth is required'}
    )

    doctor_name = forms.CharField(
        max_length=200,
        error_messages={'required': 'Doctor Name is required'}
    )

    def clean_mobile(self):
        mobile = self.cleaned_data['mobile']
        mobile_regex = r'^(98|97|96)\d{8}$'
        if not re.match(mobile_regex, mobile):
            raise forms.ValidationError(
                'Mobile must be 10 digits and start with 98, 97, or 96'
            )
        return mobile

    def clean_dob(self):
        dob = self.cleaned_data['dob']
        dob_regex = r'^\d{4}-(0[1-9]|1[0-2])-(0[1-9]|[12]\d|3[01])$'
        if not re.match(dob_regex, dob):
            raise forms.ValidationError(
                'Date of Birth must be in YYYY-MM-DD format'
            )
        from datetime import datetime
        try:
            dob_date = datetime.strptime(dob, '%Y-%m-%d').date()
        except ValueError:
            raise forms.ValidationError('Invalid calendar date')
        return dob_date
```

_Listing 3.124: Patient form with mobile and date-of-birth validation._

**View:**

```python
# views.py
from django.shortcuts import render, redirect
from django.contrib import messages
from .forms import PatientForm
from .models import Patient
import uuid


def patient_registration(request):
    if request.method == 'POST':
        form = PatientForm(request.POST)

        if form.is_valid():
            patient_id = form.cleaned_data.get('patient_id')
            if not patient_id:
                patient_id = 'PAT-' + str(uuid.uuid4())[:8].upper()

            patient = Patient(
                name=form.cleaned_data['name'],
                patient_id=patient_id,
                mobile=form.cleaned_data['mobile'],
                gender=form.cleaned_data['gender'],
                address=form.cleaned_data.get('address', ''),
                dob=form.cleaned_data['dob'],
                doctor_name=form.cleaned_data['doctor_name'],
            )
            patient.save()

            messages.success(request, 'Patient registered successfully!')
            return redirect('patient_list')
    else:
        form = PatientForm()

    return render(request, 'patient/patient_form.html', {'form': form})


def patient_list(request):
    patients = Patient.objects.all()
    return render(request, 'patient/patient_list.html', {'patients': patients})
```

_Listing 3.125: Patient registration and list views._

**Form template:**

```html
<pre>
<!-- templates/patient/patient_form.html -->
<!DOCTYPE html>
<html>
    <head>
        <title>Patient Registration</title>
        <style>
            .errorlist { color: red; }
            input, select, textarea { padding: 8px; width: 300px; }
        </style>
    </head>
    <body>
        <h1>Patient Registration Form</h1>
        <form method="post">
            {% csrf_token %}
            {{ form.as_p }}
            <button type="submit">Submit</button>
        </form>
    </body>
</html>
</pre>
```

_Listing 3.126: Patient registration form template._

**List template:**

```html
<pre>
<!-- templates/patient/patient_list.html -->
<!DOCTYPE html>
<html>
    <head>
        <title>Patient List</title>
        <style>
        table { margin-top: 20px; border-collapse: collapse; }
        th, td { border: 1px solid black; padding: 10px; }
        </style>
    </head>
    <body>
        <h1>Registered Patients</h1>
        <a href="{% url 'patient_register' %}">+ Add New Patient</a>
        <table>
            <tr>
                <th>Patient ID</th>
                <th>Name</th>
                <th>Mobile</th>
                <th>Gender</th>
                <th>DOB</th>
                <th>Doctor</th>
            </tr>
            {% for patient in patients %}
                <tr>
                    <td>{{ patient.patient_id }}</td>
                    <td>{{ patient.name }}</td>
                    <td>{{ patient.mobile }}</td>
                    <td>{{ patient.get_gender_display }}</td>
                    <td>{{ patient.dob }}</td>
                    <td>{{ patient.doctor_name }}</td>
                </tr>
            {% empty %}
                <tr>
                    <td colspan="6">No patients registered yet.</td>
                </tr>
            {% endfor %}
        </table>
    </body>
</html>
</pre>
```

_Listing 3.127: Patient list template with tabular display._

**URL configuration:**

```python
# app-level urls.py
from django.urls import path
from . import views

urlpatterns = [
    path('register/', views.patient_registration, name='patient_register'),
    path('', views.patient_list, name='patient_list'),
]
```

```python
# project-level urls.py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('patient/', include('patient.urls')),
]
```

_Listing 3.128: URL configurations for the patient app._

### Exercise: User Registration with Username and Password Validation

**Problem statement.** Design a registration form and write the corresponding server-side code to store user data after satisfying the following validation rules:

- Full name must not exceed 40 characters.
- Email address must be a valid email.
- Username must start with one or more letters followed by one or more digits.
- Password must be at least 8 characters long.

**Model:**

```python
# models.py
from django.db import models


class User(models.Model):
    full_name = models.CharField(max_length=40)
    email = models.EmailField(unique=True)
    username = models.CharField(max_length=100, unique=True)
    password = models.CharField(max_length=128)
    created_at = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return self.username
```

_Listing 3.129: User model._

**Form with validation:**

```python
# forms.py
from django import forms
import re


class UserRegistrationForm(forms.Form):
    full_name = forms.CharField(
        max_length=40,
        error_messages={
            'required': 'Full name is required',
            'max_length': 'Full name must be up to 40 characters'
        }
    )

    email = forms.EmailField(
        error_messages={
            'required': 'Email is required',
            'invalid': 'Please enter a valid email address'
        }
    )

    username = forms.CharField(
        max_length=100,
        error_messages={'required': 'Username is required'}
    )

    password = forms.CharField(
        widget=forms.PasswordInput,
        error_messages={'required': 'Password is required'}
    )

    def clean_username(self):
        """Username must start with letters and end with numbers."""
        username = self.cleaned_data.get('username')
        username_regex = r'^[a-zA-Z]+\d+$'
        if not re.match(username_regex, username):
            raise forms.ValidationError(
                'Username must start with letters and end with numbers'
            )
        return username

    def clean_password(self):
        """Password must be at least 8 characters."""
        password = self.cleaned_data.get('password')
        if len(password) < 8:
            raise forms.ValidationError(
                'Password must be more than 8 characters'
            )
        return password
```

_Listing 3.130: User registration form with username pattern and password length validation._

**View:**

```python
# views.py
from django.shortcuts import render, redirect
from django.contrib import messages
from django.contrib.auth.hashers import make_password
from .forms import UserRegistrationForm
from .models import User


def user_registration(request):
    if request.method == 'POST':
        form = UserRegistrationForm(request.POST)

        if form.is_valid():
            email = form.cleaned_data['email']
            if User.objects.filter(email=email).exists():
                form.add_error('email', 'Email already registered')
                return render(request, 'user/user_form.html', {'form': form})

            username = form.cleaned_data['username']
            if User.objects.filter(username=username).exists():
                form.add_error('username', 'Username already taken')
                return render(request, 'user/user_form.html', {'form': form})

            user = User(
                full_name=form.cleaned_data['full_name'],
                email=email,
                username=username,
                password=make_password(form.cleaned_data['password']),
            )
            user.save()

            messages.success(request, 'User registered successfully!')
            return redirect('user_list')
    else:
        form = UserRegistrationForm()

    return render(request, 'user/user_form.html', {'form': form})


def user_list(request):
    users = User.objects.all()
    return render(request, 'user/user_list.html', {'users': users})
```

_Listing 3.131: User registration view with duplicate checks._

**Form template:**

```html
<pre>
<!-- templates/user/user_form.html -->
<!DOCTYPE html>
<html>
    <head>
        <title>User Registration</title>
        <style>
            .errorlist { color: red; }
            input, select, textarea { padding: 8px; width: 300px; }
        </style>
    </head>
    <body>
        <h1>User Registration Form</h1>
        <form method="post">
            {% csrf_token %}
            {{ form.as_p }}
            <button type="submit">Submit</button>
        </form>
    </body>
</html>
</pre>
```

_Listing 3.132: User registration form template._

**List template:**

```html
<pre>
<!-- templates/user/user_list.html -->
<!DOCTYPE html>
<html>
    <head>
        <title>User List</title>
        <style>
        table { margin-top: 20px; border-collapse: collapse; }
        th, td { border: 1px solid black; padding: 10px; }
        </style>
    </head>
    <body>
        <h1>Registered Users</h1>
        <a href="{% url 'user_register' %}">+ Add New User</a>
        <table>
            <tr>
                <th>ID</th>
                <th>Full Name</th>
                <th>Email</th>
                <th>Username</th>
                <th>Created At</th>
            </tr>
            {% for user in users %}
                <tr>
                    <td>{{ user.id }}</td>
                    <td>{{ user.full_name }}</td>
                    <td>{{ user.email }}</td>
                    <td>{{ user.username }}</td>
                    <td>{{ user.created_at }}</td>
                </tr>
            {% empty %}
                <tr>
                    <td colspan="5">No users registered yet.</td>
                </tr>
            {% endfor %}
        </table>
    </body>
</html>
</pre>
```

_Listing 3.133: User list template._

**URL configuration:**

```python
# app-level urls.py
from django.urls import path
from . import views

urlpatterns = [
    path('register/', views.user_registration, name='user_register'),
    path('', views.user_list, name='user_list'),
]
```

```python
# project-level urls.py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('user/', include('user.urls')),
]
```

_Listing 3.134: URL configurations for the user app._

### Exercise: File Upload with Validation

**Problem statement.** Write a Django view to upload a file and validate:

- The file extension must be an image format (JPG, JPEG, PNG, or GIF only).
- The file size must be less than 2 MB.

**Model:**

```python
# models.py
from django.db import models


class UploadedFile(models.Model):
    file = models.FileField(upload_to='uploads/')
    uploaded_at = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return self.file.name
```

_Listing 3.135: UploadedFile model._

**Form with validation:**

```python
# forms.py
from django import forms


class FileUploadForm(forms.Form):
    file = forms.FileField(
        error_messages={'required': 'Please select a file to upload!'}
    )

    def clean_file(self):
        file = self.cleaned_data.get('file')

        # Validate file extension
        allowed_extensions = ['jpg', 'jpeg', 'png', 'gif']
        file_extension = file.name.split('.')[-1].lower()
        if file_extension not in allowed_extensions:
            raise forms.ValidationError(
                f'Invalid file type. Allowed: {", ".join(allowed_extensions)}'
            )

        # Validate file size (max 2 MB)
        max_size = 2 * 1024 * 1024
        if file.size > max_size:
            raise forms.ValidationError('File size must be less than 2 MB')

        return file
```

_Listing 3.136: File upload form with extension and size validation._

**View:**

```python
# views.py
from django.shortcuts import render, redirect
from django.contrib import messages
from .forms import FileUploadForm
from .models import UploadedFile


def upload_file(request):
    if request.method == 'POST':
        form = FileUploadForm(request.POST, request.FILES)

        if form.is_valid():
            uploaded_file = UploadedFile(
                file=form.cleaned_data['file']
            )
            uploaded_file.save()

            messages.success(request, 'File uploaded successfully!')
            return redirect('upload_success')
    else:
        form = FileUploadForm()

    return render(request, 'fileupload/upload.html', {'form': form})


def upload_success(request):
    files = UploadedFile.objects.all().order_by('-uploaded_at')
    return render(request, 'fileupload/success.html', {'files': files})
```

_Listing 3.137: File upload and success views._

**Upload template:**

```html
<pre>
<!-- templates/fileupload/upload.html -->
<!DOCTYPE html>
<html>
    <head>
        <title>File Upload</title>
        <style>
            .errorlist { color: red; }
        </style>
    </head>
    <body>
        <h1>Upload File</h1>
        <form method="post" enctype="multipart/form-data">
            {% csrf_token %}
            {{ form.as_p }}
            <p style="color: gray; font-size: 0.9em;">Allowed: JPG, JPEG, PNG, GIF (Max 2 MB)</p>
            <button type="submit">Upload</button>
        </form>
    </body>
</html>
</pre>
```

_Listing 3.138: File upload template._

> **Important:** The `enctype="multipart/form-data"` attribute on the `<form>` element is essential when the form includes file upload fields. Without it, the browser sends only the filename rather than the file contents. The view must also pass `request.FILES` as the second argument to the form constructor.

**Success template:**

```html
<pre>
<!-- templates/fileupload/success.html -->
<!DOCTYPE html>
<html>
    <head>
        <title>Upload Success</title>
    </head>
    <body>
        <h2>Uploaded Files</h2>
        <a href="{% url 'upload_file' %}">Upload Another</a>
        <ul>
            {% for file in files %}
                <li>
                    <a href="{{ file.file.url }}" target="_blank">{{ file.file.name }}</a>
                    ({{ file.uploaded_at }})
                </li>
            {% empty %}
                <li>No files uploaded yet.</li>
            {% endfor %}
        </ul>
    </body>
</html>
</pre>
```

_Listing 3.139: Upload success template listing all uploaded files._

**URL and media configuration:**

```python
# app-level urls.py
from django.urls import path
from . import views

urlpatterns = [
    path('', views.upload_file, name='upload_file'),
    path('success/', views.upload_success, name='upload_success'),
]
```

```python
# project-level urls.py
from django.contrib import admin
from django.urls import path, include
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
    path('admin/', admin.site.urls),
    path('upload/', include('fileupload.urls')),
]

if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

_Listing 3.140: URL and media configurations for the file upload app._

```python
# settings.py
import os

MEDIA_URL = '/media/'
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
```

_Listing 3.141: Media settings for file uploads._

---

### Exercise: Project File Submission

**Problem statement.** Design a form to upload and submit a student project file into a database. The form fields are: TU Registration Number, Email Address, and Project File. The validation rules are:

- All fields are mandatory.
- Email address must be in a valid format.
- The uploaded file must be in PDF, DOC, DOCX, PPT, PPTX, or JPEG format.
- The file size must be less than 5 MB.

**Model:**

```python
# models.py
from django.db import models


class ProjectSubmission(models.Model):
    tu_registration_number = models.CharField(max_length=50, unique=True)
    email = models.EmailField()
    project_file = models.FileField(upload_to='projects/')
    uploaded_at = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return f"{self.tu_registration_number} - {self.email}"
```

_Listing 3.142: ProjectSubmission model._

**Form with validation:**

```python
# forms.py
from django import forms


class ProjectSubmissionForm(forms.Form):
    tu_registration_number = forms.CharField(
        max_length=50,
        error_messages={'required': 'TU Registration Number is required'}
    )

    email = forms.EmailField(
        error_messages={
            'required': 'Email Address is required',
            'invalid': 'Please enter a valid email address'
        }
    )

    project_file = forms.FileField(
        error_messages={'required': 'Project File is required'}
    )

    def clean_project_file(self):
        """Validate file type and size."""
        project_file = self.cleaned_data.get('project_file')

        allowed_extensions = ['pdf', 'doc', 'docx', 'ppt', 'pptx', 'jpeg', 'jpg']
        file_extension = project_file.name.split('.')[-1].lower()
        if file_extension not in allowed_extensions:
            raise forms.ValidationError(
                'File format must be PDF, DOC, DOCX, PPT, PPTX, or JPEG'
            )

        max_size = 5 * 1024 * 1024    # 5 MB in bytes
        if project_file.size > max_size:
            raise forms.ValidationError('File size must be less than 5 MB')

        return project_file
```

_Listing 3.143: Project submission form with file type and size validation._

**View:**

```python
# views.py
from django.shortcuts import render, redirect
from django.contrib import messages
from .forms import ProjectSubmissionForm
from .models import ProjectSubmission


def project_upload(request):
    if request.method == 'POST':
        form = ProjectSubmissionForm(request.POST, request.FILES)

        if form.is_valid():
            reg_number = form.cleaned_data['tu_registration_number']
            if ProjectSubmission.objects.filter(tu_registration_number=reg_number).exists():
                form.add_error('tu_registration_number',
                               'This registration number has already submitted')
                return render(request, 'projectsubmission/project_form.html', {'form': form})

            submission = ProjectSubmission(
                tu_registration_number=reg_number,
                email=form.cleaned_data['email'],
                project_file=form.cleaned_data['project_file'],
            )
            submission.save()

            messages.success(request, 'Project submitted successfully!')
            return redirect('submission_list')
    else:
        form = ProjectSubmissionForm()

    return render(request, 'projectsubmission/project_form.html', {'form': form})


def submission_list(request):
    submissions = ProjectSubmission.objects.all()
    return render(request, 'projectsubmission/submission_list.html', {'submissions': submissions})
```

_Listing 3.144: Project upload and submission list views._

**Form template:**

```html
<pre>
<!-- templates/projectsubmission/project_form.html -->
<!DOCTYPE html>
<html>
  <head>
    <title>Project Submission</title>
    <style>
      .errorlist { color: red; }
      input { padding: 8px; width: 300px; }
    </style>
  </head>
  <body>
    <h1>Project File Submission</h1>
    <form method="POST" enctype="multipart/form-data">
      {% csrf_token %}
      {{ form.as_p }}
      <button type="submit">Submit Project</button>
    </form>
  </body>
</html>
</pre>
```

_Listing 3.145: Project submission form template._

**Submission list template:**

```html
<pre>
<!-- templates/projectsubmission/submission_list.html -->
<!DOCTYPE html>
<html>
    <head>
        <title>Submissions List</title>
    </head>
    <body>
        <h1>Project Submissions</h1>
        <a href="{% url 'project_upload' %}">+ Submit New Project</a>
        <table border="1" cellpadding="10" style="margin-top: 20px;">
            <tr>
                <th>Registration No.</th>
                <th>Email</th>
                <th>Project File</th>
                <th>Uploaded At</th>
            </tr>
            {% for submission in submissions %}
                <tr>
                    <td>{{ submission.tu_registration_number }}</td>
                    <td>{{ submission.email }}</td>
                    <td>
                        <a href="{{ submission.project_file.url }}" target="_blank">View File</a>
                    </td>
                    <td>{{ submission.uploaded_at }}</td>
                </tr>
            {% empty %}
                <tr>
                    <td colspan="4">No submissions yet.</td>
                </tr>
            {% endfor %}
        </table>
    </body>
</html>
</pre>
```

_Listing 3.146: Submission list template._

**URL and media configuration:**

```python
# app-level urls.py
from django.urls import path
from . import views

urlpatterns = [
    path('', views.submission_list, name='submission_list'),
    path('submit/', views.project_upload, name='project_upload'),
]
```

```python
# project-level urls.py
from django.contrib import admin
from django.urls import path, include
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
    path('admin/', admin.site.urls),
    path('projectsubmission/', include('projectsubmission.urls')),
]

if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

_Listing 3.147: URL and media configurations for the project submission app._

---

## Middleware

### What Is Middleware?

Middleware is a layer of software that sits between the incoming HTTP request and the outgoing HTTP response, processing each as it flows through the Django application. Middleware components form a chain—each request passes through every middleware in order before reaching the view, and each response passes back through the same chain in reverse order.

Each middleware component can:

- Inspect or modify an incoming request before it reaches the view.
- Inspect or modify an outgoing response before it is sent to the client.
- Short-circuit the request processing by returning a response early (without calling the view).
- Add headers, log information, handle errors, or perform other cross-cutting concerns.

_Table 3.19: Common middleware use cases._

| Use Case               | Description                                         |
| :--------------------- | :-------------------------------------------------- |
| **Authentication**     | Verify that the user is logged in                   |
| **Logging**            | Record all requests for debugging and auditing      |
| **Security**           | Add security headers (HSTS, X-Frame-Options)        |
| **CORS**               | Handle cross-origin resource sharing                |
| **Compression**        | Compress response bodies (e.g., gzip)               |
| **Caching**            | Cache responses to improve performance              |
| **Session Management** | Read and write session data                         |
| **Error Handling**     | Catch exceptions and produce custom error responses |

### Django's Built-In Middleware

Django ships with several middleware classes enabled by default in `settings.py`. Each serves a specific purpose:

```python
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    # Adds security headers; redirects HTTP to HTTPS

    'django.contrib.sessions.middleware.SessionMiddleware',
    # Reads the session ID from cookies and loads session data

    'django.middleware.common.CommonMiddleware',
    # URL normalisation (e.g., appending trailing slashes)

    'django.middleware.csrf.CsrfViewMiddleware',
    # Validates CSRF tokens on POST requests

    'django.contrib.auth.middleware.AuthenticationMiddleware',
    # Attaches request.user by resolving the user from the session

    'django.contrib.messages.middleware.MessageMiddleware',
    # Enables the messages framework

    'django.middleware.clickjacking.XFrameOptionsMiddleware',
    # Adds X-Frame-Options header to prevent clickjacking
]
```

_Listing 3.148: Django's default middleware stack with annotations._

> **Important:** The order of middleware in the `MIDDLEWARE` list matters. Middleware is processed top-to-bottom for requests and bottom-to-top for responses. For example, `SecurityMiddleware` should be near the top to ensure security headers are added to all responses, while `AuthenticationMiddleware` must come after `SessionMiddleware` because it depends on session data.

### Creating Custom Middleware

Django supports function-based middleware. A function-based middleware is a callable that accepts a `get_response` callable and returns a middleware function:

**Example: Request Timing Middleware.** The following middleware logs the processing time for each request:

Create a `middleware.py` file in the project configuration package:

```python
# myproject/middleware.py
import time


def request_timing_middleware(get_response):
    """Log the time taken to process each request."""

    def middleware(request):
        start_time = time.time()

        response = get_response(request)

        duration = time.time() - start_time
        print(f"{request.path} took {duration:.2f} seconds. "
              f"Completed with status {response.status_code}.")

        return response

    return middleware
```

_Listing 3.149: Custom function-based middleware for request timing._

**Register the middleware** in `settings.py`:

```python
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',

    # Custom middleware
    'myproject.middleware.request_timing_middleware',

    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
]
```

_Listing 3.150: Registering custom middleware in settings.py._

When a request is processed, the terminal displays output such as:

```text
/login/ took 0.08 seconds. Completed with status 200.
/dashboard/ took 0.34 seconds. Completed with status 200.
```

_Listing 3.151: Terminal output from the request timing middleware._

### Error Handling Middleware

A more advanced middleware example intercepts exceptions raised by views and returns structured JSON error responses instead of Django's default error pages:

```python
# myproject/middleware.py
from django.http import JsonResponse


def error_handling_middleware(get_response):
    """Catch exceptions and return structured JSON error responses."""

    def middleware(request):
        return get_response(request)

    def process_exception(request, exception):
        print(f"Error {request.method} on {request.path}")

        if isinstance(exception, PermissionError):
            return JsonResponse(
                {"error": "Forbidden", "detail": str(exception)}, status=403)

        if isinstance(exception, FileNotFoundError):
            return JsonResponse(
                {"error": "Not Found", "detail": str(exception)}, status=404)

        if isinstance(exception, ValueError):
            return JsonResponse(
                {"error": "Bad Request", "detail": str(exception)}, status=400)

        return JsonResponse(
            {"error": "Internal Server Error",
             "detail": "An unexpected error occurred."}, status=500)

    middleware.process_exception = process_exception
    return middleware
```

_Listing 3.152: Error handling middleware that returns JSON responses for different exception types._

To test this middleware, create views that raise different exceptions:

```python
# views.py
from django.http import HttpResponse

def trigger_value_error(request):
    raise ValueError("Invalid input: age cannot be negative")

def trigger_permission_error(request):
    raise PermissionError("You do not have access to this resource")

def trigger_generic_error(request):
    raise RuntimeError("Something went catastrophically wrong")

def safe_view(request):
    return HttpResponse("Everything is fine")
```

_Listing 3.153: Test views that raise various exceptions._

```python
# app-level urls.py
from django.urls import path
from . import views

urlpatterns = [
    path('value-error/', views.trigger_value_error, name='value_error'),
    path('permission-error/', views.trigger_permission_error, name='permission_error'),
    path('generic-error/', views.trigger_generic_error, name='generic_error'),
    path('safe/', views.safe_view, name='safe'),
]
```

_Listing 3.154: URL patterns for testing error handling middleware._

Register the error handling middleware above `SessionMiddleware` so that it can catch exceptions from all subsequent middleware and views:

```python
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'myproject.middleware.error_handling_middleware',    # Error handling
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'myproject.middleware.request_timing_middleware',    # Timing
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]
```

_Listing 3.155: Middleware stack with both custom middleware components._

---

## Comparison of Backend Frameworks

### What Is a Web Framework?

A web framework is a software library designed to support the development of web applications. It provides pre-built components for common tasks (routing, database access, authentication), a structure and set of conventions for organising code, tools for handling HTTP requests and responses, database integration, security features, and development utilities (development servers, debugging tools, migration systems).

Using a framework saves development time, enforces best practices, and reduces the likelihood of security vulnerabilities.

### Types of Web Frameworks

_Table 3.20: Categories of web frameworks._

| Type           | Description                                       | Examples               |
| :------------- | :------------------------------------------------ | :--------------------- |
| **Full-Stack** | Comprehensive—includes ORM, templates, admin      | Django, Rails, Laravel |
| **Micro**      | Minimal core; developers add extensions as needed | Flask, Express.js      |
| **API-First**  | Optimised for building APIs                       | FastAPI, .NET Web API  |

### Django (Python)

Django is a full-stack framework following the "batteries included" philosophy. It ships with a built-in admin panel, ORM, authentication system, form handling, template engine, CSRF/XSS protection, URL routing, and middleware support.

Django is best suited for content management systems, e-commerce sites, social networks, and data-driven applications. It powers large-scale platforms such as Instagram and Pinterest.

_Table 3.21: Django — advantages and disadvantages._

| Advantages                                  | Disadvantages                                |
| :------------------------------------------ | :------------------------------------------- |
| Complete package with all features built in | Can be excessive for small projects          |
| Automatic admin interface                   | Steeper learning curve due to many concepts  |
| Built-in protection against common attacks  | Encourages the "Django way" over flexibility |
| Excellent official documentation            |                                              |
| Large, active community                     |                                              |

### Flask (Python)

Flask is a micro framework with a minimal core and maximum flexibility. It provides Jinja2 templating and the Werkzeug WSGI toolkit but does not include an ORM, form validation, or admin panel—these are added through extensions (e.g., SQLAlchemy, WTForms).

Flask is best suited for small to medium applications, APIs, microservices, prototypes, and learning web development.

_Table 3.22: Flask — advantages and disadvantages._

| Advantages                           | Disadvantages                           |
| :----------------------------------- | :-------------------------------------- |
| Simple and easy to learn             | Must add extensions for common features |
| Full control over component choices  | No standard project structure           |
| Lightweight footprint                | No built-in admin, forms, or ORM        |
| Explicit code with no hidden "magic" |                                         |

### FastAPI (Python)

FastAPI is an API-first micro framework designed for speed, modern Python features, and automatic documentation. It supports native async/await, automatic API documentation (Swagger/OpenAPI), type-hint-based validation, dependency injection, and OAuth2.

FastAPI is best suited for REST APIs, microservices, high-performance applications, and machine learning APIs.

_Table 3.23: FastAPI — advantages and disadvantages._

| Advantages                                   | Disadvantages                              |
| :------------------------------------------- | :----------------------------------------- |
| One of the fastest Python frameworks         | Not ideal for traditional web applications |
| Native async/await and type hints            | Smaller community than Django or Flask     |
| Automatic Swagger UI documentation           | Async code can be harder to debug          |
| Automatic request validation from type hints |                                            |

### ASP.NET Core MVC (C#/.NET)

ASP.NET Core is a full-stack, cross-platform framework built on .NET Core. It features MVC architecture, Entity Framework ORM, Razor templating, dependency injection, strong typing, and the Identity system for authentication.

It is best suited for enterprise applications, Windows environments, large-scale systems, and projects within the Microsoft ecosystem.

_Table 3.24: ASP.NET Core — advantages and disadvantages._

| Advantages                       | Disadvantages                       |
| :------------------------------- | :---------------------------------- |
| Very fast (compiled language)    | Steeper learning curve              |
| Designed for large organisations | More verbose than dynamic languages |
| Excellent Visual Studio tooling  | Some tools require licences         |
| Compile-time type safety         |                                     |

### Ruby on Rails (Ruby)

Ruby on Rails is a full-stack framework following the "convention over configuration" philosophy. It features MVC architecture, Active Record ORM, scaffold generation, database migrations, an asset pipeline, and Action Cable for WebSockets.

It is best suited for startups, rapid prototyping, MVPs, e-commerce platforms, and content-heavy sites.

_Table 3.25: Ruby on Rails — advantages and disadvantages._

| Advantages                              | Disadvantages                       |
| :-------------------------------------- | :---------------------------------- |
| Very rapid development                  | Slower than compiled languages      |
| Strong conventions reduce configuration | Fewer hosting options               |
| Clean, readable Ruby syntax             | Requires learning the Ruby language |
| Mature ecosystem (20+ years)            |                                     |

### Spring Boot (Java)

Spring Boot is a full-stack framework that simplifies enterprise Java development. It provides standalone applications, embedded servers, auto-configuration, the Spring ecosystem (Security, Data JPA), and dependency injection.

It is best suited for enterprise applications, microservices, large-scale systems, and Java ecosystem projects.

_Table 3.26: Spring Boot — advantages and disadvantages._

| Advantages                          | Disadvantages                 |
| :---------------------------------- | :---------------------------- |
| Huge Spring ecosystem               | Verbose with boilerplate code |
| Industry standard for large systems | Many concepts to learn        |
| JVM performance optimisation        | JVM memory overhead           |
| Massive community support           |                               |

### Express.js (JavaScript/Node.js)

Express.js is a minimalist, unopinionated micro framework for Node.js. It provides JavaScript on the server, non-blocking I/O, the npm ecosystem, middleware-based architecture, template engine support, and WebSocket support.

It is best suited for real-time applications, APIs and microservices, single-page applications (with React or Vue), and JavaScript full-stack development.

_Table 3.27: Express.js — advantages and disadvantages._

| Advantages                                         | Disadvantages                              |
| :------------------------------------------------- | :----------------------------------------- |
| Same language (JavaScript) on frontend and backend | Async code can become complex              |
| Millions of npm packages                           | No enforced project structure              |
| Excellent for real-time and streaming              | Single-threaded; CPU-intensive tasks block |
| Fast V8 engine with non-blocking I/O               |                                            |

### Framework Comparison

_Table 3.28: Side-by-side comparison of popular backend frameworks._

| Feature         | Django    | Flask     | FastAPI   | ASP.NET Core   | Rails  | Spring Boot | Express.js |
| :-------------- | :-------- | :-------- | :-------- | :------------- | :----- | :---------- | :--------- |
| **Language**    | Python    | Python    | Python    | C#             | Ruby   | Java        | JavaScript |
| **Type**        | Full      | Micro     | API       | Full           | Full   | Full        | Micro      |
| **ORM**         | Built-in  | Extension | Extension | EF Core        | AR     | JPA         | Extension  |
| **Admin**       | Built-in  | No        | No        | No             | No     | No          | No         |
| **Learning**    | Medium    | Easy      | Easy      | Hard           | Medium | Hard        | Easy       |
| **Performance** | Good      | Good      | Excellent | Excellent      | Fair   | Excellent   | Good       |
| **Async**       | Limited   | Limited   | Native    | Native         | No     | Yes         | Native     |
| **Used By**     | Instagram | Netflix   | Microsoft | Stack Overflow | GitHub | Alibaba     | Netflix    |

### Choosing the Right Framework

_Table 3.29: Framework recommendations by project type._

| Consideration         | Recommendation                          |
| :-------------------- | :-------------------------------------- |
| **Beginner**          | Flask, Express.js                       |
| **Rapid Development** | Django, Rails                           |
| **APIs**              | FastAPI, Express.js                     |
| **Enterprise**        | Spring Boot, ASP.NET Core               |
| **Full-Stack**        | Django, Rails, ASP.NET Core             |
| **Microservices**     | FastAPI, Express.js, Spring Boot        |
| **Real-Time**         | Express.js (Socket.io), Django Channels |

---

## Database Types

### What Is a Database?

A database is an organised collection of structured data stored electronically. Databases enable applications to store data permanently, retrieve data efficiently, update and delete data, maintain data integrity through constraints, and handle concurrent access from multiple users.

### Relational Databases (SQL)

Relational databases store data in tables consisting of rows and columns. Tables can be related to one another through keys (primary keys and foreign keys). Relational databases use SQL (Structured Query Language) for data manipulation and are ACID compliant (Atomicity, Consistency, Isolation, Durability), ensuring reliable transactions.

Key characteristics:

- Data is stored in structured tables with a predefined schema.
- Strong data integrity enforced through constraints and relationships.
- Complex queries involving joins across multiple tables are supported.
- Transactions guarantee that a series of operations either all succeed or all fail.

_Table 3.30: Popular relational databases._

| Database       | Description                         |
| :------------- | :---------------------------------- |
| **PostgreSQL** | Advanced, open-source, feature-rich |
| **MySQL**      | Popular, fast, widely used          |
| **SQLite**     | Lightweight, file-based, embedded   |
| **Oracle**     | Enterprise, commercial              |
| **SQL Server** | Microsoft, enterprise               |

**Example table structure:**

```text
┌─────────────────────────────────────────────────────┐
│                    users TABLE                      │
├─────┬──────────┬─────────────────────┬──────────────┤
│ id  │ username │ email               │ created_at   │
├─────┼──────────┼─────────────────────┼──────────────┤
│ 1   │ john     │ john@example.com    │ 2026-01-15   │
│ 2   │ jane     │ jane@example.com    │ 2026-01-16   │
│ 3   │ bob      │ bob@example.com     │ 2026-01-17   │
└─────┴──────────┴─────────────────────┴──────────────┘
```

_Listing 3.156: Visual representation of a relational database table._

### NoSQL Databases

NoSQL databases store data in flexible formats other than traditional tables. They are designed for scalability and handling unstructured or semi-structured data.

_Table 3.31: Types of NoSQL databases._

| Type              | Description                     | Examples         |
| :---------------- | :------------------------------ | :--------------- |
| **Document**      | Stores JSON-like documents      | MongoDB, CouchDB |
| **Key-Value**     | Simple key-value pairs          | Redis, DynamoDB  |
| **Column-Family** | Columns grouped into families   | Cassandra, HBase |
| **Graph**         | Nodes and edges (relationships) | Neo4j, ArangoDB  |

Key characteristics:

- Flexible, schema-less design allows evolving data structures.
- Horizontal scaling (adding more servers) is straightforward.
- High performance for specific access patterns.
- Suited to unstructured or semi-structured data.
- Often eventually consistent rather than strictly ACID compliant.

**Example document (MongoDB):**

```json
{
  "_id": "507f1f77bcf86cd799439011",
  "username": "john",
  "email": "john@example.com",
  "profile": {
    "bio": "Software developer",
    "avatar": "avatar.jpg"
  },
  "tags": ["developer", "python", "django"]
}
```

_Listing 3.157: A MongoDB document illustrating the flexible, nested structure of a document database._

### Relational versus NoSQL — Comparison

_Table 3.32: Comparison of relational and NoSQL databases._

| Aspect             | Relational (SQL)               | NoSQL                              |
| :----------------- | :----------------------------- | :--------------------------------- |
| **Schema**         | Fixed, predefined              | Flexible, dynamic                  |
| **Scaling**        | Vertical (larger server)       | Horizontal (more servers)          |
| **Relationships**  | Strong (foreign keys, joins)   | Weak or embedded                   |
| **Query Language** | SQL                            | Varies (e.g., MongoDB uses JSON)   |
| **Transactions**   | ACID compliant                 | Often eventually consistent        |
| **Best For**       | Structured data, relationships | Flexibility, large-scale workloads |
| **Examples**       | Banking, ERP, CRM              | Social media, IoT, logging         |

**Choose a relational database when:** data has clear relationships, data integrity is critical, complex queries and joins are required, transactions are important, and the schema is well-defined.

**Choose a NoSQL database when:** the schema is evolving or unknown, horizontal scaling is required, handling large volumes of data, data is semi-structured or unstructured, and speed is more important than strict consistency.

### CRUD — The Four Fundamental Database Operations

CRUD represents the four basic operations for persistent storage:

_Table 3.33: CRUD operations mapped to SQL commands and HTTP methods._

| Operation  | Description          | SQL Command | HTTP Method |
| :--------- | :------------------- | :---------- | :---------- |
| **C**reate | Add new data         | INSERT      | POST        |
| **R**ead   | Retrieve data        | SELECT      | GET         |
| **U**pdate | Modify existing data | UPDATE      | PUT/PATCH   |
| **D**elete | Remove data          | DELETE      | DELETE      |

**CRUD with raw SQL:**

```sql
-- CREATE: Insert a new record
INSERT INTO users (username, email) VALUES ('john', 'john@example.com');

-- READ: Select records
SELECT * FROM users;                    -- All users
SELECT * FROM users WHERE id = 1;       -- Specific user
SELECT username, email FROM users;      -- Specific columns

-- UPDATE: Modify a record
UPDATE users SET email = 'newemail@example.com' WHERE id = 1;

-- DELETE: Remove a record
DELETE FROM users WHERE id = 1;
```

_Listing 3.158: CRUD operations using raw SQL._

---

## Building a Complete CRUD Application

This section presents a comprehensive, optimised CRUD (Create, Read, Update, Delete) application that integrates the concepts covered throughout this chapter: project setup, models, forms with validation, views, template inheritance, static files, the admin interface, and middleware. The application is a notes manager built from scratch.

### Project Setup

**Installing Python and Django:**

```bash
# Verify Python installation
python --version

# Create a project folder and virtual environment
mkdir crud
cd crud
python -m venv venv

# Activate the virtual environment
# Windows (Command Prompt):
venv\Scripts\activate
# macOS/Linux:
source venv/bin/activate

# Install Django
pip install django
django-admin --version
```

_Listing 3.159: Setting up Python, a virtual environment, and Django._

**Creating the project and app:**

```bash
django-admin startproject crud .
python manage.py startapp notes
```

_Listing 3.160: Creating the CRUD project and the notes app._

### Model Definition

Register the app in `settings.py`:

```python
# crud/settings.py
INSTALLED_APPS = [
    # ... default apps ...
    'notes',
]
```

_Listing 3.161: Registering the notes app._

Define the model:

```python
# notes/models.py
from django.db import models


class Note(models.Model):
    title = models.CharField(max_length=100)
    description = models.TextField()

    def __str__(self):
        return self.title
```

_Listing 3.162: Note model._

The equivalent SQL for this model is:

```sql
CREATE TABLE notes_note (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    title VARCHAR(100) NOT NULL,
    description TEXT NOT NULL
);
```

_Listing 3.163: SQL equivalent of the Note model._

Run migrations and create sample data:

```bash
python manage.py makemigrations
python manage.py migrate
```

_Listing 3.164: Running migrations._

```bash
python manage.py shell
```

```python
from notes.models import Note

Note.objects.create(title="First Note", description="This is my first note with enough characters")
Note.objects.create(title="Django Tutorial", description="Learning Django CRUD operations with forms and validation")
Note.objects.create(title="Shopping List", description="Buy groceries: milk, eggs, bread, and vegetables")

Note.objects.all()
exit()
```

_Listing 3.165: Creating sample notes via the Django shell._

### Form with Validation

The form class encapsulates validation logic and provides helper methods for creating and updating model instances:

```python
# notes/forms.py
from django import forms
from .models import Note


class NoteForm(forms.Form):
    title = forms.CharField(
        max_length=100,
        widget=forms.TextInput(attrs={'id': 'title'}),
        error_messages={'required': 'Title is required'}
    )
    description = forms.CharField(
        widget=forms.Textarea(attrs={'id': 'description'}),
        error_messages={'required': 'Description is required'}
    )

    def clean_description(self):
        description = self.cleaned_data.get('description', '')
        if len(description) < 10:
            raise forms.ValidationError(
                'Description must be at least 10 characters long.'
            )
        return description

    def create(self, validated_data):
        return Note.objects.create(**validated_data)

    def update(self, instance, validated_data):
        instance.title = validated_data.get('title', instance.title)
        instance.description = validated_data.get('description', instance.description)
        instance.save()
        return instance
```

_Listing 3.166: NoteForm with custom validation and create/update helper methods._

### CRUD Views

```python
# notes/views.py
from django.shortcuts import render, redirect, get_object_or_404
from django.contrib import messages
from .models import Note
from .forms import NoteForm


# READ — Display all notes
def index(request):
    notes = Note.objects.all().order_by('-id')
    return render(request, 'notes/index.html', {'notes': notes})


# CREATE — Add a new note
def add_note(request):
    if request.method == 'POST':
        form = NoteForm(request.POST)
        if form.is_valid():
            form.create(form.cleaned_data)
            messages.success(request, 'Data added successfully!')
            return redirect('notes:index')
    else:
        form = NoteForm()

    return render(request, 'notes/add.html', {'form': form})


# UPDATE — Edit an existing note
def edit_note(request, note_id):
    note = get_object_or_404(Note, id=note_id)

    if request.method == 'POST':
        form = NoteForm(request.POST)
        if form.is_valid():
            form.update(note, form.cleaned_data)
            messages.success(request, 'Data updated successfully!')
            return redirect('notes:index')
    else:
        form = NoteForm(initial={
            'title': note.title,
            'description': note.description
        })

    return render(request, 'notes/edit.html', {'form': form})


# DELETE — Remove a note
def delete_note(request, note_id):
    note = get_object_or_404(Note, id=note_id)
    note.delete()
    messages.success(request, 'Note deleted successfully!')
    return redirect('notes:index')
```

_Listing 3.167: Complete CRUD views using the NoteForm._

### Static Files

Create a CSS stylesheet for the app:

```css
/* notes/static/notes/css/style.css */
table {
  border-collapse: collapse;
}

th,
td {
  border: 1px solid black;
  padding: 10px;
}

.success {
  color: green;
}

.error {
  color: red;
}

.errorlist {
  color: red;
  list-style: none;
  padding: 0;
}
```

_Listing 3.168: Application stylesheet._

Configure static files in `settings.py`:

```python
STATIC_URL = 'static/'

STATICFILES_DIRS = [
    BASE_DIR / 'notes' / 'static',
]
```

_Listing 3.169: Static file configuration._

### Templates with Inheritance

**Navigation partial** (`notes/templates/notes/partials/navigation.html`):

```html
<pre>
<nav>
    <p>
        <a href="{% url 'notes:index' %}">Home</a>
        |
        <a href="{% url 'notes:add' %}">Add New Data</a>
    </p>
</nav>
</pre>
```

_Listing 3.170: Navigation partial template._

**Base template** (`notes/templates/notes/base.html`):

```html
<pre>
{% load static %}
<!DOCTYPE html>
<html>
    <head>
        <title>
            {% block title %}
                Notes App
            {% endblock title %}
        </title>
        <link rel="stylesheet" href="{% static 'notes/css/style.css' %}" />
    </head>
    <body>
        {% include 'notes/partials/navigation.html' %}
        {% if messages %}
            {% for message in messages %}<p class="{{ message.tags }}">{{ message }}</p>{% endfor %}
        {% endif %}
        {% block content %}
        {% endblock content %}
    </body>
</html>
</pre>
```

_Listing 3.171: Base template with static CSS, navigation include, messages, and content block._

**Index template** (`notes/templates/notes/index.html`):

```html
<pre>
{% extends 'notes/base.html' %}
{% block title %}
    Homepage
{% endblock title %}
{% block content %}
    <h2>Homepage</h2>
    <table width="80%">
        <tr>
            <th>Title</th>
            <th>Description</th>
            <th>Action</th>
        </tr>
        {% for note in notes %}
            <tr>
                <td>{{ note.title }}</td>
                <td>{{ note.description }}</td>
                <td>
                    <a href="{% url 'notes:edit' note.id %}">Edit</a>
                    |
                    <a href="{% url 'notes:delete' note.id %}" onclick="return confirm('Are you sure you want to delete this note?');">Delete</a>
                </td>
            </tr>
        {% empty %}
            <tr>
                <td colspan="3">No results found.</td>
            </tr>
        {% endfor %}
    </table>
{% endblock content %}
</pre>
```

_Listing 3.172: Index template listing all notes with edit and delete actions._

**Add template** (`notes/templates/notes/add.html`):

```html
<pre>
{% extends 'notes/base.html' %}
{% block title %}
    Add Note
{% endblock title %}
{% block content %}
    <h2>Add Note</h2>
    <form action="{% url 'notes:add' %}" method="post">
        {% csrf_token %} {{ form.as_p }}
        <input name="submit" type="submit" value="Submit" />
    </form>
{% endblock content %}
</pre>
```

_Listing 3.173: Add note template using form rendering._

**Edit template** (`notes/templates/notes/edit.html`):

```html
<pre>
{% extends 'notes/base.html' %}
{% block title %}
    Edit Note
{% endblock title %}
{% block content %}
    <h2>Edit Note</h2>
    <form method="post">
        {% csrf_token %} {{ form.as_p }}
        <input name="submit" type="submit" value="Update" />
    </form>
{% endblock content %}
</pre>
```

_Listing 3.174: Edit note template._

### URL Configuration

```python
# notes/urls.py
from django.urls import path
from . import views

app_name = 'notes'

urlpatterns = [
    path('', views.index, name='index'),
    path('add/', views.add_note, name='add'),
    path('edit/<int:note_id>/', views.edit_note, name='edit'),
    path('delete/<int:note_id>/', views.delete_note, name='delete'),
]
```

```python
# crud/urls.py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('notes/', include('notes.urls')),
]
```

_Listing 3.175: URL configurations for the notes app._

Start the development server and navigate to `http://127.0.0.1:8000/notes/`:

```bash
python manage.py runserver
```

_Listing 3.176: Starting the development server._

### Admin Registration

Create a superuser and register the model with the admin site:

```bash
python manage.py createsuperuser
```

_Listing 3.177: Creating a superuser._

```python
# notes/admin.py
from django.contrib import admin
from .models import Note

admin.site.register(Note)
```

_Listing 3.178: Registering the Note model with the admin site._

Navigate to `http://127.0.0.1:8000/admin` to access the admin panel.

---

## Testing

### Automated Unit Testing

Unit testing verifies that individual functions or methods work correctly in isolation, independent of other components. Django includes a test framework built on Python's `unittest` module. Tests are defined as classes that inherit from `django.test.TestCase` and contain methods whose names begin with `test_`.

**Testing a utility function.** Create a simple utility and its corresponding test:

```python
# notes/utils.py
def add(a, b):
    return a + b
```

_Listing 3.179: A simple utility function._

```python
# notes/tests.py
from django.test import TestCase
from .utils import add


class AddFunctionTestCase(TestCase):
    def test_add(self):
        self.assertEqual(add(2, 3), 5)
```

_Listing 3.180: Unit test for the add function._

Run the tests:

```bash
python manage.py test
```

```text
Ran 1 test in 0.001s

OK
```

_Listing 3.181: Running Django tests and expected output._

### Integration Testing

Integration testing verifies that multiple components—such as URLs, views, models, and templates—work together correctly as a whole. Django's test client allows testing views without starting a web server. The following tests verify that the homepage renders correctly and that the note creation and validation logic work as expected:

```python
# notes/tests.py
from django.test import TestCase
from django.urls import reverse
from .utils import add


class AddFunctionTestCase(TestCase):
    def test_add(self):
        self.assertEqual(add(2, 3), 5)


class HomePageTestCase(TestCase):
    def test_home_page_contains_homepage_text(self):
        response = self.client.get(reverse('notes:index'))
        self.assertEqual(response.status_code, 200)
        self.assertContains(response, 'Homepage')


class NotesTestCase(TestCase):
    def test_notes_can_be_created(self):
        response = self.client.post(reverse('notes:add'), {
            'title': 'Django Course1',
            'description': 'Complete course with urls, templates, models, etc'
        })
        # Redirect after successful creation
        self.assertEqual(response.status_code, 302)

        # Follow redirect and verify the note appears
        response = self.client.get(response.url)
        self.assertContains(response, 'Django Course')

    def test_error_occurs_if_description_is_less_than_10_chars_long(self):
        response = self.client.post(reverse('notes:add'), {
            'title': 'Django Course',
            'description': 'dj'
        })
        # Form re-rendered with errors (no redirect)
        self.assertEqual(response.status_code, 200)
        self.assertContains(
            response, 'Description must be at least 10 characters long')
```

_Listing 3.182: View and integration tests for the notes app._

```text
Ran 4 tests in 0.062s

OK
```

_Listing 3.183: Output after running all four tests._

### Continuous Integration with GitHub Actions

GitHub Actions automates the testing process by running tests on every push to the repository. Create a workflow file:

```text
.github/workflows/django.yml
```

```yaml
name: Django Tests

on:
  push:
    branches: ["main"]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.14"

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt

      - name: Run tests
        run: |
          python manage.py test
```

_Listing 3.184: GitHub Actions workflow for automated testing._

Generate the `requirements.txt` file before pushing:

```bash
pip freeze > requirements.txt
```

_Listing 3.185: Generating requirements.txt._

Once pushed, every commit to the `main` branch triggers the workflow. GitHub creates a virtual machine, installs Python and project dependencies, and runs the test suite. Results are visible under the **Actions** tab of the GitHub repository.

---

## Deployment

### Deploying on Render

Render is a cloud platform that supports automatic deployments from a Git repository. The following steps configure the CRUD application for production deployment.

**Step 1: Update `settings.py` for production.**

```python
import os

# ...

DEBUG = 'RENDER' not in os.environ

ALLOWED_HOSTS = []

RENDER_EXTERNAL_HOSTNAME = os.environ.get('RENDER_EXTERNAL_HOSTNAME')
if RENDER_EXTERNAL_HOSTNAME:
    ALLOWED_HOSTS.append(RENDER_EXTERNAL_HOSTNAME)
```

_Listing 3.186: Production settings for Render deployment._

**Step 2: Set up static file serving with WhiteNoise.** In production, Django's development server is not available to serve static files. WhiteNoise serves static files directly from the application:

```bash
pip install "whitenoise[brotli]"
```

_Listing 3.187: Installing WhiteNoise with Brotli compression support._

Update `settings.py`:

```python
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'whitenoise.middleware.WhiteNoiseMiddleware',
    # ... remaining middleware ...
]

# ...

STATIC_URL = '/static/'

if not DEBUG:
    STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')
    STATICFILES_STORAGE = 'whitenoise.storage.CompressedManifestStaticFilesStorage'
```

_Listing 3.188: WhiteNoise middleware and static file settings for production._

**Step 3: Create a build script** (`build.sh`):

```sh
set -o errexit

pip install -r requirements.txt

python manage.py collectstatic --no-input

python manage.py migrate
```

_Listing 3.189: Build script for Render._

The `set -o errexit` flag causes the script to exit immediately if any command fails. The `--no-input` flag runs `collectstatic` without prompting for confirmation.

**Step 4: Install Gunicorn and Uvicorn.** Gunicorn is a production-grade WSGI/ASGI HTTP server:

```bash
pip install gunicorn uvicorn
pip freeze > requirements.txt
```

_Listing 3.190: Installing Gunicorn and Uvicorn._

Test locally:

```bash
uvicorn crud.asgi:application
```

_Listing 3.191: Running the application with Uvicorn locally._

**Step 5: Define the web service** (`render.yaml`):

```yaml
services:
  - type: web
    plan: free
    name: crud
    runtime: python
    buildCommand: "./build.sh"
    startCommand: "python -m gunicorn crud.asgi:application -k uvicorn.workers.UvicornWorker"
```

_Listing 3.192: Render blueprint configuration._

**Step 6: Deploy from the Render dashboard.**

1. In the Render Dashboard, navigate to the **Blueprints** page and click **New Blueprint Instance**.
2. Select the repository containing the blueprint and click **Connect**.
3. Assign a name to the project and click **Deploy**.
4. The application will be live at its `.onrender.com` URL as soon as the build completes.

---

## UI Testing with Selenium

### Introduction to Selenium

Selenium is a browser automation framework that enables developers to write tests that interact with a web application through a real browser. Unlike Django's built-in test client (which simulates HTTP requests without rendering), Selenium tests execute JavaScript, render the DOM, and simulate user interactions such as clicking buttons and filling in forms.

### Writing Selenium Tests

Install Selenium and pytest:

```bash
pip install selenium pytest
```

_Listing 3.193: Installing Selenium and pytest._

Create a test file (`uitest.py`) at the project root:

```python
import time
from selenium import webdriver
from selenium.webdriver.common.by import By


def test_notes_can_be_created():
    # Arrange
    driver = webdriver.Chrome()
    driver.get('http://127.0.0.1:8000/notes/add/')

    # Act
    driver.find_element(By.NAME, 'title').send_keys('Django Course')
    driver.find_element(By.NAME, 'description').send_keys(
        'Complete course with urls, templates, models, etc')
    driver.find_element(By.NAME, 'submit').click()
    time.sleep(2)

    # Assert
    title = driver.find_element(By.TAG_NAME, 'td').text
    assert 'Django Course' in title

    driver.quit()


def test_error_occurs_if_description_is_less_than_10_chars_long():
    # Arrange
    driver = webdriver.Chrome()
    driver.get('http://127.0.0.1:8000/notes/add/')

    # Act
    driver.find_element(By.NAME, 'title').send_keys('Django Course')
    driver.find_element(By.NAME, 'description').send_keys('dj')
    driver.find_element(By.NAME, 'submit').click()
    time.sleep(2)

    # Assert
    error = driver.find_element(By.TAG_NAME, 'li').text
    assert 'Description must be at least 10 characters long' in error

    driver.quit()
```

_Listing 3.194: Selenium UI tests for the notes app._

Run the tests (the development server must be running separately):

```bash
pytest uitest.py
```

_Listing 3.195: Running Selenium tests with pytest._

> **Note:** The `time.sleep(2)` calls are a simple but crude approach to waiting for page loads. In production test suites, explicit waits using `WebDriverWait` and expected conditions are preferred, as they are both faster and more reliable.

### Selenium Test for a Static HTML Page

Selenium can also test simple HTML pages served by any web server. Consider a static login form:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Login</title>
  </head>
  <body>
    <form action="#" onsubmit="handleSubmit(event)">
      Username: <input type="text" name="username" required /><br /><br />
      Password: <input type="password" name="password" required /><br /><br />
      <button type="submit" id="login-button">Submit</button>
    </form>
    <script>
      function handleSubmit(event) {
        event.preventDefault();
        document.body.innerHTML += "<h3 id='welcome'>Welcome, testuser</h3>";
      }
    </script>
  </body>
</html>
```

_Listing 3.196: A static HTML login form for Selenium testing._

```python
from selenium import webdriver
from selenium.webdriver.common.by import By


def test_user_can_login():
    driver = webdriver.Chrome()
    driver.get('http://127.0.0.1:5500/login.html')

    driver.find_element(By.NAME, 'username').send_keys('testuser')
    driver.find_element(By.NAME, 'password').send_keys('testpass123')
    driver.find_element(By.ID, 'login-button').click()

    welcome_message = driver.find_element(By.ID, 'welcome').text
    assert 'Welcome, testuser' in welcome_message

    driver.quit()
```

_Listing 3.197: Selenium test for the static login form._

---

## Common Web Security Vulnerabilities

### Cross-Site Scripting (XSS)

Cross-Site Scripting (XSS) is a security vulnerability that allows attackers to inject malicious scripts into web pages viewed by other users. When a victim visits the compromised page, the injected script executes in their browser with the same privileges as legitimate scripts from the trusted website, potentially stealing cookies, session tokens, or other sensitive data.

**Demonstration.** Django's template engine auto-escapes variables by default, which prevents XSS. However, using the `|safe` filter disables this protection:

```html
<!-- Vulnerable: auto-escaping disabled -->
<td>{{ note.description|safe }}</td>
```

_Listing 3.198: Disabling auto-escaping with the safe filter (vulnerable)._

If an attacker submits the following as a note description:

```html
<script>
  alert(
    `You are hacked. Stealing your cookies. Your cookie data is: ${document.cookie}`,
  );
</script>
```

_Listing 3.199: Example XSS payload._

The script will execute in every user's browser when they view the note. Django's default auto-escaping prevents this by converting `<script>` tags into harmless `&lt;script&gt;` entities.

> **Important:** Never use the `|safe` filter or `{% autoescape off %}` on user-supplied content. Django's auto-escaping is a critical defence against XSS attacks.

### SQL Injection

SQL injection is a code injection technique that exploits applications which construct SQL queries using unsanitised user input. It allows attackers to read, modify, or delete data in the database.

**Demonstration.** The following view constructs a SQL query using string interpolation—a dangerously insecure approach:

```python
# views.py (VULNERABLE — do not use in production)
from django.db import connection

def add_note_sql_injection(request):
    if request.method == 'POST':
        form = NoteForm(request.POST)
        if form.is_valid():
            title = form.cleaned_data['title']
            desc = form.cleaned_data['description']

            sql = f"""
            INSERT INTO notes_note (title, description)
            VALUES ('{title}', '{desc}');
            """

            with connection.cursor() as cursor:
                cursor.executescript(sql)

            messages.success(request, 'Data added successfully!')
            return redirect('notes:index')
    else:
        form = NoteForm()

    return render(request, 'notes/add.html', {'form': form})
```

_Listing 3.200: A view vulnerable to SQL injection (string interpolation in SQL queries)._

An attacker who enters the following as a description can delete all data from the table:

```text
'); DELETE FROM notes_note; --
```

This input causes the server to execute:

```sql
INSERT INTO notes_note (title, description)
VALUES ('My Test Note', ''); DELETE FROM notes_note; -- ');
```

_Listing 3.201: SQL injection payload that deletes all records._

An even more destructive payload can drop the entire table:

```text
'); DROP TABLE notes_note; --
```

> **Important:** Never construct SQL queries using string formatting or interpolation with user input. Always use Django's ORM (`Note.objects.create(...)`) or parameterised queries (`cursor.execute("INSERT INTO ... VALUES (%s, %s)", [title, desc])`) to prevent SQL injection.

### Cross-Site Request Forgery (CSRF)

Cross-Site Request Forgery (CSRF) is an attack that forces authenticated users to submit unwanted requests to a web application. It exploits the trust that a website has in the user's browser—specifically, the fact that the browser automatically includes cookies (including session cookies) with every request to a domain.

**Demonstration.** An attacker creates a malicious page that auto-submits a form to the target application:

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Win iPhone</title>
  </head>
  <body>
    <h1>Win iPhone</h1>
    <form
      id="evilForm"
      action="http://127.0.0.1:8000/notes/add/"
      method="POST"
      style="display:none"
    >
      <input type="text" name="title" value="I was hacked!" />
      <input
        type="text"
        name="description"
        value="Attacker added this note using CSRF!"
      />
    </form>
    <script>
      window.onload = function () {
        document.getElementById("evilForm").submit();
      };
    </script>
  </body>
</html>
```

_Listing 3.202: A malicious page that performs a CSRF attack._

When a victim visits this page, the hidden form is automatically submitted to the Django application. Because the browser includes the victim's session cookie, the server processes the request as if it were legitimate.

Django's CSRF middleware (`CsrfViewMiddleware`) blocks this attack by default. The server rejects the request with a `403 Forbidden` error because the form does not include a valid CSRF token.

**What makes the application vulnerable.** If CSRF protection is disabled—either by removing `{% csrf_token %}` from the template or by decorating the view with `@csrf_exempt`—the attack succeeds:

```python
from django.views.decorators.csrf import csrf_exempt

@csrf_exempt
def add_note(request):
    # ... view code ...
```

_Listing 3.203: Disabling CSRF protection (vulnerable)._

> **Important:** Never disable CSRF protection on views that process state-changing requests (POST, PUT, DELETE). Always include `{% csrf_token %}` in Django forms and allow `CsrfViewMiddleware` to validate tokens.

**Running the CSRF demo with authentication.** To demonstrate the attack with an authenticated user—which reflects a real-world scenario—follow these steps:

1. Create a superuser:

```bash
python manage.py createsuperuser
```

2. Start the Django development server and navigate to `http://127.0.0.1:8000/admin`. Log in with the superuser credentials. This stores a session cookie in the browser.

3. Close the admin tab (but do not log out—the session cookie remains in the browser).

4. Update `notes/views.py` to disable CSRF protection and require authentication:

```python
from django.contrib.auth.decorators import login_required
from django.views.decorators.csrf import csrf_exempt

@login_required
@csrf_exempt
def add_note_sql_injection(request):
    # ... view code ...
```

_Listing 3.204: Combining login_required with csrf_exempt (vulnerable)._

5. Save the malicious HTML page (Listing 3.202) as `csrf.html` and open it via a Live Server at `http://127.0.0.1:5500/csrf.html`.

When the user is not authenticated, the `@login_required` decorator redirects to the login page. Since the malicious page submits to a Django URL, the redirect results in a `404` error on the attacker's origin—the attack fails.

However, when the user is authenticated (i.e., has already logged in via the admin panel), the browser automatically includes the session cookie with the forged request. Because `@csrf_exempt` disables token validation, Django accepts the request as legitimate and the attacker's data is inserted into the database. This demonstrates why CSRF is dangerous: the attack exploits the browser's automatic cookie-sending behaviour, not any flaw in the user's credentials.

---

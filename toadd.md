**At end of data and model:**

**Using slug instead of id**

**Update model**

```py
from django.core import validators
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
        # return reverse("book-detail", args=[self.id])

    def save(self, *args, **kwargs):
        self.slug = slugify(self.title)
        super().save(*args, **kwargs)

    def __str__(self):
        return f"{self.title} ({self.rating})"
```

makemigrations and migrate

**Just call save again to add slug**

```py
Book.objects.get(title="Harry Potter 1").save()
Book.objects.get(title="Harry Potter 1").slug
# 'harry-potter-1'
Book.objects.get(title="Lord of the Rings").save()
Book.objects.get(title="Lord of the Rings").slug
# 'lord-of-the-rings'
Book.objects.get(title="My Story").save()
Book.objects.get(title="Some random book").save()
```

**Update urls**

```py
from django.urls import path

from . import views

urlpatterns = [
    path("", views.index),
    path("<slug:slug>", views.book_detail, name="book-detail")
]
```

**Update view**

```py
def book_detail(request, slug):
  book = get_object_or_404(Book, slug=slug)
  return render(request, "book_outlet/book_detail.html", {
    "title": book.title,
    "author": book.author,
    "rating": book.rating,
    "is_bestseller": book.is_bestselling
  })
```

**Update index page**

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

---

**Adding Summary**

**Update view**

```py
from django.db.models import Avg

def index(request):
  books = Book.objects.all().order_by("-rating")
  num_books = books.count()
  avg_rating = books.aggregate(Avg("rating")) # rating__avg, rating__min

  return render(request, "book_outlet/index.html", {
    "books": books,
    "total_number_of_books": num_books,
    "average_rating": avg_rating
  })
```

**Update index page**

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

---

---

---

---

<!-- On Admin Lesson -->

<!-- editable: False in django model fields -->

```py
prepopulated_fields = {"slug": ("title",)}
# save on models is removed for testing prepopulated_fields
```

---

---

---

---

### Relationships

describe:
one-to-one
one-to-many
many-to-many

---

one to many

```py

class Author(models.Model):
    first_name = models.CharField(max_length=100)
    last_name = models.CharField(max_length=100)

    def full_name(self):
        return f"{self.first_name} {self.last_name}"

    def __str__(self):
        return self.full_name()

class Book(models.Model):
    author = models.ForeignKey(Author, on_delete=models.CASCADE, null=True, related_name="books")
```

makemigrations and migrate

on python shell:

```text
book_store $ python3 manage.py shell
Python 3.9.1 (v3.9.1:1e5d33e9b9, Dec 7 2020, 12:10:52)
[Clang 6.0 (clang-600.0.57)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
(InteractiveConsole)
>>> from book_outlet.models import Book, Author
>>> jkrowling = Author(first_name="J.K.", last_name="Rowling")
>>> jkrowling.save()
>>> Author.objects.all()
<QuerySet [<Author: Author object (1)>]>
>>> Author.objects.all()[0].first_name
'J.K.'
>>> hp1 = Book(title="Harry Potter 1", rating=5, is_bestselling=True, slug="harry-potter-1"
, author=jkrowling)
>>> hp1.save()
>>> Book.objects.all()
<QuerySet [<Book: Harry Potter 1 (5)>]>
>>>
>>> harrypotter = Book.objects.get(title="Harry Potter 1")
>>> harrypotter
<Book: Harry Potter 1 (5)>
>>> harrypotter.author
<Author: Author object (1)>
>>> harrypotter.author.first_name
'J.K.'
>>> harrypotter.author.last_name
'Rowling'
>>> books_by_rowling = Book.objects.filter(author__last_name="Rowling")
>>> books_by_rowling
<QuerySet [<Book: Harry Potter 1 (5)>]>
>>> books_by_rowling = Book.objects.filter(author__last_name__contains="wling")
>>> books_by_rowling
<QuerySet [<Book: Harry Potter 1 (5)>]>
>>>
>>> jkr = Author.objects.get(first_name="J.K.")
>>> jkr
<Author: Author object (1)>
>>> jkr.book_set
<django.db.models.fields.related_descriptors.create_reverse_many_to_one_manager.<locals>.Re
latedManager object at 0x7fbc181f8ac0>
>>> jkr.book_set.all()
<QuerySet [<Book: Harry Potter 1 (5)>]>
>>> ^D
now exiting InteractiveConsole...

>>> from book_outlet.models import Book, Author
>>> jkr = Author.objects.get(first_name="J.K.")
>>> jkr.books.all()
<QuerySet [<Book: Harry Potter 1 (5)>]>
>>> jkr.books.get(title="Harry Potter 1")
<Book: Harry Potter 1 (5)>
>>> jkr.books.filter()

>>> jkr.books.filter(rating__gt=3)
<QuerySet [<Book: Harry Potter 1 (5)>]>
>>>
```

admin.site.register(Author)

---

one to one

```py
class Address(models.Model):
    street = models.CharField(max_length=80)
    postal_code = models.CharField(max_length=5)
    city = models.CharField(max_length=50)


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

makemigrations and migrate

```text
book_store $ python3 manage.py shell
Python 3.9.1 (v3.9.1:1e5d33e9b9, Dec 7 2020, 12:10:52)
[Clang 6.0 (clang-600.0.57)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
(InteractiveConsole)
>>> from book_outlet.models import Author, Address, Book
>>> Author.objects.all()
<QuerySet [<Author: J.K. Rowling>, <Author: Max Schwarzmüller>, <Author: J.R.R. Tolkien>]>
>>> Author.objects.all()[0].address
>>> addr1 = Address(street="Some Street", postal_code="12345", city="London")
>>> addr2 = Address(street="Another Street", postal_code="67890", city="New York")
>>> addr1.save()
>>> addr2.save()
>>> jkr = Author.objects.get(first_name="J.K.")
>>> jkr.address
>>> jkr.address = addr1
>>> jkr.save()
>>>
>>> new_author = Author(address=addr1)

>>> jkr.address
<Address: Address object (1)>
>>> jkr.address.street
'Some Street'
>>> Address.objects.all()
<QuerySet [<Address: Address object (1)>, <Address: Address object (2)>]>
>>> Address.objects.all()[0].author
<Author: J.K. Rowling>
>>> Address.objects.all()[0].author.first_name
'J.K.'
>>>
```

admin.site.register(Address)

```py

class Address(models.Model):
    street = models.CharField(max_length=80)
    postal_code = models.CharField(max_length=5)
    city = models.CharField(max_length=50)

    def __str__(self):
        return f"{self.street}, {self.postal_code}, {self.city}"

    class Meta:
        verbose_name_plural = "Address Entries"
```

admin.site.register(Country)

---

many to many

```py
class Country(models.Model):
    name = models.CharField(max_length=80)
    code = models.CharField(max_length=2)

    def __str__(self):
        return self.name

    class Meta:
        verbose_name_plural = "Countries"

class Book(models.Model):
    published_countries = models.ManyToManyField(Country, null=False)
```

```text
>>> from book_outlet.models import Country, Book
>>> Book.objects.all()
<QuerySet [<Book: Harry Potter 1 (5)>, <Book: My Story (3)>, <Book: Lord of the Rings (4)>]>
>>>
>>> hp1 = Book.objects.all()[0]
>>> hp1.published_countries
<django.db.models.fields.related_descriptors.create_forward_many_to_many_manager.<locals>.ManyRelatedManager object at 0x7f92f0490ac0>
>>> hp1.published_countries.all()
<QuerySet []>
>>>

>>> germany = Country(name="Germany", code="DE")
>>> Book.objects.all()
<QuerySet [<Book: Harry Potter 1 (5)>, <Book: My Story (3)>, <Book: Lord of the Rings (4)>]>
>>>
>>> mys = Book.objects.all()[1]
>>> mys.published_countries.add(germany)

>>> germany.save()
>>> mys.published_countries.add(germany)
>>> mys.published_countries.all()
<QuerySet [<Country: Country object (1)>]>
>>> mys.published_countries.filter(code="DE")
<QuerySet [<Country: Country object (1)>]>
>>> mys.published_countries.filter(code="UK")
<QuerySet []>
>>> Country.objects.all()
<QuerySet [<Country: Country object (1)>]>
>>>

>>> ger = Country.objects.all()[0]
>>> ger.book_set.all()
<QuerySet [<Book: My Story (3)>]>
>>> ger.books.all()
Traceback (most recent call last):
  File "<console>", line 1, in <module>
AttributeError: 'Country' object has no attribute 'books'
>>>
```

---

---

---

---

startapp reviews

reviews in installed app

```py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    include("", include("reviews.urls"))
]
```

templates/reviews

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Your Review</title>
  </head>
  <body>
    <form action="/" method="POST">
      {% csrf_token %}
      <label for="username">Your name</label>
      <input id="username" name="username" type="text" />
      <button type="submit">Send</button>
    </form>
  </body>
</html>
```

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Thank You!</title>
  </head>
  <body>
    <h1>Thank you!</h1>
  </body>
</html>
```

```py
from django.http import HttpResponseRedirect
from django.shortcuts import render

# Create your views here.

def review(request):
  if request.method == 'POST':
    entered_username = request.POST['username']
    print(entered_username)
    return HttpResponseRedirect("/thank-you")

  return render(request, "reviews/review.html")


def thank_you(request):
  return render(request, "reviews/thank_you.html")
```

```py
from django.urls import path

from . import views

urlpatterns = [
     path("", views.review),
     path("thank-you", views.thank_you)
]
```

```py
from django.http import HttpResponseRedirect
from django.shortcuts import render

# Create your views here.

def review(request):
  if request.method == 'POST':
    entered_username = request.POST['username']

    if entered_username == "" and len(entered_username) >= 100:
      return render(request, "reviews/review.html", {
        "has_error": True
      })
    print(entered_username)
    return HttpResponseRedirect("/thank-you")

  return render(request, "reviews/review.html", {
    "has_error": False
  })
```

```html
<pre>
<form action="/" method="POST">
    {% csrf_token %}
    {% if has_error %}
      <p>This form is invalid - please enter a valid username!</p>
    {% endif %}
    <label for="username">Your name</label>
    <input id="username" name="username" type="text">
    <button type="submit">Send</button>
</form>
</pre>
```

```py
from django import forms

class ReviewForm(forms.Form):
  user_name = forms.CharField()
```

```py
from .forms import ReviewForm

def review(request):
  if request.method == 'POST':
    form = ReviewForm(request.POST)

    if form.is_valid():
      print(form.cleaned_data)
      return HttpResponseRedirect("/thank-you")

  form = ReviewForm()

  return render(request, "reviews/review.html", {
    "form": form
  })
```

```html
<pre>
  <form action="/" method="POST">
    {% csrf_token %}
    {{ form }}
    <button type="submit">Send</button>
  </form>
</pre>
```

```py
from django import forms


class ReviewForm(forms.Form):
    user_name = forms.CharField(label="Your Name", max_length=100, error_messages={
        "required": "Your name must not be empty!",
        "max_length": "Please enter a shorter name!"
    })
```

```html
<pre>
  <form action="/" method="POST">
    {% csrf_token %}
    <div class="form-control {% if form.user_name.errors %}errors{% endif %}">
      {{ form.user_name.label_tag }}
      {{ form.user_name }}
      {{ form.user_name.errors }}
    </div>
    <button type="submit">Send</button>
  </form>
</pre>
```

static/reviews/styles.css

```css
body {
  font-family: sans-serif;
  margin: 0;
}

form {
  margin: 3rem auto;
  width: 90%;
  max-width: 30rem;
  padding: 1rem;
  border-radius: 12px;
  background-color: white;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.26);
}

button {
  cursor: pointer;
  padding: 0.5rem 1.5rem;
  border-radius: 6px;
  background-color: #1f0050;
  color: white;
  border: 1px solid #1f0050;
}

button:hover,
button:active {
  background-color: #320777;
  border-color: #320777;
}

.form-control {
  margin-bottom: 1rem;
}

.form-control input {
  font: inherit;
  padding: 0.25rem;
  border-radius: 4px;
  border: 1px solid #ccc;
  display: block;
  width: 10rem;
}

.form-control input:focus {
  border-color: #1f0050;
  outline: none;
  background-color: #f4eeff;
}

.form-control label {
  font-weight: bold;
  display: block;
  margin-bottom: 0.5rem;
}

.errorlist {
  list-style: none;
  margin: 0.5rem 0;
  padding: 0;
  color: #680000;
}

.errors input {
  border-color: #680000;
  background-color: #fde3e3;
}

.errors label {
  color: #680000;
}
```

```html
{% load static %}

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Your Review</title>
  <link rel="stylesheet" href="{% static "reviews/styles.css" %}">
</head>

</html>
```

custom error_class option in forms.py

```py
from django import forms


class ReviewForm(forms.Form):
    user_name = forms.CharField(label="Your Name", max_length=100, error_messages={
        "required": "Your name must not be empty!",
        "max_length": "Please enter a shorter name!"
    })
    review_text = forms.CharField(label="Your Feedback", widget=forms.Textarea, max_length=200)
    rating = forms.IntegerField(label="Your Rating", min_value=1, max_value=5)
```

```html
<pre>
<form action="/" method="POST">
  {% csrf_token %}
  <div class="form-control {% if form.user_name.errors %}errors{% endif %}">
    {{ form.user_name.label_tag }}
    {{ form.user_name }}
    {{ form.user_name.errors }}
  </div>
  <div class="form-control {% if form.review_text.errors %}errors{% endif %}">
    {{ form.review_text.label_tag }}
    {{ form.review_text }}
    {{ form.review_text.errors }}
  </div>
  <div class="form-control {% if form.rating.errors %}errors{% endif %}">
    {{ form.rating.label_tag }}
    {{ form.rating }}
    {{ form.rating.errors }}
  </div>
  <button type="submit">Send</button>
</form>
</pre>
```

```html
<pre>
  <form action="/" method="POST">
    {% csrf_token %}
    {% for field in form %}
      <div class="form-control {% if field.errors %}errors{% endif %}">
        {{ field.label_tag }}
        {{ field }}
        {{ field.errors }}
      </div>
    {% endfor %}
    <button type="submit">Send</button>
  </form>  
</pre>
```

```py
from django.db import models

# Create your models here.

class Review(models.Model):
  user_name = models.CharField(max_length=100)
  review_text = models.TextField()
  rating = models.IntegerField()
```

makemigrations and migrate

```py
from .models import Review

def review(request):
    if request.method == 'POST':
        form = ReviewForm(request.POST)

        if form.is_valid():
            review = Review(
                user_name=form.cleaned_data['user_name'],
                review_text=form.cleaned_data['review_text'],
                rating=form.cleaned_data['rating'])
            review.save()
            return HttpResponseRedirect("/thank-you")

    else:
        form = ReviewForm()

    return render(request, "reviews/review.html", {
        "form": form
    })
```

Test from localhost, then:

```text
>>> from reviews.models import Review
>>> Review.objects.all()
<QuerySet [<Review: Review object (1)>]>
>>> Review.objects.all()[0].user_name
'Bidur'
>>>
```

```py
from django import forms

from .models import Review

class ReviewForm(forms.ModelForm):
    class Meta:
      model = Review
      fields = '__all__'
      # exclude = ['owner_comment']
```

```py
class ReviewForm(forms.ModelForm):
    class Meta:
        model = Review
        fields = "__all__"
        # exclude = ['owner_comment']
        labels = {
            "user_name": "Your Name",
            "review_text": "Your Feedback",
            "rating": "Your Rating"
        }
        error_messages = {
            "user_name": {
              "required": "Your name must not be empty!",
              "max_length": "Please enter a shorter name!"
            }
        }
```

```py
def review(request):
    if request.method == 'POST':
        form = ReviewForm(request.POST)

        if form.is_valid():
            form.save()
            return HttpResponseRedirect("/thank-you")

    else:
        form = ReviewForm()

    return render(request, "reviews/review.html", {
        "form": form
    })
```

```py
def review(request):
    if request.method == 'POST':
        existing_data=Review.objects.get(pk=1)
        form = ReviewForm(request.POST, instance=existing_data)

        if form.is_valid():
            form.save()
            return HttpResponseRedirect("/thank-you")
```

```py
from django.views import View


class ReviewView(View):
    def get(self, request):
        form = ReviewForm()

        return render(request, "reviews/review.html", {
            "form": form
        })

    def post(self, request):
        form = ReviewForm(request.POST)

        if form.is_valid():
            form.save()
            return HttpResponseRedirect("/thank-you")

        return render(request, "reviews/review.html", {
            "form": form
        })
```

```py
from django.urls import path

from . import views

urlpatterns = [
     path("", views.ReviewView.as_view()),
     path("thank-you", views.thank_you)
]
```

---

---

---

---

```html
<pre>
{% load static %}

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>
    {% block title %}
    {% endblock %}
  </title>
  <link rel="stylesheet" href="{% static "reviews/styles.css" %}">
</head>
<body>
  {% block content %}
  {% endblock %}
</body>
</html>
</pre>
```

```html
<pre>
{% extends "reviews/base.html" %}

{% block title %}Your Review{% endblock %}

{% block content %}
  <form action="/" method="POST">
    {% csrf_token %}
    {% for field in form %}
      <div class="form-control {% if field.errors %}errors{% endif %}">
        {{ field.label_tag }}
        {{ field }}
        {{ field.errors }}
      </div>
    {% endfor %}
    <button type="submit">Send</button>
  </form>
{% endblock %}
</pre>
```

```html
<pre>
{% extends "reviews/base.html" %}

{% block title %}Thank you!{% endblock %}

{% block content %}
  <h1>Thank you!</h1>
{% endblock %}
</pre>
```

```py
class ThankYouView(View):
    def get(self, request):
        return render(request, "reviews/thank_you.html")
```

```py
from django.views.generic.base import TemplateView

class ThankYouView(TemplateView):
    template_name = "reviews/thank_you.html"
```

```py
from django.urls import path

from . import views

urlpatterns = [
     path("", views.ReviewView.as_view()),
     path("thank-you", views.ThankYouView.as_view())
]
```

```py
from django.views.generic.base import TemplateView

class ThankYouView(TemplateView):
    template_name = "reviews/thank_you.html"

    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context["message"] = "This works!"
        return context
```

```html
<pre>
{% extends "reviews/base.html" %}

{% block title %}Thank you!{% endblock %}

{% block content %}
  <h1>Thank you!</h1>
  <p>{{ message }}</p>
{% endblock %}
</pre>
```

```py
class ReviewsListView(TemplateView):
    template_name = "reviews/review_list.html"

    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        reviews = Review.objects.all()
        context["reviews"] = reviews
        return context
```

```html
<pre>
{% extends "reviews/base.html" %}

{% block title %}All Reviews{% endblock %}

{% block content %}
  <ul>
    {% for review in reviews %}
      <li>{{ review.user_name }} (Rating: {{ review.rating }})</li>
    {% endfor %}
  </ul>
{% endblock %}
</pre>
```

```py
path("reviews", views.ReviewsListView.as_view()),
```

```html
<pre>
{% extends "reviews/base.html" %}

{% block title %}Review Detail{% endblock %}

{% block content %}
  <h1>{{ review.user_name }}</h1>
  <p>Rating: {{ review.rating }}</p>
  <p>{{ review.review_text }}</p>
{% endblock %}
</pre>
```

```py
class SingleReviewView(TemplateView):
    template_name = "reviews/single_review.html"

    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        review_id = kwargs["id"]
        selected_review = Review.objects.get(pk=review_id)
        context["review"] = selected_review
        return context
```

```py
path("reviews/<int:id>", views.SingleReviewView.as_view())
```

```py
from django.views.generic import ListView

class ReviewsListView(ListView):
    template_name = "reviews/review_list.html"
    model = Review
```

```html
<pre>
<ul>
    {% for review in object_list %}
      <li>{{ review.user_name }} (Rating: {{ review.rating }})</li>
    {% endfor %}
  </ul>
</pre>
```

```py
class ReviewsListView(ListView):
    template_name = "reviews/review_list.html"
    model = Review
    context_object_name = "reviews"
```

```html
<pre>
<ul>
    {% for review in reviews %}
      <li>{{ review.user_name }} (Rating: {{ review.rating }})</li>
    {% endfor %}
  </ul>
</pre>
```

```py
class ReviewsListView(ListView):
    template_name = "reviews/review_list.html"
    model = Review
    context_object_name = "reviews"

    def get_queryset(self):
        base_query = super().get_queryset()
        data = base_query.filter(rating__gt=4)
        return data
```

```py
from django.views.generic import ListView, DetailView

class SingleReviewView(DetailView):
    template_name = "reviews/single_review.html"
    model = Review
```

```py
path("reviews/<int:pk>", views.SingleReviewView.as_view())
```

model name all lowercase or object in template context name

```py
from django.views.generic.edit import FormView

class ReviewView(FormView):
    form_class = ReviewForm
    template_name = "reviews/review.html"
    success_url = "/thank-you"

    def form_valid(self, form):
        form.save()
        return super().form_valid(form)
```

```py
from django.views.generic.edit import CreateView

class ReviewView(CreateView):
    model = Review
    # fields = '__all__'
    form_class = ReviewForm # either fields or form_class
    template_name = "reviews/review.html"
    success_url = "/thank-you"
```

also there are update view and delete view.

---

---

---

---

startapp profiles

add profiles to settings

```html
<pre>
{% load static %}

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Create a Profile</title>
  <link rel="stylesheet" href="{% static "profiles/styles.css" %}">
</head>
<body>
  <form action="/profiles/" method="POST" enctype="multipart/form-data">
    {% csrf_token %}
    <input type="file" name="image" />
    <button>Upload!</button>
  </form>
</body>
</html>
</pre>
```

```py
from django.shortcuts import render
from django.views import View
from django.http import HttpResponseRedirect

def store_file(file):
    with open("temp/image.jpg", "wb+") as dest:
        for chunk in file.chunks():
            dest.write(chunk)


class CreateProfileView(View):
    def get(self, request):
        return render(request, "profiles/create_profile.html")

    def post(self, request):
        # print(request.FILES['image'])
        store_file(request.FILES["image"])
        return HttpResponseRedirect("/profiles")
```

```py
from django.urls import path

from . import views

urlpatterns = [
    path("", views.CreateProfileView.as_view())
]
```

create folder temp inside project folder (outside app folder)

---

```py
from django import forms

class ProfileForm(forms.Form):
    user_image = forms.FileField()
```

```html
<pre>
{% load static %}

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Create a Profile</title>
  <link rel="stylesheet" href="{% static "profiles/styles.css" %}">
</head>
<body>
  <form action="/profiles/" method="POST" enctype="multipart/form-data">
    {% csrf_token %}
    {{ form }}
    <button>Upload!</button>
  </form>
</body>
</html>
</pre>
```

```py
from django.shortcuts import render
from django.views import View
from django.http import HttpResponseRedirect

from .forms import ProfileForm

def store_file(file):
    with open("temp/image.jpg", "wb+") as dest:
        for chunk in file.chunks():
            dest.write(chunk)


class CreateProfileView(View):
    def get(self, request):
        form = ProfileForm()
        return render(request, "profiles/create_profile.html", {
            "form": form
        })

    def post(self, request):
        submitted_form = ProfileForm(request.POST, request.FILES)

        if submitted_form.is_valid():
            store_file(request.FILES["image"]) # user_image ??
            return HttpResponseRedirect("/profiles")

        return render(request, "profiles/create_profile.html", {
            "form": submitted_form
        })
```

---

```py
from django.db import models

class UserProfile(models.Model):
    image = models.FileField(upload_to="images")
```

makemigrations and migrate

```py
MEDIA_ROOT = BASE_DIR / "uploads"
```

```py
from django.shortcuts import render
from django.views import View
from django.http import HttpResponseRedirect

from .forms import ProfileForm
from .models import UserProfile

# Create your views here

class CreateProfileView(View):
    def get(self, request):
        form = ProfileForm()
        return render(request, "profiles/create_profile.html", {
            "form": form
        })

    def post(self, request):
        submitted_form = ProfileForm(request.POST, request.FILES)

        if submitted_form.is_valid():
            profile = UserProfile(image=request.FILES["user_image"])
            profile.save()
            return HttpResponseRedirect("/profiles")

        return render(request, "profiles/create_profile.html", {
            "form": submitted_form
        })
```

```bash
>>> from profiles.models import UserProfile
>>> UserProfile.objects.all()
<QuerySet [<UserProfile: UserProfile object (1)>]>
>>> UserProfile.objects.all()[0].user_image
Traceback (most recent call last):
  File "<console>", line 1, in <module>
AttributeError: 'UserProfile' object has no attribute 'user_image'
>>> >>> UserProfile.objects.all()[0].image
<FieldFile: images/me_kthmaSa.jpg>
>>> UserProfile.objects.all()[0].image.path
'...django-project/uploads/images/me_kthmaSa.jpg'
>>> UserProfile.objects.all()[0].image.size
125067
>>>
```

---

```py
from django.db import models

class UserProfile(models.Model):
    image = models.ImageField(upload_to="images")
```

```bash
pip install Pillow
```

```py
from django import forms

class ProfileForm(forms.Form):
    user_image = forms.ImageField()
```

try locahost

---

```py
from django.shortcuts import render
from django.views import View
from django.http import HttpResponseRedirect
from django.views.generic.edit import CreateView

from .forms import ProfileForm
from .models import UserProfile

# Create your views here

class CreateProfileView(CreateView):
    template_name = "profiles/create_profile.html"
    model = UserProfile
    fields = "__all__"
    success_url = "/profiles"
```

replace veiw, try localhost

---

```html
<pre>
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>User Profiles</title>
</head>
<body>
  <ul>
    {% for profile in profiles %}
      <li>
        <img src="{{ profile.image.url }}">
      </li>
    {% endfor %}
  </ul>
</body>
</html>
</pre>
```

```py
from django.shortcuts import render
from django.views import View
from django.http import HttpResponseRedirect
from django.views.generic.edit import CreateView
from django.views.generic import ListView

from .forms import ProfileForm
from .models import UserProfile

# Create your views here

class CreateProfileView(CreateView):
    template_name = "profiles/create_profile.html"
    model = UserProfile
    fields = "__all__"
    success_url = "/profiles"


class ProfilesView(ListView):
    model = UserProfile
    template_name = "profiles/user_profiles.html"
    context_object_name = "profiles"
```

```py
from django.urls import path

from . import views

urlpatterns = [
    path("", views.CreateProfileView.as_view()),
    path("list", views.ProfilesView.as_view())
]
```

```py
MEDIA_ROOT = BASE_DIR / "uploads"
MEDIA_URL = "/user-media/"
```

```py
from django.contrib import admin
from django.urls import path, include
from django.conf.urls.static import static
from django.conf import settings

urlpatterns = [
    path('admin/', admin.site.urls),
    path("", include("reviews.urls")),
    path("profiles/", include("profiles.urls"))
] + static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

---

---

---

---

```html
<pre>
{% extends "reviews/base.html" %}

{% block title %}Review Detail{% endblock %}

{% block content %}
  <h1>{{ review.user_name }}</h1>
  <p>Rating: {{ review.rating }}</p>
  <p>{{ review.review_text }}</p>
  <form action="/reviews/favorite" method="POST">
    {% csrf_token %}
    <input type="hidden" name="review_id" value="{{ review.id }}">
    <button>Favorite</button>
  </form>
{% endblock %}
</pre>
```

```py
class AddFavoriteView(View):
    def post(self, request):
        review_id = request.POST["review_id"]
        request.session["favorite_review"] = review_id
        return HttpResponseRedirect("/reviews/" + review_id)
```

```py
from django.urls import path

from . import views

urlpatterns = [
     path("", views.ReviewView.as_view()),
     path("thank-you", views.ThankYouView.as_view()),
     path("reviews", views.ReviewsListView.as_view()),
     path("reviews/favorite", views.AddFavoriteView.as_view()),
     path("reviews/<int:pk>", views.SingleReviewView.as_view())
]
```

```py
class SingleReviewView(DetailView):
    template_name = "reviews/single_review.html"
    model = Review

    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        loaded_review = self.object
        request = self.request
        favorite_id = request.session.get("favorite_review")
        context["is_favorite"] = favorite_id == str(loaded_review.id) # why str ??
        return context
```

---

---

---

---

start project my_site
start app blog

add blog in settings

```py
from django.shortcuts import render

# Create your views here.

def starting_page(request):
  pass

def posts(request):
  pass

def post_detail(request):
  pass
```

```py
from django.urls import path

from . import views

urlpatterns = [
    path("", views.starting_page, name="starting-page"),
    path("posts", views.posts, name="posts-page"),
    path("posts/<slug:slug>", views.post_detail,
         name="post-detail-page")  # /posts/my-first-post
]
```

```py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path("", include("blog.urls"))
]
```

```html
<pre>
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>{% block title %}{% endblock %}</title>
  {% block css_files %}{% endblock %}
</head>
<body>
  {% block content %}
  {% endblock %}
</body>
</html>
</pre>
```

```html
<pre>
{% extends "base.html" %}

{% block title %}
  My Blog
{% endblock %}

{% block content %}
  <h1>Welcome to my blog</h1>
{% endblock %}
</pre>
```

```css
@import url("https://fonts.googleapis.com/css2?family=Lato:wght@400;700&family=Open+Sans:wght@400;700&display=swap");

* {
  box-sizing: border-box;
}

html {
  font-family: "Open Sans", "Lato", sans-serif;
}

body {
  margin: 0;
}

h1,
h2,
h3 {
  font-family: "Lato", sans-serif;
  font-weight: bold;
}
```

```py
STATICFILES_DIRS = [
    BASE_DIR / "static"
]
```

```css
#main-navigation {
  width: 100%;
  height: 5rem;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0 10%;
  position: absolute;
  top: 0;
  left: 0;
}

#main-navigation a {
  text-decoration: none;
  color: white;
  font-weight: bold;
}

#main-navigation a:hover,
#main-navigation a:active {
  color: #cf79f1;
}

#main-navigation h1 a:hover,
#main-navigation h1 a:active {
  color: white;
}

#welcome {
  background: linear-gradient(to right top, #6305dd, #390281);
  padding: 6rem 12%;
}

#welcome header {
  display: flex;
  align-items: flex-start;
  margin: 3rem auto;
}

#welcome img {
  width: 10rem;
  height: 10rem;
  object-fit: cover;
  object-position: top;
  border: 5px solid white;
  border-radius: 12px;
  background-color: #ffe1bf;
  transform: rotateZ(-5deg);
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.25);
}

#welcome h2 {
  font-size: 3.5rem;
  margin: 0 0 0 2rem;
  color: #e4e4e4;
  width: 10rem;
}

#welcome p {
  color: white;
  font-size: 1.5rem;
}

#latest-posts {
  background-color: white;
  padding: 2rem;
  border-radius: 12px;
  width: 60rem;
  margin: -6rem auto 2rem auto;
  box-shadow: 1px 1px 12px rgba(0, 0, 0, 0.4);
}

#latest-posts h2 {
  text-align: center;
}

#latest-posts ul {
  list-style: none;
  margin: 0;
  padding: 0;
  display: flex;
  gap: 1rem;
}

#latest-posts li {
  flex: 1;
}

#about {
  text-align: center;
  padding: 3rem;
  background-color: #e48900;
  margin-top: 5rem;
}

#about h2 {
  font-size: 3rem;
}

#about p {
  font-size: 1.5rem;
  color: #353535;
}

.post {
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: center;
}

.post a {
  text-decoration: none;
  color: black;
  transition: all 0.2s ease;
  padding: 1rem;
}

.post a:hover,
.post a:active {
  transform: scale(1.1);
  background-color: #390281;
  color: white;
}

.post img {
  width: 5rem;
  height: 5rem;
  object-fit: cover;
  border-radius: 50%;
}

.post h3 {
  margin: 0.25rem 0;
}

.post time {
  color: #666666;
  margin: 0.25rem;
  font-style: italic;
  font-size: 0.85rem;
}

.post:hover time,
.post:active time {
  color: white;
}
```

```html
<pre>
{% load static %}

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>{% block title %}{% endblock %}</title>
  <link rel="stylesheet" href="{% static "app.css" %}">
  {% block css_files %}{% endblock %}
</head>
<body>
  {% block content %}
  {% endblock %}
</body>
</html>
</pre>
```

```html
<pre>
{% extends "base.html" %}
{% load static %}

{% block title %}
My Blog
{% endblock %}

{% block css_files %}
  <link rel="stylesheet" href="{% static "blog/index.css" %}" />
{% endblock %}

{% block content %}
<header id="main-navigation">
  <h1><a href="">Bidur' Blog</a></h1>
  <nav>
    <a href="">All Posts</a>
  </nav>
</header>

<section id="welcome">
  <header>
    <img src="{% static "blog/images/bidur.png" %}" alt="Bidur - The Author Of This Blog" />
    <h2>BIDUR'S BLOG</h2>
  </header>
  <p>Hi, I am Bidur and I love to blog about tech and the world!</p>
</section>

<section id="latest-posts">
  <h2>My Latest Thoughts</h2>

  <ul>
    <li>
      <article class="post">
        <a href="">
          <img src="{% static "blog/images/mountains.jpg" %}" alt="Mountain Hiking" />
          <div class="post__content">
            <h3>Mountain Hiking</h3>
            <p>
              There's nothing like the views you get when hiking in the
              mountains! And I wasn't even prepared for what happened whilst I
              was enjoying the view!
            </p>
          </div>
        </a>
      </article>
    </li>
  </ul>
</section>

<section id="about">
  <h2>What I Do</h2>
  <p>
    I love programming, I love to help others and I enjoy exploring new
    technologies in general!
  </p>
  <p>
    My goal is to keep on growing as a developer - and if I could help you do
    the same, I'd be very happy!
  </p>
</section>
{% endblock %}
</pre>
```

```html
<pre>
{% load static %}

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>{% block title %}{% endblock %}</title>
  <link rel="stylesheet" href="{% static "app.css" %}">
  {% block css_files %}{% endblock %}
</head>
<body>
  <header id="main-navigation">
    <h1><a href="{% url "starting-page" %}">Bidur' Blog</a></h1>
    <nav>
      <a href="{% url "posts-page" %}">All Posts</a>
    </nav>
  </header>

  {% block content %}
  {% endblock %}
</body>
</html>
</pre>
```

```html
<pre>
{% extends "base.html" %}
{% load static %}

{% block title %}
My Blog
{% endblock %}

{% block css_files %}
  <link rel="stylesheet" href="{% static "blog/post.css" %}" />
  <link rel="stylesheet" href="{% static "blog/index.css" %}" />
{% endblock %}

{% block content %}
<!-- ... -->
<section id="latest-posts">
  <h2>My Latest Thoughts</h2>

  <ul>
    {% include "blog/includes/post.html" %}
  </ul>
</section>
<!-- ... -->
{% endblock %}
</pre>
```

```html
<pre>
{% extends "base.html" %}
{% load static %}

{% block title %}
All My Posts
{% endblock %}

{% block css_files %}
  <link rel="stylesheet" href="{% static "blog/post.css" %}" />
  <link rel="stylesheet" href="{% static "blog/all-posts.css" %}" />
{% endblock %}

{% block content %}
<section id="all-posts">
  <h2>My Collected Posts</h2>

  <ul>
    {% include "blog/includes/post.html" %}
    {% include "blog/includes/post.html" %}
    {% include "blog/includes/post.html" %}
    {% include "blog/includes/post.html" %}
    {% include "blog/includes/post.html" %}
    {% include "blog/includes/post.html" %}
  </ul>
</section>
{% endblock %}
</pre>
```

```html
<pre>
{% load static %}

<li>
  <article class="post">
    <a href="{% url "post-detail-page" "the-mountains" %}">
      <img src="{% static "blog/images/mountains.jpg" %}" alt="Mountain Hiking"
      />
      <div class="post__content">
        <h3>Mountain Hiking</h3>
        <p>
          There's nothing like the views you get when hiking in the mountains!
          And I wasn't even prepared for what happened whilst I was enjoying the
          view!
        </p>
      </div>
    </a>
  </article>
</li>
</pre>
```

```css
/* cut from index.css */
.post {
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: center;
}

.post a {
  text-decoration: none;
  color: black;
  transition: all 0.2s ease;
  padding: 1rem;
}

.post a:hover,
.post a:active {
  transform: scale(1.1);
  background-color: #390281;
  color: white;
}

.post img {
  width: 5rem;
  height: 5rem;
  object-fit: cover;
  border-radius: 50%;
}

.post h3 {
  margin: 0.25rem 0;
}

.post time {
  color: #666666;
  margin: 0.25rem;
  font-style: italic;
  font-size: 0.85rem;
}

.post:hover time,
.post:active time {
  color: white;
}
```

```css
body {
  background-color: #e7e7e7;
}

#main-navigation {
  background-color: #390281;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.25);
}

#all-posts {
  margin: 7rem auto;
  width: 90%;
  max-width: 60rem;
}

#all-posts h2 {
  text-align: center;
  font-size: 2rem;
  color: #2e2e2e;
  margin: 3rem 0;
}

ul {
  list-style: none;
  margin: 0;
  padding: 0;
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(17rem, 1fr));
  gap: 1.5rem;
}

.post img {
  width: 7rem;
  height: 7rem;
}

.post a {
  height: 22rem;
  transform-origin: center;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.25);
  border-radius: 12px;
  background-color: white;
}
```

```py
from django.shortcuts import render

# Create your views here.

def starting_page(request):
  return render(request, "blog/index.html")

def posts(request):
  return render(request, "blog/all-posts.html")

def post_detail(request, slug):
  return render(request, "blog/post-detail.html")
```

```html
<pre>
{% extends "base.html" %}
{% load static %}

{% block title %}
This Post Title
{% endblock %}

{% block css_files %}
<link rel="stylesheet" href="{% static "blog/post-detail.css" %}" />
{% endblock %}

{% block content %}
<section id="summary">
  <h2>Post Title</h2>
  <article>
    <img src="{% static "blog/images/mountains.jpg" %}" alt="Post Title" />
    <address>By Bidur</address>
    <div>Last updated on <time>July 10th</time></div>
  </article>
</section>

<main>
  <p>
    Lorem ipsum dolor sit amet consectetur adipisicing elit. Officiis nobis
    aperiam est praesentium, quos iste consequuntur omnis exercitationem quam
    velit labore vero culpa ad mollitia? Quis architecto ipsam nemo. Odio.
  </p>
  <p>
    Lorem ipsum dolor sit amet consectetur adipisicing elit. Officiis nobis
    aperiam est praesentium, quos iste consequuntur omnis exercitationem quam
    velit labore vero culpa ad mollitia? Quis architecto ipsam nemo. Odio.
  </p>
  <p>
    Lorem ipsum dolor sit amet consectetur adipisicing elit. Officiis nobis
    aperiam est praesentium, quos iste consequuntur omnis exercitationem quam
    velit labore vero culpa ad mollitia? Quis architecto ipsam nemo. Odio.
  </p>
</main>
{% endblock %}
</pre>
```

```css
body {
  background-color: #e7e7e7;
}

#main-navigation {
  background-color: #390281;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.25);
}

#summary {
  margin: 8rem auto 3rem auto;
  padding: 2rem;
  width: 90%;
  max-width: 90rem;
  border-radius: 16px;
  background-color: #5706c0;
  position: relative;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.25);
  height: 22rem;
}

#summary h2 {
  color: white;
  font-size: 4.5rem;
}

#summary article {
  position: absolute;
  top: 2rem;
  right: 3rem;
  padding: 1rem 2rem;
  border-radius: 12px;
  text-align: right;
  color: white;
}

#summary article img {
  width: 12rem;
  height: 12rem;
  border-radius: 12px;
  transform: rotateZ(5deg);
  border: 5px solid white;
  margin-bottom: 1rem;
}

#summary address {
  font-weight: bold;
}

#summary time {
  font-weight: bold;
}

main {
  width: 90%;
  max-width: 90rem;
  margin: 3rem auto;
  background-color: white;
  padding: 1rem;
  border-radius: 12px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.25);
}
```

```py
from datetime import date

from django.shortcuts import render

all_posts = [
    {
        "slug": "hike-in-the-mountains",
        "image": "mountains.jpg",
        "author": "Bidur",
        "date": date(2021, 7, 21),
        "title": "Mountain Hiking",
        "excerpt": "There's nothing like the views you get when hiking in the mountains! And I wasn't even prepared for what happened whilst I was enjoying the view!",
        "content": """
          Lorem ipsum dolor sit amet consectetur adipisicing elit. Officiis nobis
          aperiam est praesentium, quos iste consequuntur omnis exercitationem quam
          velit labore vero culpa ad mollitia? Quis architecto ipsam nemo. Odio.

          Lorem ipsum dolor sit amet consectetur adipisicing elit. Officiis nobis
          aperiam est praesentium, quos iste consequuntur omnis exercitationem quam
          velit labore vero culpa ad mollitia? Quis architecto ipsam nemo. Odio.

          Lorem ipsum dolor sit amet consectetur adipisicing elit. Officiis nobis
          aperiam est praesentium, quos iste consequuntur omnis exercitationem quam
          velit labore vero culpa ad mollitia? Quis architecto ipsam nemo. Odio.
        """
    },
    {
        "slug": "programming-is-fun",
        "image": "coding.jpg",
        "author": "Bidur",
        "date": date(2022, 3, 10),
        "title": "Programming Is Great!",
        "excerpt": "Did you ever spend hours searching that one error in your code? Yep - that's what happened to me yesterday...",
        "content": """
          Lorem ipsum dolor sit amet consectetur adipisicing elit. Officiis nobis
          aperiam est praesentium, quos iste consequuntur omnis exercitationem quam
          velit labore vero culpa ad mollitia? Quis architecto ipsam nemo. Odio.

          Lorem ipsum dolor sit amet consectetur adipisicing elit. Officiis nobis
          aperiam est praesentium, quos iste consequuntur omnis exercitationem quam
          velit labore vero culpa ad mollitia? Quis architecto ipsam nemo. Odio.

          Lorem ipsum dolor sit amet consectetur adipisicing elit. Officiis nobis
          aperiam est praesentium, quos iste consequuntur omnis exercitationem quam
          velit labore vero culpa ad mollitia? Quis architecto ipsam nemo. Odio.
        """
    },
    {
        "slug": "into-the-woods",
        "image": "woods.jpg",
        "author": "Bidur",
        "date": date(2020, 8, 5),
        "title": "Nature At Its Best",
        "excerpt": "Nature is amazing! The amount of inspiration I get when walking in nature is incredible!",
        "content": """
          Lorem ipsum dolor sit amet consectetur adipisicing elit. Officiis nobis
          aperiam est praesentium, quos iste consequuntur omnis exercitationem quam
          velit labore vero culpa ad mollitia? Quis architecto ipsam nemo. Odio.

          Lorem ipsum dolor sit amet consectetur adipisicing elit. Officiis nobis
          aperiam est praesentium, quos iste consequuntur omnis exercitationem quam
          velit labore vero culpa ad mollitia? Quis architecto ipsam nemo. Odio.

          Lorem ipsum dolor sit amet consectetur adipisicing elit. Officiis nobis
          aperiam est praesentium, quos iste consequuntur omnis exercitationem quam
          velit labore vero culpa ad mollitia? Quis architecto ipsam nemo. Odio.
        """
    }
]

def get_date(post):
  return post['date']

def starting_page(request):
    sorted_posts = sorted(all_posts, key=get_date)
    latest_posts = sorted_posts[-3:]
    return render(request, "blog/index.html", {
      "posts": latest_posts
    })
```

```html
<pre>
<section id="latest-posts">
  <h2>My Latest Thoughts</h2>

  <ul>
    {% for post in posts %}
      {% include "blog/includes/post.html" %}
    {% endfor %}
  </ul>
</section>
</pre>
```

```html
<pre>
{% load static %}

<li>
  <article class="post">
    <a href="{% url "post-detail-page" post.slug %}">
      <img src="{% static "blog/images/"|add:post.image %}" alt="{{ post.title }}"
      />
      <div class="post__content">
        <h3>{{ post.title }}</h3>
        <p>
          {{ post.excerpt }}
        </p>
      </div>
    </a>
  </article>
</li>
</pre>
```

```py
def posts(request):
    return render(request, "blog/all-posts.html", {
      "all_posts": all_posts
    })


def post_detail(request, slug):
    identified_post = next(post for post in all_posts if post['slug'] == slug)
    return render(request, "blog/post-detail.html", {
      "post": identified_post
    })
```

```html
<pre>
<section id="all-posts">
  <h2>My Collected Posts</h2>
  <ul>
    {% for post in all_posts %}
      {% include "blog/includes/post.html" %}
    {% endfor %}
  </ul>
</section>
</pre>
```

```html
<pre>
{% extends "base.html" %}
{% load static %}

{% block title %}
{{ post.title }}
{% endblock %}

{% block css_files %}
<link rel="stylesheet" href="{% static "blog/post-detail.css" %}" />
{% endblock %}

{% block content %}
<section id="summary">
  <h2>{{ post.title }}</h2>
  <article>
    <img src="{% static "blog/images/"|add:post.image %}" alt="{{ post.title }}" />
    <address>By {{ post.author }}</address>
    <div>Last updated on <time>{{ post.date|date:"d M Y" }}</time></div>
  </article>
</section>

<main>
  {{ post.content|linebreaks }}
</main>
{% endblock %}
</pre>
```

```html
<pre>
{% extends "base.html" %}

{% block title %}
We didn't find that page!
{% endblock %}

{% block content %}
  <h2>We're sorry!</h2>
  <p>But we couldn't find that page!</p>
{% endblock %}
</pre>
```

---

---

---

---

```py
from django.db import models
from django.core.validators import MinLengthValidator

# Create your models here.


class Tag(models.Model):
    caption = models.CharField(max_length=20)

    def __str__(self):
      return self.caption


class Author(models.Model):
    first_name = models.CharField(max_length=100)
    last_name = models.CharField(max_length=100)
    email_address = models.EmailField()

    def full_name(self):
        return f"{self.first_name} {self.last_name}"

    def __str__(self):
        return self.full_name()


class Post(models.Model):
    title = models.CharField(max_length=150)
    excerpt = models.CharField(max_length=200)
    image = models.ImageField(upload_to="posts", null=True)
    date = models.DateField(auto_now=True)
    slug = models.SlugField(unique=True, db_index=True)
    content = models.TextField(validators=[MinLengthValidator(10)])
    author = models.ForeignKey(
        Author, on_delete=models.SET_NULL, null=True, related_name="posts")
    tags = models.ManyToManyField(Tag)

    def __str__(self):
        return self.title
```

```py
from django.contrib import admin

from .models import Post, Author, Tag

# Register your models here


class PostAdmin(admin.ModelAdmin):
    list_filter = ("author", "tags", "date",)
    list_display = ("title", "date", "author",)
    prepopulated_fields = {"slug": ("title",)}


admin.site.register(Post, PostAdmin)
admin.site.register(Author)
admin.site.register(Tag)
```

makemigrations and migrate

createsuperuser

add array data from admin panel

```py
from django.shortcuts import render, get_object_or_404

from .models import Post

# Create your views here.


def starting_page(request):
    latest_posts = Post.objects.all().order_by("-date")[:3]
    return render(request, "blog/index.html", {
      "posts": latest_posts
    })


def posts(request):
    all_posts = Post.objects.all().order_by("-date")
    return render(request, "blog/all-posts.html", {
      "all_posts": all_posts
    })


def post_detail(request, slug):
    identified_post = get_object_or_404(Post, slug=slug)
    return render(request, "blog/post-detail.html", {
      "post": identified_post,
      "post_tags": identified_post.tags.all()
    })
```

```html
<!-- post detail -->
<address>
  By <a href="mailto:{{ post.author.email_address }}">{{ post.author }}</a>
</address>
```

```css
#summary h2 {
  /* ... */
  margin-bottom: 0.25rem;
}
#summary a {
  color: white;
  text-decoration: none;
}
.tag {
  background-color: white;
  color: #390281;
  padding: 0.5rem 1rem;
  border-radius: 8px;
}
```

---

---

---

---

install pillow

```py
# post model
image = models.ImageField(upload_to="posts", null=True)
```

edit from admin panel and upload each posts image

```py
MEDIA_ROOT = BASE_DIR / "uploads"
MEDIA_URL = "/files/"
```

```html
<!-- post includes and in post detail -->
<img src="{{ post.image.url }}" alt="{{ post.title }}" />
```

```py
urlpatterns = [
    path('admin/', admin.site.urls),
    path("", include("blog.urls"))
] + static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

---

```py
from django.shortcuts import render, get_object_or_404
from django.views.generic import ListView, DetailView

from .models import Post

# Create your views here.

class StartingPageView(ListView):
    template_name = "blog/index.html"
    model = Post
    ordering = ["-date"]
    context_object_name = "posts"

    def get_queryset(self):
        queryset = super().get_queryset()
        data = queryset[:3]
        return data


class AllPostsView(ListView):
    template_name = "blog/all-posts.html"
    model = Post
    ordering = ["-date"]
    context_object_name = "all_posts"


class SinglePostView(DetailView):
    template_name = "blog/post-detail.html"
    model = Post

    def get_context_data(self, **kwargs):
        context =  super().get_context_data(**kwargs)
        context["post_tags"] = self.object.tags.all()
        return context
```

```py
from django.urls import path

from . import views

urlpatterns = [
    path("", views.StartingPageView.as_view(), name="starting-page"),
    path("posts", views.AllPostsView.as_view(), name="posts-page"),
    path("posts/<slug:slug>", views.SinglePostView.as_view(),
         name="post-detail-page")  # /posts/my-first-post
]
```

```py
class Comment(models.Model):
    user_name = models.CharField(max_length=120)
    user_email = models.EmailField()
    text = models.TextField(max_length=400)
    post = models.ForeignKey(
        Post, on_delete=models.CASCADE, related_name="comments")
```

```py
from django import forms

from .models import Comment

class CommentForm(forms.ModelForm):
    class Meta:
        model = Comment
        exclude = ["post"]
        labels = {
          "user_name": "Your Name",
          "user_email": "Your Email",
          "text": "Your Comment"
        }
```

```py
class SinglePostView(DetailView):
    template_name = "blog/post-detail.html"
    model = Post

    def get_context_data(self, **kwargs):
        context =  super().get_context_data(**kwargs)
        context["post_tags"] = self.object.tags.all()
        context["comment_form"] = CommentForm()
        return context
```

```html
<!-- detail page -->
<section>
  <form>
    {{ comment_form }}
    <button>Save Comment</button>
  </form>
</section>
```

```html
<section id="comment-form">
  <h2>Your Comment</h2>
  <form action="{% url "post-detail-page" post.slug %}" method="POST">
    {% csrf_token %}
    {% for form_field in comment_form %}
      <div class="form-control">
        {{ form_field.label_tag }}
        {{ form_field }}
        {{ form_field.errors }}
      </div>
    {% endfor %}
    <button>Save Comment</button>
  </form>
</section>
```

```css
#comment-form {
  margin: 3rem auto;
  width: 90%;
  max-width: 40rem;
  border-radius: 12px;
  background-color: white;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.25);
  padding: 1rem;
}

#comment-form button {
  font: inherit;
  background-color: #390281;
  color: white;
  border: 1px solid #390281;
  padding: 0.5rem 1.5rem;
  border-radius: 6px;
  cursor: pointer;
}

#comment-form button:hover,
#comment-form button:active {
  background-color: #4f0ba7;
  border-color: #4f0ba7;
}

.form-control {
  margin-bottom: 1rem;
}

.form-control label {
  font-weight: bold;
  margin-bottom: 0.5rem;
  display: block;
}

.form-control input,
.form-control textarea {
  display: block;
  width: 100%;
  font: inherit;
  padding: 0.25rem;
  border-radius: 6px;
  border: 1px solid #ccc;
}
```

```py
# necessary new imports

class SinglePostView(View):
    def get(self, request, slug):
        post = Post.objects.get(slug=slug)
        context = {
          "post": post,
          "post_tags": post.tags.all(),
          "comment_form": CommentForm()
        }
        return render(request, "blog/post-detail.html", context)

    def post(self, request, slug):
        comment_form = CommentForm(request.POST)
        post = Post.objects.get(slug=slug)

        if comment_form.is_valid():
          comment = comment_form.save(commit=False)
          comment.post = post
          comment.save()

          return HttpResponseRedirect(reverse("post-detail-page", args=[slug]))

        context = {
          "post": post,
          "post_tags": post.tags.all(),
          "comment_form": comment_form
        }
        return render(request, "blog/post-detail.html", context)
```

```text
>>> from blog.models import Comment
>>> Comment.objects.all()
<QuerySet [<Comment: Comment object (1)>]>
>>> Comment.objects.all()[0].user_name
'Max'
>>> Comment.objects.all()[0].text
'This is awesome!'
>>> Comment.objects.all()[0].post
<Post: Post object (1)>
>>> Comment.objects.all()[0].post.title
'Mountain Hiking'
>>> ^D
```

```html
<pre>
{% block content %}

{% if comment_form.errors %}
  <div id="alert">
    <h2>Saving the comment failed!</h2>
    <p>Please check the comment form below the post and fix your erros.</p>
    <a href="#comment-form">Fix!</a>
  </div>
{% endif %}

<!-- ... summary, main -->

<section id="comment-form">
  <h2>Your Comment</h2>
  <form action="{% url "post-detail-page" post.slug %}" method="POST">
    {% csrf_token %}
    {% for form_field in comment_form %}
      <div class="form-control {% if form_field.errors %}invalid{% endif %}">
        {{ form_field.label_tag }}
        {{ form_field }}
        {{ form_field.errors }}
      </div>
    {% endfor %}
    <button>Save Comment</button>
  </form>
</section>
{% endblock %}
</pre>
```

```css
.errorlist {
  list-style: none;
  margin: 0.5rem 0;
  padding: 0;
  color: #d6000b;
}

.invalid label {
  color: #d6000b;
}

.invalid input,
.invalid textarea {
  border-color: #d6000b;
  background-color: #ffe6e7;
}

#alert {
  margin: 8rem auto 3rem auto;
  border: 1px solid #d6000b;
  background-color: #ffe6e7;
  padding: 1rem;
  width: 90%;
  max-width: 40rem;
}

#alert a {
  text-decoration: none;
  border: 1px solid #d6000b;
  background-color: #d6000b;
  color: white;
  padding: 0.25rem 1.5rem;
  border-radius: 6px;
}
```

```py
class SinglePostView(View):
    def get(self, request, slug):
        post = Post.objects.get(slug=slug)
        context = {
          "post": post,
          "post_tags": post.tags.all(),
          "comment_form": CommentForm(),
          "comments": post.comments.all().order_by("-id")
        }
        return render(request, "blog/post-detail.html", context)

    def post(self, request, slug):
        comment_form = CommentForm(request.POST)
        post = Post.objects.get(slug=slug)

        if comment_form.is_valid():
          comment = comment_form.save(commit=False)
          comment.post = post
          comment.save()

          return HttpResponseRedirect(reverse("post-detail-page", args=[slug]))

        context = {
          "post": post,
          "post_tags": post.tags.all(),
          "comment_form": comment_form,
          "comments": post.comments.all().order_by("-id")
        }
        return render(request, "blog/post-detail.html", context)
```

```html
<pre>
<section id="comments">
  <ul>
    {% for comment in comments %}
      <li>
        <h2>{{ comment.user_name }}</h2>
        <p>{{ comment.text|linebreaks }}</p>
      </li>
    {% endfor %}
  </ul>
</section>
<!-- comment form section -->
</pre>
```

```css
#comments {
  margin: 3rem auto;
  width: 90%;
  max-width: 60rem;
  border-radius: 12px;
  background-color: white;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.25);
  padding: 1rem 2rem;
}

#comments ul {
  list-style: none;
  margin: 0;
  padding: 0;
}

#comments li {
  margin-bottom: 1rem;
  border-bottom: 2px solid #ccc;
}

#comments li:last-of-type {
  border-bottom: none;
}

#comments h2 {
  color: #464646;
}
```

```py
class CommentAdmin(admin.ModelAdmin):
    list_display = ("user_name", "post")

admin.site.register(Comment, CommentAdmin)
```

```py
class ReadLaterView(View):
    def post(self, request):
        stored_posts = request.session.get("stored_posts")

        if stored_posts is None:
          stored_posts = []

        post_id = int(request.POST["post_id"])

        if post_id not in stored_posts:
          stored_posts.append(post_id)
        else: # can do later
          stored_posts.remove(post_id)

        request.session["stored_posts"] = stored_posts

        return HttpResponseRedirect("/")
```

```py
path("read-later", views.ReadLaterView.as_view(), name="read-later")
```

```html
<!-- in detail page inside summary section -->
<pre>
  <div id="read-later">
    <form action="{% url "read-later" %}" method="POST">
      {% csrf_token %}
      <input type="hidden" value="{{ post.id }}" name="post_id">
      <button>
          Read Later
      </button>
    </form>
  </div>
  <!-- article -->
</pre>
```

```py
class SinglePostView(View):
    def is_stored_post(self, request, post_id):
        stored_posts = request.session.get("stored_posts")
        if stored_posts is not None:
          is_saved_for_later = post_id in stored_posts
        else:
          is_saved_for_later = False

        return is_saved_for_later

    def get(self, request, slug):
        post = Post.objects.get(slug=slug)

        context = {
            "post": post,
            "post_tags": post.tags.all(),
            "comment_form": CommentForm(),
            "comments": post.comments.all().order_by("-id"),
            "saved_for_later": self.is_stored_post(request, post.id) # also for post req context
        }
        return render(request, "blog/post-detail.html", context)
```

```html
<pre>
<button>
  {% if saved_for_later %}
    Remove from "Read Later" List
  {% else %}
    Read Later
  {% endif %}
</button>
</pre>
```

```html
<!-- update base template -->
<nav>
  <a href="{% url "read-later" %}">Stored Posts</a>
  <a href="{% url "posts-page" %}">All Posts</a>
</nav>
```

```css
/* app.css */
#main-navigation a {
  /* ... */
  margin-left: 1rem;
}
```

```py
class ReadLaterView(View):
    def get(self, request):
        stored_posts = request.session.get("stored_posts")

        context = {}

        if stored_posts is None or len(stored_posts) == 0:
            context["posts"] = []
            context["has_posts"] = False
        else:
          posts = Post.objects.filter(id__in=stored_posts)
          context["posts"] = posts
          context["has_posts"] = True

        return render(request, "blog/stored-posts.html", context)
```

```html
<pre>
{% extends "base.html" %}
{% load static %}

{% block title %}
My Stored Posts
{% endblock %}

{% block css_files %}
<link rel="stylesheet" href="{% static "blog/stored-posts.css" %}">
{% endblock %}

{% block content %}
  <section id="stored-posts">
    {% if not has_posts %}
      <p>You didn't save any posts for later.</p>
    {% endif %}
    <ul>
      {% for post in posts %}
        <li><a href="{% url "post-detail-page" post.slug %}">{{ post.title }}</a></li>
      {% endfor %}
    </ul>
  </section>
{% endblock %}
</pre>
```

```css
#main-navigation {
  background-color: #390281;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.25);
}

#stored-posts {
  margin: 8rem auto 3rem auto;
  padding: 2rem;
  border-radius: 12px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.25);
  background-color: white;
  width: 90%;
  max-width: 30rem;
}

#stored-posts ul {
  list-style: none;
  margin: 0;
  padding: 0;
}

#stored-posts li {
  margin: 1rem 0;
}

#stored-posts a {
  display: block;
  padding: 1rem;
  width: 100%;
  text-decoration: none;
  border: 1px solid #ccc;
  color: #1b1b1b;
  font-weight: bold;
}

#stored-posts a:hover,
#stored-posts a:active {
  background-color: #ccc;
}
```

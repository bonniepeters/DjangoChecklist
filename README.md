*all information in this README.md is courtesy of General Assembly/SEIR*

# DjangoChecklist

## Setup 

- [ ] Create a directory for our project: i.e. ```my_project```

- [ ] ```cd``` into ```my_project```

- [ ] Install Django: ```pipenv install django```

- [ ] Install library to connect Django to PostgreSQL: ```pipenv install psycopg2-binary```

- [ ] Open ```my_project```: ```code .```

- [ ] Start Django project: ```pipenv run django-admin startproject my_project .```

- [ ] Activate virtual environment: ```pipenv shell```

- [ ] Create app: ```django-admin startapp myapp``` 

NOTE: ```my_project``` is the base django project where we will handle our routes.  ```myapp``` is where we write our models, controllers, and templates.

## Database Connection

- [ ] Login to psql: ```psql -d postgres```

- [ ] Create a database:
```
> CREATE DATABASE somedb;
> CREATE USER myUser WITH PASSWORD 'user';
> GRANT ALL PRIVILEGES ON DATABASE somedb TO myuser;
> \q
```

- [ ] Add Database details:
```python
# my_project/settings.py
DATABASES = {
  'default': {
    'ENGINE': 'django.db.backends.postgresql',
    'NAME': 'somedb',
    'USER': 'myuser',
    'PASSWORD': 'user',
    'HOST': 'localhost'
   }
}
```

- [ ] Add ```myapp``` to installed apps:
```python
# my_project/settings.py
INSTALLED_APPS = [
  '...',
  'myapp'
]
```

- [ ] Start server: ```python3 manage.py runserver```

- [ ] Navigate to ```localhost:8000```. (You should see a page welcoming you to Django!)

## Models

- [ ] Create desired models in ```myapp/models.py```:

```python
class Primary(models.Model):
    name = models.CharField(max_length=100)

    def __str__(self):
        return self.name

class Secondary(models.Model):
    primary = models.ForeignKey(Primary, on_delete=models.CASCADE, related_name='secondaries')
    title = models.CharField(max_length=100, default='untitled')

    def __str__(self):
        return f"{self.primary.name} - {self.title}"
```

## Migrations

- [ ] Stage migration to the database: ```python3 manage.py makemigrations```

NOTE: Every time you make changes to your models, run ```makemigrations``` again.

- [ ] Migrate staged changes: ```python3 manage.py migrate```

## Admin Console

- [ ] Create a super user: ```python3 manage.py createsuperuser```

- [ ] Fill in the info in the boxes that pop up.

- [ ] Import models in ```myapp/admin.py```:

```python
from django.contrib import admin
from .models import Primary, Secondary

admin.site.register(Primary, Secondary)
```
## Views

- [ ] Import models in ```myapp/views.py```:
```python
from django.shortcuts import render

from .models import Primary, Secondary
```
- [ ] Create views:
```python
# List View
def primary_list(request):
    primaries = Primary.objects.all()
    return render(request, 'myapp/primary_list.html', {'primaries': primaries})
    
# Show View
def primary_detail(request, pk):
    primary = Primary.objects.get(id=pk)
    return render(request, 'mya/artist_detail.html', {'artist': artist})
 ```
 ## URLs
 
 - [ ] Import paths, include, and admin in ```my_project/urls.py```:
 ```python
from django.conf.urls import include
from django.urls import path
from django.contrib import admin

urlpatterns = [
    path('admin', admin.site.urls),
    path('', include('myapp.urls')),
]
```

- [ ] Create ```myapp/urls.py```:
```python
from django.urls import path
from . import views

urlpatterns = [
# List URL
    path('', views.primary_list, name='primary_list'),
# Show URL
    path('primaries/<int:pk>', views.primary_detail, name='primary_detail'),
]
```

## READ

- [ ] In ```myapp``` create a ```templates``` directory with a ```myapp``` subdirectory

### Base

- [ ] In ```templates```'s ```myapp``` subdirectory create base ```base.html```

- [ ] Add HTML5 boilerplate to ```base.html```

- [ ] Build out ```base.html``` template:
```html
  <body>
    <h1>MyApp</h1>
    <nav>
      <a href="{% url 'primary_list' %}">Primaries</a>
    </nav>
    {% block content %} {% endblock %}
  </body>
```

### List Template

- [ ] In ```templates```'s ```myapp``` subdirectory create base ```primary_list.html```

- [ ] Build out ```primary_list.html``` template:
```html
{% extends 'tunr/base.html' %}
{% block content %}

<h2>Primaries <a href="">(+)</a></h2>
<ul>
  {% for primary in primaries %}
  <li>
    <a href="{% url 'primary_detail' pk=primary.pk %}">{{ primary.name }}</a>
  </li>
  {% endfor %}
</ul>

{% endblock %}
```
### Show Template

- [ ] In ```templates```'s ```myapp``` subdirectory create base ```primary_detail.html```

- [ ] Build out ```primary_detail.html``` template:
```html
{% extends 'tunr/base.html' %}
{% block content %}

<h2>{{ primary.name }} <a href="">(edit)</a></h2>

<h3>Secondaries <a href="">(+)</a></h3>
<ul>
  {% for secondary in primary.secondaries.all %}
  <li>
    <a href="">{{ secondary.title }}</a>
  </li>
  {% endfor %}
</ul>

{% endblock %}
```
## CREATE(Form)

- [ ] Create a ```forms.py``` file in ```myapp```:
```python
from django import forms
from .models import Primary

class PrimaryForm(forms.ModelForm):

    class Meta:
        model = Primary
        fields = ('name',)
```
- [ ] Add Form View:
```python
# myapp/views.py
from django.shortcuts import render, redirect

from .forms import PrimaryForm

def primary_create(request):
    if request.method == 'POST':
        form = PrimaryForm(request.POST)
        if form.is_valid():
            primary = form.save()
            return redirect('primary_detail', pk=primary.pk)
    else:
        form = PrimaryForm()
    return render(request, 'myapp/primary_form.html', {'form': form})
```
- [ ] Add Form URL:
```python
# myapp/urls.py
path('primary/new', views.primary_create, name='primary_create'),
```
- [ ] In ```templates```'s ```myapp``` subdirectory create base ```primary_form.html```
- [ ] Build out ```primary_form.html``` template:
```html
{% extends 'tunr/base.html' %} {% block content %}
<h1>New Artist</h1>
<form method="POST" class="artist-form">
  {% csrf_token %} {{ form.as_p }}
  <button type="submit" class="save btn btn-default">Save</button>
</form>
{% endblock %}
```
- [ ] Add Create Link to ```primary_list.html```:
```html
<h2>Primary <a href="{% url 'primary_create' %}">(+)</a></h2>
```

## UPDATE

- [ ] Add Edit View:
```python
# tunr/views.py
def primary_edit(request, pk):
    artist = Primary.objects.get(pk=pk)
    if request.method == "POST":
        form = PrimaryForm(request.POST, instance=primary)
        if form.is_valid():
            primary = form.save()
            return redirect('primary_detail', pk=primary.pk)
    else:
        form = PrimaryForm(instance=primary)
    return render(request, 'tunr/primary_form.html', {'form': form})
```
- [ ] Add Edit URL:
```python
# tunr/urls.py
path('primary/<int:pk>/edit', views.primary_edit, name='primary_edit'),
```
- [ ] Add Edit Link in Detail Template:
```html
<!-- tunr/templates/myapp/primary_detail.html -->
<h2>
  {{ primary.name }} <a href="{% url 'primary_edit' pk=primary.pk %}">(edit)</a>
</h2>
```

## DELETE

- [ ] Add Delete View
```python
# myapp/views.py
def primary_delete(request, pk):
    Primary.objects.get(id=pk).delete()
    return redirect('primary_list')
```
- [ ] Add Delete URL:
```python
# tunr/urls.py
path('artists/<int:pk>/delete', views.artist_delete, name='artist_delete'),
```

## CSS Styling

- [ ] In ```myapp``` create a ```static``` directory with a ```css``` subdirectory

- [ ] In ```static```'s ```css``` subdirectory create base ```myapp.css```

- [ ] Add static folder to ```base.html```:
```html
{% load staticfiles %}
  <head>
    <title>MyApp</title>
    <link rel="stylesheet" href="{% static 'css/myapp.css' %}" />
  </head>
```

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
```
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

- [ ] Add ```myapp``` to the last line of ```INSTALLED_APPS``` in ```my_project/settings.py```

- [ ] Start server: ```python3 manage.py runserver```

- [ ] Navigate to ```localhost:8000```. (You should see a page welcoming you to Django!)

## Models

- [ ] Create desired models in ```myapp/models.py```:

```
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

- [ ] Update ```myapp/admin.py```:

```
from django.contrib import admin
from .models import Primary, Secondary

admin.site.register(Primary, Secondary)
```
## Views

- [ ] Import models in ```myapp/views.py``` and create list view endpoint:
```
from django.shortcuts import render

from .models import Primary, Secondary

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
 
 - [ ] Update ```my_project/urls.py```:
 ```
from django.conf.urls import include
from django.urls import path
from django.contrib import admin

urlpatterns = [
    path('admin', admin.site.urls),
    path('', include('myapp.urls')),
]
```

- [ ] Create ```myapp/urls.py```:
```
from django.urls import path
from . import views

urlpatterns = [
# List URL
    path('', views.primary_list, name='primary_list'),
# Show URL
    path('primaries/<int:pk>', views.primary_detail, name='primary_detail'),
]
```

## READ / Templates

- [ ] In ```myapp``` create a ```templates``` directory with a ```myapp``` subdirectory

### Base

- [ ] Create base HTML file: ```myapp/templates/myapp/base.html```

- [ ] Add HTML5 boilerplate to ```base.html```

- [ ] Build out base template:
```
  <body>
    <h1>MyApp</h1>
    <nav>
      <a href="{% url 'primary_list' %}">Primaries</a>
    </nav>
    {% block content %} {% endblock %}
  </body>
```

### List Template

- [ ] Create list HTML file: ```myapp/templates/myapp/primary_list.html```

- [ ] Build out list template:
```
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

- [ ] Create detail/show HTML file: ```myapp/templates/myapp/primary_detail.html```

- [ ] Build out detail template:
```
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
## CREATE Form / Template

- [ ] Create form HTML file: ```myapp/templates/myapp/primary_form.html```


## CSS Styling

- [ ] In ```myapp``` create a ```static``` directory with a ```css``` subdirectory

- [ ] Create myapp.css file: ```myapp/static/css/myapp.css```

- [ ] Add static folder to ```base.html```:
```
{% load staticfiles %}
  <head>
    <title>MyApp</title>
    <link rel="stylesheet" href="{% static 'css/myapp.css' %}" />
  </head>
```

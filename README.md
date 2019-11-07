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
  'myapp'
]
```

- [ ] Start server: ```python3 manage.py runserver```

- [ ] Navigate to ```localhost:8000```. (You should see a page welcoming you to Django!)

## Models

- [ ] Create desired models in ```myapp/models.py```:

```python
class Post(models.Model):
    name = models.CharField(max_length=100)

    def __str__(self):
        return self.name

class Comment(models.Model):
    post = models.ForeignKey(Post, on_delete=models.CASCADE, related_name='comments')
    title = models.CharField(max_length=100, default='untitled')

    def __str__(self):
        return f"{self.post.name} - {self.title}"
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
from .models import Post, Comment

admin.site.register(Post, Comment)
```
## Views

- [ ] Import models in ```myapp/views.py```:
```python
from django.shortcuts import render

from .models import Post, Comment
```
- [ ] Create views:
```python
# List View
def post_list(request):
    posts = Post.objects.all()
    return render(request, 'myapp/post_list.html', {'posts': posts})
    
# Show View
def post_detail(request, pk):
    post = Post.objects.get(id=pk)
    return render(request, 'myapp/post_detail.html', {'post': post})
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
    path('', views.post_list, name='post_list'),
# Show URL
    path('posts/<int:pk>', views.post_detail, name='post_detail'),
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
      <a href="{% url 'post_list' %}">Primaries</a>
    </nav>
    {% block content %} {% endblock %}
  </body>
```

### List Template

- [ ] In ```templates```'s ```myapp``` subdirectory create base ```post_list.html```

- [ ] Build out ```post_list.html``` template:
```html
{% extends 'myapp/base.html' %}
{% block content %}

<h2>Primaries <a href="">(+)</a></h2>
<ul>
  {% for post in posts %}
  <li>
    <a href="{% url 'post_detail' pk=post.pk %}">{{ post.name }}</a>
  </li>
  {% endfor %}
</ul>

{% endblock %}
```
### Show Template

- [ ] In ```templates```'s ```myapp``` subdirectory create base ```post_detail.html```

- [ ] Build out ```post_detail.html``` template:
```html
{% extends 'myapp/base.html' %}
{% block content %}

<h2>{{ post.name }} <a href="">(edit)</a></h2>

<h3>Secondaries <a href="">(+)</a></h3>
<ul>
  {% for comment in post.comments.all %}
  <li>
    <a href="">{{ comment.title }}</a>
  </li>
  {% endfor %}
</ul>

{% endblock %}
```
## CREATE(Form)

- [ ] Create a ```forms.py``` file in ```myapp```:
```python
from django import forms
from .models import Post

class PostForm(forms.ModelForm):

    class Meta:
        model = Post
        fields = ('name',)
```
- [ ] Add Form View:
```python
# myapp/views.py
from django.shortcuts import render, redirect

from .forms import PostForm

def post_create(request):
    if request.method == 'POST':
        form = PostForm(request.POST)
        if form.is_valid():
            post = form.save()
            return redirect('post_detail', pk=post.pk)
    else:
        form = PostForm()
    return render(request, 'myapp/post_form.html', {'form': form})
```
- [ ] Add Form URL:
```python
# myapp/urls.py
path('post/new', views.post_create, name='post_create'),
```
- [ ] In ```templates```'s ```myapp``` subdirectory create base ```post_form.html```
- [ ] Build out ```post_form.html``` template:
```html
{% extends 'myapp/base.html' %} {% block content %}
<h1>New Post</h1>
<form method="POST" class="post-form">
  {% csrf_token %} {{ form.as_p }}
  <button type="submit" class="save btn btn-default">Save</button>
</form>
{% endblock %}
```
- [ ] Add Create Link to ```post_list.html```:
```html
<h2>Post <a href="{% url 'post_create' %}">(+)</a></h2>
```

## UPDATE

- [ ] Add Edit View:
```python
# myapp/views.py
def post_edit(request, pk):
    post = Post.objects.get(pk=pk)
    if request.method == "POST":
        form = PostForm(request.POST, instance=post)
        if form.is_valid():
            post = form.save()
            return redirect('post_detail', pk=post.pk)
    else:
        form = PostForm(instance=post)
    return render(request, 'myapp/post_form.html', {'form': form})
```
- [ ] Add Edit URL:
```python
# myapp/urls.py
path('post/<int:pk>/edit', views.post_edit, name='post_edit'),
```
- [ ] Add Edit Link in Detail Template:
```html
<!-- myapp/templates/myapp/post_detail.html -->
<h2>
  {{ post.name }} <a href="{% url 'post_edit' pk=post.pk %}">(edit)</a>
</h2>
```

## DELETE

- [ ] Add Delete View
```python
# myapp/views.py
def post_delete(request, pk):
    Post.objects.get(id=pk).delete()
    return redirect('post_list')
```
- [ ] Add Delete URL:
```python
# myapp/urls.py
path('posts/<int:pk>/delete', views.post_delete, name='post_delete'),
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

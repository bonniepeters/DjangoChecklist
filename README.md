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

- [ ] Login to psql: ```$ psql -d postgres```

- [ ] Create a database:
```
> CREATE DATABASE somedb;
> CREATE USER myUser WITH PASSWORD 'user';
> GRANT ALL PRIVILEGES ON DATABASE somedb TO myUser;
> \q
```

- [ ] Add Database details to ```my_project/settings.py```:
```
DATABASES = {
  'default': {
    'ENGINE': 'django.db.backends.postgresql',
    'NAME': 'myapp',
    'USER': 'myUser',
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
class Main(models.Model):
    name = models.CharField(max_length=100)

    def __str__(self):
        return self.name

class Secondary(models.Model):
    main = models.ForeignKey(Main, on_delete=models.CASCADE, related_name='secondaries')
    title = models.CharField(max_length=100, default='untitled')

    def __str__(self):
        return f"{self.main.name} - {self.title}"
```

## Migrations

- [ ] Stage migration to the database: ```python3 manage.py makemigrations```
NOTE: Every time you make changes to your models, run ```makemigrations``` again.

- [ ] Migrate staged changes: ```python3 manage.py migrate```

## Django Admin Console

- [ ] Create a super user: ```python3 manage.py createsuperuser```

- [ ] Fill in the info in the boxes that pop up.

- [ ] Update ```myapp/admin.py```:

```
from django.contrib import admin
from .models import Example

admin.site.register(Example)
```

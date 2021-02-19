# Django-ext-utils
### Useful utilities for your django project

## Requirements
+ Python >= 3.0
+ Django >= 2.0
+ Pillow
+ (optional) Djangorestframework

## Installation
```commandline
pip install django-ext-utils
```
Add to your settings:
`settings.py`

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    ...
    'django_ext_utils',  # you have to add this
]
```

## Usage

##### ResizeImageMixin
Resize image on the fly before save to `ImageField`:
```python
from django_ext_utils.utils import ResizeImageMixin
from django.contrib.auth.models import AbstractUser

class User(AbstractUser, ResizeImageMixin):
    avatar = models.ImageField(upload_to='avatars')
    ...
    
    def save(self, *args, **kwargs):
        if self.pk is None:
            self.resize(self.avatar, (200, 200))

        super().save(*args, **kwargs)
```

##### DeletedModel
Fake deletion. On `delete` instance - mark `is_mark_as_delete` instead 
```python
from django_ext_utils.utils import DeletedModel
from django.db import models

class Post(models.Model, DeletedModel):
    title = models.CharField(max_length=250)
    ...
```

##### SingletonModel
Create only one model instance
```python
from django_ext_utils.utils import SingletonModel
```

##### Service
Service-form-oriented helper
```python
from django_ext_utils.utils import Service
from .models import Subscription

# Services/service.py
class AddSubscriptionService(Service):
    def process(self):
        email = self.cleaned_data.get('email')
        name = self.cleaned_data.get('name')
        
        subscription, created = Subscription.objects.get_or_create(
            'email': email,
            defaults = {
                'name': name
            }
        )
        
        # some another actions...
        
        return subscription

# views.py       
subscription = NotificationService.execute({
    'email': 'foo@example.com',
    'name': 'foo'
})
...
return Response(subscription.serialize_data)

```

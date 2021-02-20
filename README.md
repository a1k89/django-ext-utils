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
Add to your settings 
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
On `delete` mark `is_mark_as_delete` instead 
```python
from django_ext_utils.utils import DeletedModel
```

##### SingletonModel
Create only one instance of model
```python
from django_ext_utils.utils import SingletonModel
```

##### Service
Service-form-oriented helper (I really like services)
```python
from django_ext_utils.utils import Service
from .models import Subscription

# Services/service.py
class AddSubscriptionService(Service):
    def go(self):
        email = self.cleaned_data.get('email')
        name = self.cleaned_data.get('name')
        
        subscription, created = Subscription.objects.get_or_create(
            email = email,
            defaults = {
                'name': name
            }
        )
        if created:
            # notify user
            ......
        return subscription

# views.py       
subscription = NotificationService.exec({
    'email': 'foo@example.com',
    'name': 'foo'
})
...
return Response(subscription.serialize_data)
```

### Rest api helpers
#### ResponsesMixin
* wrapper for Response
* simple_text_response/success_objects_response/error_response

```python
class UserProfileViewSet(ResponsesMixin, GenericAPIView):
    serializer_class = UserSerializer
    
    def post(self, request, *args, **kwargs):
        user = self.request.user
        serializer = UserSerializer(instance=self.request.user,
                                    data=self.request.data,
                                    context={'request': self.request})
        if serializer.is_valid():
            serializer.save()
            return self.success_objects_response(serializer.data) 

        return self.error_response(serializer.errors)

    def get(self, request, *args, **kwargs):
        user = request.user
        data = user._serialize_data(request)

        return self.success_objects_response(data)
```
##### Base
Abstract class to help serialize your models
In your `settings.py` add 
```
SERIALIZERS_PATH = 'api.v1.serializers' # path to serializer folder
```
* get_serializer # Return serializer by name (<Model>Serializer)
* get_short_serializer # Retrun another serializer by name (<Model>ShortSerializer)
* serialize_data # Find serializer by name and serialize data
* _serialize_data # Find serializer by name and serialize data with request

```python
from django_ext_utils.rest_utils import Base

# models.py
class User(AbstractUser, Base,  ResizeImageMixin):
    avatar = models.ImageField(upload_to='avatars')
    ....

# serializer.py
class UserSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = [
            'pk',
            'first_name',
            'last_name'
        ]

# views.py
from rest_framework.generics import GenericAPIView

class UserApiView(GenericAPIView):
    def get(self, request, *args, **kwargs):
        user = request.user
        data = user._serialize_data(request)
        return self.success_objects_response(data)
```

#### ImageBase64Field
* Wrapper for `serializers.ImageField()`
* Got `base64` string and save in model `ImageField`
* Return `link to image` (http://example.com/media/avatars/my.jpeg)

```python
# serializer.py

class SomeUserApiView(serializers.ModelSerializer):
    image = ImageBase64Field()
    created_at = TimestampField(read_only=True)
  
    class Meta:
        model = User
        fields = [
            'uid',
            'image',
            'created_at'
        ]
```


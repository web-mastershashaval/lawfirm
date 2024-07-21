# Cloudinary

## Why Cloudinary
Heroku can not server media files. AWS S3 is be best option. But S3 is free for 12 months and needs credit card to get and AWS account.
Cloudinary provides 10GB of free storage and account can be created without credit card.
N.B.: With the free package, only image files can be server. Raw files like .pdf, .docx, .txt, etc can not be served. Make sure all your file fields are image fields if you are using free package. Else, you can upload the file but can't access the file.

## Installation
### Packages
- [cloudinary](https://pypi.org/project/cloudinary/)
- [django-cloudinary-storage](https://pypi.org/project/django-cloudinary-storage/)

### Commands

```bash
pip install cloudinary
pip install django-cloudinary-storage
```

### Instructions
- [Official](https://pypi.org/project/django-cloudinary-storage/)
- [By dothedev](https://www.dothedev.com/blog/heroku-django-store-your-uploaded-media-files-for-free/)

### settings.py Integration
- For this project, it's settings/base.py
- If cloudinary is used for both static and media files, cloudinary_storage must be before django.contrib.staticfiles this way.

```python
INSTALLED_APPS = [
    # ...
    'cloudinary_storage',
    'django.contrib.staticfiles',
    'cloudinary',
    # ...
]
```

- As we are serving only media files, cloudinary_storage must be at the end of INSTALLED_APPS, otherwise it will not get our static files.

```python
INSTALLED_APPS = [
    # ...
    'cloudinary_storage',
    'cloudinary',
    # ...
]

### Cloudinary Credentials
- Generally this information is added in the settings.py
- In env we save CLOUDINARY_INFO = 'CLOUD_NAME:API_KEY:API_SECRET'
- Then in settings/production.py we add the following code.
- Thus way we save only one variable in the env.

```bash
CLOUDINARY_INFO = config('CLOUDINARY_INFO').split(':')

CLOUDINARY_STORAGE = {
    'CLOUD_NAME': CLOUDINARY_INFO[0],
    'API_KEY': CLOUDINARY_INFO[1],
    'API_SECRET': CLOUDINARY_INFO[2]
}
```

### Storage for Media Files

- There are 3 media storages. You should include DEFAULT_FILE_STORAGE in the settings.py. For us, we'll include it in the settings/production.py

```python
# For images
DEFAULT_FILE_STORAGE = 'cloudinary_storage.storage.MediaCloudinaryStorage'

# For videos
DEFAULT_FILE_STORAGE = 'cloudinary_storage.storage.VideoMediaCloudinaryStorage'

# For raw files, like txt, pdf
DEFAULT_FILE_STORAGE = 'cloudinary_storage.storage.RawMediaCloudinaryStorage'
```

- If we want to use both images and raw files storages, we can define the most used storage as DEFAULT_FILE_STORAGE, and for other types we can define the storage in the model.

```python
# settings.py
DEFAULT_FILE_STORAGE = 'cloudinary_storage.storage.MediaCloudinaryStorage'

# models.py
from django.db import models

from cloudinary_storage.storage import RawMediaCloudinaryStorage

class UserProfile(models.Model):
    name = models.CharField(max_length=100)
    image = models.ImageField(upload_to='images/', blank=True)  # no need to set storage, field will use the default one
    document = models.ImageField(upload_to='raw/', blank=True, storage=RawMediaCloudinaryStorage())
```

## Cloudinary Account Settings
- In the Media Library create a folder 'media'. This folder will store the images.
- Create a folder 'media/lawyers'. Here lawyers is our APP_NAME and we are using this to distinguish among various projects.
- Now create the file for default profile picture.
'/media/lawyers/defaults/img/user_pro_pic.jpg'.
- After the upload, file name may change. If it changes, update the change in the default field of the model and the associated file in the local media storage.
- For this project, you need to make changes in these 2 places:
    - /media/lawyers/media/lawyers/defaults/img/user_pro_pic.jpg
    - /users/functions.py variable: user_default_pro_pic
- Solution to this issue:
    - Go to Cloudinary Upload Settings
    - Scroll down to Upload presets
    - Select ml_default or another other present being used for the image uploads
    - Uncheck Unique filename
    - Now it won't add any character after the name of the uploaded image.
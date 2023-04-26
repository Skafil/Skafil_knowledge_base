I have made following steps to get my project ready for development:
1. Create virtualenv and activate it.
    ```
    pip install virtualenv
    python -m venv <directory>
    source <directory>/bin/activate (for Linux)
    ```
2. Create django project
    ```
    pip install django
    django-admin startproject <directory>
    ```
3. Setting up environment variables.
    - **Create .env file** in root directory    
        ```
        export SECRET_KEY=...
        export DEBUG=...
        ...
        ```
    - **Create .env-sample file** (to let everyone know how it is structured)
        ```
        export SECRET_KEY=<>
        export DEBUG=<>
        ...
        ```
    - **to use env variables** in settings.py use syntax:  
    `os.environ['<env variable name>']`  
    `DEBUG = os.environ['DEBUG'] == 'True'` // for boolean
    - **to generate new secret** key use this command in the terminal:  
    `python3 -c 'import secrets; print(secrets.token_hex(100))'`  
    This is better method than using *generate_random_secret_key()*. For more read [S Ghosh post on Stack Overflow](https://stackoverflow.com/questions/41298963/is-there-a-function-for-generating-settings-secret-key-in-django).
4. Create Git repository in root directory(there are several ways, so I just mentioned it).

5. Add new apps
    - `python manage.py startapp <name>`
    - add app to INSTALLED_APPS 
    ```
    INSTALLED_APPS = [
        'django.contrib.admin',
        ...
        'django.contrib.staticfiles',

        # My apps
        '<app name>.apps.<App Name>Config',
    ]
    ```
    - include app urls.py in project urls.py
        - create urls.py in app
        ```
        from django.urls import path
        from . import views

        urlpatterns = [
            path('', views.index, name="index"),
        ]
        ```
        - in project's urls.py
        ```
        from django.urls import path, include
        
        urlpatterns = [
        path('admin/', admin.site.urls),
        path('', include('<app name>.urls')),
        ]
        ```
6. Add templates (there are two ways)
    1. Create general templates folder in project folder and put there all HTMLS or
    2. Create app-specific templates folder in app folder ('app name' --> templates --> 'app name').
    - finally, add templates dir's path to settings.py
    ```
    TEMPLATES = [
        {
            'BACKEND': 'django.template.backends.django.DjangoTemplates',
            'DIRS': [
                BASE_DIR / 'templates', # general templates dir
            ], 
            ...
        }
    ]
    ```
7. **Add staticfiles** (media, js, css):
    - add paths to settings.py
        ```
        STATIC_ROOT = BASE_DIR / 'staticfiles'
        STATICFILES_DIRS = [
            BASE_DIR / 'static',
        ]

        MEDIA_URL = 'media/'
        MEDIA_ROOT = BASE_DIR / 'media'
        ```
    - add paths to project's urls.py
        ```
        ...
        from django.conf import settings
        from django.conf.urls.static import static

        urlpatterns = [
            ...
        ]

        urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
        ```
    - if using ImageField in models.py:
        - `pip install Pillow`
        -  use ImageField like this
            ```
            profileimg  = models.ImageField(upload_to='profile_images',                      default='blank-profile-picture.png')
            ```
        All images inserted through this field will be uploaded to 'profile_images' folder inside media folder.  
        What's more, we are using default image from media folder.

**File tree should look like this**:  
root  
- env
- project dir
    - project dir
    - app dirs
        - ...
        - templates
            - 'app dir name'
    - static
    - staticfiles
    - media
    - templates
    - manage.py
    - database
- .env
- .env-sample
- .gitignore

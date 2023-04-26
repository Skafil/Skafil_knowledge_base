- [User model](#user-model)
  - [***What model to use?***](#what-model-to-use)
    - [*USER AS A FIELD*](#user-as-a-field)
  - [***Don't use User model directly!***](#dont-use-user-model-directly)
- [Views.py](#viewspy)
  - [***How to authenticate user?***](#how-to-authenticate-user)
  - [***How to register user?***](#how-to-register-user)
  - [***How to login user?***](#how-to-login-user)
  - [***How to logout user?***](#how-to-logout-user)
  - [***Secure password hashing and checking***](#secure-password-hashing-and-checking)
  - [***How to update user's data?***](#how-to-update-users-data)
  - [***How to prevent unlogged user from going to specific page?***](#how-to-prevent-unlogged-user-from-going-to-specific-page)
  - [**Send email to reset password**](#send-email-to-reset-password)


## User model


### ***What model to use?***
You can **default User model** or **create a custom own**. You can also create new class (for example Profile) that **takes User as a ForeignKey** (that way you can add fields that don't concern auth).

---

#### *USER AS A FIELD*

```
class Profile(models.Model):
    user = models.ForeignKey(User, on_delete=models.CASCADE)
    id_user = models.IntegerField()
    bio = models.TextField(blank=True, null=True)
    profileimg = models.ImageField(upload_to='profile_images', default='blank-profile-picture.png')
    location = models.CharField(max_length=100, blank=True, null=True)

    def __str__(self):
        return self.user.get_username()
```

### ***Don't use User model directly!***

You can have several User models but your user will use only one during its session, so there comes problem: "Which model will be used?". Instead of trying to guess, better **just use reference** to currenct active User model this way:
```
from django.contrib.auth import get_user_model
User = get_user_model()
```
---

## Views.py

Below explanations are written assuming that you use **function views**.

---

### ***How to authenticate user?***

1. Make sure that data came to you in POST method.  
    ```
    if request.method == 'POST':
    ```

2. Get the data from the input fields from the form. **You need to use input field's name**.
    ```
    username = request.POST['username']
    ```

3. Use **`authenticate(username, password)`** function.
    ```
    from django.contrib.auth import authenticate
    user = authenticate(username=username, password=password)
    ```
    Note that if the username and password don't match than **the user will be set to None**.  

Great, user is authenticated!

---

### ***How to register user?***

1. Make sure that data came to you in POST method. 
2. Get the data from the input fields from the form. **You need to use input field's name**.
3. (Optional) check if the password1 and password2 match.
4. Check if user with given username or email doesn't exist.
    ```
    if User.objects.filter(email=email).exists():
        ...
    elif User.objects.filter(username=username).exists():
    ```
5. If username and email aren't taken, create new user and save it.
    ```
    user = User.objects.create_user(username=username, email=email, password=password)
    user.save()
    ```
It would be good to know more about **`save()`** so for more info check    
Django --> explanations.md --> save() 



---

### ***How to login user?***

1. Authenticated user.
2. Check if the authenticated user is not set to None.
    ```
    if user is not None:
    ```
3. Use 
    ```
    from django.contrib.auth import login
    login(request, user)
    ``` 
    function. The second argument is user model returned by authenticate() function. 

---

### ***How to logout user?***
Just use 
```
from django.contrib.auth import login
logout(request)
``` 
and that's all. There is only one user during session so the program know which one to logout.

---

### ***Secure password hashing and checking***

To hash password, use `make_password(password, salt=None, hasher='default')`.

To compare plain password against hashed password, use `check_password(password, encoded, setter=None, preferred='default')`. It returns True or False. *preffered* argument let you choose hash algorithm, default is the first one in PASSWORD_HASHERS setting (PBKDF2)

---

### ***How to update user's data?***

1. Get model.
2. Make sure that data came to you in POST method. 
3. Get the data from the input fields from the form. **You need to use input field's name**.
    - if you need to take image, do something like this:
        ```
        # We need yet to add enctype in form because of file
        if request.FILES.get('image') == None:
            image = user_profile.profileimg
        else:
            image = request.FILES.get('image')
        ```
4. change user's data using the ones from input fields. You can use just **assignment operator** (`user_profile.bio = bio`) **or F expression**   
(check Django --> explanations.md --> save() --> Pracital use of F expression)

5. use `save()` method.

---

### ***How to prevent unlogged user from going to specific page?***

Above function view use `@login_required(login_url)` decorator:
```
from django.contrib.auth.decorators import login_required

@login_required(login_url='signin')
def settings(request):
```

---

### **Send email to reset password**

Most of the works is done by Django already. Just create own templates and make some configurations.

1. **Set up SMTP Server in settings.py**.
    ```
    EMAIL_BACKEND = "django.core.mail.backends.smtp.EmailBackend"
    EMAIL_HOST=
    EMAIL_PORT=
    EMAIL_HOST_USERNAME=
    EMAIL_HOST_PASSWORD=
    EMAIL_USE_TLS=True/False
    ```
    Here we will store sent emails in directory.

2. **Get templates**:
    - create your own:
       - password_reset_form.html
       - password_reset_done.html 
       - password_reset_confirm.html
       - password_reset_complete.html
    - and use built-in views:
        ```
        from django.contrib.auth import views as auth_views

        ......    
        path('password-reset/', auth_views.PasswordResetView.as_view(template_name='registration/password_reset_form.html'),
                name='password_reset'),
        path('password-reset-confirm/<uidb64>/<token>/',
            auth_views.PasswordResetConfirmView.as_view(template_name='registration/password_reset_confirm.html'),
            name='password_reset_confirm'),
        path('password-reset/done/',
            auth_views.PasswordResetDoneView.as_view(template_name='registration/password_reset_done.html'),
            name='password_reset_done'),
        path('password-reset-complete/',
            auth_views.PasswordResetCompleteView.as_view(template_name='registration/password_reset_complete.html')),
        ....
        ```

3. **Send email**
    - use functions
        ```
        from django.core.mail import send_mail

        # Send mails over multiple connections
        send_mail(
            'Subject here',
            'Here is the message.',
            'from@example.com',
            ['to@example.com'],
            fail_silently=False, # Show error
            auth_user=None, 
            auth_password=None, # For auth with SMTP server
        )
        ```

        ```
        # Send mails over one connection
        send_mass_mail(
            datatuple, 
            fail_silently=False
            auth_user=None, 
            auth_password=None
        )

        datatuple is a tuple with tuples in format: (subject, message, from_email, recipient_list)
        ```

        You can also use `mail_admins()` and `mail_managers()` functions.

        **Be careful!** There is cyberattack called Header Injection. Django prevents this attack by forbidding newlines in header. So if you try header injection in send_email(), the error will be thrown. Validate your data!
    - use `EmailMessage` class,
    - use an instance of an email backend.

    For more [read Django docs](https://docs.djangoproject.com/en/3.2/topics/email/#)


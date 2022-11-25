# Crime Scene Reporting Portal

## Background
There are cases when people have either witnessed a crime, have a suspicion, or are 
aware that crime is being planned, but hesitant come forward to report. Forcing people 
to identify themselves as part of providing a tip-off might therefore lead to missing 
out on a lot of useful information. Crime Activity Reporting Portal will be a platform 
through which a person can give tip-off about any suspicious activity or crime in a secure
and completely anonymous manner. The platform will allow people to give out the most 
useful information in the shortest amount of time. 

I have chosen the Django framework to build the project because it is a beginner-friendly web development framework. For frontend experience tailwind and automation purpose celery, Redis.
Courier API for email sendign purposes. Newscatcherapi to get lates News.So we have divided this project in parts. you have to follow the each steps and see its results if you
are getting the right ones.

## Instructions

### Part 1: Setting up the Django + tailwind project
In first Part we are going to set django with tailwindcss for that purpose you will have to follow along with the steps.

1. Run the following command in the terminal to create a new Django project with the name of your choice 
    `django-admin startproject [project_name]`

2. To go inside the project folder
    `cd [project_name]/`
   
2. Create and activate virtual environment for the project
    `python -m venv venv_name`
    `venv_name\Scripts\activate`

    
3. To install all required Django packagesfor this create requirements.txt and include
   following packages in it.
    ``` 
    celery==5.2.7
    Django==4.1.2
    django-celery-beat==2.4.0
    django-celery-results==2.4.0
    django-compressor==4.1
    newscatcherapi==0.7.1
    redis==4.3.4
    social-auth-app-django==5.0.0
    social-auth-core==4.3.0
    sqlparse==0.4.3
    trycourier==4.4.0
    ```
   After entering above packages open your terminal and enter `pip install -r requirements.txt`
   it will install all the required packages in virtual environment.

   Note: While following this tutorial if you get module not found error then you will have to 
   install that package by entering 'pip install module_name' in the terminal.

4. To check if we have successfully set the project run the command 
   in terminal
   ```  
    python manage.py runserver
 
   ```
    

5. Install django-compressor by running the following command in your terminal:
    ```
    python -m pip install django-compressor
    ```

6. Add compressor to the installed apps inside the settings.py file:
    ```
    # project_folder/project_name/settings.py

    INSTALLED_APPS = [
        'django.contrib.admin',
        'django.contrib.auth',
        'django.contrib.contenttypes',
        'django.contrib.sessions',
        'django.contrib.messages',
        'django.contrib.staticfiles',
        'compressor',  # new
    ]

    ```

7. Create a new templates and static directory inside the project folder and update settings.py folder:

    ```    
    TEMPLATES = [
        {
            ...
            'DIRS': ['templates'],
            ...
        },
    ]
 
    import os

    COMPRESS_ROOT = os.path.join(BASE_DIR, 'static')

    COMPRESS_ENABLED = True

    STATICFILES_FINDERS = ('compressor.finders.CompressorFinder',)
    ```

9. Create two new folders and an input.css file inside the static/src/ folder:
    ```
    static
    └── src
        └── input.css

    ```


10. Later we will import the Tailwind CSS directives and use it as the source file for the final stylesheet.
   Create a new views.py file inside app_name/ next to urls.py and add the following content:
   
    ```
        from django.shortcuts import render

        def index(request):
            return render(request, 'index.html')

    ```

11. Add index function route in the urls.py file 
    ```
    from django.contrib import admin
    from django.urls import path, include
    from . import views

    urlpatterns = [
        path('admin/', admin.site.urls),
        path('', views.index, name='index'),
    ]

    ```

12. Create a new base.html file inside the templates/ directory:
    ```
    {% load compress %}
    {% load static %}

    <!DOCTYPE html>
    <html lang="en">

    <head>
        <meta charset="UTF-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Django + Tailwind CSS</title>

        {% compress css %}
        <link rel="stylesheet" href="{% static 'src/output.css' %}">
        {% endcompress %}

    </head>

    <body class="bg-green-50">
        <div class="container mx-auto mt-4">
            {% block content %}
            {% endblock content %}
        </div>
    </body>

    </html>

    ```

13. Create an index.html file that will be served as the homepage:
    ```
    <!-- templates/index.html -->

    {% extends "base.html" %}

    {% block content %}
    <h1 class="text-3xl text-green-800">Django + Tailwind CSS</h1>
    {% endblock content %}


    ```

14. Finally, create a local server instance by running the following command: This will give us
an error because we did not installed tailwind yet. 
    ```
    python manage.py runserver

    ```

15. Run the following command the install Tailwind CSS as a dev dependency using NPM:
    ```
    npm install -D tailwindcss

    ```
16. Using the Tailwind CLI create a new tailwind.config.js file:
    ```
    npx tailwindcss init

    ```

17. Configure the template paths using the content value inside the Tailwind configuration file:
    ```
    module.exports = {
    content: [
        './templates/**/*.html'
    ],
    theme: {
        extend: {},
    },
    plugins: [],
    }
    ```

18. Import the Tailwind CSS directives inside the input.css file:
    ```
    /* static/src/input.css */

    @tailwind base;
    @tailwind components;
    @tailwind utilities;

    ```

19. Run the following command to watch for changes and compile the Tailwind CSS code:
    ```
    npx tailwindcss -i ./static/src/input.css -o ./static/src/output.css --watch

    ```
20. create another terminal then activate the virtual environment, run and check if it works.

You should have got final Structure of files and the output as you see in the below images.




### Part 2:  Create a superuser and models for the project

In this part we are going to build the models/tables and setting up the superuser details To
see each changes we are making in our database.

1. Create accounts app inside our project folder to manage user's details.
    ```
    python manage.py startapp accounts
    //you can give the name of your choice
    ```

2. Let's add this accounts app in setting.py/INSTALLED_APPS and path of it in the urls.py file

    ```
    ...
    path('accounts/', include('accounts.urls')),
    ...
    ```

3. To store the data we have to create the tables/ models. so lets create them by adding 
   below code in the accounts/models.py file 
   ```
    from django.db import models
    from django.utils import timezone
    from django.contrib.auth.models import AbstractBaseUser, BaseUserManager, PermissionsMixin

    class MyAccountManager(BaseUserManager):
        def create_user(self, first_name, last_name, username, email, password=None):
            if not email:
                raise ValueError('User must have an email address')
            
            if not username:
                raise ValueError('User must have an Username')
            
            user = self.model(
                email = self.normalize_email(email), #  If you enter email address in captital format it will make it normalize
                username = username,
                first_name = first_name,
                last_name = last_name,
            )
            
            user.set_password(password)
            user.save(using=self._db)
            return user

        def create_superuser(self, first_name, last_name, email, username, password):
            user = self.create_user(
                email = self.normalize_email(email), #  If you enter email address in captital format it will make it normalize
                username = username,
                password = password,
                first_name = first_name,
                last_name = last_name,
            )
            
            user.is_admin = True
            user.is_staff = True
            user.is_active = True
            user.is_superadmin = True
            user.save(using=self._db)
            return user



    class Account(AbstractBaseUser, PermissionsMixin):
        first_name  = models.CharField(max_length=50)
        last_name   = models.CharField(max_length=50)
        username    = models.CharField(max_length=50, unique=True)
        email       = models.EmailField(max_length=100,unique=True)
        newsletter  = models.BooleanField(default=False)
        # required
        date_joined = models.DateTimeField(auto_now_add=True)
        last_login  = models.DateTimeField(auto_now_add=True)
        is_admin    = models.BooleanField(default=False)
        is_staff    = models.BooleanField(default=False)
        is_active    = models.BooleanField(default=False)
        is_superadmin    = models.BooleanField(default=False)

        USERNAME_FIELD = 'email'    
        REQUIRED_FIELDS = ['username', 'first_name','last_name']

        objects = MyAccountManager()

        def __str__(self): # When we return the object in side the template so this should return the email address
            return self.email
        
        def has_perm(self, perm, obj=None): # if the user is the admin then he has all the permissions
            return self.is_admin 

        def has_module_perms(self, add_label):
            return True

        def name(self):
            return f'{self.first_name} {self.last_name}' 


    class Report(models.Model):
        user = models.ForeignKey(Account, on_delete=models.CASCADE)
        country = models.CharField(max_length=10)
        state   = models.CharField(max_length=10)
        crime = models.TextField(max_length=10)
        description =  models.TextField(max_length=200, null=True, default=True, blank=True)
        proof = models.FileField(upload_to='dashboard\proof', null=False, default=False)
        reported_at = models.DateTimeField(auto_now_add = True)



        def name(self):
            return f'{self.user.f_name} {self.user.l_name}'


    class Anonymous_report(models.Model):
        country = models.CharField(max_length=10)
        state   = models.CharField(max_length=10)
        crime = models.TextField(max_length=10)
        description =  models.TextField(max_length=200, null=True, default=True, blank=True)
        proof = models.FileField(upload_to='dashboard\proof', null=False, default=False)
        reported_at = models.DateTimeField(auto_now_add = True)

        def name(self):
            return f'{self.user.f_name} {self.user.l_name}'

   ```
   
   After adding this code in the models.py enter the following command in the terminal.
   makemigrations will creates models for us and migrate will reflect it in the database.
   ```
    python manage.py makemigrations 
    python manage.py migrate 

   ```

    To register this models into admin section we have to the the models inside admin.py file
    ```
    //accounts/admin.py

    from django.contrib import admin
    from django.contrib.auth.admin import UserAdmin
    from .models import Account, Report, Anonymous_report
    from django.utils.html import format_html
    # Register your models here.

    class AccountAdmin(UserAdmin):
        list_display = ('email', 'first_name', 'last_name', 'username', 'last_login', 'date_joined', 'is_active')
        list_display_links = ('email', 'first_name', 'last_name')
        readonly_fields = ('last_login', 'date_joined')
        ordering = ('-date_joined',)

        filer_horizontal = ()
        list_filter = ()
        fieldsets = ()

    class ReportAdmin(admin.ModelAdmin):
        list_display = ('user', 'country', 'state', 'crime', 'description', 'proof',)
        list_display_links = ('user',)
        readonly_fields = ('crime', 'proof',)
        ordering = ('-country',)

    
        filer_horizontal = ()
        list_filter = ()
        fieldsets = ()




    admin.site.register(Account, AccountAdmin)
    admin.site.register(Report, ReportAdmin)
    admin.site.register(Anonymous_report)


    ```
Note : if any errors occurs while creating models e.g dependency error initial_1001 file 
then follow the steps
```
    i] delete sqlite3.db file
   ii] delete migrations folder that present in you project and app folder.
  iii] run the command 'python manage.py runserver'
   iv] python manage.py makemigrations app_names
    v] python manage.py migrate
```
This should solve your problem.

4. Let's create superuser to view all the tables successfully created or not. After entering below command in the terminal it will ask you for the details.
   Enter the details carefully and save it somewhere as it will be required every time you visit admin section.
 
    ```
    python manage.py createsuperuser

    ```

5. visit https://localhost:8000/admin/
   You should see the login page. Here Enter the credentials that you saved in the previous step.
   

   Here you can see the models that we have created.



### Part 3:  Create Pages along with the backend logic
In this part, we are going to make all the templates that we required and the login and registerations 

1. Here are the functions you have to add in the accounts/views.py file 
    ```
    from django.shortcuts import render, redirect
    from django.http import HttpResponse
    from django.contrib import messages, auth
    from django.shortcuts import render
    from .models import Account
    from django.contrib.auth.decorators import login_required
    from django.views.decorators.csrf import csrf_exempt

    # Create your views here.

    @csrf_exempt
    def login(request):
        if request.method == 'POST':
            email = request.POST.get('email')
            password = request.POST.get('password')
            print(password)
            user = auth.authenticate(email=email, password=password)
            print(user)
            if user is not None:
                auth.login(request, user)
                url = request.META.get('HTTP_REFERER')
                return redirect('dashboard')
            else:
                messages.warning(request, 'Invalid login credentials')
                return redirect('login')
        
        return render(request, 'accounts/login.html')

    @csrf_exempt
    def register(request):
        if request.method=='POST':
            first_name = request.POST['f_name']
            last_name = request.POST['l_name']
            email = request.POST['email']
            password = request.POST['password']
            username = email.split('@')[0]
            user = Account.objects.create_user(first_name=first_name, last_name=last_name, email=email, username=username, password=password)
            user.is_active = True
            user.save()
            return redirect('login')
            
        return render(request, 'accounts/register.html')


    @csrf_exempt
    @login_required(login_url='login')
    def logout(request):
        auth.logout(request)
        return redirect('login')

    ```
2. Here are the functions you have to add in the project_app/views.py file 
    ```
    from django.shortcuts import render
    from django.http import HttpResponse
    from accounts.models import Report, Anonymous_report, Account
    from django.contrib.auth.decorators import login_required
    from django.views.decorators.csrf import csrf_exempt
    from django.db.models import Q
    from trycourier import Courier
    from pprint import pprint
    from accounts.models import Report
    from django.db.models import Q



    @csrf_exempt
    def dashboard(request):
        return render(request, 'dashboard.html')


    @csrf_exempt
    @login_required(login_url='login')
    def report_crime(request):
        if request.method == 'POST':
            state = request.POST['state']
            country = request.POST['country']
            crime = request.POST['crime']
            crime_description = request.POST.get('crime_description')
            proof = request.FILES['proof']
            user = request.user
            report = Report.objects.create(user=user, country=country, state=state, crime=crime, description=crime_description, proof=proof)
            report.save()
            return render(request, 'successfully_reported.html')
        else:
            return render(request, 'report.html')
        


    @csrf_exempt
    def report_anonymously(request):
        if request.method == 'POST':
            state = request.POST['state']
            country = request.POST['country']
            crime = request.POST['crime']
            crime_description = request.POST.get('crime_description')
            proof = request.FILES['proof']
            report = Anonymous_report.objects.create(country=country, state=state, crime=crime, description=crime_description, proof=proof)
            report.save()


        return render(request, 'report_anonymously.html')

    @csrf_exempt
    def search(request):
        return render(request, 'dashboard.html')

    ```


3. Add the urls inside urls.py to route the request to appropriate functions
    ```
    # accounts/urls.py
    
    ...
    urlpatterns = [
        path('login/', views.login, name="login"),
        path('register/', views.register, name="register"),
        path('logout/', views.logout, name="logout"),   
    ]

    # project_app/views.py

    ...
    urlpatterns = [
        path('admin/', admin.site.urls),
        path('accounts/', include('accounts.urls')),
        path('', views.dashboard, name='dashboard'),
        path('report_crime/', views.report_crime, name="report_crime"),
        path('report_anonymously/', views.report_anonymously, name="report_anonymously"),
    ]
    ...
    ```


4. Copy the static folder from github repo and paste and replace in our project.

5. Add following codes in their respected templates files. 
   you can find these templates in the github repor also.

    ```
    // templates/accounts/login.html
    {% extends 'base.html' %}
    {% block content %}
    <div style="overflow: hidden">
    <div
        class="bg-purple-900 absolute top-0 left-0 bg-gradient-to-b from-gray-900 via-gray-900 to-purple-800 bottom-0 leading-5 h-full w-full overflow-hidden">
        
    </div>
    <div
        class="relative   min-h-screen  sm:flex sm:flex-row  justify-center bg-transparent rounded-3xl shadow-xl">
        <div class="flex-col flex  self-center lg:px-14 sm:max-w-4xl xl:max-w-md  z-10">
            <div class="self-start hidden lg:flex flex-col  text-gray-300">
                
                <h1 class="my-3 font-semibold text-4xl">Welcome back</h1>
                <p class="pr-3 text-sm opacity-75">Welcome to Crime Scene Reporting Portal</p>
            </div>
        </div>
        <div class="flex justify-center self-center  z-10">
            <div class="p-12 bg-white mx-auto rounded-3xl w-96 ">
                <div class="mb-7">
                    <h3 class="font-semibold text-2xl text-gray-800">Sign In </h3>
                    <p class="text-gray-400">Don'thave an account? <a href="{% url 'register' %}"
                            class="text-sm text-purple-700 hover:text-purple-700">Sign Up</a></p>
                </div>

            <form class="text-center" action="{% url 'login' %}" method="POST">
                {% csrf_token %}
                <div class="space-y-6">
                    <div class="">
                        <input name="email" class=" w-full text-sm  px-4 py-3 bg-gray-200 focus:bg-gray-100 border  border-gray-200 rounded-lg focus:outline-none focus:border-purple-400" type="" placeholder="Email">
                    </div>


                        <div class="relative" x-data="{ show: true }">
                            <input name="password" placeholder="Password" :type="show ? 'password' : 'text'" class="text-sm text-gray-200 px-4 py-3 rounded-lg w-full bg-gray-200 focus:bg-gray-100 border border-gray-200 focus:outline-none focus:border-purple-400">
                            <div class="flex items-center absolute inset-y-0 right-0 mr-3  text-sm leading-5">

                                <svg @click="show = !show" :class="{'hidden': !show, 'block':show }"
                                    class="h-4 text-purple-700" fill="none" xmlns="http://www.w3.org/2000/svg"
                                    viewbox="0 0 576 512">
                                    <path fill="currentColor"
                                        d="M572.52 241.4C518.29 135.59 410.93 64 288 64S57.68 135.64 3.48 241.41a32.35 32.35 0 0 0 0 29.19C57.71 376.41 165.07 448 288 448s230.32-71.64 284.52-177.41a32.35 32.35 0 0 0 0-29.19zM288 400a144 144 0 1 1 144-144 143.93 143.93 0 0 1-144 144zm0-240a95.31 95.31 0 0 0-25.31 3.79 47.85 47.85 0 0 1-66.9 66.9A95.78 95.78 0 1 0 288 160z">
                                    </path>
                                </svg>

                                <svg @click="show = !show" :class="{'block': !show, 'hidden':show }"
                                    class="h-4 text-purple-700" fill="none" xmlns="http://www.w3.org/2000/svg"
                                    viewbox="0 0 640 512">
                                    <path fill="currentColor"
                                        d="M320 400c-75.85 0-137.25-58.71-142.9-133.11L72.2 185.82c-13.79 17.3-26.48 35.59-36.72 55.59a32.35 32.35 0 0 0 0 29.19C89.71 376.41 197.07 448 320 448c26.91 0 52.87-4 77.89-10.46L346 397.39a144.13 144.13 0 0 1-26 2.61zm313.82 58.1l-110.55-85.44a331.25 331.25 0 0 0 81.25-102.07 32.35 32.35 0 0 0 0-29.19C550.29 135.59 442.93 64 320 64a308.15 308.15 0 0 0-147.32 37.7L45.46 3.37A16 16 0 0 0 23 6.18L3.37 31.45A16 16 0 0 0 6.18 53.9l588.36 454.73a16 16 0 0 0 22.46-2.81l19.64-25.27a16 16 0 0 0-2.82-22.45zm-183.72-142l-39.3-30.38A94.75 94.75 0 0 0 416 256a94.76 94.76 0 0 0-121.31-92.21A47.65 47.65 0 0 1 304 192a46.64 46.64 0 0 1-1.54 10l-73.61-56.89A142.31 142.31 0 0 1 320 112a143.92 143.92 0 0 1 144 144c0 21.63-5.29 41.79-13.9 60.11z">
                                    </path>
                                </svg>

                            </div>
                        </div>
                    </form>

                        <div class="flex items-center justify-between">

                            <div class="text-sm ml-auto">
                                <a href="#" class="text-purple-700 hover:text-purple-600">
                                    Forgot your password?
                                </a>
                            </div>
                        </div>
                        <div>
                            <button type="submit" class="w-full flex justify-center bg-purple-800  hover:bg-purple-700 text-gray-100 p-3  rounded-lg tracking-wide font-semibold  cursor-pointer transition ease-in duration-500">
                    Sign in
                </button>
                        </div>
                        <div class="flex items-center justify-center space-x-2 my-3">
                            <span class="h-px w-16 bg-gray-100"></span>
                            <span class="text-gray-300 font-normal">or</span>
                            <span class="h-px w-16 bg-gray-100"></span>
                        </div>
                        <div class="flex justify-center gap-5 w-full ">

                            <button type="submit" class="w-full flex items-center justify-center mb-6 md:mb-0 border border-gray-300 hover:border-gray-900 hover:bg-gray-900 text-sm text-gray-500 p-3  rounded-lg tracking-wide font-medium  cursor-pointer transition ease-in duration-500">
        <svg  class="w-4 mr-2" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path fill="#EA4335" d="M5.266 9.765A7.077 7.077 0 0 1 12 4.909c1.69 0 3.218.6 4.418 1.582L19.91 3C17.782 1.145 15.055 0 12 0 7.27 0 3.198 2.698 1.24 6.65l4.026 3.115Z"/><path fill="#34A853" d="M16.04 18.013c-1.09.703-2.474 1.078-4.04 1.078a7.077 7.077 0 0 1-6.723-4.823l-4.04 3.067A11.965 11.965 0 0 0 12 24c2.933 0 5.735-1.043 7.834-3l-3.793-2.987Z"/><path fill="#4A90E2" d="M19.834 21c2.195-2.048 3.62-5.096 3.62-9 0-.71-.109-1.473-.272-2.182H12v4.637h6.436c-.317 1.559-1.17 2.766-2.395 3.558L19.834 21Z"/><path fill="#FBBC05" d="M5.277 14.268A7.12 7.12 0 0 1 4.909 12c0-.782.125-1.533.357-2.235L1.24 6.65A11.934 11.934 0 0 0 0 12c0 1.92.445 3.73 1.237 5.335l4.04-3.067Z"/></svg>
                    <!-- <svg class="w-4" fill="#fff" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path d="M11.99 13.9v-3.72h9.36c.14.63.25 1.22.25 2.05 0 5.71-3.83 9.77-9.6 9.77-5.52 0-10-4.48-10-10S6.48 2 12 2c2.7 0 4.96.99 6.69 2.61l-2.84 2.76c-.72-.68-1.98-1.48-3.85-1.48-3.31 0-6.01 2.75-6.01 6.12s2.7 6.12 6.01 6.12c3.83 0 5.24-2.65 5.5-4.22h-5.51v-.01Z"></path></svg> -->
            

                            <button type="submit" class="w-full flex items-center justify-center mb-6 md:mb-0 border border-gray-300 hover:border-gray-900 hover:bg-gray-900 text-sm text-gray-500 p-3  rounded-lg tracking-wide font-medium  cursor-pointer transition ease-in duration-500 px-">
                    <svg class="w-4 mr-2" viewBox="0 0 100 100" style="enable-background:new 0 0 100 100" xml:space="preserve" xmlns="http://www.w3.org/2000/svg"><style>.st0{fill:#fff}.st1{fill:#f5bb41}.st2{fill:#2167d1}.st3{fill:#3d84f3}.st4{fill:#4ca853}.st5{fill:#398039}.st6{fill:#d74f3f}.st7{fill:#d43c89}.st8{fill:#b2005f}.st9{stroke:#000}.st10,.st11,.st9{fill:none;stroke-width:3;stroke-linecap:round;stroke-linejoin:round;stroke-miterlimit:10}.st10{fill-rule:evenodd;clip-rule:evenodd;stroke:#000}.st11{stroke:#040404}.st11,.st12,.st13{fill-rule:evenodd;clip-rule:evenodd}.st13{fill:#040404}.st14{fill:url(#SVGID_1_)}.st15{fill:url(#SVGID_2_)}.st16{fill:url(#SVGID_3_)}.st17{fill:url(#SVGID_4_)}.st18{fill:url(#SVGID_5_)}.st19{fill:url(#SVGID_6_)}.st20{fill:url(#SVGID_7_)}.st21{fill:url(#SVGID_8_)}.st22{fill:url(#SVGID_9_)}.st23{fill:url(#SVGID_10_)}.st24{fill:url(#SVGID_11_)}.st25{fill:url(#SVGID_12_)}.st26{fill:url(#SVGID_13_)}.st27{fill:url(#SVGID_14_)}.st28{fill:url(#SVGID_15_)}.st29{fill:url(#SVGID_16_)}.st30{fill:url(#SVGID_17_)}.st31{fill:url(#SVGID_18_)}.st32{fill:url(#SVGID_19_)}.st33{fill:url(#SVGID_20_)}.st34{fill:url(#SVGID_21_)}.st35{fill:url(#SVGID_22_)}.st36{fill:url(#SVGID_23_)}.st37{fill:url(#SVGID_24_)}.st38{fill:url(#SVGID_25_)}.st39{fill:url(#SVGID_26_)}.st40{fill:url(#SVGID_27_)}.st41{fill:url(#SVGID_28_)}.st42{fill:url(#SVGID_29_)}.st43{fill:url(#SVGID_30_)}.st44{fill:url(#SVGID_31_)}.st45{fill:url(#SVGID_32_)}.st46{fill:url(#SVGID_33_)}.st47{fill:url(#SVGID_34_)}.st48{fill:url(#SVGID_35_)}.st49{fill:url(#SVGID_36_)}.st50{fill:url(#SVGID_37_)}.st51{fill:url(#SVGID_38_)}.st52{fill:url(#SVGID_39_)}.st53{fill:url(#SVGID_40_)}.st54{fill:url(#SVGID_41_)}.st55{fill:url(#SVGID_42_)}.st56{fill:url(#SVGID_43_)}.st57{fill:url(#SVGID_44_)}.st58{fill:url(#SVGID_45_)}.st59{fill:#040404}.st60{fill:url(#SVGID_46_)}.st61{fill:url(#SVGID_47_)}.st62{fill:url(#SVGID_48_)}.st63{fill:url(#SVGID_49_)}.st64{fill:url(#SVGID_50_)}.st65{fill:url(#SVGID_51_)}.st66{fill:url(#SVGID_52_)}.st67{fill:url(#SVGID_53_)}.st68{fill:url(#SVGID_54_)}.st69{fill:url(#SVGID_55_)}.st70{fill:url(#SVGID_56_)}.st71{fill:url(#SVGID_57_)}.st72{fill:url(#SVGID_58_)}.st73{fill:url(#SVGID_59_)}.st74{fill:url(#SVGID_60_)}.st75{fill:url(#SVGID_61_)}.st76{fill:url(#SVGID_62_)}.st77,.st78{fill:none;stroke-miterlimit:10}.st77{stroke:#000;stroke-width:3}.st78{stroke:#fff}.st79{fill:#4bc9ff}.st80{fill:#50d}.st81{fill:#ff3a00}.st82{fill:#e6162d}.st84{fill:#f93}.st85{fill:#b92b27}.st86{fill:#00aced}.st87{fill:#bd2125}.st89{fill:#6665d2}.st90{fill:#ce3056}.st91{fill:#5bb381}.st92{fill:#61c3ec}.st93{fill:#e4b34b}.st94{fill:#181ef2}.st95{fill:red}.st96{fill:#fe466c}.st97{fill:#fa4778}.st98{fill:#f70}.st99{fill-rule:evenodd;clip-rule:evenodd;fill:#1f6bf6}.st100{fill:#520094}.st101{fill:#4477e8}.st102{fill:#3d1d1c}.st103{fill:#ffe812}.st104{fill:#344356}.st105{fill:#00cc76}.st106{fill-rule:evenodd;clip-rule:evenodd;fill:#345e90}.st107{fill:#1f65d8}.st108{fill:#eb3587}.st109{fill-rule:evenodd;clip-rule:evenodd;fill:#603a88}.st110{fill:#e3ce99}.st111{fill:#783af9}.st112{fill:#ff515e}.st113{fill:#ff4906}.st114{fill:#503227}.st115{fill:#4c7bd9}.st116{fill:#69c9d0}.st117{fill:#1b92d1}.st118{fill:#eb4f4a}.st119{fill:#513728}.st120{fill:#f60}.st121{fill-rule:evenodd;clip-rule:evenodd;fill:#b61438}.st122{fill:#fffc00}.st123{fill:#141414}.st124{fill:#94d137}.st125,.st126{fill-rule:evenodd;clip-rule:evenodd;fill:#f1f1f1}.st126{fill:#66e066}.st127{fill:#2d8cff}.st128{fill:#f1a300}.st129{fill:#4ba2f2}.st130{fill:#1a5099}.st131{fill:#ee6060}.st132{fill-rule:evenodd;clip-rule:evenodd;fill:#f48120}.st133{fill:#222}.st134{fill:url(#SVGID_63_)}.st135{fill:#0077b5}.st136{fill:#fc0}.st137{fill:#eb3352}.st138{fill:#f9d265}.st139{fill:#f5b955}.st140{fill:#dd2a7b}.st141{fill:#66e066}.st142{fill:#eb4e00}.st143{fill:#ffc794}.st144{fill:#b5332a}.st145{fill:#4e85eb}.st146{fill:#58a45c}.st147{fill:#f2bc42}.st148{fill:#d85040}.st149{fill:#464eb8}.st150{fill:#7b83eb}</style><g id="Layer_1"/><g id="Layer_2"><path d="M50 2.5c-58.892 1.725-64.898 84.363-7.46 95h14.92c57.451-10.647 51.419-93.281-7.46-95z" style="fill:#1877f2"/><path d="M57.46 64.104h11.125l2.117-13.814H57.46v-8.965c0-3.779 1.85-7.463 7.781-7.463h6.021V22.101c-12.894-2.323-28.385-1.616-28.722 17.66V50.29H30.417v13.814H42.54V97.5h14.92V64.104z" style="fill:#f1f1f1"/></g></svg>
                    <!-- <svg class="w-4" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path fill="#fff" fill-rule="evenodd" d="M9.945 22v-8.834H7V9.485h2.945V6.54c0-3.043 1.926-4.54 4.64-4.54 1.3 0 2.418.097 2.744.14v3.18h-1.883c-1.476 0-1.82.703-1.82 1.732v2.433h3.68l-.736 3.68h-2.944L13.685 22"></path></svg> -->
                <span>

                    Facebook</span>
                </button>

                        </div>
                    </div>
                    <div class="mt-7 text-center text-gray-300 text-xs">
                        <span>
                    Copyright © 2021-2023
                    <a href="https://codepen.io/uidesignhub" rel="" target="_blank" title="Codepen aji" class="text-purple-500 hover:text-purple-600 ">Kunal Patil</a></span>
                    </div>
                </div>
            </div>
        </div>
        </div>

    </div>
    <svg class="absolute bottom-0 left-0 " xmlns="http://www.w3.org/2000/svg" viewBox="0 0 1440 320"><path fill="#fff" fill-opacity="1" d="M0,0L40,42.7C80,85,160,171,240,197.3C320,224,400,192,480,154.7C560,117,640,75,720,74.7C800,75,880,117,960,154.7C1040,192,1120,224,1200,213.3C1280,203,1360,149,1400,122.7L1440,96L1440,320L1400,320C1360,320,1280,320,1200,320C1120,320,1040,320,960,320C880,320,800,320,720,320C640,320,560,320,480,320C400,320,320,320,240,320C160,320,80,320,40,320L0,320Z"></path></svg>
    <script src="https://cdn.jsdelivr.net/gh/alpinejs/alpine@v2.x.x/dist/alpine.js"></script>

    {% endblock content %}



    // templates/accounts/register.html

    {% extends 'base.html' %}
    {% block content %}

    <div
        class="bg-purple-900 absolute top-0 left-0 bg-gradient-to-b from-gray-900 via-gray-900 to-purple-800 bottom-0 leading-5 h-full w-full overflow-hidden">
        
    </div>
    <div
        class="relative   min-h-screen  sm:flex sm:flex-row  justify-center bg-transparent rounded-3xl shadow-xl">
        <div class="flex-col flex  self-center lg:px-14 sm:max-w-4xl xl:max-w-md  z-10">
            <div class="self-start hidden lg:flex flex-col  text-gray-300">
                
                <h1 class="my-3 font-semibold text-4xl">Welcome</h1>
                <p class="pr-3 text-sm opacity-75">Welcome to Crime Scene Reporting Portal</p>
            </div>
        </div>
        <div class="flex justify-center self-center  z-10">
            <div class="p-12 bg-white mx-auto rounded-3xl w-96 ">
                <div class="mb-2">
                    <h3 class="font-semibold text-2xl text-gray-800">Sign Up </h3>
                    <p class="text-gray-400">have an account? <a href="{% url 'login' %}"
                            class="text-sm text-purple-700 hover:text-purple-700">Sign in</a></p>
                </div>

            <form class="text-center" action="{% url 'register' %}" method="POST">
                {% csrf_token %}
                <div class="grid grid-cols-2 gap-4">
                    <div>
                        <input name="f_name"  class=" w-full text-sm  px-4 py-3 bg-gray-200 focus:bg-gray-100 border  border-gray-200 rounded-lg focus:outline-none focus:border-purple-400" type="text" placeholder="First Name">
                    </div>
                    <div>
                        <input name="l_name" class=" w-full text-sm  px-4 py-3 bg-gray-200 focus:bg-gray-100 border  border-gray-200 rounded-lg focus:outline-none focus:border-purple-400" type="text" placeholder="Last Name">
                    </div>
                </div>
                <br>

                <div class="grid grid-cols-1 gap-4">
                    <div>
                        <input name="email" class=" w-full text-sm  px-4 py-3 bg-gray-200 focus:bg-gray-100 border  border-gray-200 rounded-lg focus:outline-none focus:border-purple-400" type="text" placeholder="Email">
                    </div>
                </div>
                <br>

                <div class="space-y-3">
                    <div class="relative" x-data="{ show: true }">
                        <input name="password" placeholder="Password" :type="show ? 'password' : 'text'" class="text-sm text-gray-200 px-4 py-3 rounded-lg w-full bg-gray-200 focus:bg-gray-100 border border-gray-200 focus:outline-none focus:border-purple-400">
                        <div class="flex items-center absolute inset-y-0 right-0 mr-3  text-sm leading-5">

                            <svg @click="show = !show" :class="{'hidden': !show, 'block':show }"
                                class="h-4 text-purple-700" fill="none" xmlns="http://www.w3.org/2000/svg"
                                viewbox="0 0 576 512">
                                <path fill="currentColor"
                                    d="M572.52 241.4C518.29 135.59 410.93 64 288 64S57.68 135.64 3.48 241.41a32.35 32.35 0 0 0 0 29.19C57.71 376.41 165.07 448 288 448s230.32-71.64 284.52-177.41a32.35 32.35 0 0 0 0-29.19zM288 400a144 144 0 1 1 144-144 143.93 143.93 0 0 1-144 144zm0-240a95.31 95.31 0 0 0-25.31 3.79 47.85 47.85 0 0 1-66.9 66.9A95.78 95.78 0 1 0 288 160z">
                                </path>
                            </svg>

                            <svg @click="show = !show" :class="{'block': !show, 'hidden':show }"
                                class="h-4 text-purple-700" fill="none" xmlns="http://www.w3.org/2000/svg"
                                viewbox="0 0 640 512">
                                <path fill="currentColor"
                                    d="M320 400c-75.85 0-137.25-58.71-142.9-133.11L72.2 185.82c-13.79 17.3-26.48 35.59-36.72 55.59a32.35 32.35 0 0 0 0 29.19C89.71 376.41 197.07 448 320 448c26.91 0 52.87-4 77.89-10.46L346 397.39a144.13 144.13 0 0 1-26 2.61zm313.82 58.1l-110.55-85.44a331.25 331.25 0 0 0 81.25-102.07 32.35 32.35 0 0 0 0-29.19C550.29 135.59 442.93 64 320 64a308.15 308.15 0 0 0-147.32 37.7L45.46 3.37A16 16 0 0 0 23 6.18L3.37 31.45A16 16 0 0 0 6.18 53.9l588.36 454.73a16 16 0 0 0 22.46-2.81l19.64-25.27a16 16 0 0 0-2.82-22.45zm-183.72-142l-39.3-30.38A94.75 94.75 0 0 0 416 256a94.76 94.76 0 0 0-121.31-92.21A47.65 47.65 0 0 1 304 192a46.64 46.64 0 0 1-1.54 10l-73.61-56.89A142.31 142.31 0 0 1 320 112a143.92 143.92 0 0 1 144 144c0 21.63-5.29 41.79-13.9 60.11z">
                                </path>
                            </svg>

                        </div>
                    </div>

                    <div class="relative" x-data="{ show: true }">
                        <input name="confirm_password" placeholder="Confirm Password" :type="show ? 'password' : 'text'" class="text-sm text-gray-200 px-4 py-3 rounded-lg w-full bg-gray-200 focus:bg-gray-100 border border-gray-200 focus:outline-none focus:border-purple-400">
                        <div class="flex items-center absolute inset-y-0 right-0 mr-3  text-sm leading-5">

                            <svg @click="show = !show" :class="{'hidden': !show, 'block':show }"
                                class="h-4 text-purple-700" fill="none" xmlns="http://www.w3.org/2000/svg"
                                viewbox="0 0 576 512">
                                <path fill="currentColor"
                                    d="M572.52 241.4C518.29 135.59 410.93 64 288 64S57.68 135.64 3.48 241.41a32.35 32.35 0 0 0 0 29.19C57.71 376.41 165.07 448 288 448s230.32-71.64 284.52-177.41a32.35 32.35 0 0 0 0-29.19zM288 400a144 144 0 1 1 144-144 143.93 143.93 0 0 1-144 144zm0-240a95.31 95.31 0 0 0-25.31 3.79 47.85 47.85 0 0 1-66.9 66.9A95.78 95.78 0 1 0 288 160z">
                                </path>
                            </svg>

                            <svg @click="show = !show" :class="{'block': !show, 'hidden':show }"
                                class="h-4 text-purple-700" fill="none" xmlns="http://www.w3.org/2000/svg"
                                viewbox="0 0 640 512">
                                <path fill="currentColor"
                                    d="M320 400c-75.85 0-137.25-58.71-142.9-133.11L72.2 185.82c-13.79 17.3-26.48 35.59-36.72 55.59a32.35 32.35 0 0 0 0 29.19C89.71 376.41 197.07 448 320 448c26.91 0 52.87-4 77.89-10.46L346 397.39a144.13 144.13 0 0 1-26 2.61zm313.82 58.1l-110.55-85.44a331.25 331.25 0 0 0 81.25-102.07 32.35 32.35 0 0 0 0-29.19C550.29 135.59 442.93 64 320 64a308.15 308.15 0 0 0-147.32 37.7L45.46 3.37A16 16 0 0 0 23 6.18L3.37 31.45A16 16 0 0 0 6.18 53.9l588.36 454.73a16 16 0 0 0 22.46-2.81l19.64-25.27a16 16 0 0 0-2.82-22.45zm-183.72-142l-39.3-30.38A94.75 94.75 0 0 0 416 256a94.76 94.76 0 0 0-121.31-92.21A47.65 47.65 0 0 1 304 192a46.64 46.64 0 0 1-1.54 10l-73.61-56.89A142.31 142.31 0 0 1 320 112a143.92 143.92 0 0 1 144 144c0 21.63-5.29 41.79-13.9 60.11z">
                                </path>
                            </svg>

                        </div>
                    </div>
            <form>

                        <div class="flex items-center justify-between">

                            <div class="text-sm ml-auto">
                                <a href="#" class="text-purple-700 hover:text-purple-600">
                                    Forgot your password?
                                </a>
                            </div>
                        </div>
                        <div>
                            <button type="submit" class="w-full flex justify-center bg-purple-800  hover:bg-purple-700 text-gray-100 p-3  rounded-lg tracking-wide font-semibold  cursor-pointer transition ease-in duration-500">
                    Sign Up
                </button>
                        </div>
                        <div class="flex items-center justify-center space-x-2 my-3">
                            <span class="h-px w-16 bg-gray-100"></span>
                            <span class="text-gray-300 font-normal">or</span>
                            <span class="h-px w-16 bg-gray-100"></span>
                        </div>
                        <div class="flex justify-center gap-5 w-full ">

                            <button type="submit" class="w-full flex items-center justify-center mb-6 md:mb-0 border border-gray-300 hover:border-gray-900 hover:bg-gray-900 text-sm text-gray-500 p-3  rounded-lg tracking-wide font-medium  cursor-pointer transition ease-in duration-500">
        <svg  class="w-4 mr-2" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path fill="#EA4335" d="M5.266 9.765A7.077 7.077 0 0 1 12 4.909c1.69 0 3.218.6 4.418 1.582L19.91 3C17.782 1.145 15.055 0 12 0 7.27 0 3.198 2.698 1.24 6.65l4.026 3.115Z"/><path fill="#34A853" d="M16.04 18.013c-1.09.703-2.474 1.078-4.04 1.078a7.077 7.077 0 0 1-6.723-4.823l-4.04 3.067A11.965 11.965 0 0 0 12 24c2.933 0 5.735-1.043 7.834-3l-3.793-2.987Z"/><path fill="#4A90E2" d="M19.834 21c2.195-2.048 3.62-5.096 3.62-9 0-.71-.109-1.473-.272-2.182H12v4.637h6.436c-.317 1.559-1.17 2.766-2.395 3.558L19.834 21Z"/><path fill="#FBBC05" d="M5.277 14.268A7.12 7.12 0 0 1 4.909 12c0-.782.125-1.533.357-2.235L1.24 6.65A11.934 11.934 0 0 0 0 12c0 1.92.445 3.73 1.237 5.335l4.04-3.067Z"/></svg>
                    <!-- <svg class="w-4" fill="#fff" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path d="M11.99 13.9v-3.72h9.36c.14.63.25 1.22.25 2.05 0 5.71-3.83 9.77-9.6 9.77-5.52 0-10-4.48-10-10S6.48 2 12 2c2.7 0 4.96.99 6.69 2.61l-2.84 2.76c-.72-.68-1.98-1.48-3.85-1.48-3.31 0-6.01 2.75-6.01 6.12s2.7 6.12 6.01 6.12c3.83 0 5.24-2.65 5.5-4.22h-5.51v-.01Z"></path></svg> -->
                <span>Google</span>
                </button>

                            <button type="submit" class="w-full flex items-center justify-center mb-6 md:mb-0 border border-gray-300 hover:border-gray-900 hover:bg-gray-900 text-sm text-gray-500 p-3  rounded-lg tracking-wide font-medium  cursor-pointer transition ease-in duration-500 px-">
                    <svg class="w-4 mr-2" viewBox="0 0 100 100" style="enable-background:new 0 0 100 100" xml:space="preserve" xmlns="http://www.w3.org/2000/svg"><style>.st0{fill:#fff}.st1{fill:#f5bb41}.st2{fill:#2167d1}.st3{fill:#3d84f3}.st4{fill:#4ca853}.st5{fill:#398039}.st6{fill:#d74f3f}.st7{fill:#d43c89}.st8{fill:#b2005f}.st9{stroke:#000}.st10,.st11,.st9{fill:none;stroke-width:3;stroke-linecap:round;stroke-linejoin:round;stroke-miterlimit:10}.st10{fill-rule:evenodd;clip-rule:evenodd;stroke:#000}.st11{stroke:#040404}.st11,.st12,.st13{fill-rule:evenodd;clip-rule:evenodd}.st13{fill:#040404}.st14{fill:url(#SVGID_1_)}.st15{fill:url(#SVGID_2_)}.st16{fill:url(#SVGID_3_)}.st17{fill:url(#SVGID_4_)}.st18{fill:url(#SVGID_5_)}.st19{fill:url(#SVGID_6_)}.st20{fill:url(#SVGID_7_)}.st21{fill:url(#SVGID_8_)}.st22{fill:url(#SVGID_9_)}.st23{fill:url(#SVGID_10_)}.st24{fill:url(#SVGID_11_)}.st25{fill:url(#SVGID_12_)}.st26{fill:url(#SVGID_13_)}.st27{fill:url(#SVGID_14_)}.st28{fill:url(#SVGID_15_)}.st29{fill:url(#SVGID_16_)}.st30{fill:url(#SVGID_17_)}.st31{fill:url(#SVGID_18_)}.st32{fill:url(#SVGID_19_)}.st33{fill:url(#SVGID_20_)}.st34{fill:url(#SVGID_21_)}.st35{fill:url(#SVGID_22_)}.st36{fill:url(#SVGID_23_)}.st37{fill:url(#SVGID_24_)}.st38{fill:url(#SVGID_25_)}.st39{fill:url(#SVGID_26_)}.st40{fill:url(#SVGID_27_)}.st41{fill:url(#SVGID_28_)}.st42{fill:url(#SVGID_29_)}.st43{fill:url(#SVGID_30_)}.st44{fill:url(#SVGID_31_)}.st45{fill:url(#SVGID_32_)}.st46{fill:url(#SVGID_33_)}.st47{fill:url(#SVGID_34_)}.st48{fill:url(#SVGID_35_)}.st49{fill:url(#SVGID_36_)}.st50{fill:url(#SVGID_37_)}.st51{fill:url(#SVGID_38_)}.st52{fill:url(#SVGID_39_)}.st53{fill:url(#SVGID_40_)}.st54{fill:url(#SVGID_41_)}.st55{fill:url(#SVGID_42_)}.st56{fill:url(#SVGID_43_)}.st57{fill:url(#SVGID_44_)}.st58{fill:url(#SVGID_45_)}.st59{fill:#040404}.st60{fill:url(#SVGID_46_)}.st61{fill:url(#SVGID_47_)}.st62{fill:url(#SVGID_48_)}.st63{fill:url(#SVGID_49_)}.st64{fill:url(#SVGID_50_)}.st65{fill:url(#SVGID_51_)}.st66{fill:url(#SVGID_52_)}.st67{fill:url(#SVGID_53_)}.st68{fill:url(#SVGID_54_)}.st69{fill:url(#SVGID_55_)}.st70{fill:url(#SVGID_56_)}.st71{fill:url(#SVGID_57_)}.st72{fill:url(#SVGID_58_)}.st73{fill:url(#SVGID_59_)}.st74{fill:url(#SVGID_60_)}.st75{fill:url(#SVGID_61_)}.st76{fill:url(#SVGID_62_)}.st77,.st78{fill:none;stroke-miterlimit:10}.st77{stroke:#000;stroke-width:3}.st78{stroke:#fff}.st79{fill:#4bc9ff}.st80{fill:#50d}.st81{fill:#ff3a00}.st82{fill:#e6162d}.st84{fill:#f93}.st85{fill:#b92b27}.st86{fill:#00aced}.st87{fill:#bd2125}.st89{fill:#6665d2}.st90{fill:#ce3056}.st91{fill:#5bb381}.st92{fill:#61c3ec}.st93{fill:#e4b34b}.st94{fill:#181ef2}.st95{fill:red}.st96{fill:#fe466c}.st97{fill:#fa4778}.st98{fill:#f70}.st99{fill-rule:evenodd;clip-rule:evenodd;fill:#1f6bf6}.st100{fill:#520094}.st101{fill:#4477e8}.st102{fill:#3d1d1c}.st103{fill:#ffe812}.st104{fill:#344356}.st105{fill:#00cc76}.st106{fill-rule:evenodd;clip-rule:evenodd;fill:#345e90}.st107{fill:#1f65d8}.st108{fill:#eb3587}.st109{fill-rule:evenodd;clip-rule:evenodd;fill:#603a88}.st110{fill:#e3ce99}.st111{fill:#783af9}.st112{fill:#ff515e}.st113{fill:#ff4906}.st114{fill:#503227}.st115{fill:#4c7bd9}.st116{fill:#69c9d0}.st117{fill:#1b92d1}.st118{fill:#eb4f4a}.st119{fill:#513728}.st120{fill:#f60}.st121{fill-rule:evenodd;clip-rule:evenodd;fill:#b61438}.st122{fill:#fffc00}.st123{fill:#141414}.st124{fill:#94d137}.st125,.st126{fill-rule:evenodd;clip-rule:evenodd;fill:#f1f1f1}.st126{fill:#66e066}.st127{fill:#2d8cff}.st128{fill:#f1a300}.st129{fill:#4ba2f2}.st130{fill:#1a5099}.st131{fill:#ee6060}.st132{fill-rule:evenodd;clip-rule:evenodd;fill:#f48120}.st133{fill:#222}.st134{fill:url(#SVGID_63_)}.st135{fill:#0077b5}.st136{fill:#fc0}.st137{fill:#eb3352}.st138{fill:#f9d265}.st139{fill:#f5b955}.st140{fill:#dd2a7b}.st141{fill:#66e066}.st142{fill:#eb4e00}.st143{fill:#ffc794}.st144{fill:#b5332a}.st145{fill:#4e85eb}.st146{fill:#58a45c}.st147{fill:#f2bc42}.st148{fill:#d85040}.st149{fill:#464eb8}.st150{fill:#7b83eb}</style><g id="Layer_1"/><g id="Layer_2"><path d="M50 2.5c-58.892 1.725-64.898 84.363-7.46 95h14.92c57.451-10.647 51.419-93.281-7.46-95z" style="fill:#1877f2"/><path d="M57.46 64.104h11.125l2.117-13.814H57.46v-8.965c0-3.779 1.85-7.463 7.781-7.463h6.021V22.101c-12.894-2.323-28.385-1.616-28.722 17.66V50.29H30.417v13.814H42.54V97.5h14.92V64.104z" style="fill:#f1f1f1"/></g></svg>
                    <!-- <svg class="w-4" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path fill="#fff" fill-rule="evenodd" d="M9.945 22v-8.834H7V9.485h2.945V6.54c0-3.043 1.926-4.54 4.64-4.54 1.3 0 2.418.097 2.744.14v3.18h-1.883c-1.476 0-1.82.703-1.82 1.732v2.433h3.68l-.736 3.68h-2.944L13.685 22"></path></svg> -->
                <span>Facebook</span>
                </button>

                        </div>
                    </div>
                    <div class="mt-7 text-center text-gray-300 text-xs">
                        <span>
                    Copyright © 2021-2023
                    <a href="https://codepen.io/uidesignhub" rel="" target="_blank" title="Codepen aji" class="text-purple-500 hover:text-purple-600 ">Kunal Patil</a></span>
                    </div>
                </div>
            </div>
        </div>
        </div>


    <svg class="absolute bottom-0 left-0 " xmlns="http://www.w3.org/2000/svg" viewBox="0 0 1440 320"><path fill="#fff" fill-opacity="1" d="M0,0L40,42.7C80,85,160,171,240,197.3C320,224,400,192,480,154.7C560,117,640,75,720,74.7C800,75,880,117,960,154.7C1040,192,1120,224,1200,213.3C1280,203,1360,149,1400,122.7L1440,96L1440,320L1400,320C1360,320,1280,320,1200,320C1120,320,1040,320,960,320C880,320,800,320,720,320C640,320,560,320,480,320C400,320,320,320,240,320C160,320,80,320,40,320L0,320Z"></path></svg>
    <script src="https://cdn.jsdelivr.net/gh/alpinejs/alpine@v2.x.x/dist/alpine.js"></script>

    {% endblock content %}



    // templates/report.html

    {% extends 'base.html' %}

    {% block content %}


    <form action="{% url 'report_crime' %}" method="POST" enctype="multipart/form-data">
        {% csrf_token %}
    <div class="mt-10 sm:mt-0">
        <div class="md:grid md:grid-cols-3 md:gap-6">
        <div class="md:col-span-1">
            <div class="px-4 sm:px-0">
            <h3 class="text-lg font-medium leading-6 text-gray-900">Report a crime scene or give any tip off</h3>
            <p class="mt-1 text-sm text-gray-600">Use a permanent address where you can receive mail.</p>
            </div>
        </div>
        <div class="mt-5 md:col-span-2 md:mt-0">
            <form action="#" method="POST">
            <div class="overflow-hidden shadow sm:rounded-md">
                <div class="bg-white px-4 py-5 sm:p-6">
                <div class="grid grid-cols-6 gap-6">
                    <div class="col-span-6 sm:col-span-3">
                    <label for="first-name" class="block text-sm font-medium text-gray-700">State</label>
                    <input type="text" name="state" id="first-name" autocomplete="given-name" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm">
                    </div>
    
                    <div class="col-span-6 sm:col-span-3">
                        <label for="country" class="block text-sm font-medium text-gray-700">Country</label>
                        <select id="country" name="country" autocomplete="country-name" class="mt-1 block w-full rounded-md border border-gray-300 bg-white py-2 px-3 shadow-sm focus:border-indigo-500 focus:outline-none focus:ring-indigo-500 sm:text-sm">
                        <option>United States</option>
                        <option>India</option>
                        <option>Japan</option>
                        </select>
                    </div>

                    <div class="col-span-6 sm:col-span-3 lg:col-span-2">
                        <label for="postal-code" class="block text-sm font-medium text-gray-700">ZIP / Postal code</label>
                        <input type="text" name="zip_code" id="postal-code" autocomplete="postal-code" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm">
                    </div>

                    <div class="col-span-6">
                        <label for="street-address" class="block text-sm font-medium text-gray-700">Street address</label>
                        <input type="text" name="street_address" id="street-address" autocomplete="street-address" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm">
                    </div>
    
                    <div class="col-span-6 sm:col-span-4">
                        <label for="email-address" class="block text-sm font-medium text-gray-700">Email address</label>
                        <input name="email" type="email" name="email-address" id="email-address" autocomplete="email" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm">
                    </div>
                    
                    <div class="col-span-6 sm:col-span-4">
                        <label for="email-address" class="block text-sm font-medium text-gray-700">Crime </label>
                        <input name="crime" type="text"  id="email-address" autocomplete="email" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm">
                    </div>

                    <div class="col-span-6 sm:col-span-4">
                        <label for="crime-description" class="block text-sm font-medium text-gray-700">crime-description </label>
                        <input name="crime-description" type="text" id="large-input" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm">
                    </div>

                    <div class="col-span-6 sm:col-span-4">                    
                        <label class="block mb-2 text-sm font-medium text-gray-900 dark:text-gray-300" for="file_input">Upload file</label>
                        <input name="proof" class="block w-full text-sm text-gray-900 bg-gray-50 rounded-lg border border-gray-300 cursor-pointer dark:text-gray-400 focus:outline-none dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400" id="file_input" type="file">
                    </div>
                </div>
                <div class="bg-gray-50 px-4 py-3 text-right sm:px-6">
                <button type="submit" class="inline-flex justify-center rounded-md border border-transparent bg-indigo-600 py-2 px-4 text-sm font-medium text-white shadow-sm hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2">Submit</button>
                <a href="{% url 'dashboard' %}" class="inline-flex justify-center rounded-md border border-transparent bg-indigo-600 py-2 px-4 text-sm font-medium text-white shadow-sm hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2">Go to home</a>
                </div>
            </form>
            </div>
            </form>
        </div>
        </div>
    </div>
    
    <div class="mb-10 p-6 container md:w-2/3 xl:w-auto mx-auto flex flex-col xl:items-stretch justify-between xl:flex-row">
        <div class="xl:w-1/2 md:mb-14 xl:mb-0 relative h-auto flex items-center justify-center">
            <img src="https://cdn.tuk.dev/assets/components/26May-update/newsletter-1.png" alt="Envelope with a newsletter" role="img" class="h-full xl:w-full lg:w-1/2 w-full" />
        </div>
        <div class="w-full xl:w-1/2 xl:pl-40 xl:py-28">
            {% if user.newsletter == True %}
            <div class="flex items-stretch mt-12">
            <h2 class="text-2xl md:text-4xl xl:text-5xl font-bold leading-10 text-gray-800 mb-4 text-center xl:text-left md:mt-0 mt-4">You have already Subscribed Our Newsletter</h2>
            </div>
            {% endif %}
            {% if user.newsletter == False %}
            <form action="#" method="POST">
            <h2 class="text-2xl md:text-4xl xl:text-5xl font-bold leading-10 text-gray-800 mb-4 text-center xl:text-left md:mt-0 mt-4">Subscribed To Our Newsletter</h2>
            <div class="flex items-stretch mt-12">
            <input class="bg-gray-100 rounded-lg rounded-r-none text-base leading-none text-gray-800 p-5 w-4/5 border border-transparent focus:outline-none focus:border-gray-500" type="email" placeholder="Your Email" />
            <button class="w-32 rounded-l-none hover:bg-indigo-600 bg-indigo-700 rounded text-base font-medium leading-none text-white p-5 uppercase focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-700">subscribe</button>
            </form>
            </div>
            {% endif %}
        </div>

    {% endblock content %}



    // templates/report_anonymously.html

    {% extends 'base.html' %}

    {% block content %}


    <form action="{% url 'report_anonymously' %}" method="POST" enctype="multipart/form-data">
        {% csrf_token %}
    <div class="mt-10 sm:mt-0">
        <div class="md:grid md:grid-cols-3 md:gap-6">
        <div class="md:col-span-1">
            <div class="px-4 sm:px-0">
            <h3 class="text-lg font-medium leading-6 text-gray-900">Report Anonymously</h3>
            <p class="mt-1 text-sm text-gray-600"></p>
            </div>
        </div>
        <div class="mt-5 md:col-span-2 md:mt-0">
            <form action="#" method="POST">
            <div class="overflow-hidden shadow sm:rounded-md">
                <div class="bg-white px-4 py-5 sm:p-6">
                <div class="grid grid-cols-6 gap-6">
                    <div class="col-span-6 sm:col-span-3">
                    <label for="first-name" class="block text-sm font-medium text-gray-700">State</label>
                    <input type="text" name="state" id="first-name" autocomplete="given-name" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm">
                    </div>
    
                    <div class="col-span-6 sm:col-span-3">
                        <label for="country" class="block text-sm font-medium text-gray-700">Country</label>
                        <select id="country" name="country" autocomplete="country-name" class="mt-1 block w-full rounded-md border border-gray-300 bg-white py-2 px-3 shadow-sm focus:border-indigo-500 focus:outline-none focus:ring-indigo-500 sm:text-sm">
                        <option>United States</option>
                        <option>India</option>
                        <option>Mexico</option>
                        </select>
                    </div>

                    <div class="col-span-6 sm:col-span-3 lg:col-span-2">
                        <label for="postal-code" class="block text-sm font-medium text-gray-700">ZIP / Postal code</label>
                        <input type="text" name="zip_code" id="postal-code" autocomplete="postal-code" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm">
                    </div>

                    <div class="col-span-6">
                        <label for="street-address" class="block text-sm font-medium text-gray-700">Street address</label>
                        <input type="text" name="street_address" id="street-address" autocomplete="street-address" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm">
                    </div>
    
                    <div class="col-span-6 sm:col-span-4">
                        <label for="email-address" class="block text-sm font-medium text-gray-700">Crime </label>
                        <input name="crime" type="text"  id="email-address" autocomplete="email" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm">
                    </div>

                    <div class="col-span-6 sm:col-span-4">
                        <label for="crime-description" class="block text-sm font-medium text-gray-700">crime-description </label>
                        <input name="crime-description" type="text" id="large-input" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm">
                    </div>

                    <div class="col-span-6 sm:col-span-4">                    
                        <label class="block mb-2 text-sm font-medium text-gray-900 dark:text-gray-300" for="file_input">Upload file</label>
                        <input name="proof" class="block w-full text-sm text-gray-900 bg-gray-50 rounded-lg border border-gray-300 cursor-pointer dark:text-gray-400 focus:outline-none dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400" id="file_input" type="file">
                    </div>
                </div>
                <div class="g-recaptcha mt-5" data-sitekey="6Leg_rQiAAAAADk3TlVE4gYpbixVIpnYrCAkLOnV"></div>
                <br/>
                <input type="submit" value="">
                    
            </form>
            <div class="bg-gray-50 px-4 py-3 text-right sm:px-6">
            <button type="submit" class="inline-flex justify-center rounded-md border border-transparent bg-indigo-600 py-2 px-4 text-sm font-medium text-white shadow-sm hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2">Submit</button>
            <a href="{% url 'dashboard' %}" class="inline-flex justify-center rounded-md border border-transparent bg-indigo-600 py-2 px-4 text-sm font-medium text-white shadow-sm hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2">Go to home</a>
            </div>

            </div>
            </form>
        </div>
        </div>
    </div>
    
    {% endblock content %}



    // templates/successfully_reported.html

    {% extends 'base.html' %}

    {% block content %}


    <h2 class="text-2xl md:text-4xl xl:text-5xl font-bold leading-10 text-gray-800 mb-4 text-center xl:text-left md:mt-0 mt-4">Thanks for Reporting. After Verification We'll forward it to the respected Authorities</h2>
    <div class="bg-gray-50 px-4 py-3 text-right sm:px-6">
        <a href="{% url 'dashboard' %}" class="inline-flex justify-center rounded-md border border-transparent bg-indigo-600 py-2 px-4 text-sm font-medium text-white shadow-sm hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2">Go to home</a>
    </div>

    {% endblock content %}


    ```

6. Once you done this try to run the project and you will see all the templates. Try to register yourself
   and check if your details getting saved inside admin site by visiting the 'localhost/admin' url.
    ```
    python manage.py runserver 8000

    ```




## Conclusions
In the future, Blockchain will be implemented for security purposes. Users will be able to schedule
the tasks.

[Call to action: e.g. try building this project and tag me @shreythecray when you do!]

## About the Author
```
👋 Hi, I’m Kunal Patil
👀 I’m interested in DevOps
🌱 I’m currently learning AWS, Django
💞️ I’m looking to collaborate on New Projects where I can apply my knowledge and skills.
```

## Quick Links
[Demo](https://youtu.be/-Vlfuwni_H8)

[Documentation](https://docs-six-gamma.vercel.app)

[Github](https://github.com/Kunalp02/Crime_Reporting_Portal)

[Django Docs](https://docs.djangoproject.com/en/4.1/)


## Environment Variables

To run this project, you will need to add the following environment variables to your .env file

`Newscatcherapi`[Link](https://newscatcherapi.com/)

`Courier API`[Link](https://www.courier.com/)















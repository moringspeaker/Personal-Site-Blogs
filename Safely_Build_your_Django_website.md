# Build Your Django Website from 0

---
In this tutorial, you will learn:
- How to own and configure your server using Droplets
- Configure ssh connection on the server
- How to develop a simply django back-end application 
- How to use Vue3 to build a simple front-end project
- How to configure and use Nginx in your front-end

---
## How to own and configure your server using Droplets:
 Let's start from Digital Ocean:
![Droplet](https://i.imgur.com/8SvlTEo.png)
First, we need to create a droplet and do some neccessary configurations. Remeber to select ssh Authentication Method: 
![SSH](https://i.imgur.com/qEodSoc.png)

If you find some difficulties on creating ssh key, you can follow this tutorial:
[How to Create SSH Keys with OpenSSH on MacOS or Linux](https://docs.digitalocean.com/products/droplets/how-to/add-ssh-keys/create-with-openssh/)

After finish creating our droplets, we need to apply a reserved IP so that our website can be stably visited.

![Reserve IP](https://i.imgur.com/2T13YuR.png)

After done this, the next step is to create some firewall rules for our website:
Choose Networking on the left side under MANAGE:
![Firewall](https://i.imgur.com/5COHOG3.png)

And then go to the firewall and create a new one:
![firewall](https://i.imgur.com/o6GoJVZ.png)

We can just make some basic settings and configure a very simple set of firewall rules, but in order to avoid cumbersome subsequent configuration, we can set it fully at the first time.

Rules are consisted of Inbound rules and Outbound rules. For Inbound rules, we want to accept all possible visiting on Http and Https(You can just use Http) and our SSH connections(This part will be discussed later). 
![Inbound](https://i.imgur.com/UhHB4ue.png)

After that, we need to finish the Outbound rules:
![Outbound](https://i.imgur.com/rNSQT8g.png)

But remeber, above is just a very basic firewall rules, these rules are platform layer's security restrictions. We need to configure a more detailed set of firewall rules on our server.

---
## Configure ssh connection on the server:

If you've already uploaded your ssh key while creating your new droplet, you can just use your command line tool(Terminal on Mac and CMD/PowerShell on Windows) to remote connect to the droplet, but if you want to change it or you just accidentally set the password authentication as the default way, you may need below tutorial:

[How to Upload an SSH Public Key to an Existing Droplet](https://docs.digitalocean.com/products/droplets/how-to/add-ssh-keys/to-existing-droplet/)

If you want to have more than 1 ssh keys so that your droplet can be reached through different equipments, you can simply copy your existed and available private key to your other PCs and use 
```
ssh -i \PATH_TO_YOUR_KEY User@YOUR_IP
```
 command to specify the private key. Or you can just manually add it using:
 ```
 cd ~/.ssh/
 vi authorized_keys
 ```
 And copy your new ssh private pub key on your computer and paste it in that file on remote server.

When all steps above are done, let's leave the server aside for a moment and let's see how to build a local django application。

---
## How to develop a simply django back-end application

To build a django application, we need to install it first. So create a new virtual environment, and install django: 

`pip install django`

And then enter the directory which will contain your new project, start a django application(let say we want to create a new project named myproject and it has an app called school):

```
django-admin startproject myproject
cd myproject
python manage.py startapp school
```

Then we need to add it to our installed app list in `./myproject/settings.py`:

```python
INSTALLED_APPS = [
    # ...
    'school',
    # ...
]
```

Then we create the model of students:
![StudentsModel](https://i.imgur.com/6qrXSiF.png)

Because we're planning to build a seperated backend and frontend, so we just need to return the data to the frontend and there's no need to write any html files in our backend project. 

Before the next step, we need to install a new library named: `Django rest_framework(DRF)` .
```
pip install djangorestframework
```
and add this framework to out app list
```python
INSTALLED_APPS = [
    # ...
    'rest_framework',
    # ...
]
```

Then we want to create a serializer for our model. The reason why we need a serializer is because our backend program will only do things like receive data from the frontend and pass data to the backend. During this process, we want to make sure the front and back end can exchange data in some form. We need a special mechanism to put python data. For example, lists and dicts are converted into a data format that the front-end can recognize directly. This is why we need to serialize and deserialize our data. In fact, our backend is mainly doing 2 things: serialize and deserialize. That's why we encapsulate a specialized serializer to do this.

So we need to create a serializers.py under our app school's directory:
![serializers.py](https://i.imgur.com/DafzKux.png)

After that, we can create the view function (A simple function which can return all students' name), here is what our view.py looks like:
![View.py](https://i.imgur.com/WOEgPYt.png)

Add and a url pattern for this. We need to create a `urls.py ` in our app school's directory and add:
```python
from django.urls import path
from .views import StudentNamesView

urlpatterns = [
    path('names/', StudentNamesView.as_view()),
]
```

Also, you may want to add this to your `/myproject/urls.py`:
```python
from django.contrib import admin
from django.urls import path,include

urlpatterns = [
    path("admin/", admin.site.urls),
    path("school/", include("school.urls")),
]
```
We also need to add an `admin.py` to register our model to django's built-in admin panel
```python 
from django.contrib import admin
from .models import Student

admin.site.register(Student)


```

After done all above steps, it's time to test our backend!
``` 
python manage.py makemigrations

python manage.py migrate

# create a superuser
python manage.py createsuperuser

python manage.py runserver
```

We can use [Postman](https://www.postman.com/) to test our backend, or just leave it and write our frontend first.
![Postman](https://i.imgur.com/VcB8IOl.png)

Because we don't have any data, so it's reasonable to have an empty return.

We need to add some test data to our database, so we can open (https://127.0.0.1:8000/admin/) and add a student:
![student](https://i.imgur.com/5YcXyN5.png)

Now we are able to get correct data:
![result](https://i.imgur.com/n3P8gMf.png)

---

## How to use Vue3 to build a simple front-end project

First, if there's no node.js on your computer, you should install it first:

1.  Install Node.js

    You can download it from the [official Node.js website](https://nodejs.org/)
2. Install Vue CLI

    ```
    npm install -g @vue/cli
    ```
3. Create a New Vue 3 Project   
    vue create my-vue3-project

4. Choose Vue 3
    ```
        vue create --preset vuejs/vue-next my-vue3-project
    ```

5. Run the project
    After done creating, we can use command 
    ```
    npm run serve
    ``` 
    to run this project, and if everything goes well , you can see below page on http://localhost:8080/ :
    ![Page](https://i.imgur.com/ZIUGuGS.png)

6. Build our frontend page

    To keep it simple, here I'll just create a simple page which can display all student's names provided by our backend, you may need more functions and pages for your frontend.

    Let's edit our App.vue with some chatGPT codes for scripts:
    ```javascript
    import axios from 'axios';

    export default {
    name: 'App',
    data() {
        return {
        names: []
        };
    },
    mounted() {
        this.fetchNames();
    },
    methods: {
        async fetchNames() {
        try {
            const response = await axios.get('http://127.0.0.1:8000/school/names/');
            this.names = response.data;
        } catch (error) {
            console.error('An error occurred while fetching the names:', error);
        }
        }
    }
    };
    ```
    And the html template:
    ```html
    <template>
    <div>
        <img alt="Vue logo" src="./assets/logo.png">
        <h1>All Names:</h1>
        <ul>
        <li v-for="name in names" :key="name">{{ name }}</li>
        </ul>
    </div>
    </template>
    ```
    then use command:
    ``` 
    npm run serve
    ```

    to see what we get now


7. Add CORS to your backend settings:
    When you try to opem you newly built page, you may find an error there:
    ![error](https://i.imgur.com/06Vwj7K.png)
    That's because there's something called **Same-origin Policy**: Browsers implement this security measure to prevent malicious sites from making unauthorized requests to a different site. It restricts web pages from making requests to a different domain than the one that served the web page.
    To solve this, we can use **CORS (Cross-Origin Resource Sharing)** technolohy. In order to realize that, we should do some configurations at our backend.

8. Add CORS settings to our **Django** project:
    - Modify Your settings.py File:
    ```python
        INSTALLED_APPS = [
        # ...
        'corsheaders',
        # ...
        ]
    ```
    - Then, add the middleware class to the top of the MIDDLEWARE list:
    ```python
        MIDDLEWARE = [
        'corsheaders.middleware.CorsMiddleware',
        # Place this at the first 
        ]
    ```
    - Configure CORS Settings:
    ```python
        CORS_ALLOWED_ORIGINS = [
            'http://localhost:8080',
            'https://example.com',
        ]
    ```
    After these steps, we can now visit our vue3 project and open the webpage. If everythings is correct, we are supposed to see this page:
    ![correctpage](https://i.imgur.com/GYWPzMF.png)

    This means our frontend and backend can communicate with each other smoothly. 

    Our frontend's project file structure is:
    ![FrontStruct](https://i.imgur.com/o4OQ570.png)
    And our backend's file structure is:
    ![](https://i.imgur.com/S1ZrZwD.png)


    Before we talk about Nginx and Gunoiorn, you may want to upload your existed codes to github to keep it safe. Before doing that, please make sure that all your sensitive information (password and secretkey) is replaced with environment variables and your environment variables files( if it exist) are added to the .gitignore file like this:
    ```python
        SECRET_KEY = "django-insecure-o6w@a46mx..." #replace this line
    ```
    And replace it with :
    ```python
    import os

    # ...

    try:
        SECRET_KEY = os.environ["SECRET_KEY"]
    except KeyError as e:
        raise RuntimeError("Could not find a SECRET_KEY in environment") from e
    ```

    In this case you have to set all those environment variables on your server, and for the later steps, we can use github repo secrets when we use git workflow to build the docker containers.

    Or you can keep them as a file with installing:
    ```python
    pip install python-decouple
    ```
    and use this library in your settings.py:
    ```python
    from decouple import config,Csv
    ...

    SECRET_KEY = config('SECRET_KEY')
    ALLOWED_HOSTS =  config('ALLOWED_HOSTS', cast=Csv())
    ```
    And load all those secrets in your .env file, and add it to your .gitignore file.

    ---

    ## Nginx and Cunicorn

    So far we've already had a separate project for frontend and backend. They can work perfectly on local. But there are more steps needed to be done if we want to make it visitable on our server.

    So you may have a question, why we need Nginx and Gunicorn? 

    ### What is Nginx?
    ![Nginx](https://i.imgur.com/QvuSCox.jpg)
    Nginx  is a lightweight web server, reverse proxy server, and email (IMAP/POP3) proxy server. There are many advantages for using Nginx, you can check them by yourself. But in this article, we just use it in a pretty basic way. To install the nginx on your platform, you can follow the following tutorial:
  

    [Nginx Install](https://www.nginx.com/resources/wiki/start/topics/tutorials/install/)

    Once the Nginx is installed, we can test it by command:
    ```shell
    sudo nginx -t
    ```
    And if everything goes well, you should see a welocome page offered by Nginx with command:

    ```
    sudo nginx
    ```

    To make nginx useable, we need to make some changes to nginx.conf. This configuration file is loacted variously on different platforms so you may need to check its path on your own equipment.

    After making sure Nginx is successfully installed, we can open `nginx.conf` and edit it:
    ```
    server {
    listen 80; # Listen on port 80 for HTTP
    server_name http://localhost;
    return 301 https://$host$request_uri;
    access_log /var/log/nginx/chenyu-site-http-access.log;
    error_log /var/log/nginx/chenyu-site-http-error.log;

    location / {
        root /usr/share/nginx/html/dist;
        try_files $uri $uri/ /index.html;
       
        }
    }
    ```
    set the limit of connections
    ```
    worker_processes auto;

    events {
        worker_connections 1024;
    }
    ```

    After done setting, we are able to use Nginx to proxy our frontend! We should place all our static files in a certain directory and then specify the folder to be accessed by nginx. So our next step is to package all our frontend stuffs and place this under specific path which is available for Nginx. We need to add:
    ```javascript
    module.exports = {
        ...#
        assetsDir: "static",
        ...#
        }
    ```
    in the `vue.config.js`to aquire a pure static folder and make all static files(frontend's codes) inside it.
    Then we can build the entire project with command:
    ```
    npm run build
    ```
    After that, we can see there's a newly created directory named `dist` in our frontend's root directory. we move this `/dist` file to the specific location we set in our Nginx's nginx.conf file. 

    After that, we can try to restart Nginx service and visit our website's frontend successfully.




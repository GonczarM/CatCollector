<img src="https://i.imgur.com/efjnAna.jpg">

# Deploying a Django App to Heroku

## Road Map

1. Preparation
2. Ready the Django Project
3. Commit the Changes
4. Deploy to Heroku
5. Migrate the Database Migrations
6. Set Environment Variables
7. Open the Application
8. Troubleshooting
9. Create the superuser
10. Test the Admin Portal

## 1. Preparation

### `cd` Into the Project's Folder

- `cd` into the the Django project's root folder

- Open the project in VS Code: `code .`

- Open a terminal in VS Code: `ctrl + backtick`

- Make sure that the `main` branch is checked out

- Make sure you are in your virtual environment

## 2. Ready the Django Project

Django projects need to be configured to be deployed.

Django has detailed deployment [docs](https://docs.djangoproject.com/en/3.0/howto/deployment/) and a [checklist](https://docs.djangoproject.com/en/3.0/howto/deployment/checklist/), however, there is dedicated package we will use to make deploying to Heroku much easier.

> Note:  If you ever deploy a Django app used in production, you'll want to review the [checklist](https://docs.djangoproject.com/en/3.0/howto/deployment/checklist/) mentioned above and implement the additional security precautions.
### Install `django-on-heroku`

First, let's install [`django-on-heroku`](https://github.com/pkrefta/django-on-heroku) which is a Python package that will help with the deployment process:

```
$ pip install django-on-heroku
```

### Update `settings.py`

There are several changes we would have to make to `settings.py` in order to be able to deploy.

However, the `django-on-heroku` package makes the necessary changes to `settings.py` for us. All we need to do is add the following to the **very bottom** of **settings.py**:

```python
# Other settings above
# Configure Django App for Heroku.
import django_on_heroku
django_on_heroku.settings(locals())
```

> Note that the import name is `django_on_heroku` instead of `django-on-heroku` we used when installing.

### Install `gunicorn`

The built-in development server we've been running with `python3 manage.py runserver` is not suitable for deployment.

`gunicorn` is a Python HTTP Server designed to work with Linux/Unix servers such as Heroku's.

Let's install it:

```
$ pip install gunicorn
```

### Create & Configure `Procfile`

Heroku needs a file named **Procfile** to know how to run a Python app.

Let's create one - be sure to name it **exactly** as `Procfile` (capitalized and without a file extension):

```
$ touch Procfile
```

We only need to add a single line of code in **Procfile**. However, it's important to replace the `<your project name here>` with your actual project name:

```
web: gunicorn <your project name here>.wsgi
```

The project name should be the same as your project's folder name, however, you can also verify the project's name by examining this line in `settings.py`:

```python
WSGI_APPLICATION = 'catcollector.wsgi.application'
# ^^^ catcollector is the project name
```
> Note:  If you install any additional Python packages during development after your initial deployment, you will need to run `pip3 freeze > requirements.txt` again to update the **requirements.txt** after the install of the additional Python package.

We just installed two packages to get us deployed so lets go ahead and update our requirements.txt file

```
$ pip3 freeze > requirements.txt
```

## 3. Connect a Remote Database

- Navigate to [bit.io](bit.io)
- Create a free account
- Create a Database and name it
- Navigate to the Connect tab

Here is your connection strings you will supply to your Django app.

- Open the 'settings.py' file
- Match your Database connection to your bit.io values like shown below

```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': '<Database name>',
        'USER': '<Username>',
        'PASSWORD': '<API Key / Password>',
        'HOST': 'db.bit.io',
        'PORT': '5432',
    }
}
```

This will reconnect your Django app to your remote database.

## 4. Set Environment Variables

We need to set environment variables (secrets) on Heroku in the same way we needed to set our environmental variables in Unit 3.

You'll notice that the settings.py contains a warning not to leave your secret key in production so copy the value and replace the line with the following:

```
# SECURITY WARNING: keep the secret key used in production secret! 
SECRET_KEY = os.environ['SECRET_KEY']
```
Then navigate to your .env/activate file and at the very bottom of the file create your env variable.

```
# Env Variables
export SECRET_KEY='<your secret key>'
```
> Note: If you accidentally publish your secret key to GitHub you can generate a new one [here](https://djecrety.ir/).
Note the message to not run debug in production. Instead, we'll create an environment variable called MODE and set it to 'dev' in our .env/activate file locally and 'production' in the Heroku configvars. We can use a ternary operator to set the DEBUG to True or False based on the environment variable.


settings.py
```py
# SECURITY WARNING: don't run with debug turned on in production! 
# Replace the DEBUG = True with:
DEBUG = True if os.environ['MODE'] == 'dev' else False
```

.env/activate
```
# Env Variables
export SECRET_KEY='<your secret key>'
export MODE='dev'
```

Create a .gitignore file in the root of your repo and add `.env` to it. 


## 5. Commit the Changes

Now let's commit the changes made to the project (make sure that you're on the `main` branch):

```
$ git add -A
$ git commit -m "Config deployment"
```

## 6. Deploy to Heroku

The `heroku` remote was added to the repo with the `heroku create` command ran earlier.

So, deploying the first time and re-deploying later is as easy as running this command:

```
$ git push heroku main
```

Read the output during deployment carefully. You'll need to address the errors if the deployment fails.

## 7. Open the Application

Let's check it out!

```
$ heroku open
```

Since the database is new, there will not be any users or data.  After creating a new cat, celebrate! 

## 8. Troubleshooting

The following command shows Heroku's log for our app and is useful for troubleshooting.  The log also contains the output from your app's `print()` statements:

```
$ heroku logs
``` 


### Congrats!

<img src="https://i.imgur.com/efjnAna.jpg">

# Deploying a Django App to Heroku

## Road Map

1. Preparation
2. Ready the Django Project
3. Connect a Remote Database
4. Set Environment Variables
5. Commit Changes
6. Deploy to Heroku

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

There you will find the connection strings you will supply to your Django app.

- Open the `settings.py` file
- Match your Database connection to your bit.io values like shown below

```py
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

This will change the connection on your Django app to your remote database.

## 4. Set Environment Variables

We need to set environment variables (secrets) on Heroku in the same way we needed to set our environmental variables in Unit 3.

### Secret Key

You'll notice that the settings.py contains a warning not to leave your secret key in production so copy the value and replace the line with the following:

```py
# SECURITY WARNING: keep the secret key used in production secret! 
SECRET_KEY = os.environ['SECRET_KEY']
```
Then navigate to your .env/activate file and at the very bottom of the file create your env variable.

```py
# Env Variables
export SECRET_KEY='<your secret key>'
```
> Note: If you accidentally publish your secret key to GitHub you can generate a new one [here](https://djecrety.ir/).

### Debug Mode

Note the message to not run debug in production. Instead, we'll create an environment variable called MODE and set it to 'dev' in our .env/activate file locally and 'production' in the Heroku configvars. We can use a ternary operator to set the DEBUG to True or False based on the environment variable.



```py
# settings.py

# SECURITY WARNING: don't run with debug turned on in production! 
# Replace the DEBUG = True with:
DEBUG = True if os.environ['MODE'] == 'dev' else False
```


```py
# .env/activate

# Env Variables
export SECRET_KEY='<your secret key>'
export MODE='dev'
```

### Database Password

We need to limit the accessibility of our database to just this application. We can set the database password to an env variable for that security.

```py
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': '<Database name>',
        'USER': '<Username>',
        'PASSWORD': os.environ['DB_PASSWORD'],
        'HOST': 'db.bit.io',
        'PORT': '5432',
    }
}
```

Add this env variable to your .env/activate file.

Create a .gitignore file in the root of your repo and add `.env` to it. 


## 5. Commit the Changes

Now let's commit the changes made to the project (make sure that you're on the `main` branch):

```
$ git add -A
$ git commit -m "Config deployment"
$ git push origin main
```

## 6. Deploy to Heroku

### Create Heroku App

- Open up your [Heroku Dashboard](https://dashboard.heroku.com) and create a new app.
- Name your app and then create.
- After creating your application you'll be taken to the application page. From here, naviagte to your application settings.

### Config Vars

- Reveal Config Vars
- This is where you will create the key value pairs for each of your env variables. 
- DB_PASSWORD and SECRET_KEY should remain the same
- MODE will change its value to production
> Note: you do not need qoutation marks for your values on heroku

### Deploy 

Now that your Heroku application is properly configured, it's time to connect your GitHub account and deploy your app from GitHub.

1. Select the Deploy option in your application page toolbar, select GitHub as your deployment method, and then connect your GitHub account.
2. Once you've connected your GitHub account, specify which repository you'll use to deploy your application.
3. Upon selecting your repository, you can select a specific branch to deploy your app from. In this readme we specified the main branch. Enable automatic deploys so that your Heroku app updates every time the branch you selected is pushed to.
4. On first deploy of a postgres db heroku will create a PostgreSQL "mini" database add-on. We will using bit.io instead so luckily, once removed, Heroku won't add it back in subsequent deployments.
5. To remove this add-on navigate to the Resources tab and delete the PostgreSQL add-on


Your app should then deploy and build successfully!

Since the database is new, there will not be any users or data.  After creating a new cat, celebrate! 

You can view the build logs in the Activity Tab on heroku for troubleshooting.

### Congrats!

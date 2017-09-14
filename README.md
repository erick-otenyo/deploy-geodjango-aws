# deploy-geospatial
Guides on deploying Geodjango/Geospatial applications on different platforms


## Heroku

An installation guide for getting GeoDjango setup on Heroku

----------

1. Sign up for  [Heroku](https://signup.heroku.com/)

2. Download & Install [Heroku Command Line Interface](http://cli.heroku.com)

3. Open Terminal or Command Prompt (Windows users)

4. Authenticate with Heroku:
    ```
    $ heroku login
    Enter your Heroku credentials.
    Email: <your-heroku-account-email>
    Password (typing will be hidden): 
    Logged in as <your-heroku-account-email>
    ```

5. Install Python:
    [Python.org](http://www.kirr.co/52lk1y/)
    or
    Mac watch [here](http://www.kirr.co/a9u645/)
    Linux watch [here](http://www.kirr.co/yoywdh/)
    Windows watch [here](http://www.kirr.co/xeaocj/)

### Start GeoDjango Project Locally. Ensure you have the following done:
    cd ~/path/to/your/project
    pip install gunicorn dj-database-url psycopg2
    pip freeze  > requirements.txt

### Setup your GeoDjango Project on Git
1. Initialize Git in the root of your Django Project (where `manage.py` is)
    ```
    cd /path/to/your/project
    git init 
    ```

2. Create `.gitignore` file with the exact contents of [this .gitignore file](http://www.kirr.co/mbehan/). Put at the same path as where you did the `git init`


3. First Commit
    ```
    git add --all
    git commit -m "Intial Commit"
    ```


### Heroku Setup
1. Navigate to the root of your Django Project (where `manage.py` is):
    ```
    cd ~/path/to/your/project
    ```
2. Create Procfile with the contents:
    ```
    web: gunicorn projectname.wsgi --log-file -
    ```


3. Add Procfile to Git
    ```
    git add .
    git commit -m "Added Procfile"
    ```

5. Create Heroku App
    Use `heroku create <your-app-name>` or just `heroku create`, such as:
    ```
    heroku create myappname
    ```

    Successful Result should be:
    
    ```
    (env) $ heroku create myappname
    Creating ⬢ myappname... done
    https://myappname.herokuapp.com/ | https://git.heroku.com/myappname.git
    ```

6. Test Heroku Locally
    ```
    heroku local web
    ```
    Open [http://localhost:5000/](http://localhost:5000/)

7. Create `runtime.txt` file for Python Version:
    In Django Project Root (where manage.py is), create a file `runtime.txt` with the contents:
    ```
    python-2.7.13
    ```
    Add to git
    ```
    git add runtime.txt
    git commit -m "Added runtime file"
    ```
8. Now we need to tell Heroku that our application is not the normal django apps, but one that should support geospatial functionalities.

To have GIS functionalities available for use in our Django app, we need to have Geospatial libraries in our heroku instance.

In this case, what we need is a custom buildpack that will set up the gospatial environment on Heroku.
  
We will use the [heroku-geo-builpack](https://github.com/cyberdelia/heroku-geo-buildpack) for this.
  
Unfortunately, when creating the heroku app on #5, the stack used by Heroku is heroku-16, which in the time of writing this is not compatible with heroku-geo-buildpack. But the cedar-14 stack is compatible.
  
This means we will have to change our app's stack to cedar-14 using the command below:
  ```
    heroku stack:set cedar-14 -a <myappname>
  ```
This will have the app using the compatible cedar-14 stack once we push it.
  
Now we need to tell heroku to use the heroku-geo-buildpack, and ofcourse the the python buildpack:
  ```
     heroku buildpacks:set https://github.com/cyberdelia/heroku-geo-buildpack.git
     heroku buildpacks:add heroku/python
  ```



### Update Django Settings for Production:
1. Update `settings.py`:

    - Change `DEBUG = TRUE` to `DEBUG = FALSE`

    - Add `myappname.herokuapp.com` to `ALLOWED_HOSTS`:
        ```
        ALLOWED_HOSTS = ['myappname.herokuapp.com']
        ```

3. Create `heroku` Live Database:

    ```
    heroku addons:create heroku-postgresql:hobby-dev
    ```

4. Configure Live Database on `settings.py`:

    ```
    # keep this
    DATABASES = {
        'default': {
            'ENGINE': ''django.contrib.gis.db.backends.postgis'',
            'NAME':'YOUR_DB_NAME',
            'USER':'YOUR_DB_USER',
            'PASSWORD':'YOUR_DB_PASSWORD',
            'HOST':'localhost',
            
        }
    }

    # add this
    import dj_database_url
    db_from_env = dj_database_url.config()
    DATABASES['default'].update(db_from_env)
    DATABASES['default']['ENGINE'] = 'django.contrib.gis.db.backends.postgis'
    ```
5. Commit:

    ```
    git add --all
    git commit -m "Update Django for database"
    ```


### Setup Static Files:
    
1. Install whitenoise and add to `requirements.txt`:
    ```
    pip install whitenoise
    pip freeze > requirements.txt
    ```
2. Update our  Django Settings file:
    ```
    MIDDLEWARE = [
        'django.middleware.security.SecurityMiddleware',
        'whitenoise.middleware.WhiteNoiseMiddleware',
        'django.contrib.sessions.middleware.SessionMiddleware',
        'django.middleware.common.CommonMiddleware',
        'django.middleware.csrf.CsrfViewMiddleware',
        'django.contrib.auth.middleware.AuthenticationMiddleware',
        'django.contrib.messages.middleware.MessageMiddleware',
        'django.middleware.clickjacking.XFrameOptionsMiddleware',
    ]

    # Simplified static file serving.
    # https://warehouse.python.org/project/whitenoise/

    STATIC_URL = '/static/'

    STATICFILES_DIRS = (
        os.path.join(BASE_DIR, "static"),
    )

    STATIC_ROOT = os.path.join(BASE_DIR, "live-static-files", "static-root")

    STATICFILES_STORAGE = 'whitenoise.django.GzipManifestStaticFilesStorage'

4. Run `collectstatic` locally:

    ```
    python manage.py collectstatic
    ```

5. Commit:
    ```
    git add --all
    git commit -m "Update Django for whitenoise static"
    ```
    

### Push to Heroku
1. After all `git` commits, run:
    ``` 
    git push heroku master
    ```

2. Useful Django commands on Heroku:
 
    `heroku bash` opens the shell to do:
    
    ```
    python manage.py migrate
    python manage.py createsuperuser
    python manage.py shell
    ```
    
    **Shortcut commands**:
    
    `heroku run python manage.py migrate`

    `heroku run python manage.py createsuperuser`

    `heroku run python manage.py shell`

    `heroku restart`

3. When you make changes to Django on your local project:
    1. Ensure `makemigrations` is ran prior to deployment:
        ```
        python manage.py makemigrations
        ```
    2. View local changes:
        ```
        heroku local web
        ```
        open [http://localhost:5000/](http://localhost:5000/)
    
    3. Commit all changes:
    
        ```
        git add --all

        git commit -m "Update something"
        ```
    4. Push & Migrate:
    
        ```
        git push heroku master && heroku run python manage.py migrate
        ```



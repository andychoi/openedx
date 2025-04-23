

From source
-----------
1. git clone URL
2. pip install -e '.[dev]'
3. pyinstaller tutor.spec
4. ./dist/tutor --version

plugin update/install
---------------------
```
tutor plugins update   
tutor plugins install indigo mfe
tutor plugins enable indigo mfe

tutor dev run lms bash  
python manage.py cms makemigrations enterprise
```

Run tutor
---------
```
tutor local launch # for local installations
tutor dev launch   # for local development installations
tutor k8s launch   # for Kubernetes installation
```

## Common tasks
https://docs.tutor.edly.io/local.html#main-commands

### Creating a new user with staff and admin rights
You will most certainly need to create a user to administer the platform. Just run:

```tutor local do createuser --staff --superuser yourusername user@email.com```
You will be asked to set the user password interactively.

### Importing the demo course
After a fresh installation, your platform will not have a single course. To import the Open edX demo course, run:

```tutor local do importdemocourse```
### Setting a new theme
The default Open edX theme is rather bland, so Tutor makes it easy to switch to a different theme:

```tutor local do settheme mytheme```

### stop & start
```
tutor config save

git clone https://github.com/openedx/frontend-app-learning.git
cd frontend-app-learning

tutor dev stop
tutor dev launch
```
If you’re running locally with production settings:
```
tutor local stop
tutor local start
```


## Access site
```
    http://local.openedx.io:8000
    http://studio.local.openedx.io:8001
    http://meilisearch.local.openedx.io:7700
```
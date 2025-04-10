# fanfare-on-the-web

## Docker structure

Installer [Docker et Docker-Compose](https://www.docker.com/)

structure initiale :
```bash
.
├── Django
│   ├── media
│   └── static
├── Docker_setup
│   ├── docker-compose.yml
│   ├── Dockerfile
│   ├── env-template.txt
│   └── requirements.txt
├── LICENSE
└── README.md
```

La configuration de la base de donnée ce fait dans le fichier `.env` (renommer et personnaliser [`env-template.txt`](/env-template.txt)).

5 conteneur docker lancé avec [docker compose](/Docker_setup//docker-compose.yml) :
- **db** : basé sur l'image docker [postgres:15](https://hub.docker.com/_/postgres) avec un accès par le port 5432.
    - Gestion de la base de donnée
- **pgadmin** : basé sur [dpage/pgadmin4](https://hub.docker.com/r/dpage/pgadmin4) avec une interface web sur le port 5050.
    - permet d'utiliser l'interface web de pgadmin
- **web** : basé sur le [Dockerfile](/Docker_setup/Dockerfile)
- **minio** : basé sur l'image docker [minio](https://hub.docker.com/r/minio/minio) avec un accès par les ports 9000 et 9001. 
    - Gestion des média uploads par les utilisateurs et des fichiers statics (comme les logos, images et autres)
- **mc** : basé sur l'image docker [mc](https://hub.docker.com/r/minio/mc) génère un [bucket](https://code-garage.com/blog/qu-est-ce-qu-un-bucket-s3/)
    - <u>def bucker chatGPT</u> : *Un bucket est comme un dossier principal où sont stockés les fichiers (objets). Il ne contient pas de fichiers de manière "classique" comme un disque dur, mais plutôt des objets qui sont des fichiers accompagnés de métadonnées.*

### Docker generation conteneurs

Pour lancer les conteneur (à l'exception de **web** qui necessite plus d'initialisation) utiliser dans le dossier `Docker_setup` la commande :
```bash
docker-compose up db pgadmin minio mc
```

Possibiliter de supprimer les conteneur et les volumes docker ouvert avec la commande 
```bash
docker-compose down -v
```

### PostgreSQL

Une fois le conteneur crée il est possible de s'y connecter avec pgadmin. Pour cela il faut se connecté à l'adresse IP du conteneur. Cette adresse peut être trouvé avec la commande 
```bash
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' postgres_db
```

### PGAdmin

Une fois les conteneur crée l'espace de stockage est accéssible à l'adresse [http://127.0.0.1:5050/](http://localhost:5050/login?next=/browser/).

### MinIO

Une fois les conteneur crée l'espace de stockage est accéssible à l'adresse [http://127.0.0.1:9001/](http://127.0.0.1:9001/).

## Django setup

Création du project avec la commande :

```bash
docker-compose run --user "$(id -u):$(id -g)" web django-admin startproject fanfare_on_the_web .
```

ce qui va générer les fichiers suivant dans le dossier [Django](/Django/):
```bash
├── fanfare_on_the_web
│   ├── asgi.py
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
├── manage.py
```

Dans [setting.py](/Django/fanfare_on_the_web/settings.py) du nouveaux dossier `fanfare_on_the_web`, modifier :
```python
import os
import dj_database_url

# Database postgreSQL
DATABASES = {
    'default': dj_database_url.config(default=os.getenv("DATABASE_URL"))
}

#Static et media folder
STATIC_URL = '/static/'
STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')  # pour collectstatic

MEDIA_URL = '/media/'
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')  # fichiers uploadés

# MinIO
# Activer django-storages pour les fichiers
DEFAULT_FILE_STORAGE = 'storages.backends.s3boto3.S3Boto3Storage'

AWS_ACCESS_KEY_ID = os.environ.get('AWS_ACCESS_KEY_ID')
AWS_SECRET_ACCESS_KEY = os.environ.get('AWS_SECRET_ACCESS_KEY')
AWS_STORAGE_BUCKET_NAME = os.environ.get('AWS_STORAGE_BUCKET_NAME')
AWS_S3_ENDPOINT_URL = os.environ.get('AWS_S3_ENDPOINT_URL')
AWS_S3_REGION_NAME = os.environ.get('AWS_S3_REGION_NAME')
AWS_S3_ADDRESSING_STYLE = os.environ.get('AWS_S3_ADDRESSING_STYLE', 'path')
AWS_QUERYSTRING_AUTH = False  # rend les fichiers publics si policy public sur le bucket
INSTALLED_APPS = [
    ...
    'storages',
]

# Debug mode
DEBUG = os.environ.get('DEBUG', '0') == '1'
``` 

Il faut construire et lancé les conteneur avec : `docker-compose up -d --build`.

Pour générer les [migrations](https://docs.djangoproject.com/fr/5.1/topics/migrations/) lancer la commande `docker-compose exec web python manage.py migrate`.

Le site est à présent accéssible par [http://localhost:8000](http://localhost:8000)

création du profil super-utilisateur avec la commande `docker-compose exec web python manage.py createsuperuser`

**Cela demande un identifiant et un mot de passe à noté quelque part.**

L'accès à la page admin du serveur django se fait à l'adresse [http://localhost:8000/admin](http://localhost:8000/admin)




### TailwindCSS

1.  Ajouter à [settings.py](/Django/fanfare_on_the_web/settings.py):
    ```python
    INSTALLED_APPS = [
    ...,
    'tailwind',
    ]
    ```

2.  Initialisation avec la commande `docker-compose exec web python manage.py tailwind init`

    Ajouter à [settings.py](/Django/fanfare_on_the_web/settings.py):
    ```python
    INSTALLED_APPS = [
    # Autre app django,
    'tailwind',
    'theme',
    'django_browser_reload',
    ]
    # ...
    TAILWIND_APP_NAME = 'theme'
    # ...
    INTERNAL_IPS = ["127.0.0.1"]
    # ...
    MIDDLEWARE = [
    # Autre middleware,
    "django_browser_reload.middleware.BrowserReloadMiddleware",
    ]
    ```

    Ajouter à [](/backend/myproject/urls.py)
    ```python
    from django.urls import include, path
    urlpatterns = [
        ...,
        path("__reload__/", include("django_browser_reload.urls")),
    ]
    ```

3.  Installer avec la commande `docker-compose exec web python manage.py tailwind install`

4.  Lancer avec la commande `docker-compose exec web python manage.py tailwind start`
    Une fois lancé (nécessite un terminale exploité), les fichier CSS seront générer automatiquement à partir du fichier html.


Dans les fichier .html ajouter à <head>:
```html
    {% load static tailwind_tags %}
    ...
    <head>
        ...
        {% tailwind_css %}
    </head>
```

S'il y a une erreur avec `npm`, modifier dans [settings.py](/Django/fanfare_on_the_web/settings.py) avec le chemin vers npm. Il peut être trouvé avec la commande `docker-compose exec web which npm`
```python
NPM_BIN_PATH = #'path to npm'
```

## App setup

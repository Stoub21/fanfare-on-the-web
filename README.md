# fanfare-on-the-web

## Initialisation

Installer [Docker et Docker-Compose](https://www.docker.com/)

### Docker

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

### PostgreSQL

Une fois le conteneur crée il est possible de s'y connecter avec pgadmin. Pour cela il faut se connecté à l'adresse IP du conteneur. Cette adresse peut être trouvé avec la commande 
```bash
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' postgres_db
```

### PGAdmin

Une fois les conteneur crée l'espace de stockage est accéssible à l'adresse [http://127.0.0.1:5050/](http://localhost:5050/login?next=/browser/).

### MinIO

Une fois les conteneur crée l'espace de stockage est accéssible à l'adresse [http://127.0.0.1:9001/](http://127.0.0.1:9001/).
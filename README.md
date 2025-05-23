# fanfare-on-the-web

Pour comprendre comment la structure du projet à été construite voir [la documentation](/docs/Initialisation_from_0.md).

## Installation

Installer [Docker et Docker-Compose](https://www.docker.com/)

Faire une copie de `.env` avec la commande `cp Docker_setup/env-template.txt Docker_setup/.env` et modifier le fichier `.env` avec vos identifiants.

générer les conteneur avec la commande executer dans le dossier `Docker_setup` :

```bash
cd Docker_setup
docker-compose up -d --build
```

- accéder à l'accueil du serveur Django à l'adresse [http://localhost:8000](http://localhost:8000).
    - à l'espace administrateur à l'adresse [http://localhost:8000/admin](http://localhost:8000/admin)
- accéder à l'interface pgAdmin à l'adresse [http://localhost:5050](http://localhost:5050) puis se connecter à la base de donnée :
    - trouver l'adresse du serveur postgree avec la commande `docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' postgres_db` et 
- accéder à l'interface minIO à l'adresse [http://localhost:9001](http://localhost:9001).

Si le serveur Django ne fonctionne pas directement utiliser les commandes : 
```bash
docker-compose exec web python manage.py migration
docker-compose restart web
```
Ce qui devrait effectuer les migrations et redémarer le serveur. Vérifier que le serveur est à présent fonctionnel.
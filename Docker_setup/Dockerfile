# Utiliser une image Python officielle
FROM python:3.11

# Installer Node.js et npm
RUN apt update && apt install -y nodejs npm

# Définir le répertoire de travail
WORKDIR /app

# Copier les fichiers et installer les dépendances
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Exposer le port 8000
EXPOSE 8000

# Commande pour démarrer le serveur Django
CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]

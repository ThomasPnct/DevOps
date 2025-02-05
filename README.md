# DevOps
## 1-1 Why should we run the container with a flag -e to give the environment variables?
En passant les variables par la ligne de commande, cela nous évite d'écrire ces dernières en clair dans le fichier Dockerfile. De ce fait, la sécurité du projet est améliorée.

## 1-2 Why do we need a volume to be attached to our postgres container?
Pour avoir des données persistentes

## 1-3
Dockerfile :
FROM postgres:14.1-alpine

ENV POSTGRES_DB=db \
   POSTGRES_USER=usr \
   POSTGRES_PASSWORD=pwd

COPY *.sql /docker-entrypoint-initdb.d

docker build -t ThomasPct/databse .

docker run -v ./data:/var/lib/postgresql/data  --net=app-network  --name postgres ThomasPct/databse

## 1-4 
Cela permet d'avoir des images docker moins lourdes et de construire une image docker a partir d'autres images.

### Étape de build : utiliser une image Maven avec Corretto JDK 21
FROM maven:3.9.9-amazoncorretto-21 AS myapp-build

### Définir une variable d'environnement pour le répertoire de l'application
ENV MYAPP_HOME=/opt/myapp 

### Définir le répertoire de travail
WORKDIR $MYAPP_HOME

### Copier le fichier pom.xml (gestionnaire de dépendances Maven)
COPY simpleapi/pom.xml .

### Copier tout le code source du projet
COPY simpleapi/src ./src

### Compiler le projet et générer le fichier JAR (sans exécuter les tests)
RUN mvn package -DskipTests

### Étape d'exécution : utiliser uniquement le JRE (runtime)
FROM amazoncorretto:21

### Définir la variable d'environnement pour le répertoire de l'application
ENV MYAPP_HOME=/opt/myapp 

### Définir le répertoire de travail
WORKDIR $MYAPP_HOME


### Copier le fichier .jar généré lors de la phase de build
COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar

### Exécuter l'application avec Java
ENTRYPOINT ["java", "-jar", "myapp.jar"]

### 1-5 Why do we need a reverse proxy?

sécurité : SSL

haute disponibilité : répartition de la charge

authentification/autorisation centralisée : un serveur pour toutes les applications

## 1-6 Why is docker-compose so important?

**Docker Compose** est un outil qui facilite la gestion de plusieurs conteneurs Docker. Voici pourquoi il est important :

- **Orchestration facile** : Il permet de gérer facilement plusieurs conteneurs, de leur démarrage à leur arrêt, en utilisant un seul fichier de configuration (`docker-compose.yml`). Cela simplifie grandement le déploiement de projets multi-conteneurs.
  
- **Gestion de configurations** : Docker Compose permet de définir les configurations de chaque service (base de données, API, frontend, etc.) dans un même fichier, avec des variables d'environnement et des volumes. Cela rend les déploiements cohérents, reproductibles et faciles à configurer.
  
- **Automatisation** : En combinant plusieurs services dans un fichier `docker-compose.yml`, on peut automatiser des tâches comme la construction des images, le démarrage des conteneurs, le nettoyage des volumes, etc., avec une seule commande.
  
- **Simplicité** : Grâce à Compose, il est plus facile de travailler en équipe sur des projets complexes qui impliquent plusieurs conteneurs. Chaque membre peut travailler avec la même configuration sans avoir à gérer des commandes Docker complexes.


## 1-7 Document docker-compose most important commands.

### Commandes Docker Compose les plus importantes :

- **`docker-compose up`** : Cette commande démarre tous les services définis dans le fichier `docker-compose.yml`. Elle crée et démarre les conteneurs en fonction de la configuration du fichier. L'option `-d` permet de démarrer les conteneurs en arrière-plan (mode détaché).
  
  ```bash
  docker-compose up -d
  ```

- **`docker-compose down`** : Cette commande arrête et supprime tous les conteneurs, réseaux et volumes associés au projet. Cela permet de nettoyer l'environnement de travail.
  
  ```bash
  docker-compose down
  ```

- **`docker-compose build`** : Permet de reconstruire les images des services définis dans le fichier `docker-compose.yml`. Cela est utile lorsque les Dockerfiles ou les fichiers associés ont été modifiés.
  
  ```bash
  docker-compose build
  ```

- **`docker-compose logs`** : Affiche les logs de tous les conteneurs ou d'un conteneur spécifique dans le projet Docker Compose.
  
  ```bash
  docker-compose logs
  ```

- **`docker-compose exec <service> <command>`** : Exécute une commande dans un conteneur en cours d'exécution d'un service spécifié.
  
  ```bash
  docker-compose exec backend bash
  ```

- **`docker-compose ps`** : Affiche l'état des conteneurs gérés par Docker Compose, y compris leurs ID et leurs ports associés.
  
  ```bash
  docker-compose ps
  ```

- **`docker-compose restart`** : Redémarre les services définis dans le fichier `docker-compose.yml`.
  
  ```bash
  docker-compose restart
  ```


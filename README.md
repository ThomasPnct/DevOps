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
## 1-8
```
services:
  java:
    container_name: backend
    build:
      context: ./java
    networks:
      - app-network-2
    depends_on:
      - postgres

  postgres:
    build: database
    environment:
      POSTGRES_DB: db
      POSTGRES_USER: usr
      POSTGRES_PASSWORD: pwd
    volumes:
      - data:/var/lib/postgresql/data
    networks:
      - app-network-2

  http:
    build:
      context: ./http
    container_name: http
    ports:
      - "80:80"
    networks:
      - app-network-2
    depends_on:
      - java

networks:
  app-network-2:
    name: app-network-2

volumes:
  data:
```

### 2-1 What are testcontainers?

**Testcontainers** est une bibliothèque Java qui permet de créer des instances temporaires et jetables de bases de données, de courtiers de messages ou d'autres services via des conteneurs Docker, afin de faciliter les tests d'intégration.  

### 2-2 

```
name: CI devops 2025
on:
  #to begin you want to launch this job in main and develop
  push:
    branches: main 
  pull_request:

jobs:
  test-backend:
    runs-on: ubuntu-22.04
    steps:
      # Checkout your GitHub code using actions/checkout@v2.5.0
      - uses: actions/checkout@v4

      # Set up JDK 21 using actions/setup-java@v3 with the correct distribution
      - name: Set up JDK21
        uses: actions/setup-java@v4
        with:
          java-version: '21'
          distribution: 'corretto'  # You can replace this with your preferred distribution

      # Build and test your app with Maven
      - name: Build and test with Maven
        run: mvn clean verify
        working-directory: simple-api
```
### 2-3 For what purpose do we need to push docker images?

Pousser des images Docker permet de partager et déployer facilement son application sur différents serveurs ou machines. Cela permet aussi de faire en sorte que tout le monde utilise la même version de l'application.

### 2-4 Document your quality gate configuration.

```
name: "SonarQube"
on:
  push:
    branches:
      - main 
      - develop 
  pull_request:
    branches:
      - main
      - develop
jobs:
  test-backend:
    runs-on: ubuntu-22.04
    steps:
      # Checkout your GitHub code using actions/checkout@v2.5.0
      - uses: actions/checkout@v4

      # Set up JDK21 using actions/setup-java@v3 with the correct distribution
      - name: Set up JDK21
        uses: actions/setup-java@v4
        with:
          java-version: '21'
          distribution: 'corretto'  # You can replace this with your preferred distribution

      # Build and test your app with Maven
      - name: Build and test with Maven
        run: mvn clean verify
        working-directory: simple-api
  build:
    name: Build and analyze
    runs-on: ubuntu-latest
    needs: test-backend
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Shallow clones should be disabled for a better relevancy of analysis
      - name: Set up JDK 21
        uses: actions/setup-java@v4
        with:
          java-version: 21
          distribution: 'corretto' # Alternative distribution options are available.
      - name: Cache SonarQube packages
        uses: actions/cache@v4
        with:
          path: ~/.sonar/cache
          key: ${{ runner.os }}-sonar
          restore-keys: ${{ runner.os }}-sonar
      - name: Cache Maven packages
        uses: actions/cache@v4
        with:
          path: ~/.m2
          key: ${{ runner.os }}-m2-${{ hashFiles('**/pom.xml') }}
          restore-keys: ${{ runner.os }}-m2
      - name: Build and analyze
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
        run: mvn -B verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.projectKey=ThomasPnct_tp-devops-correction-docker 
        working-directory: simple-api
```
```
name: "Docker Hub"

on:
  workflow_run:
    workflows: ["SonarQube"]
    types: [completed]
  pull_request:
    branches:
      - main

jobs:
  # define job to build and publish docker image
  build-and-push-docker-image:
    runs-on: ubuntu-22.04  # Specifies the environment to run the job on (Ubuntu 22.04)
    if: github.event.workflow_run.conclusion == 'success' && github.event.workflow_run.head_branch == 'main' # Ensure deploy only runs for main
    
      
    steps:
      - name: Checkout repository  # Checkout the code from the repository
        uses: actions/checkout@v4

      - name: Login to Docker Hub  # Log in to Docker Hub using credentials stored in secrets
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}  # Docker Hub username from secrets
          password: ${{ secrets.DOCKERHUB_TOKEN }}  # Docker Hub token from secrets

      # Build and push the backend Docker image
      - name: Build and push backend image
        uses: docker/build-push-action@v3
        with:
          context: ./simple-api  # Directory containing the backend code
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/tp-devops-backend:latest  # Docker image tag
          push: ${{ github.ref == 'refs/heads/main' }}  # Push image only if commit is on the 'main' branch

      # Build and push the database Docker image
      - name: Build and push database image
        uses: docker/build-push-action@v3
        with:
          context: ./database  # Directory containing the database code
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/tp-devops-db:latest  # Docker image tag
          push: ${{ github.ref == 'refs/heads/main' }}  # Push image only if commit is on the 'main' branch

      # Build and push the HTTP Docker image
      - name: Build and push http-server image
        uses: docker/build-push-action@v3
        with:
          context: ./http-server  # Directory containing the HTTP service code
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/tp-devops-http:latest  # Docker image tag
          push: ${{ github.ref == 'refs/heads/main' }}  # Push image only if commit is on the 'main' branch
```
### 3-1 Document your inventory and base commands

```
all:
 vars:
   ansible_user: admin
   ansible_ssh_private_key_file: ../id_rsa
 children:
   prod:
     hosts: thomas.poncet.takima.cloud
```
```
root@DESKTOP-2KODCED:/tp-devops-correction-docker/ansible# ssh -i id_rsa admin@thomas.poncet.takima.cloud
```
### 3-2 Document your inventory and base commands

```
- hosts: all
  gather_facts: true
  become: true

  tasks:
    - name: Gather facts
      setup:

  roles:
    - docker
    - env
    - network
    - database
    - app
    - proxy
```

---
# tasks file for docker

- name: Install Docker dependencies
  apt:
    name: "{{ item }}"
    state: present
  with_items:
    - apt-transport-https
    - ca-certificates
    - curl
    - software-properties-common

- name: Add Docker's official GPG key
  apt_key:
    url: https://download.docker.com/linux/ubuntu/gpg
    state: present

- name: Add Docker repository
  apt_repository:
    repo: deb [arch=amd64] https://download.docker.com/linux/ubuntu {{ ansible_distribution_release | lower }} stable
    state: present

- name: Install Docker
  apt:
    name: docker-ce
    state: latest

- name: Start and enable Docker service
  service:
    name: docker
    state: started
    enabled: yes

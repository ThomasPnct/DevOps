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

# Étape de build : utiliser une image Maven avec Corretto JDK 21
FROM maven:3.9.9-amazoncorretto-21 AS myapp-build

# Définir une variable d'environnement pour le répertoire de l'application
ENV MYAPP_HOME=/opt/myapp 

# Définir le répertoire de travail
WORKDIR $MYAPP_HOME

# Copier le fichier pom.xml (gestionnaire de dépendances Maven)
COPY simpleapi/pom.xml .

# Copier tout le code source du projet
COPY simpleapi/src ./src

# Compiler le projet et générer le fichier JAR (sans exécuter les tests)
RUN mvn package -DskipTests

# Étape d'exécution : utiliser uniquement le JRE (runtime)
FROM amazoncorretto:21

# Définir la variable d'environnement pour le répertoire de l'application
ENV MYAPP_HOME=/opt/myapp 

# Définir le répertoire de travail
WORKDIR $MYAPP_HOME

# Copier le fichier .jar généré lors de la phase de build
COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar

# Exécuter l'application avec Java
ENTRYPOINT ["java", "-jar", "myapp.jar"]


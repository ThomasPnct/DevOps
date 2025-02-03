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

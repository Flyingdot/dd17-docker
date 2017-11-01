Docker für Entwickler mit .NET Core
-------------------------------------

***
Umgebung:
 - Notebook: MBP
 - Notebook Backup: P50
 - Software
 	- Docker
*******

```
# Preparation
iterm
~\dev\dd-docker
	frontend
	backend

code .
```

0. Intro
	* Docker als DevOps oder Ops-Thema
	* Sehr nützlich Dev-Workflow

1. Docker Theorie (10')
    * was ist Container-Virtualisierung?
    * Layer-Modell
    * Stand Windows vs Linux

2. Demo (1): Init App Szenario (2')
   	* webapi-backend
   	* angular-frontend
   	* mongodb
   	* webapi/angular lokal laufen lassen

```
# frontend
ng new frontend
ng serve
localhost:4200

# backend
dotnet new webapi
dotnet run
http://localhost:5000/api/values
```

```
.dockerignore
packages.json
	ng serve -H 0.0.0.0

# frontend/Dockerfile
FROM node:latest
# hub.docker.com

RUN mkdir -p /usr/src/app

WORKDIR /usr/src/app

COPY package.json /usr/src/app

RUN npm install

COPY . /usr/src/app

EXPOSE 4200

CMD ["npm", "start"]
```
```
docker build -t frontend .
docker run -d --name frontend -p 4200:4200 frontend
```

```
.dockerignore
packages.json
	ng serve -H 0.0.0.0

# backend/Dockerfile
FROM microsoft/aspnetcore-build:2.0 AS build-env
WORKDIR /app

COPY *.csproj ./
RUN dotnet restore

COPY . ./
RUN dotnet publish -c Release -o out

FROM microsoft/aspnetcore:2.0
WORKDIR /app
COPY --from=build-env /app/out .
ENTRYPOINT ["dotnet", "backend.dll"]
```
```
docker build -t backend .
docker run -d --name backend -p 8080:80 backend
```

```
docker run -d --name mongodb -p 27017:27017 mongo
```

4. Demo (3): Docker-Compose (10')

```
version: '2'

services:
  frontend:
    build: frontend
    ports:
      - "4200:4200"
  backend:
    build: backend
    ports:
      - "8080:80"
  db:
    image: mongo
    ports:
      - "27017:27017"
```
```
docker kill mongodb frontend backend
docker-compose up
```

5. Demo Volumes (5')
```
# docker-compose.yml
volumes:
      - ./frontend:/usr/src/app
```
```
# frontend/Dockerfile
FROM node:8

# RUN mkdir -p /usr/src/app

VOLUME [ "/usr/src/app" ]
WORKDIR /usr/src/app

#COPY package.json /usr/src/app

#RUN npm install

#COPY . /usr/src/app

EXPOSE 4200

CMD ["npm", "start"]
```

```
frontend/npm install
docker-compose up --build

6. Demo (4): Volumes integrieren
	* Volume für Angular App definieren
	* Reload nach View-Änderung
	* Summary
		* Entwicklungs-Workflow definiert
		* Live-reloading etc. möglich

7. Linking Container

8. Demo (5): Linking Container
	* API erstellen
	* Angular-Component erstellen
	* Links definieren

Was noch fehlt:
 - Application Links (aktiver API-Call z.B.)
 - Environment-Variablen integrieren
 - Debug



Cheatsheet:
	docker build ...
	docker run ...
	docker rm
	docker rm -v
	docker kill


docker build -t frontend .
docker build -t backend .
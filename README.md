# Introduction

I decided to create this repository because the process of creating a server using Ubuntu Server is quite arduous and has several steps that I might later forget.

## Problems

I needed to run an application on a computer that would serve as a server. Therefore, I saw Ubuntu as a better option. So I decided to run a test environment with a VM, but I couldn't run docker-compose.

### Question
First I asked myself if I just did a `git clone` of my code, would it run on another machine, even if it was running inside Ubuntu?

Of course, I would use **SSH** with **tunneling**. 

Well, I don't think I would be able to access the database connected to the API, since I would be using `localhost`, and that would be a problem. So I thought about using Docker and, to use it, I would have to configure my application to run on **Docker**.

In the next sections I will show part of the Dockerization processes of an example application.

# .env file

The .env file used must have the following environment variables defined.

```ini
DB_HOST=Some_Host_Name
DB_NAME=Some_DB_Name
DB_USER=Some_DB_Username
DB_PASSWORD=Some_DB_Password
DB_PORT=Your_DB_Port
DB_CONTAINER_PORT=Your_DB_Container_Port
SERVER_PORT=Your_server_Port
SERVER_CONTAINER_PORT=Your_server_Container_Port
```

It should be on the same page as `docker-compose.yml`.

# Docker Configuration
I will show the settings, but in the repository I show what my folder system looks like for my application.

## Docker Compose
```yml
# Creating services docker declarative commands
services:
  db:
    # import .env file
    env_file:
      - .env
    # used image
    image: mongo:latest
    # name that identifies the container
    container_name: db
    # if there is any error it should restart every time
    restart: always
    ports:
      # HOST_PORT:CONTAINER_PORT
      - ${DB_PORT}:${DB_CONTAINER_PORT}
    environment:
      # Environment variables for mongodb
      MONGO_INITDB_DATABASE: ${DB_NAME}
      MONGO_INITDB_ROOT_USERNAME: ${DB_USER}
      MONGO_INITDB_ROOT_PASSWORD: ${DB_PASSWORD}
    volumes:
      # defining the volume used by my database
      - mongo_data:/data/db

  # If you want to upload more images, just copy the configuration below
  server:
    # I'm defining the node with a predefined version, but you can use anything else
    image: node:20.11.1
    container_name: server

    # Here I define whether I will use a Dockerfile and where it is
    build:
      context: ./server
      dockerfile: Dockerfile

    restart: always
    env_file:
      - .env
    environment:
      DB_NAME: ${DB_NAME}
      DB_USER: ${DB_USER}
      DB_PASSWORD: ${DB_PASSWORD}
      DB_HOST: ${DB_HOST}
      BD_PORT: ${DB_PORT}
      SERVER_PORT: ${SERVER_PORT}
    ports:
      # HOST_PORT:CONTAINER_PORT
      - ${SERVER_PORT}:${SERVER_CONTEINER_PORT}
    
    # Here I define that this image will start running after the db is started
    depends_on:
      - db
    volumes:
      - ./server:/src

    # defining the directory in which we will work
    working_dir: /src

    #defining which command should run
    command: npm start

    # setting a limit for the logs that docker produces
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "5"

volumes:
  mongo_data:
```

## Dockerfile
```Dockerfile
FROM node

WORKDIR /src

COPY package*.json ./

RUN npm install

COPY . .

CMD ["npm", "start"]
```

## Makefile
It must be in the same directory as `docker-compose.yml`

```make
include .env

.PHONY: up

up: 
	docker-compose up -d

.PHONY: down

down: 
	docker-compose down

```

### Run make

#### Up docker
```sh
make up
```

#### Down docker
```sh
make down
```

## Docker - Extra

If I want to monitor the console of the application I am running, I can run the following command in the terminal.

```sh
docker logs -f container_name 
```

### Ubuntu Server
To interact in your application's terminal/console.
```sh
docker exec -it container_name /bin/bash
```

## Restore your database

### Mongodump gzip
You must do this on your local computer.
```sh
mongodump --db <database name> --gzip --out <.:/your/directory>
```
### up the docker-compose

### Copy dump to docker
```sh
docker cp <.:/your/directory> <.:/your/docker/directory>
```

### Mongorestore

```sh
docker exec -it <container name> mongorestore --authenticationDatabase admin -u <user name> -p <password> --db <database name> --gzip --dir <directory>
```

# Ubuntu Server Docker Instalation
By default, Docker is already installed on **Ubuntu server**.

## Problems
I couldn't run docker compose

## Solution
I decided to do the correct **Docker** installation with **Compose** support myself.

## Check **Docker** version
```sh
sudo docker --version
```

## apt-get update
```sh
sudo apt-get update
```

## apt-get installations
```sh
sudo apt-get install curl
```

```sh
sudo apt-get install gnupg
```

```sh
sudo apt-get install ca-certificates
```

```sh
sudo apt-get install lsb-release
```

## Create directory
```sh
sudo mkdir -p /etc/apt/keyrings
```

## Create docker.gpg
```sh
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```
You can enter the created folder and see if the file was created
```sh
cd /etc/apt/keyrings
```

```sh
ls
```
## Create repository
```sh
sudo echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

## Update
```sh
sudo apt-get update
```

## Install Docker

```sh
sudo apt-get isntall docker-ce docker -ce-cli containerd.io docker-compose-plugin
```

## Teste Docker
```sh
sudo docker run hello-world
```

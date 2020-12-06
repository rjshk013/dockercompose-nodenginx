# dockercompose-nodenginx

Step 1:Clone the repo in your local system

https://github.com/rjshk013/dockercompose-nodenginx

contents will look like 

client  controller  docker-compose.yml  README.md  server

remember to run npm install on client & server  folder individually if you want to test locally.

so after that each folder look like 

#depends on your cloned repository location
docker@docker:~/testrepos/dockercompose-nodenginx/server$ ls
Dockerfile  index.js  node_modules  package.json  yarn.lock

docker@docker:~/testrepos/dockercompose-nodenginx/client$ ls
Dockerfile  node_modules  package.json  public  src  yarn.lock

on controller folder  we need to create file named default.conf with below contents 
upstream client {
  server client:3000;
}

upstream server {
  server server:4000;
}

server {
  listen 80;

  location / {
    proxy_pass http://client;
  }

  location /api {
    proxy_pass http://server;
  }
}

So the Dockerfile inside  controller folder look like 

FROM nginx
COPY ./default.conf /etc/NGINX/conf.d/default.conf

Dockerfile under client folder 

FROM node:alpine
WORKDIR '/app'
COPY ./package.json ./
RUN npm install
COPY . .
CMD ["npm", "run", "start"]


Dockerfile under server/

WORKDIR '/app'
COPY ./package.json ./
RUN npm install
COPY . .
CMD ["npm", "run", "start"]

docker-compose.yml will look like 

version: '3'
services:
    server:
        build:
            dockerfile: Dockerfile
            context: ./server
        volumes:
            - /app/node_modules
            - ./server:/app
    nginx:
        restart: always
        build:
          dockerfile: Dockerfile
          context: ./controller
        ports:
          - '5000:80'
    client:
        build:
            dockerfile: Dockerfile
            context: ./client
        volumes:
            - /app/node_modules
            - ./client:/app

Finally create our services and attach our containers together using the docker-compose up command and the --build flag to build out our Dockerfiles.

docker-compose up --build



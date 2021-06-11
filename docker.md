# Docker
github for containers: DockerHub
Container (layers of images) - linus Base Image, application image. (Tag, Port)

`docker pull postgres:13.3`  => to pull from dockerhub
`docker run postgres:13.3`  => to pull and run from dockerhub
`docker ps`		     => all the running container

image can be moved around and when you start a image it becocomes a container
	IMAGE - NOT RUNNING
	CONTAINER - RUNNING

OS have 2 layers
 - Kernel
 - Applications
 
different linux distro all use the same Linux Kernel but implement different applicartion on top

Docker virtualize the application layer - Small images, speed startups 
VM virtualize both kernel and apllication layer (boots up a new kernal on VM)  - vm of any OS can run on any OS hosts

Installation: 
- DockerCE
- Docker Enginer
- Docker Compose

`docker images`		   => all the images
`docker run -d postgres:13.3`     => De-attach mode
`docker stop container-id`        => stop running the container
`docker start container-id`        => start running the container
`docker ps -a`		     => all the running or stopped container

## port 
which port the container is listening to is given by port
conatiner ports can be same but host ports must be different

How to solve this binding the port while bringing up the container
`docker run -p6000:6379 redis`
`docker run -p6000:6379 -d redis:40`


naming container:
`docker run -d -p6002:6379 --name redis-older redis:4.0`

clean up docker
images -
container -

troubleshooting with container
`docker logs container-id` or `docker logs names` or `docker logs names | tail` or `docker logs names -f`
`docker exec -it container-id /bin/bash` use `exit` quit terminal 

`env` lists all env variables

------------------------------------------------------------
PRACTICLE workflow

development  -> CI/Delivery -> Deployment
------------------------------------------------------------

 develop a app with images from dockerhub for db 
 -commit
 git
 -Jenkins(builds the app and creates an image)
 -push
 Dockr Repository (private)
 -pull
 Dev sever pull all images(app and for db)


----------------------------------------------------

 Docker Network

Mongo - mongo-express

DOcker network 
- use just the container name to talk to each other when on same network

`docker network ls`
`docker network create mongo-network`

use the network to run containers if they want to talk to each other
`docker run -p27017:27017 -d mongo `

```
$ docker run -d --network some-network --name some-mongo \
    -e MONGO_INITDB_ROOT_USERNAME=mongoadmin \
    -e MONGO_INITDB_ROOT_PASSWORD=secret \
    mongo

$ docker run -it --rm --network some-network mongo \
    mongo --host some-mongo \
        -u mongoadmin \
        -p secret \
        --authenticationDatabase admin \
        some-db
> db.getName();
some-db
```

cant keep doing this on terminal 

# Docker compose
my-docker-compose.yaml is stored within the application

docker compose takes care of creating a common network :O

```
version:'3'
service:
    mongodb:    => CONTAINER NAME
        image:mongo:4.0
        ports:
            - 27017:27017  => HOST:CONTAINER
        environment:
            - MONGO_INITDB_ROOT_USERNAME=mongoadmin

    mongo-express:
        image:mongo-express
        port:
            - 8080:8080
        environment:
            - ME_CONFIG_MONGODB_SERVER=mongodb

````
how to use this file
docker-compose -f my-docker-compose.yaml up  => to bring up all containers

docker-compose -f my-docker-compose.yaml down  => to bring down all containers and network

Note: we can add the waiting logic here

when you restart a container all the data is lost :\ ( data persistant is not available on container by default)

## Dockerfile - Building our own Docker Image
we need to package our app a own container to deploy 
docker file to docker image

Jenkins builds app and creates a docker image

simulate what jenkins does- like how builds app and creates image on local
1. copy artifact into dockerfile
2. use Blueprint for creating docker image

call it `Dockerfile`
`
FROM node:tag => base it of other image (instead of linux)

ENV MONGO_DB_USERNAME=admin \
    MONGO_DB_PASSWORD=password

RUN mkdir -p /home/app => can run any linux command(inside container)

COPY ./app /home/app => copy from host to conatiner

CMD ["node","/home/app/server.js"] => start the app with (entry command) only 1 can be specified

`

to build the image from dockerfile
give our a name and a tag
`docker build -t my-app:1.0 .`

we cant override it so delete image with
`docker rmi image-id`

delete container with
`docker rm container-id`


we need aws cli need to be installed on local

Docker private repository (AWS elastic container registry)(1000 image versions limit)
Registry options
build and tag an image
docker login 
docker push registryDomain/imageName:tag

Image naming in docker registeries
registryDomain/imageName:tag
in docker hub 
docker pull mongo.io/library/mongo:4.2


# Deploy on development server our containerized app

use docker-compose to pull from both private repo and dockerhub

We need to add our app image as well

```
version:'3'
service:
    mongodb:    => CONTAINER NAME
        image:mongo:4.0
        ports:
            - 27017:27017  => HOST:CONTAINER
        volumes:
            - db-data:/data/db
        environment:
            - MONGO_INITDB_ROOT_USERNAME=mongoadmin

    mongo-express:
        image:mongo-express
        ports:
            - 8080:8080
        environment:
            - ME_CONFIG_MONGODB_SERVER=mongodb
    
    my-app:
        image:12o3u3.dkr.ecr.eu-central-1.amazonaws.com/my-app:1.0
        ports:
            -3000:3000

volumes:
    db-data:
        driver: local
```
# Docker Volumes
For data persistence
 - databases
 - other statefull applications

Folder in physical host file system is mounted into the virtual file system of docker

3 types of docker volume
docker run -v host:container
docker run -v container (anonymous volume)
docker run -v name:/var/lib/mysql/data (named volumes) use in prod

where are these located???:O
/var/lib/docker/volumes
_data 




real example 
```
version: "3.7"
services:
  # Third party services

  postgres:
    build: containers/postgres
    environment:
      POSTGRES_HOST_AUTH_METHOD: trust
    init: true
    ports:
      - 5432:5432
    restart: unless-stopped
    volumes:
      - postgres:/var/lib/postgresql/data

  mongo:
    init: true
    image: mongo:4.2.11
    ports: ["27018:27017"]

  localstack:
    init: true
    privileged: true
    build: containers/localstack
    ports: ["4566:4566", "4600:4600"]
    environment:
      - SERVICES=s3,sns,sqs,cloudwatch
      - DATA_DIR=/tmp/localstack/data
      - PORT_WEB_UI=4600
      - DOCKER_HOST=unix:///var/run/docker.sock
      - HOST_TMP_FOLDER=/tmp/
      - FORCE_NONINTERACTIVE=1
      - SKIP_INFRA_DOWNLOADS=1
    volumes:
      - /tmp/localstack:/tmp/localstack
      - /var/run/docker.sock:/var/run/docker.sock
    logging:
      driver: "json-file"
      options:
        max-size: "10M"
        max-file: "10"
 ```
 
 
 
 *https://learnxinyminutes.com/docs/yaml/*
 
 ```
 #######################
# EXTRA YAML FEATURES #
#######################

# YAML also has a handy feature called 'anchors', which let you easily duplicate
# content across your document. Both of these keys will have the same value:
anchored_content: &anchor_name This string will appear as the value of two keys.
other_anchor: *anchor_name

# Anchors can be used to duplicate/inherit properties
base: &base
  name: Everyone has same name

# The regexp << is called Merge Key Language-Independent Type. It is used to
# indicate that all the keys of one or more specified maps should be inserted
# into the current map.

foo:
  <<: *base
  age: 10

bar:
  <<: *base
  age: 20

# foo and bar would also have name: Everyone has same name
```











# https://www.youtube.com/watch?v=AYAh6YDXuho

Intro to container services on aws
1. ECS (Elastic Container Service) - EC2 vs Fargate
2. EKS (Elastic Kubernetes Service)
3. ECR (Elastic Container Registry)


Some container Orchestration tools are  ECS, Docker Swarn, Kubernetes, Mesos, Nomad 



ECS cluster - Control Plane

Services are runnin on EC2 Instances [container runtime, ECS Agent]
	- headache need to take care of EC@ infra

ECS Fargate - (I'm here to help) - serveless way to launch a container 




what is VPC, Elastic load Balance for load balancing

a defaut comes with every aws account
Virtual Private Cloud

Region > VPC > Availabilty zone (subnet)
https://www.youtube.com/watch?v=bGDMeD6kOz0 best
https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints.html
https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/vpc_endpoint

https://www.sentinelone.com/blog/what-is-hash-how-does-it-work/


























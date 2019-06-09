---
title: Building a secure & optimised  docker image using multistage docker build approach
layout: post
category: docker
date: '2019-06-09 03:20:33'
New field 1: golang
---

In this post, I am going to walk  through how to build  a secure and optimized docker image. 

In the previous post, I have developed a simple API server using go. we will use that project to develop the docker image.  So, I would recommend having a look on that post.
https://tanverhasan.com/golang/2019/05/05/creating-restfull-api-in-go.html

# Creating docker iamge :

* Create a docker file in the project directory:

```
touch Dockerfile.basic && touch .dockerignore

```

In that stage, we are going to pull the golang:1.12 image from docker hub and then copy the source code from host machine to docker golang:1.12 image. Then,  set the environment port and pull the go packages required for this project. Note that , this projcet uses the go module architecture. Therefore, we do not need to configure the go src path. Finally, run the application by executing the project binary. 

```
# pulling golang image
FROM golang:1.12   

# setting up working directory
WORKDIR /app   

 # copying the project form source
COPY . /app      

# Setting up environment variable
ENV PORT 7200

# Remvoe the unnecessary dependency and ensure the current go.mod file reflects all possible tags/os/architecture combinations
RUN go mod tidy

# Build go project
RUN  go build  -o api-server

#RUN Go project
ENTRYPOINT [ "./api-server" ]
```

* Build  image:
```
docker build -t api-server:basic -f Dockerfile.basic . 
```
* Run image :
```
docker run -it -p 7200:7200 api-server:basic 
```

If you now look at the image size , you may find it surprising how big is   image.
```
docker image ls | grep  api-server | grep basic
```
The command should find the docker image and display teh information about the image
```
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
api-server          basic               d427ed65f721        13 minutes ago      1.01GB
```
As you can see, the image size is unacceptable. 

# Optimizing docker image: 

Let's optimize the image size.   First of all, go generates a binary file which is a massive advantage. Secondly, Docker supports multistage build. So, we will  build the image using the golang:1.12 image . Then, copy the project binary file to a minimum alpine linux image and run the project. 

Create a dockerfile `touch Dockerfile.prod`

```

FROM golang:1.12 as build
COPY . /app
WORKDIR /app
RUN go mod tidy

ENV PORT 7001
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o api-server .


FROM alpine 
ENV PORT 7001
WORKDIR /app
COPY --from=build /app/api-server .
CMD [ "./api-server" ]
```



* Build image : 

```
docker build -t api-server:prod -f Dockerfile.prod . 
```


* Run  image :
```
docker run -it -p 7001:7001 api-server:prod 
```

Look at the image size :` docker image ls `
```
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
api-server          prod                7c45fc663892        8 minutes ago       16.7MB
```
We managed to reduce the image size significantly. 

# Securing the docker image. 
Let's step back and understand how the docker image works. Docker takes advantage of the Linux Namespace feature to isolate the  docker container from host machine. For example, the ame process can have different PID in the conainer and the host machine and work perfectly. However, container can still have some access to host operating system. Such as /proc  file system and the system time. As a solution, we can use linux capabilities and seccomp to reduce the syscall from the container. For more detailed discussion on docker security, I would highly recommend reading the [docker security guidelines](https://docs.docker.com/engine/security/security/) and [Red Hat container security guidelines](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux_atomic_host/7/html/container_security_guide/index) 



Now , monitor the process priviliges in the docker container. 
* List docker conainer : 
```
docker container ls
```

* Monitor the process in the container : 

``` 
docker container top [conainer_id]

```
Copy the container ID from the previous command.  The result should look like following 

```
PID                 USER                TIME                COMMAND
78953               root                0:00                ./api-server
```

Interestingly, our api server is running with root priviliges. This is extremely dangerious. 

create another docker file , `touch Dockerfile.secure`

In this satage, we are going to create a non privilige user and run project as non priviliege user. 

```

FROM golang:1.12 as build
COPY . /app
WORKDIR /app
RUN go mod tidy

ENV PORT 7002
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o api-server .


FROM alpine 
RUN apk --no-cache add ca-certificates && \
    apk add --no-cache strace && \
    addgroup -S -g 500 api && \
    adduser -S -u 500 -G api api 

    
ENV PORT 7002
WORKDIR /app
COPY --from=build /app/api-server .

USER api:api
CMD [ "./api-server" ]
```


* Build  image : 
```
docker build -t api-server:secure -f Dockerfile.secure . 
```

* Run  image :
```
docker run -it -p 7002:7002 api-server:secure 
```

Monitor the process PID  and it privilege. It is running the user 500 which we created and assigned using the USER command in the docker file. 
```
PID                 USER                TIME                COMMAND
86058               500                 0:00                ./api-server
```

As promised, we drastically reducied the size of the docker image and droped the priviliege to reduce the attack surface in the container. Finally, never store secret information in the container and do not expose docker service using tcp port. The reasoning is that docker service to a TCP port allows non-root users to gain root access on the host.
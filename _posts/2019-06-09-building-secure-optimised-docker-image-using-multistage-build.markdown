---
title: Building secure & optimised  docker image using multistage build
layout: post
category: docker
date: '2019-06-09 03:20:33'
---

In this post, I am going to walkthrough how to build secure and optimized docker image. 

In the previous post, I have developed a simple API server using go. we will use that project to develop the docker image.  So, I would recommend to have a look on that post.
https://tanverhasan.com/golang/2019/05/05/creating-restfull-api-in-go.html

Creating docker iamge :

Create docker file 

```
touch Dockerfile && touch .dockerignore

```

In that stage, we are going to pull the golang:1.12 image from docker hub and then copy the source code from host machine to docker golang:1.12 image. Then,  set the environment port and pull the go packages requird for this project. Note that , this projcet uses the go module architecture. Therefore, we do not need to use the go src path. Finally, run the application by executing the project binary. 

```
FROM golang:1.12
WORKDIR /app
COPY . /app
ENV PORT 7200
RUN go mod tidy
RUN  go build  -o api-server
ENTRYPOINT [ "./api-server" ]
```

Build the image 
```
docker build -t api-server:basic -f Dockerfile.basic . 
```
Run image :
```
docker run -it -p 7200:7200 api-server:basic 
```

If you now look at the image size , you may find it surprising how the the image.

```
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
api-server          basic               d427ed65f721        13 minutes ago      1.01GB
```

Optimizing docker image: 

As we already noticed, the size of the image is huge. However, it is possible to optimize the image size. Generelly, go generates binary file which is massive advantage. Docker supports multistage build. So, we will  build the image using the golang:1.12 image . Then, copy the project binary file to a minimum linux image and run the project. 

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

Build the image : 

```
docker build -t api-server:prod -f Dockerfile.prod . 
```

Run the image :
```
docker run -it -p 7001:7001 api-server:prod 
```
Look at the image size :` docker image ls `
```
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
api-server          prod                7c45fc663892        8 minutes ago       16.7MB
```
We managed to reduce the image size significantly. 

Securing the docker image. 

Now , monitor the process priviliges in the docker container. 
List docker conainer : `docker container ls`
Monitor the process in the container : docker container top [conainer_id]

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
#RUN go get -d -v golang.org/x/net/html  

ENV PORT 7002
#RUN  go build  -o api-server
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o api-server .
#ENTRYPOINT [ "./api-server" ]


FROM alpine 
RUN apk --no-cache add ca-certificates && \
    apk add --no-cache strace && \
    addgroup -S -g 500 api && \
    adduser -S -u 500 -G api api 

    
ENV PORT 7002
# ENV HOME /app
WORKDIR /app
COPY --from=build /app/api-server .

USER api:api
CMD [ "./api-server" ]
```

Build the image : 

```
docker build -t api-server:secure -f Dockerfile.secure . 
```

Run the image :
```
docker run -it -p 7002:7002 api-server:secure 
```
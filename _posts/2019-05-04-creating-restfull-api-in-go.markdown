---
title: Developing scalable RESTFull API using GO, POSTGRES, GORM
layout: post
date: '2019-05-03 21:33:43 +0100'
categories: golang
ORM: gorm
Secure API: JWT (JSON web token)
---

**Summary**

The following is the planning what we will build in this blog post. 
					
* 					Creating RESTFull API  in go
* 					Using Postgress database to store data
* 					Using GORM as ORM
* 					Testing the API using CURL



# Setting up the environment. 

* lnstalling  the golang and  configure the enviroment. The following documentation shows the necesary information [https://golang.org/doc/install](http://). 
* Installing the nodejs server for front end application 
* Installing docker to speed up the setup of local environment
* Installing the Postgress in your local machine. You can use the following docker  compose file  to pull the postgress and pgAdmin image and run in your local machine 

Create a file and name it `docker-compose.yml ` . Copy the follwoing code. 

```
# Use postgres/example user/password credentials
version: '3.1'

services:
  #https://hub.docker.com/_/postgres
  pgdb:
    container_name: pgdb
    image: postgres:latest
    restart: always
    ports: 
      - 5432:5432
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    volumes:
      - db-data:/var/lib/postgresql/data
    networks: 
      - postgressNetwork
  adminer:
    image: adminer
    restart: always
    ports:
      - 8080:8080
    networks: 
      - postgressNetwork

  #https://hub.docker.com/r/dpage/pgadmin4/
  pgadmin:
    container_name: pgadmin
    image: dpage/pgadmin4
    environment:
      PGADMIN_DEFAULT_EMAIL: "pgadmin4@pgadmin.org"
      PGADMIN_DEFAULT_PASSWORD: "admin"
    ports: 
      - 5050:80
    networks: 
      - postgressNetwork

networks:
  postgressNetwork:
    driver: bridge    
    
volumes:
  db-data:
	
```

Save the file and run `docker-compose up` . It should pull the postgres docker images and run the postgress on port `8080` . Secondly, it will pull the pgadmin docker image and run the project on port `5050` . In a terminal , run `docker ps`  to see the active container.  PGADMIN is a admin UI for postgress database. You can interact with postgress database using a nice user interface in your browser. Navigagte to `localhost:5050` to login to pgadmin.


# Developing the API server
At first we are going to develop the API server in Go.  Go provides  `net/http` package for http client and server implementation. Secondly, we will use `gorila/mux package` which provide fast and easy request router and HTTP dispatcher for incoming HTTP request. 

Lets create a go project (demoapi) in src folder. Create a file `main.go`.  Before writing any code, we need to import appropriate package to use in this project. 

Ruen the following in the command line to import the package. 

```
 got get -u github.com/jinzhu/gorm
 go get -u github.com/lib/pq
 go get   -u github.com/gorilla/mux
 go get  -u github.com/gorilla/handlers
 go get  -u github.com/gorilla/context
 
```

#### Deffining the HTTP server

Project summary: In this porject, I am going to develop a simple CURD opertions and expose as RESTFull api endpint. 

Create a file `main.go` in the project folder. This file is the main entry point for the entire API server. It will contain API endpoint defination and routing to appropriate handler and finally start the API server. 
     
Lets examine the minimum code to set up the HTTP server. We first initizlized the `gorila/mux `router. Then, defined a simple endpoint which should only returns *Hello world* if the API request match the route `/` . After that , we passed the route definenation to `http.handle` method. Finally we called the `http.ListenAndServe` method to start the application 
     
```

package main

import (
	"log"
	"net/http"

	"github.com/gorilla/context"
	"github.com/gorilla/mux"
)

func main() {

	// Initilizing the gorila/mux router
	r := mux.NewRouter()

	// Test API endpoints
	r.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		w.WriteHeader(http.StatusOK)
		w.Write([]byte("Hello world"))
	})

	http.Handle("/", r)
	
	log.Print("Project is serving on port 7000 : http://localhost:7000")
	http.ListenAndServe(":7000", context.ClearHandler(http.DefaultServeMux))
}
```

Run `go run` command in the project directory. Open a browser window and navigate to [http://localhost:7000/](http://http://localhost:7000/) to see the message **Hello world**. Thats awsome, we managed to setup a HTTP server just writing a few line of code. I personally liked the simplicity of go.

#### Defining the API endpoints

At this stage, lets define our product endpoint. Here we register routes mapping URL paths to handlers. This is equivalent to how `http.HandleFunc()` works: if an incoming request URL matches one of the paths, the corresponding handler is called passing (http.ResponseWriter, *http.Request) as parameters.


```
	// Defining API endpoints
	// Retrieve the list of products
	r.HandleFunc("/api/products", api.ProductHandlerGETALL).Methods("GET")
	//Return Individual Products
	r.HandleFunc("/api/products/{id}", api.ProductHandlerGETBYID).Methods("GET")
	// Creating product
	r.HandleFunc("/api/products", api.ProductHandlerPOST).Methods("POST")
	//Deleting product
	r.HandleFunc("/api/products/{id}", api.ProductHandlerDELETE).Methods("DELETE")
	// Update  product
	r.HandleFunc("/api/products/{id}", api.ProductHandlerUPDATE).Methods("UPDATE")

```



Create a folder in the project root directory (api). Secondly, create a file ProductHandler which will  contain product API endpoint implementation. For the testing purpose, lets just create the boilerplate endpoint implementation. 

**ProductHandler.go**

```
package api

import (
	"net/http"
)

// ProductHandlerGETALL returns all products
func ProductHandlerGETALL(w http.ResponseWriter, r *http.Request) {

	w.Write([]byte("product handler GETALL endpoint"))
}

// ProductHandlerGETBYID returns product by id
func ProductHandlerGETBYID(w http.ResponseWriter, r *http.Request) {

	w.Write([]byte("product handler GETBYID endpoint"))
}

// ProductHandlerPOST delete product by id
func ProductHandlerPOST(w http.ResponseWriter, r *http.Request) {

	w.Write([]byte("product handler POST point"))
}

// ProductHandlerDELETE delete product by id
func ProductHandlerDELETE(w http.ResponseWriter, r *http.Request) {

	w.Write([]byte("product handler DELETE endpoint"))
}


// ProductHandlerUPDATE update product
func ProductHandlerUPDATE(w http.ResponseWriter, r *http.Request) {

	w.Write([]byte("product handler UPDATE endpoint"))
}


```

For  the testing purpose , I am going to use the CURL command. Lets run the following command  and examine the HTTP response 
```
// Test /api/products GET  endpoint 
curl --request GET http://localhost:7000/api/products 
curl --request GET http://localhost:7000/api/products/1 
curl --request POST http://localhost:7000/api/products 
curl --request  DELETE http://localhost:7000/api/products/1 
curl --request  UPDATE http://localhost:7000/api/products/1 

```

#### Configuring database and establish connection to save data in database

I like the concept of single responsibiliy. For that reason, I created a new file` ProductService.go` in **service** folder and `connector.go` in **db** folder. 

**connector.go**

```
package db

import (
	"github.com/jinzhu/gorm"
	_ "github.com/lib/pq"
)

var database *gorm.DB

func init() {
	db, err := gorm.Open("postgres", "user=postgres password=postgres dbname=postgres sslmode=disable")
	if err != nil {
		panic(err.Error())
	}

	database = db
	
}

/*
	Return instance of Db connection
*/
func GetDb() *gorm.DB {
	return database
}

```

It just establish the connection with postgress using **jinzhu/gorm** package and return an instance of **db** which will be used in the service file. Note that, I hardcoded all the parameter to keep the example simple. In real world, we store the database configuration in the environment variable.  But at the end of this blog post , I will perform test and modify code  to make server secure , scalable and elegant. Lets foucs on service layer and develop the basic CURD operation. 

```
package services

import (
	"demoapi/db"
	"log"

	"github.com/jinzhu/gorm"
)

type Product struct {
	gorm.Model
	Name        string
	Description string
	Category    string
	Sku         string
	Price       float32
	ImageURL    string
}

// This funtion is automatically initialized  whenever this  handler  is called
func init() {
	db.GetDb().Debug().DropTable(&Product{})  //Droping the Table to postgress. Do do this in production environment. 
	db.GetDb().Debug().CreateTable(&Product{}) //re-creating the Table
	prod := Product{
		Name:        "Mac book air",
		Description: "Mac product",
		Category:    "laptop",
		Sku:         "sddf",
		Price:       6.6,
		ImageURL:    "https://imaga.com/png",
	}
	db.GetDb().Debug().Save(&prod)   // Inserting sample row. 
}

/*
 Method GetAllProduct
 Returns all the produce in the product database
*/
func (product Product) GetAllProduct() []Product {
	products := []Product{}
	res := db.GetDb().Find(&products)

	if res.Error != nil {
		log.Print(res.Error)
	}
	return products
}

/*
	Method - GetByID(id uint) (*Product, error)
	Description - The function takes product ID as input prameter and returns the product with the relevant parameter
	Erro - In case of err message, it returns err mesage in the response
*/
func (product Product) GetByID(id uint) (*Product, error) {

	prod := Product{}
	res := db.GetDb().Where("id = ?", id).First(&prod)

	if res.Error != nil {
		log.Print(res.Error)
		return nil, res.Error
	}
	return &prod, nil
}

/*
 method - CreateProduct
 create proudct entry in database
*/
func (product Product) CreateProduct(prod *Product) uint {

	res := db.GetDb().Create(prod)

	if res.Error != nil {
		log.Print(res.Error)
	}

	return product.ID

}

/*
method - DeleteProduct(id int) error
Description - Delete the product by ID
*/
func (product Product) DeleteProduct(id uint) error {

	res := db.GetDb().Where("id = ?", id).Delete(&Product{})

	if res.Error != nil {
		log.Print(res.Error)
		return res.Error
	}

	return nil

}
```

`init()` funciton creates the schema and populate the database with a simple record.  gorm `Debug()` method logs the generated SQL command in the consol. It is really helpful as we can look at the SQL command to see exactly gorm transform our code to SQL syntex. Each method is comendted with it is functionality and I think the code is self explanatory. So, I am not going into the details. But in case, if you want to undestand more, I would recommend to look ato gorm official documentation. 

It is time to change our API handler to use service layer and returns the appriate data as HTTP response. 

**ProductHandler.go**
```
package api

import (
	"demoapi/services"
	"encoding/json"
	"log"
	"net/http"
	"strconv"

	"github.com/gorilla/mux"
)

// ProductHandlerGETALL should handle all the http request to product endpoint
func ProductHandlerGETALL(w http.ResponseWriter, r *http.Request) {
	s := services.Product{}
	prod := s.GetAllProduct()
	res, _ := json.Marshal(prod)

	w.Header().Set("Content-Type", "application/json")
	w.WriteHeader(http.StatusOK)

	w.Write([]byte(res))
}

// ProductHandlerGETBYID should handle all the http request to product endpoint
func ProductHandlerGETBYID(w http.ResponseWriter, r *http.Request) {

	vars := mux.Vars(r)
	id := vars["id"]
	s := services.Product{}
	uid, _ := strconv.ParseUint(id, 10, 16)
	prod, err := s.GetByID(uint(uid))

	if err != nil {
		w.WriteHeader(http.StatusNotFound)
		w.Write([]byte(err.Error()))
	}

	res, _ := json.Marshal(prod)

	w.Header().Set("Content-Type", "application/json")
	w.WriteHeader(http.StatusOK)

	w.Write([]byte(res))
}

// ProductHandlerPOST should handle all the http request to product endpoint
func ProductHandlerPOST(w http.ResponseWriter, r *http.Request) {

	product := &services.Product{}
	err := json.NewDecoder(r.Body).Decode(product)

	if err != nil {
		w.Write([]byte("Invliad Request"))
	}
	log.Print(&product)
	id := product.CreateProduct(product)

	w.WriteHeader(http.StatusCreated)
	w.Header().Set("Content-Type", "application/json")

	data, _ := json.Marshal(id)
	w.Write([]byte(data))
}

// ProductHandlerDELETE delete product
func ProductHandlerDELETE(w http.ResponseWriter, r *http.Request) {

	vars := mux.Vars(r)
	id := vars["id"]
	s := services.Product{}
	uid, _ := strconv.ParseUint(id, 10, 16)
	err := s.DeleteProduct(uint(uid))
	w.Header().Set("Content-Type", "application/json")
	if err != nil {
		w.WriteHeader(http.StatusInternalServerError)
		w.Write([]byte(err.Error()))
	}
	w.WriteHeader(http.StatusNoContent)

	w.Write([]byte("product deleted"))
}

// ProductHandlerUPDATE update product
func ProductHandlerUPDATE(w http.ResponseWriter, r *http.Request) {

	w.Write([]byte("product handler UPDATE endpoint"))
}
```

Now, we have a fine grained RESTfull API to perfomr CURD operation. Lets do the test using the CURL command. 

```
curl --request GET http://localhost:7000/api/products
curl --request GET http://localhost:7000/api/products/1

curl -X POST \
  http://localhost:7000/api/products \
  -H 'content-type: application/json' \
  -d '{

        "Name": "Mac book air 2",
        "Description": "Mac product",
        "Category": "laptop",
        "Sku": "sddSf",
        "Price": 7.6,
        "ImageURL": "https://imaga.com/png"
    }'
		
curl -X DELETE http://localhost:7000/api/products/2
```
In the upcoming  blog post, I will show step by step process on how to 

*           Analyzing the source code and potentially tweek the code to make it production ready. 
*           Securing the API using JWT 
* 					Developing the front end application using angular 
* 					Dockerizing the API server and angular application
* 					Deploying the API server and angular for testing 
* 					Scalling the API server using docker swrm
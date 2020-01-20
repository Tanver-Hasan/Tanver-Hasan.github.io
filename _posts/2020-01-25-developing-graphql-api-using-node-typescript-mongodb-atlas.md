---
title: Developing GraphQL API using Node (Typescript) + MongoDB Atlas
category: GraphQL
date: '2020-01-20 09:25:57'
layout: post
comments: true
---

> In this post, I am going to discuss how to develop GraphQL API and consume the GraphQL API from frontend client. 

### Summary 

Yet another framework / mechanism to expose data from backend.  Dont we have RESTFull API already. Every developer spends significant amount of time to understand and develop the RESTfull API. Most organisation uses RESTFull API for production application. Then

> Why we need GraphQL? 

[GraphQL](https://graphql.org/) provides significatnt advantage and flexibility of querying data. It reduces number of  Network request from your frontend to backend which is a win for low powered device. GraphQL is developed by Facebook. Yes, the purpose is clear. It has to handle vast amount of data transfer from backend to froned in inteligent manner.  It also offers the data agreegration facility without any cost. You will get what you ask for. I remember developing DTO  for RESTFull API to reduce the unwanted data exposure. 

Inspiration!

Github uses GraphQL in production. The folloiwing blog post in Github is my source of Inspiration and confidence on GraphQL. 
[https://github.blog/2016-09-14-the-github-graphql-api/](https://github.blog/2016-09-14-the-github-graphql-api/)

Netflix is another comapny adapted Graphql: [https://netflixtechblog.com/our-learnings-from-adopting-graphql-f099de39ae5f](https://netflixtechblog.com/our-learnings-from-adopting-graphql-f099de39ae5f)

Paypal adapted Graphql: [https://medium.com/paypal-engineering/graphql-a-success-story-for-paypal-checkout-3482f724fb53
](https://medium.com/paypal-engineering/graphql-a-success-story-for-paypal-checkout-3482f724fb53
)


To develop the GraphQL API, I will usse Node with typescript. Typescript is another part I love. It helps to minimize the sily mistake we regularly do in javascript. It is a strongly typed language which then compiles to javascript. 

We are not going to spin up our own local database. Rather, I will take advantage of MongoDB atlas. It offers free MongoDB cluster for development purpose.


### Develop Backend 

Let's get started!

Create a project directory in local machine.  Make sure you have `node` and `npm` installed. Run the following command

```

npm init -y      
mkdir src
touch index.ts

``` 

Configure Typescript in the project. 

````
npm install -g typescript
tsc --init  // generate tsconfig.json file
tslint --init
```

In the tsconfig.json file, change **outDir**: **./dist**. Thefore, compiled javascript will be available in the dist folder.  

Add the following code in the tslint.json  file 

```
{
    "defaultSeverity": "error",
    "extends": [
        "tslint:recommended"
    ],
    "jsRules": {},
    "rules": {
        "object-literal-sort-keys": false,
        "no-console":false   // Disable console log warning
    },
    "rulesDirectory": []
}
```


Lets install GraphQL related packages. 

``` 

npm install --save  graphql apollo-server express assert @types/assert mongodb @types/mongodb

```

**package.json** looks like below. Feel free to copy and paste. It contains all the npm modules require for this project. 
```
{
{
  "name": "api",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "build-ts": "tsc",
    "tsc": "tsc",
    "postinstall": "npm run tsc",
    "start": "npm run serve",
    "build": "npm run build-ts",
    "serve": "node alt-src/index.js",
    "watch-node": "nodemon dist/index.js",
    "watch": "concurrently -k -p \"[{name}]\" -n \"Sass,TypeScript,Node\" -c \"yellow.bold,cyan.bold,green.bold\" \"npm run watch-sass\" \"npm run watch-ts\" \"npm run watch-node\"",
    "test": "jest --forceExit --coverage --verbose",
    "watch-test": "npm run test -- --watchAll",
    "watch-ts": "tsc -w",
    "build-sass": "node-sass src/public/css/main.scss dist/public/css/main.css",
    "watch-sass": "node-sass -w src/public/css/main.scss dist/public/css/main.css",
    "tslint": "tslint -c tslint.json -p tsconfig.json",
    "copy-static-assets": "ts-node copyStaticAssets.ts",
    "debug": "npm run build && npm run watch-debug",
    "serve-debug": "nodemon --inspect dist/server.js",
    "watch-debug": "concurrently -k -p \"[{name}]\" -n \"Sass,TypeScript,Node\" -c \"yellow.bold,cyan.bold,green.bold\" \"npm run watch-sass\" \"npm run watch-ts\" \"npm run serve-debug\""
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "@types/compression": "^1.0.1",
    "@types/cors": "^2.8.6",
    "@types/dotenv": "^6.1.1",
    "@types/express": "^4.17.1",
    "@types/jest": "^24.0.18",
    "@types/jsonwebtoken": "^8.3.5",
    "concurrently": "^5.0.0",
    "jest": "^24.9.0",
    "nodemon": "^1.19.4",
    "ts-jest": "^24.1.0",
    "tslint": "^5.20.0",
    "typescript": "^3.6.4"
  },
  "dependencies": {
    "@types/assert": "^1.4.3",
    "@types/auth0": "^2.9.23",
    "@types/check-types": "^7.3.1",
    "@types/helmet": "0.0.44",
    "@types/mongodb": "^3.3.4",
    "@types/winston": "^2.4.4",
    "apollo-link-persisted-queries": "^0.2.2",
    "apollo-server": "^2.9.4",
    "apollo-server-express": "^2.9.4",
    "apollo-server-plugin-response-cache": "^0.3.5",
    "assert": "^2.0.0",
    "auth0": "^2.20.0",
    "body-parser": "^1.19.0",
    "check-types": "^10.0.0",
    "compression": "^1.7.4",
    "cors": "^2.8.5",
    "dotenv": "^8.2.0",
    "express": "^4.17.1",
    "express-graphql": "^0.9.0",
    "express-jwt": "^5.3.1",
    "graphql": "^14.5.8",
    "graphql-subscriptions": "^1.1.0",
    "helmet": "^3.21.1",
    "http": "0.0.0",
    "jsonwebtoken": "^8.5.1",
    "jwks-rsa": "^1.6.0",
    "mongodb": "^3.3.2",
    "subscriptions-transport-ws": "^0.9.16",
    "winston": "^3.2.1"
  },
  "engines": {
    "node": "10.x",
    "npm": "6.x",
    "yarn": "1.x"
  }
}

```


Now, open a terminal and run ``` npm install ```  in the project directory 


To store environment variables, create a file `.env` in the project root directory. This file will be used to store database secret which is mongodb Atlas. I will describe database releated code end of this tutorial.  Currently, .env file looks like below. 

```
MONGO_USERNAME=demouser1
MONGO_PASSWORD=mondodb
NODE_ENV=development
MONGO_URL=mongodb_atlas_connection_url
```

Lets start working on the actual code. At first, create a folder src and create a file index.ts. This file will be responsible to start the graphql server and apply other express midleware. To keep the graphql code seperate , create a folder graphql and create a file server.ts which will contain graphql specific code.  Create another file schema.ts. It contains graphql scema defination.  Entire project structure looks like below. 


![](http://localhost:4000/graphqlprojectstructure.png)



I am not going to discuss all the code. However, comments before each  line  of code should explain the purpose of having those line. Rather, I will only focus on graphql specific code. 

```
import bodyParser from "body-parser";
import compression from "compression";
import * as dotenv from "dotenv";
import express from "express";
import helmet from "helmet";
import { createServer } from "http";
import initGraphqlServer from "./graphql/server";
import { connect } from "./mongo";

// import ENVs from .env (gitignored)
dotenv.config();

async function run() {
    const app = express();
    const PORT = process.env.PORT || 4001;


    // standard express middlewares
    app.use(helmet());
    app.use(compression());
    app.use(bodyParser.urlencoded({ extended: true }));
    app.use(bodyParser.json());

    // connect to MongoDB
    const db = await connect();

    // init graphql server
    const graphqlServer = initGraphqlServer(db);


    // start HTTP server
    const httpServer = createServer(app);

    graphqlServer.applyMiddleware({ app });
    graphqlServer.installSubscriptionHandlers(httpServer);

    httpServer.listen(PORT, () => {
        console.log(
            `🚀 Server ready at http://localhost:${PORT}${graphqlServer.graphqlPath}`
        );
        console.log(
            `🚀 Subscriptions ready at ws://localhost:${PORT}${graphqlServer.subscriptionsPath}`
        );
    });
}

process.on("unhandledRejection", (error) => {
    // Will print "unhandledRejection err is not defined"
    console.log("unhandledRejection", error);
});

run().then(() => console.log(`Server Successfully Started`));

```


The avoboe code starts importing node packages and then  loaded environment variables form `.env` file.   Express middleware is initialized to use express related middleware.  Then, it includes some common express middleare to handle request and response.  `helment` middleare includes the common security header in the http response to avoid common security issue. 

Before start working on graphql code, lets define the product types/interface. In the models folder, create a file product.ts and write following code. 

```
export interface IProduct {
    _id: number;
    title: string;
    category: string;
    price: number;
    ratings: number;
    sku: string;
}
```

initGraphqlServer(db) method is loaded from `./graphql/server` file which contains  graphql specific code and pass the db instances down the chain to be used in the graphql code. Lets look at the server.ts file in the graphql directory. 

```
import { ApolloServer } from "apollo-server-express";
import responseCachePlugin from "apollo-server-plugin-response-cache";
import { Db } from "mongodb";
import { Schema } from "./schema";

export default function server(db: Db) {
    return new ApolloServer({
        schema: Schema,
        subscriptions: {
            path: "/subscriptions",
            onConnect: async () => {
                console.log(`Subscription client connected using Apollo server's built-in SubscriptionServer.`)
            },
            onDisconnect: async () => {
                console.log(`Subscription client disconnected.`);
            },
        },
        cacheControl: {
            defaultMaxAge: 5,
        },
        introspection: true,
        plugins: [responseCachePlugin()],
    });
}

```

This code exposes the server function which takes mongodb conconnections instance as argument and return the ApolloServer instance. ApolloServer implements graphql schema and subscription and other plugins. Follow the ApolloServer documentation for understanding all the options.

`Schema`  is loaded from `schema.ts` files. 


`schema.ts` files contains the follwoing code. 

```
import { GraphQLSchema } from "graphql";
import { RootQueryType } from "./RootQueryType";

export const Schema = new GraphQLSchema({
    query: RootQueryType,
    // mutation: RootMutationType,
    // subscription: RootSubscriptionType
});
```

It initialized graphql schema and expose the schema to be used in the `server.ts` file. mutation and subscription is commented out at this moment. We are going to only implement graphql query first. 

Graphql query is implemented in the `RootQueryType.ts` file. 

```
import { GraphQLList, GraphQLNonNull, GraphQLObjectType, GraphQLString } from "graphql";
import { IProductType } from "./types/IProductType";
import { IProduct } from '../models/IProduct';


export const RootQueryType = new GraphQLObjectType({
    name: "RootQueryType",
    fields: {
        GetDeals: {
            type: new GraphQLList(IProductType),
            resolve(obj, args, ctx) {
                const products: IProduct[] = [
                    {
                        _id: 1,
                        title: "Macbook pro",
                        category: "Computer",
                        price: 1200,
                        sku: "ssfff44",
                        ratings: 4,

                    },
                ];
                return products;
            },
        },
        GetDealById: {
            type: IProductType,
            args: {
                id: { type: new GraphQLNonNull(GraphQLString) },
            },
            resolve(obj, args, ctx) {
                return "single product";
            },
        },
    },
});

```

RootQueryType creates an instance of `GraphQLObject`Type. Each object type should have a name and fileds defination. Fileds can have many filed type. Abobe code implements GetDeals and GetDealById fields. Each field should have type and resolver. Type specify what sort of data this field is going to return and resolve method loads and retuns that data. In this  implementation, I am just returns simple JSON object which is a product type. As we are using typescript, we have to define the fileds type. Lets implement graphql types for product. 

```
import { GraphQLInt, GraphQLObjectType, GraphQLString } from "graphql";

export const IProductType = new GraphQLObjectType({
    name: "IProductType",
    fields: {
        _id: { type: GraphQLString },
        title: { type: GraphQLString },
        category: { type: GraphQLString },
        price: { type: GraphQLInt},
        ratings: { type: GraphQLInt },
        sku: { type: GraphQLString },
    },
});

```

#### Run the project 

Open two seperate terminal . In the first terminal, run `npm run watch-ts` and in the second ternminal run `npm run watch-node` 

It should start the project in port 4001.  Following code should be printed in the terminal. 

```
🔗 Connected to Mongo
Server Successfully Started
🚀 Server ready at http://localhost:4001/graphql
🚀 Subscriptions ready at ws://localhost:4001/subscriptions
```

Navigate to http://localhost:4001/graphql. Apollo Graphql playground should be available to play with graphql API. 

Run a Query like below and execute. 

```
{
  GetDeals{
    _id
    title
    category
  }
}
```
Awoesome! Our graphql server is up and running and query is retuns product data. 

#### Connecting to Mongodb:

In the mongo folder, insert the follwoing code in the index.ts files to establish the connection with mongo atlas cluster and expose the mongo connection instances. 

```
import { Db, MongoClient } from "mongodb";

export async function connect() {
    // tslint:disable-next-line:max-line-length
    const URL = process.env.MONGO_URL || "local mongo instance connection URL";

    const client = new MongoClient(URL, {
        useUnifiedTopology: true,
        useNewUrlParser: true,
    });

    const conn = await client.connect();
    console.log("🔗 Connected to Mongo");
    return conn.db("allocations-prod");
}
```



It is mandatory to whitelist the machine IP address in the mongo atlas console. Otherwise, connection will be refused.

#### Project Repository: 

This project should be available in the follwing Github Repo: 

[https://github.com/Tanver-Hasan/zuul-api-gateway-with-auth/tree/node-graphql](https://github.com/Tanver-Hasan/zuul-api-gateway-with-auth/tree/node-graphql)


{% if site.disqus.shortname %}
  {% include disqus_comments.html %}
{% endif %}
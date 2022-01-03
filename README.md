# Firebase Cloud Functions with GraphQL Query

### Description
This example app demonstrate how to use firebase cloud functions with graphql query using apollo server express to perform CRUD operation.


## What is GraphQL?
>  GraphQL, on the other hand, is an open-source data query and manipulation language for APIs, and a runtime for fulfilling queries with existing data.

## What is Firebase Cloud Functions?
>  Cloud Functions for Firebase is a serverless framework that lets you automatically run backend code in response to events triggered by Firebase features and HTTPS requests.

## What is Apollo Client?
>  Platform which gives the implementation of GraphQL which transfers data between the server (i.e In the tutorial it is Firebase Cloud Functions) and client (iOS App). 

In this article, we will discuss how to set up the Firebase Cloud Function along with the apollo server via apollo-server-express which pairs well with Firebase cloud functions.

>  NOTE: We assume you know basic of [firebase CLI](https://firebase.google.com/docs/cli) and setting up Node projects. 

### Create a Firebase project

```
$ firebase init
```

### Select Firebase Features
We have to select Functions to configure and deploy our cloud functions.

```
❯ ◯ Functions: Configure and deploy Cloud Functions
```

### Configure project setup
Select Create a new project or use the existing one if you already created it from the firebase console.

### Configure functions setup
The language which we like to use to write Cloud Functions.
We are using javascript for this tutorials. 

```
❯ JavaScript
```

### Install node modules & apollo server express dependencies
After project initialization cd to functions dir

```
$ cd functions
$ npm install && npm install apollo-server-express express graphql
```

Now let's jump into the functions directory and update it step by step to serve our GraphQL API.

## Step 1: Expose express API 
In `index.js`,  import express from the `express` app which will go to the onRequest() argument for an HTTP function.

```
const functions = require("firebase-functions");
const express = require("express");

// Setup express cloud function
const app = express();

exports.graphql = functions.https.onRequest(app);
```

## Step 2: Construct a schema
In `schema/schema.js`, import `gql` from apollo-server-express and create a variable called typeDefs for your schema:

```
const { gql } = require("apollo-server-express");

const typeDefs = gql`
  type Employee {
    id: ID!
    name: String
    email: String
    role: String
  }
  
  type Query {
    employees: [Employee]
  }
  
  type Mutation {
    addEmployee(
      id: String!
      name: String!
      email: String!
      role: String!
    ): String
  }
`;

module.exports = typeDefs;

```

>  NOTE: We are using standard Employee schema for these tutorials.

>  The schema will go inside the gql function (between the backticks). The language we'll use to write the schema is GraphQL's schema definition language (SDL). [Reference](https://www.apollographql.com/docs/tutorial/schema/#define-your-schemas-types)

## Step 3: Connect to firebase database
In `database/database.js`, import `admin` from `firebase-admin` for the connection between Firebase and our server app.

```
// To generate a private key file for your service account:
// 1. In the Firebase console, open Settings > Service Accounts.
// 2. Click Generate New Private Key, then confirm by clicking Generate Key.
// 3. Store the JSON file in the `functions/resources/` directory

const admin = require("firebase-admin");

var serviceAccount = require(".././resources/employesss-managment-firebase-adminsdk-hb6cv-c63f14627d.json");

admin.initializeApp({
  credential: admin.credential.cert(serviceAccount),
  databaseURL: "https://employesss-managment-default-rtdb.firebaseio.com"
});

module.exports = admin;
```

## Step 4: Provide Resolvers 
Resolvers provide the instructions for turning a GraphQL operation (a query, mutation, or subscription) into data.

```
// Connect to firebase database
const admin = require("../database/database");

const resolvers = {
  Query: {
    employees: () =>
      admin
        .database()
        .ref("employees")
        .once("value")
        .then((snap) => snap.val())
        .then((val) => Object.keys(val).map((key) => val[key])),
  },
  Mutation: {
    addEmployee: (id, name, email, role) =>
      admin.database().ref("employees").push({
        id: id,
        name: name,
        email: email,
        role: role,
      }).key,
  },
};

module.exports = resolvers;
```

Resolvers copy your TypeDefs and communicate with Apollo Server to find our data. 

Find more about resolvers here. [Official Reference](https://www.apollographql.com/docs/tutorial/resolvers/)

## Step 4: Connecting schema and resolvers to Apollo Server
Here we update `index.js`, to connect schema and resolvers with our Apollo Server.

```
const functions = require("firebase-functions");
const express = require("express");

// Construct a schema, using GraphQL schema language
const typeDefs = require("./schema/schema");

// Provide resolver functions for your schema fields
const resolvers = require("./resolvers/resolvers");

// Create GraphQL express server
const { ApolloServer } = require("apollo-server-express");

// Setup express cloud function
const server = new ApolloServer({
  typeDefs,
  resolvers,
  playground: true,
  introspection: true,
});

const app = express();

server.start().then((res) => {
  server.applyMiddleware({ app, path: "/", cors: true });
  
});

exports.graphql = functions.https.onRequest(app);
```

### Step 5: Import data in firebase console

Add few data to Realtime database to import it in firebase console
```
{
	"employees": [
		{
			"id": 1,
			 "name": "Shahar Shilman",
			 "email": "shahar@test.com",
			 "role": "ADMIN"
		},
		{
			"id": 1,
			"name": "Sheli",
			"email": "sheli@test.com",
			"role": "MANAGER"
		},
		{
			"id": 1,
			"name": "Milan",
			"email": "milan@test.com",
			"role": "EMPLOYEE"
		}
	]
}
```

### Step 6: Test it locally using the emulator
To run the Cloud Functions emulator, use the emulators:start command: [Reference](https://firebase.google.com/docs/functions/local-emulator#run_the_emulator_suite)

```
$ firebase emulators:start

or

$ firebase serve
```

Now we should have a local URL like http://localhost:5001/<project-id>/<region>/graphql

Open this local URL in the browser to view the GraphQL Apollo web interface. 


### Step 7: Deploy it on Firebase
Run this command to deploy our functions i.e graphql on Firebase

```
$ firebase deploy --only functions 
```

That's it. We are done and we successfully deployed our first Firebase Cloud Functions with GraphQL using Apollo Server.

NOTE: Will update this repo to perform more queries like update and delete via a mutation to the GraphQL API.


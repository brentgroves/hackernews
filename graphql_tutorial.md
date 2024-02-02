# Graphql tutorial

## references

<https://www.howtographql.com/graphql-go/0-introduction/>

<git@github.com>:brentgroves/graphql-golang.git

## graphql-go Tutorial - Introduction

Motivation
Go is a modern general purpose programming language designed by google; best known for it’s simplicity, concurrency and fast performance. It’s being used by big players in the industry like Google, Docker, Lyft and Uber. If you are new to golang you can start from golang tour to learn fundamentals.

**[gqlgen](https://gqlgen.com/)** is a library for creating GraphQL applications in Go.

In this tutorial we Implement a Hackernews GraphQL API clone with golang and gqlgen and learn about GraphQL fundamentals along the way. Source code and also this tutorial are available on Github at: <https://github.com/howtographql/graphql-golang> or <git@github.com>:brentgroves/graphql-golang.git

## What is a GraphQL server?

A GraphQL server is able to receive requests in GraphQL Query Language format and return response in desired form. GraphQL is a query language for API so you can send queries and ask for what you need and exactly get that piece of data. In this sample query we are looking for address, title of the links and name of the user who added it:

```graphql
query {
 links{
     title
     address,
     user{
        name
     }
   }
}
```

## response

```graphql
{
  "data": {
    "links": [
      {
        "title": "our dummy link",
        "address": "https://address.org",
        "user": {
          "name": "admin"
        }
      }
    ]
  }
}

```

## Schema-Driven Development

In GraphQL your API starts with a schema that defines all your types, queries and mutations, It helps others to understand your API. So it’s like a contract between server and the client. Whenever you need to add a new capability to a GraphQL API you must redefine schema file and then implement that part in your code. GraphQL has its Schema Definition Language for this purpose. gqlgen is a Go library for building GraphQL servers and has a nice feature that generates code based on your schema definition.

## Getting Started

What are you going to build?

In this tutorial we are going to create a Hackernews clone with Go and gqlgen, So our API will be able to handle registration, authentication, submitting links and getting list of links.

Project Setup
Create a directory for project and initialize go modules file:

```bash
pushd .
cd ~/src
# create repo with only a readme
git clone git@github.com:brentgroves/hackernews.git
cd ~/src/hackernews
go mod init github.com/brentgroves/hackernews
```

2.Add github.com/99designs/gqlgen to your project's tools.go

```bash
printf '// +build tools\npackage tools\nimport (_"github.com/99designs/gqlgen"\n_ "github.com/99designs/gqlgen/graphql/introspection")' | gofmt > tools.go
go mod tidy
```

```go
//go:build tools
// +build tools

package tools

// Manage tool dependencies via go.mod.
// https://pkg.go.dev/cmd/compile
import (
        _ "github.com/99designs/gqlgen"
        _ "github.com/99designs/gqlgen/graphql/introspection"
)

```

Gofmt is a tool that automatically formats Go source code.

What does an underscore in front of an import statement mean?

It's for importing a package solely for its side-effects.
From the Go Specification:
after that use ‍‍gqlgen init command to setup a gqlgen project.

```bash
go run github.com/99designs/gqlgen init
Creating gqlgen.yml
Creating graph/schema.graphqls
Creating server.go
Generating...

Exec "go run ./server.go" to start GraphQL server
go mod tidy
go: downlo//go:generate goyacc -o gopher.go -p parser gopher.y
ading github.com/PuerkitoBio/goquery v1.8.1
go: downloading golang.org/x/net v0.17.0
go: downloading github.com/andybalholm/cascadia v1.3.1
```

More help to get started:

- **[Getting started tutorial](https://gqlgen.com/getting-started/)** - a comprehensive guide to help you get started
- **[Real-world examples](https://github.com/99designs/gqlgen/tree/master/_examples)** show how to create GraphQL applications
- **[Reference docs](https://pkg.go.dev/github.com/99designs/gqlgen)** for the APIs

Here is a description from gqlgen about the generated files:

- **gqlgen.yml** — The gqlgen config file, knobs for controlling the generated code.
- **graph/generated/generated.go** — The GraphQL execution runtime, the bulk of the generated code.
- **graph/model/models_gen.go** — Generated models required to build the graph. Often you will override these with your own models. Still very useful for input types.
- **graph/schema.graphqls** — This is the file where you will add GraphQL schemas.
- **graph/schema.resolvers.go** — This is where your application code lives. generated.go will call into this to get the data the user has requested.
- **server.go** — This is a minimal entry point that sets up an http.Handler to the generated GraphQL server. start the server with go run server.go and open your browser and you should see the graphql playground, So setup is right!

## Defining Our Schema

Now let’s start with defining schema we need for our API. We have two types Link and User each of them for representing Link and User to client, a links Query to return list of Links. an input for creating new links and mutation for creating link. we also need mutations to for auth system which includes Login, createUser, refreshToken(I’ll explain them later) then run the command below to regenerate graphql models.

file: graph/schema.graphqls

```graphql
type Link {
  id: ID!
  title: String!
  address: String!
  user: User!
}

type User {
  id: ID!
  name: String!
}

type Query {
  links: [Link!]!
}

input NewLink {
  title: String!
  address: String!
}

input RefreshTokenInput{
  token: String!
}

input NewUser {
  username: String!
  password: String!
}

input Login {
  username: String!
  password: String!
}

type Mutation {
  createLink(input: NewLink!): Link!
  createUser(input: NewUser!): String!
  login(input: Login!): String!
  # we'll talk about this in authentication section
  refreshToken(input: RefreshTokenInput!): String!
}
```

Now run the following command to regenerate files;

```bash
go run github.com/99designs/gqlgen generate

validation failed: packages.Load: /home/brent/src/hackernews/graph/schema.resolvers.go:54:72: undefined: model.NewTodo
/home/brent/src/hackernews/graph/schema.resolvers.go:54:89: undefined: model.Todo
/home/brent/src/hackernews/graph/schema.resolvers.go:57:62: undefined: model.Todo
```

Note: If you are getting validation failed: packages.Load error. It may occur, because gqlgen uses todo project as starter template. To get rid of this error, edit graph/schema.resolvers.go file and delete functions CreateTodo and Todos. Now run the command again.

After gqlgen generated code for us, we’ll have to implement our schema, we do that in ‍‍‍‍schema.resolvers.go, as you see there is functions for Queries and Mutations we defined in our schema.

## Queries

In the previous section we setup up the server, Now we try to implement a Query that we defined in schema.graphqls.

## What Is A Query

a query in graphql is asking for data, you use a query and specify what you want and graphql will return it back to you.

## Simple Query

open schema.resolvers.go file and take a look at Links function,

func (r *queryResolver) Links(ctx context.Context) ([]*model.Link, error) {

Notice that this function takes a Context and returns slice of Links and an error(if there are any). ctx argument contains the data from the person who sends request like which user is working with app(we’ll see how later), etc.

Let’s make a dummy response for this function, for now.

schema.resolvers.go:

```go
func (r *queryResolver) Links(ctx context.Context) ([]*model.Link, error) {
  var links []*model.Link
  dummyLink := model.Link{
    Title: "our dummy link",
    Address: "https://address.org",
    User: &model.User{Name: "admin"},
  }
 links = append(links, &dummyLink)
 return links, nil
}
```

now run the server with go run server.go and send this query in Graphiql

```bash
pushd .
cd ~/src/hackernews
go run server.go

```

```graphql
query {
 links{
    title
    address,
    user{
      name
    }
  }
}

```

## Mutations

What Is A Mutation
Simply mutations are just like queries but they can cause a data write, Technically Queries can be used to write data too however it’s not suggested to use it. So mutations are like queries, they have names, parameters and they can return data.

## A Simple Mutation

Let’s try to implement the createLink mutation, since we do not have a database set up yet(we’ll get it done in the next section) we just receive the link data and construct a link object and send it back for response!

Open schema.resolvers.go and Look at CreateLink function:

```go
func (r *mutationResolver) CreateLink(ctx context.Context, input model.NewLink) (*model.Link, error) {
// This function receives a NewLink with type of input we defined NewLink structure in our schema.graphqls.

```

This function receives a NewLink with type of input we defined NewLink structure in our schema.graphqls.

Try to Construct a Link object we defined in our schema.graphqls:

```go
func (r *mutationResolver) CreateLink(ctx context.Context, input model.NewLink) (*model.Link, error) {
 var link model.Link
 var user model.User
 link.Address = input.Address
 link.Title = input.Title
 user.Name = "test"
 link.User = &user
 return &link, nil
}
```

now run server and use the mutation to create a new link:

```graphql
mutation {
  createLink(input: {title: "new link", address:"http://address.org"}){
    title,
    user{
      name
    }
    address
  }
}
```

and you will get:

```graphql
{
  "data": {
    "createLink": {
      "title": "new link",
      "user": {
        "name": "test"
      },
      "address": "http://address.org"
    }
  }
}
```

Nice now we know what are mutations and queries we can setup our database and make these implementations more practical.

## Database

Before we jump into implementing GraphQL schema we need to setup database to save users and links, This is not supposed to be tutorial about databases in go but here is what we are going to do:

- Setup MySQL
- Create MySQL database
- Define our models and create migrations

## Setup MySQL

If you have docker you can run Mysql image from docker and use it.

```bash

docker run -p 3306:3306 --name mysql -e MYSQL_ROOT_PASSWORD=dbpass -e MYSQL_DATABASE=hackernews -d mysql:latest 

# now run  and you should see our mysql image is running:

docker ps

CONTAINER ID        IMAGE                                                               COMMAND                  CREATED             STATUS              PORTS                  NAMES
8fea71529bb2        mysql:latest                                                        "docker-entrypoint.s
```

## Create MySQL database

You have already started mysql instance in the previous step. Now we will need to create our hackernews database in that instance. To create the database run these commands.

```bash
docker exec -it mysql bash

# It will open the bash terminal inside mysql instance.

# In the next step we will open mysql repl as the root user:

mysql -u root -p

# It will ask you for root password, enter dbpass and enter.

# Now we are inside mysql repl. To create the database, run this command:

CREATE DATABASE hackernews;
ERROR 1007 (HY000): Can't create database 'hackernews'; database exists
```

## Models and migrations

We need to create migrations for our app so every time our app runs it creates tables it needs to work properly, we are going to use golang-migrate package. Create a folder structure for our database files in the project root directory:

```bash
└─hackernews
|
└── internal
│   └── pkg
│       └── db
│           └── migrations
│               └── mysql
│                   ├── 000001_create_users_table.down.sql
│                   ├── 000001_create_users_table.up.sql
│                   ├── 000002_create_links_table.down.sql
│                   └── 000002_create_links_table.up.sql
```

Install go mysql driver and golang-migrate packages then create migrations:

```bash
mkdir $GOPATH/bin/migrate
go get -u github.com/go-sql-driver/mysql
go build -tags 'mysql' -ldflags="-X main.Version=1.0.0" -o $GOPATH/bin/migrate github.com/golang-migrate/migrate/v4/cmd/migrate/
cd internal/pkg/db/migrations/
migrate create -ext sql -dir mysql -seq create_users_table
migrate create -ext sql -dir mysql -seq create_links_table
```

migrate command will create two files for each migration ending with .up and .down; up is responsible for applying migration and down is responsible for reversing it. open 000001_create_users_table.up.sql and add table for our users:

```sql
CREATE TABLE IF NOT EXISTS Users(
    ID INT NOT NULL UNIQUE AUTO_INCREMENT,
    Username VARCHAR (127) NOT NULL UNIQUE,
    Password VARCHAR (127) NOT NULL,
    PRIMARY KEY (ID)
)
```

in 000002_create_links_table.up.sql:

```sql
CREATE TABLE IF NOT EXISTS Links(
    ID INT NOT NULL UNIQUE AUTO_INCREMENT,
    Title VARCHAR (255) ,
    Address VARCHAR (255) ,
    UserID INT ,
    FOREIGN KEY (UserID) REFERENCES Users(ID) ,
    PRIMARY KEY (ID)
)
```

We need one table for saving links and one table for saving users, Then we apply these to our database using migrate command. Run this command in your project root directory.

```bash
pushd .
cd ~/src/hackernews
migrate -database mysql://root:dbpass@/hackernews -path internal/pkg/db/migrations/mysql up
```

## Next

<https://www.howtographql.com/graphql-go/4-database/>

## Execute **[Stored Procedure with GraphQL](https://stackoverflow.com/questions/73944424/execute-stored-procedure-with-graphql)**

We have database with a lot of stored procedures. I need to implement GraphQL, but I can't find information about is it possible to Execute stored procedures with GraphQL.

GraphQL has nothing to do with stored procedures. It is only a wrapper over your http requests.

In your GraphQL resolvers, you can execute any code that needs to talk to your database.

```graphql
const resolvers = {
  Query: {
    executeInsertReportRequest() {  
      return db.execute('select insert_report_request();');
    }
  }
}
```

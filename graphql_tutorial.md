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

import (
        _ "github.com/99designs/gqlgen"
        _ "github.com/99designs/gqlgen/graphql/introspection"
)

```

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
go: downloading github.com/PuerkitoBio/goquery v1.8.1
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

```graphql
file: graph/schema.graphqls

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

now run the server with go run server.go and send this query in Graphiql:

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

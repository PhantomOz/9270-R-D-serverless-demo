# Serverless Demo

This repository contains the implementation of a demo GraphQL API using the 
[Serverless framework](https://serverless.com/framework/), Node.js, 
[graphql-yoga](https://github.com/prisma-labs/graphql-yoga) and
[Amazon RDS (Aurora MySQL Serverless)](https://aws.amazon.com/rds/aurora/serverless/).

It spins up automatically:
* Lambda handlers for the GraphQL and its playground
* The API Gateway endpoints to access the above
* A VPC, with all the security and networking configuration necessary to allow 
the lambdas to connect to the database
  * Permissions for the Lambdas
  * Internet Gateway attachyed to the VPC
  * Subnets
  * Security Group
  * Route Tables
* An Aurora MySQL Compatible Cluster running in serverless mode

## Getting Started

This is a GraphQL API with the following schema.

```
type Mutation {
  createUser(input: UserInput!): User
}

type Post {
  UUID: String
  Text: String
}

input PostInput {
  Text: String
}

type Query {
  getUser(uuid: String!): User
}

type User {
  UUID: String
  Name: String
  Posts: [Post]
}

input UserInput {
  Name: String
  Posts: [PostInput]
}
```

It has one Query and one Mutation

```
createUser(
  input: UserInput!
): User

getUser(
  uuid: String!
): User
```

### Prerequisites

* An AWS account
* [AWS CLI](https://aws.amazon.com/cli/) installed locally.
* API credentials for your AWS account configured in your AWS CLI locally by running `aws configure`.
* Serverless framework installed locally via `npm -g install serverless`.
* A local MySQL database 

### Install

* Install NPM Dependencies by running `npm install`
* 
* Copy `.env.example` to `.env.development` and set the required environmental variables
  * `AURORA_HOST` set to your local mysql host 
  * `DB_NAME` the name of your local database
  * `USERNAME` the username for your local databases
  * `PASSWORD`  the password of that user

### Running locally

Execute `npm run offline`.    
Head over http://localhost:3000 to access the GraphQL playground.

### Steps to Deploy in AWS

* Create your `.env.production` file.
* Run `NODE_ENV=production serverless deploy`
* The `AURORA_HOST` is going to be automatically generated by [CloudFormation](https://github.com/whiteprompt/9270-R-D-serverless-demo/blob/d27e8ce50f7ca2729632c9700b51e2f7e08aa6a2/serverless.yml#L10-L15)

*DISCLAIMER* After 5 minutes without traffic the Aurora Serverless DB scales 
down to 0 capacity (it goes to sleep). When it gets the first connection after 
it cooled down you might experience a latency of about 20 seconds, because:

* There are no warm containers to run the lambda
* The Serverless RDS Cluster needs to scale up and spin up at least 1 instance

### Steps to Remove All Resources

* Create your `.env.production` file.
* Run `NODE_ENV=production serverless remove`

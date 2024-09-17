# Comprehensive Guide to GraphQL with TypeScript: File Structure and GUI

GraphQL has gained immense popularity in recent years for its flexibility in querying data. It allows clients to request exactly what they need, minimizing over-fetching and under-fetching of data. In this article, we'll dive into the key concepts of GraphQL with a little project, and how to use a GraphQL GUI.


## 1. Introduction to GraphQL
GraphQL allows clients to specify the structure of the response. Instead of having multiple endpoints like in REST, a GraphQL server exposes a single endpoint, and clients can query it to get exactly what they need.

Key Benefits:
- **Efficient data fetching:** Fetch multiple resources in a single request.
- **Strongly typed:** GraphQL is strongly typed, and you define your data structure in the schema.
- **Client-specific queries:** The client decides what data it needs, reducing over-fetching or under-fetching.

Example of a GraphQL query:
```graphql
query {
  user(id: "1") {
    id
    firstName
    email
  }
}
```

## Query
A query in GraphQL is used to fetch data from the server. It is analogous to a GET request in REST APIs. Queries allow clients to specify exactly what data they need, and the server responds with only that data, reducing over-fetching and under-fetching issues.

## Mutation
A mutation in GraphQL is used to modify data on the server, such as creating, updating, or deleting records. It is similar to POST, PUT, or DELETE requests in REST APIs. Mutations can also return data, often including the newly created or updated item.

# 2. Setting Up a GraphQL Project in TypeScript
Follow these steps to set up a GraphQL server using Apollo Server in TypeScript.

## Step 1: Install Dependencies
```bash
npm init -y
npm install apollo-server express graphql
npm install @types/express typescript ts-node nodemon --save-dev
```

## Step 2: Create tsconfig.json
```typescript
{
  "compilerOptions": {
    "target": "ES6",
    "module": "commonjs",
    "strict": true,
    "esModuleInterop": true,
    "outDir": "./dist"
  },
  "include": ["src/**/*.ts"],
  "exclude": ["node_modules"]
}
```
## Step 3: Apollo Server Setup
create **app.ts** file for graphql server setup which we will call in main **index.ts** file. 
```javascript
import { ApolloServer } from '@apollo/server';
import { expressMiddleware } from '@apollo/server/express4';
import express from 'express';
import { types, resolvers } from './user';

export async function initApolloServer(){
    const app = express();
    app.use(express.json());
    const server = new ApolloServer({
        typeDefs:types,
        resolvers: resolvers
      });
      await server.start();
      app.use('/graphql', expressMiddleware(server));
      return app;
}
```

We will see **typeDef** and **resolvers** further.

Now create a main **index.ts** file.
```javascript
import { initApolloServer } from "./app/app";

async function init() {
  const app = await initApolloServer();
  app.listen(8000, () => {
    console.log(`Server is started on PORT 8000`);
  });
}

init();
```

## 3. GraphQL GUI Tools
GraphQL GUIs provide an interactive environment to write and test queries/mutations.
When running Apollo Server, go to http://localhost:8000/graphql to access the built-in Apollo Server interface.

## 4. Our Project File Structure for user signup and signIn function


```plaintext
/src
  /user
    ├── mutation.ts       // Mutations like signup, login
    ├── queries.ts        // Queries like getUser
    ├── types.ts          // GraphQL types (schema)
    ├── resolver.ts       // Resolvers for queries and mutations
    └── index.ts          // Exporting all user-related logic
 └── app.ts              // Main app with Apollo server setup

```

Now we will write the code in every file. First, We will create **types** for out login and signup:

```graphql
export const types = `#graphql
  type User{
    id : ID!
    firstName: String!
    lastName: String
    email: String!
    profileImageUrl: String
    createdAt: String!
    updatedAt: String!
  }
  type AuthPayload {
    token: String!
    user: User!
  }

  type Mutation {
    signup(
      firstName: String!,
      lastName: String,
      email: String!,
      password: String!,
      profileImageUrl: String
    ): User

    login(
      email: String!,
      password: String!
    ): AuthPayload
  }
  type Query {
    sayHello: String
  }
`;
```

**Mutation.ts**
```graphql
import { prisma } from "../../client/db";
import jwt from "jsonwebtoken";
export const mutation = {
  signup: async (
    _: any,
    {
      firstName,
      lastName,
      email,
      password,
      profileImageUrl,
    }: {
      firstName: string;
      lastName?: string;
      email: string;
      password: string;
      profileImageUrl?: string;
    }
  ) => {
    // Check if the user already exists
    const existingUser = await prisma.user.findUnique({
      where: { email },
    });
    if (existingUser) {
      throw new Error("User already exists");
    }
     // Create the new user
     const user = await prisma.user.create({
        data: {
          firstName,
          lastName,
          email,
          password,
          profileImageUrl
        }
      });

      return user;
  },

  login: async(
    _:any,
    { email, password }: 
    {email: string, password: string})=>{
    const user = await prisma.user.findUnique({ where: { email } });
    if (!user) {
      throw new Error('Invalid email or password');
    }
    const valid = password === user?.password;
    if (!valid) {
      throw new Error('Invalid email or password');
    }
    // Generate a JWT token
    const token = jwt.sign({ userId: user.id }, "6261878278", { expiresIn: '1h' });

    return {
      token,
      user,
    };
  }
};
```
**queries.ts**
```graphql
export const queries = {
    sayHello: () => {
        console.log("hello")
        return "Hello from GraphQL server"
    }
  };
```

**resolvers.ts**
```graphql
import { mutation } from './mutation';
import { queries } from './queries';


export const resolvers = {
    Query: {
     ...queries
    },
    Mutation: {
      ...mutation
    }
  };
  ```
**index.ts**
```graphql
import { types } from './types';
import { resolvers } from './resolvers';

export { types, resolvers };
```

This was the baisic code in graphql for signup and login. 

**User Model**
```javascript
model User {
  id    String     @id @default(cuid())
  email String  @unique
  firstName  String
  lastName String?
  profileImageUrl String?
  password String?
  createdAt DateTime @default(now())
   updatedAt DateTime @updatedAt
}
```


## 5. Writing and Testing Queries and Mutations
Here are examples of queries and mutations to run in a GraphQL GUI:

```graphql
mutation Login($email: String!, $password: String!) {
  login(email: $email, password: $password) {
    token,
    user {
      email
    }
  }
}
```
you can query any data what you need, here I only fetched email.
and the variable are
```javascript
{
  "email": "pg150704@gmail.com",
  "password": "123456"
}
```

## Final thought
y structuring your GraphQL project effectively and using TypeScript, you can build scalable, maintainable applications. The interactive testing through GraphQL GUIs like Apollo Server makes it easy to validate your queries and mutations.






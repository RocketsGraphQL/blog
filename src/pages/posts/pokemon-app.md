---
title: "How to build a pokemon collectibles application in React 🤯"
description: "Build production grade pokemon collectibles application using React + Rocketgraph"
pubDate: "2023-11-15"
hero: "/images/pokemon-ai.jpg"
tags: ["astro"]
layout: "../../layouts/BlogPostLayout.astro"
---

## What we'll be building?

We'll be building a Pokemon collectibles app. You can have users signup and a database of Pokemon to choose from. After a user collects a Pokemon, it's his. It's first come first serve.

For this, we'll need

- Authentication for users
- DB to store, users and Pokemon
- Pokemon data
- A mapping or a foreign key to track which Pokemon belongs to who
- Realtime notification system for tracking Pokemon collection
- Front-end in React.js

Luckily we have [Rocketgraph](https://rocketgraph.io) which provides this functionality right out of the box. Let's leverage that to build this much faster.

> 💡To read this article you need to have a basic understanding of React.js and PostgreSQL

## [Rocketgraph](https://rocketgraph.io) : A complete backend that beats Firebase and is open-source

A little background. Rocketgraph provides a complete backend. It comes with a Postgres DB, Hasura console to manage your Postgres and add GraphQL layer to your data, Authentication and Serverless functions.

So to sum it up we provide auth for your web apps, GraphQL for realtime things like messages/notifications/comments etc and Serverless functions for anything you want to offload. Deploy your serverless functions by running `rgraph deploy <project name>` in your repo.

## So what the hack is GraphQL?

GraphQL is a language specification by Meta to achieve realtime queries on your data by asking for exactly what you need. This is different from traditional API approach where the query is coded into the backend and front-end has very little control over what/how to ask for data.

Think of it like a JSON query. You ask for what data you want inn a json-like query and it will return exactly those fields.

In this article we'll leverage the power of GraphQL, React Apollo and Hasura to build a real-time system for rating and commenting on movies. We can use this same system to book movie tickets too.

## TLDR Version

If you just want to see the code. Here is the [code base for this article](https://github.com/RocketsGraphQL/rgraph/tree/master/example-setups/movie-voting). Here you can see [more examples](https://github.com/RocketsGraphQL/rgraph/tree/master/example-setups). This is the [open-source software](https://github.com/RocketsGraphQL/hasura-batteries) behind Rocketgraph

Keep reading

![keep reading gif](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/yma0zrp48i934bppq3pc.jpeg)

## Before you start

Signup on [Rocketgraph](https://rocketgraph.io/signup) and create a project. This will also spin up an AWS RDS Postgres DB that we'll use to store our pokemon data. Once it's complete,

## Cool, let's start from the basics
Create a react project and develop the front-end. Forget about the backend, we'll add it later.


```bash
mkdir pokemon-app && cd pokemon-app
```

Next scaffold a basic react application.

```bash
npx create-react-app ./
```

Install [react router](https://www.npmjs.com/package/react-router-dom) to be able to navigate between pages. Install react-apollo and graphql for real-time stuff as mentioned above.

```bash
yarn add react-router-dom
yarn add @apollo/client graphql
```

Next add some Rocketgraph libraries

```
yarn add @rocketgraphql/react-apollo
yarn add @rocketgraphql/rocketgraph-js-sdk
```

Delete the redundant files such as logo from the project and update the index.js file to as follows:

```js
// src/index.js
import React from "react";
import ReactDOM from "react-dom";
import { Login } from "./components/login";
import { Signup } from "./components/signup";
import { BrowserRouter as Router, Routes, Route } from "react-router-dom";
import App from "./App";
import { RApolloProvider } from "@rocketgraphql/react-apollo";
import { auth } from "./utils/rockets";

ReactDOM.render(
  <React.StrictMode>
      <RApolloProvider auth={auth} gqlEndpoint="https://hasura-vlklxvo.rocketgraph.app/v1/graphql">
        <Router>
          <Routes>
            <Route path="/login" element={<Login />}/>
            <Route path="/signup" element={<Signup />}/>
            <Route path="/" element={<App />} />
          </Routes>
        </Router>
      </RApolloProvider>
  </React.StrictMode>,
  document.getElementById("root")
);
```

And replace your `gqlEndpoint` value with that on your Hasura console here:


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ftu33ce9oc7m4ar2fkcn.png)

App.js

```js
import './App.css';

import { gql, useMutation, useSubscription } from "@apollo/client";


const GET_TODOS = gql`
  subscription {
    pokemon(limit: 100, order_by: {id: asc}) {
      id
      identifier
      user_id
    }
  }
`;

const COLLECT = gql`
  mutation MyMutation($name: String!) {
    update_pokemon(where: {identifier: {_eq: $name}}, _set: {identifier: $name}) {
      affected_rows
    }
  }
`

const PokemonComponent = (poke, func) => {
  return (
    <>
      <li key={poke.id}>{poke.identifier}
        {
          poke.user_id ?
          " collected"
          : <button onClick={() => {console.log(poke); func(poke.identifier)}}>collect</button>
        }
      </li>
    </>
  )
}
function App() {
  const { data, loading } = useSubscription(GET_TODOS);
  console.log("data, loading", data, loading);
  const [collectPoke, { poke_mutate, poke_loading }] = useMutation(COLLECT);
  const collectPokemon = (poke_name) => {
    collectPoke({ variables: { name: poke_name} });
}
  return (
    <div className="App">
        {!data || !data.pokemon || !data.pokemon.length ? (
          "no data"
        ) : (
          <ul id="todosList">
            {data.pokemon.map((poke) => {
              return PokemonComponent(poke, collectPokemon);
            })}
          </ul>
        )}
    </div>
  );
}

export default App;
```

components/signup:

```js
import React, { useState } from "react";
import { useNavigate } from 'react-router-dom';
import { auth } from "../utils/rockets";

export function Signup(props) {
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const navigate = useNavigate();

  async function handleSubmit(e) {
    e.preventDefault();

    // login
    try {
        await auth.register({email, password});
        navigate("/")
      } catch (error) {
        alert("error signing up");
        console.error(error);
        return;
    }

  }

  return (
    <div>
      <h1>Signup</h1>
      <form onSubmit={handleSubmit}>
        <input
          id="inputUserEmail"
          type="email"
          placeholder="email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
        />
        <input
          id="inputUserPassword"
          type="password"
          placeholder="password"
          value={password}
          onChange={(e) => setPassword(e.target.value)}
        />
        <button id="signupButton">Signup</button>
      </form>
    </div>
  );
}
```

## Load the Pokemon data

Let's load the Pokemon data from the open-source db. We'll need a small standalone Python project for that.

Create a folder called `pokemon-python` and a file named `load_pokemon.py`:

```py
import pandas as pd
from sqlalchemy import create_engine
engine = create_engine('postgresql://postgres:password@xxxxx.xxxxx.us-east-2.rds.amazonaws.com:5432/postgres')
df=pd.read_csv('https://raw.githubusercontent.com/veekun/pokedex/master/pokedex/data/csv/pokemon.csv')
df.to_sql('pokemon', engine)
```

```bash
cd pokemon-python
```

```bash
# create a venv
pip install virtualenv
python3 -m venv env
```

```bash
# To activate
source env/bin/activate
```

For an easier and robust deployment of Postgres instances, checkout [Rocketgraph](https://rocketgraph.io/)

```bash
pip3 install pandas
pip3 install sqlalchemy
```

And for good measure

```bash
pip3 freeze > requirements.txt
```

Now simply run

```bash
python3 load_pokemon.py
```

if you get an error such as

> ModuleNotFoundError: No module named 'psycopg2'

simply install it:

```
pip install psycopg2-binary
```

Once you run that command, `psql` into your PostgresDB to ensure that Pokemon data is loaded:

```bash
psql postgresql://postgres:password@restless-pine.xxxxxxxxxxx.us-east-2.rds.amazonaws.com:5432/postgres
```

And:

```
SELECT * FROM pokemon LIMIT 10;
```

You should see a few Pokemon there. Congrats, you loaded the Pokemon data and setup a front-end succesfully.


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/us19o6uv240x2yl1ez7l.png)


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/z7spmvhovql1cjhyb25n.png)

Now, go to public and click on track

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/1ydq9k98l2r8xwxiwtdm.png)



## Now the permissions

Open the Hasura console from your Rocketgraph Hasura tab. We set permissions here so we know which pokemon belongs to whom. Create a role `user` with the following permissions on the permissions tab.

For select:


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/a706cne34u85lcl1s41h.png)

Do without any checks

For update:


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/vz7cf8gmfcma4j3mra3q.png)


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/yh6jpxgtyqik6fsd8mma.png)

We also need to preset column user_id so that it automatically sets it up from our rocketgraph-js-sdk:


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/otjhacmrcqfsvwhvm502.png)


## Aaaand, we are done!!

You can host your app on Vercel, use tailwind to make it look more professional.

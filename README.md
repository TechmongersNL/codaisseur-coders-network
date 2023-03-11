# The Codaisseur Coders Network API

- The API: [https://coders-network-api-okta.herokuapp.com/](https://coders-network-api-okta.herokuapp.com/)
- The code: [https://github.com/Codaisseur/codaisseur-coders-network/tree/okta](https://github.com/Codaisseur/codaisseur-coders-network/tree/okta)

## Introduction

The Codaisseur Coders Network API is just a simple REST API. Sadly, it does not have a UI yet. Will you help us make it? :)

### How-to

We've documented all the available endpoints below. Each endpoint has two examples, one that you can use with [HTTPie](https://httpie.org/) in your terminal, and one that you can use directly in your DevTools. We encourage you to do this!

### Making requests

We recommend you use `axios` to make your requests. Some request examples are:

```js
// inside an async function
try {
  // Simple GET request
  const response = await axios.get(
    "https://coders-network-api-okta.herokuapp.com/hello"
  );
  console.log(response.data);
} catch (error) {
  console.log(error.message);
}

// All these should also be wrapped in async functions and try/catch blocks:
// POST request
const response = await axios.post(
  "https://coders-network-api-okta.herokuapp.com/login",
  { email: "some@email.com" } // these are the body parameters
);

// Authorized post request
const response = await axios.post(
  "https://coders-network-api-okta.herokuapp.com/posts",
  { title: "My new post", content: "lorem ipsum" },
  {
    headers: {
      Authorization: "Bearer <yourJWTtoken>", // Setting the JWT header
    },
  }
);
```

If we want to avoid having to repeat the first part of the URL many times (what is sometimes called the `baseURL`) we can create a preconfigured axios instance. To do this you can create an `axios.js` file, maybe inside the `/store` folder in your frontend. In this file you can do the following:

```js
// /src/store/axios.js
import axios from "axios";

export default axios.create({
  baseURL: "https://coders-network-api-okta.herokuapp.com",
});
```

Then from another file (like the actions files):

```js
import axios from "../axios"; // Import axios from our file, not the library

export const fetchPosts = () => async (dispatch, getState) => {
  try {
    const response = await axios.get("/posts");
    console.log(response.data);
  } catch (e) {
    console.log(e.message);
  }
};
```

### Hello world

_The simplest endpoint of all, just to see if everything's still working._

- [Example](/hello)

- HTTPie:

  `http -v GET https://coders-network-api-okta.herokuapp.com/hello`

- JavaScript:

  ```js
  axios
    .get("https://coders-network-api-okta.herokuapp.com/hello")
    .then((response) => console.log("data", response.data))
    .catch((err) => console.log("err", err));
  ```

- Response:

  ```ts
  {
    message: "Hello world!";
  }
  ```

### Entity types

The API knows of the following entity types:

- Developers
  - Technologies
- Posts
  - Tags
  - Comments
  - Likes

To give you a feeling of what these objects look like, here are some TypeScript definitions for them:

```ts
interface Developer_Meta {
  id: number;
  name: string;
  email: string;
}

interface Developer extends Developer_Meta {
  github_username: null | string;
  website: null | string;
  posts: Post_Meta[];
  technologies: Technology[];
  createdAt: string;
}

interface Technology {
  id: number;
  title: string;
}

interface Post_Meta {
  id: number;
  title: string;
  tags: Tag[];
  post_likes: Like[];
  createdAt: string;
  updatedAt: string;
}

interface Post extends Post_Meta {
  content: string;
}

interface Like {
  developer: Developer_Meta;
  createdAt: string;
}

interface Comment {
  id: number;
  text: string;
  developer: Developer_Meta;
  createdAt: string;
  updatedAt: string;
}
```

## Authentication

Authentication is done via Okta.

### Check whether authenticated / get own profile `GET /me`

- HTTPie:

  `http -v GET https://coders-network-api-okta.herokuapp.com/me Authorization:"Bearer JWT"`

- JavaScript:

  ```js
  axios
    .get("/me", { headers: { Authorization: "Bearer <yourJWTtoken>" } })
    .then((data) => console.log("data", data))
    .catch((err) => console.log("err", err));
  ```

- Response:

  `Developer`

## Posts

### Get a single post `GET /posts/:id`

- [Example](/posts/1)

- HTTPie:

  `http -v GET https://coders-network-api-okta.herokuapp.com/posts/1`

- JavaScript:

  ```js
  axios
    .get(`/posts/1`)
    .then((data) => console.log("data", data))
    .catch((err) => console.log("err", err));
  ```

- Response:

  `Post`

### Like a post `POST /posts/:id/likes`

_This is an authenticated API endpoint._

- HTTPie:

  `http -v POST https://coders-network-api-okta.herokuapp.com/posts/1/likes Authorization:"Bearer JWT"`

- JavaScript:

  ```js
  axios
    .post(
      `/posts/1/likes`,
      {}, // empty body object, no body parameters are sent.
      { headers: { Authorization: "Bearer <yourJWTtoken>" } }
    )
    .then((data) => console.log("data", data))
    .catch((err) => console.log("err", err));
  ```

- Response:

  ```ts
  {
    ok: true;
  }
  ```

### Unlike a post `DELETE /posts/:id/likes`

_This is an authenticated API endpoint._

- HTTPie:

  `http -v DELETE https://coders-network-api-okta.herokuapp.com/posts/1/likes Authorization:"Bearer JWT"`

- JavaScript:

  ```js
  axios
    .delete(`/posts/1/likes`, {
      headers: { Authorization: "Bearer <yourJWTtoken>" },
    })
    .then((data) => console.log("data", data))
    .catch((err) => console.log("err", err));
  ```

- Response:

  ```ts
  {
    ok: true;
  }
  ```

### Get a post's comments `GET /posts/:id/comments`

- [Example](/posts/1/comments)

- HTTPie:

  `http -v GET https://coders-network-api-okta.herokuapp.com/posts/1/comments`

- JavaScript:

  ```js
  axios
    .get(`/posts/1/comments`)
    .then((data) => console.log("data", data))
    .catch((err) => console.log("err", err));
  ```

- Response:

  ```ts
  {
    count: number;
    rows: Comment[];
  }
  ```

### Add a comment `POST /posts/:id/comments`

_This is an authenticated API endpoint. The new comment is made in the name of the user currently logged in._

- HTTPie:

  `http -v POST https://coders-network-api-okta.herokuapp.com/posts/1/comments Authorization:"Bearer JWT" text="Love it!"`

- JavaScript:

  ```js
  axios
    .post(
      `/posts/1/comments`,
      {
        text: "Love it!",
      },
      {
        headers: {
          Authorization: "Bearer <yourJWTtoken>",
        },
      }
    )
    .then((data) => console.log("data", data))
    .catch((err) => console.log("err", err));
  ```

- Response:

  `Comment`

### Get a list of posts `GET /posts`

_Paginated with the optional `offset` and `limit` query parameters._

- [Example](/posts)

- HTTPie:

  `http -v GET "https://coders-network-api-okta.herokuapp.com/posts?offset=1&limit=2"`

- JavaScript:

  ```js
  axios
    .get(`/posts?offset=1&limit=2`)
    .then((data) => console.log("data", data))
    .catch((err) => console.log("err", err));
  ```

- Response:

  ```ts
  {
    count: number;
    rows: Post[];
  }
  ```

### Get a list of posts (by tag) `GET /posts?tag=XYZ`

_Paginated with the optional `offset` and `limit` query parameters._

- [Example](/posts?tag=github)

- HTTPie:

  `http -v GET "https://coders-network-api-okta.herokuapp.com/posts?tag=github"`

- JavaScript:

  ```js
  axios
    .get(`/posts?tag=github`)
    .then((data) => console.log("data", data))
    .catch((err) => console.log("err", err));
  ```

- Response:

  ```ts
  {
    count: number;
    rows: Post[];
  }
  ```

### Get a list of posts (by author) `GET /posts?author=ID`

_Paginated with the optional `offset` and `limit` query parameters._

- [Example](/posts?author=1)

- HTTPie:

  `http -v GET "https://coders-network-api-okta.herokuapp.com/posts?author=1"`

- JavaScript:

  ```js
  axios
    .get(`/posts?author=2`)
    .then((data) => console.log("data", data))
    .catch((err) => console.log("err", err));
  ```

- Response:

  ```ts
  {
    count: number;
    rows: Post[];
  }
  ```

### Create a new post `POST /posts`

_This is an authenticated API endpoint. The new post is made in the name of the user currently logged in._

- HTTPie:

  `http -v POST https://coders-network-api-okta.herokuapp.com/posts Authorization:"Bearer JWT" title="ABC" content="bla bla bla"`

- JavaScript:

  ```js
  axios
    .post(
      "/posts",
      {
        title: "ABC",
        content: "bla bla bla",
      },
      {
        headers: { Authorization: "Bearer <yourJWTtoken>" },
      }
    )
    .then((data) => console.log("data", data))
    .catch((err) => console.log("err", err));
  ```

- Response:

  `Post`

### Update a post `PUT /posts/:id`

_This is an authenticated API endpoint. The new post is made in the name of the user currently logged in._

_You don't have to send all post fields. Only the included fields will be updated._

- HTTPie:

  `http -v PUT https://coders-network-api-okta.herokuapp.com/posts Authorization:"Bearer JWT" title="DEF"`

- JavaScript:

  ```js
  axios
    .put(
      "/posts/1",
      {
        title: "DEF",
      },
      {
        headers: { Authorization: "Bearer <yourJWTtoken>" },
      }
    )
    .then((data) => console.log("data", data))
    .catch((err) => console.log("err", err));
  ```

- Response:

  `Post`

### Delete a post `DELETE /posts/:id`

_This is an authenticated API endpoint. The post must be owned by the user currently logged in._

- HTTPie:

  `http -v DELETE https://coders-network-api-okta.herokuapp.com/posts/1 Authorization:"Bearer JWT"`

- JavaScript:

  ```js
  axios
    .delete("/posts/1", {
      headers: { Authorization: "Bearer <yourJWTtoken>" },
    })
    .then((data) => console.log("data", data))
    .catch((err) => console.log("err", err));
  ```

- Response:

  ```ts
  {
    id: number;
  }
  ```

## Developers

### Create a new developer's profile

_Refer to the [`/signup`](#signup) endpoint above._

### Get a single developer's profile `GET /developers/:id`

- [Example](/developers/1)

- HTTPie:

  `http -v GET https://coders-network-api-okta.herokuapp.com/developers/1`

- JavaScript:

  ```js
  axios
    .get(`/developers/1`)
    .then((data) => console.log("data", data))
    .catch((err) => console.log("err", err));
  ```

- Response:

  `Developer`

### Get a list of developers `GET /developers`

_Paginated with the optional `offset` and `limit` query parameters._

- [Example](/developers)

- HTTPie:

  `http -v GET "https://coders-network-api-okta.herokuapp.com/developers?offset=1&limit=2"`

- JavaScript:

  ```js
  axios
    .get(`/developers?offset=1&limit=2`)
    .then((data) => console.log("data", data))
    .catch((err) => console.log("err", err));
  ```

- Response:

  ```ts
  {
    count: number;
    rows: Developer[];
  }
  ```

### Update your profile `PUT /developers/:id`

_This is an authenticated API endpoint. You can of course only edit your own profile._

- HTTPie:

  `http -v PUT https://coders-network-api-okta.herokuapp.com/developers/1 Authorization:"Bearer JWT" name="Bla" github_username="blabla"`

- JavaScript:

  ```js
  axios
    .put(
      `/developers/1`,
      {
        name: "Bla",
        github_username: "blabla",
      },
      {
        headers: { Authorization: "Bearer <yourJWTtoken>" },
      }
    )
    .then((data) => console.log("data", data))
    .catch((err) => console.log("err", err));
  ```

- Response:

  `Developer`

### Delete your account `DELETE /developers/:id`

_This is an authenticated API endpoint. You can of course only delete your own account._

- HTTPie:

  `http -v DELETE https://coders-network-api-okta.herokuapp.com/developers/1 Authorization:"Bearer JWT"`

- JavaScript:

  ```js
  axios
    .delete(`/developers/1`, {
      headers: { Authorization: "Bearer <yourJWTtoken>" },
    })
    .then((data) => console.log("data", data))
    .catch((err) => console.log("err", err));
  ```

- Response:

  ```ts
  {
    id: number;
  }
  ```

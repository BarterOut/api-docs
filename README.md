# API

### Overview
Our API is built around half of the [MERN](http://mern.io) stack. We use a combination of [Express.js](https://expressjs.com) and [Node.js](https://nodejs.org/en/) to build an HTTP compliant server that meets all general standards and behaves in a [RESTFUL](https://en.wikipedia.org/wiki/Representational_state_transfer) way. This documentation provides a detailed list of all functions along with instructions on their intended purpose and how to use them.

### Table Of Contents:
1. [Authenticating Users](#authenticating-users)
2. [Managing Books](#managing-books)

### Standards
All of our code is compliant with [ESLint](https://eslint.org), modern Javascript and [Web Standards](https://www.w3.org/standards), and all functions are documented internally with [JS Doc](http://usejsdoc.org/about-getting-started.html).

### Important Info
- For endpoints that require authorization, a valid token must be passed through the `Authorization` header in the form:
`Authorization: Token your_token`

If a valid token is not passed in this way, the server will reject the request automatically, throwing an error.


Our standard API response looks like this:
```javascript
{
  data: Object,
  time: new Date(),
  method: String
}
```
*if you parse the response as JSON.

All data will be passed back under `data` and we also provide the API method you called
as well as a time stamp of when you called it.

### Errors
If the API runs into a problem when attempting to complete your request, we will send back a response with a `status` >= `400`. If this is the case, there will be an `error` property under the `data` object in our standard response, providing an explanation as to what the error was.

### User Permissions
All users in the database have a `permissionType` property that is, by default, `0`. This allows them to acces the main part of the website, post books, buy books, etc. Any user with permission type `1` is considered an admin, and can login and view the admin dashboard, comfirming transaction details, viewing statistics, and finding users. Any user with permission type `2` is considered an owner, and as of now, this is the highest level of permission. Owners have the additional privilage of making other users admins, as well as running admin tools on the dashboard, such as the `Unlist Old Books` feature.

## Documentation

### Authenticating Users
Our API uses a node package called JWT (Javascript Web Tokens) to handle authentication. When a user is signed up, we verify all their information and then send them an email to the provided email address. Once they click the link we sent them, we store them in our database, hashing and salting the password and generating a token. We take security very seriously at BarterOut, because of this almost every request made by a client to the API requires a valid token. That token is verified serverside everytime a user makes a request, thus keeping our API and database safe from malicious attackers.

#### Sign Up
To Sign Up a user with an existing account using our API, you must provide a valid `.edu` email address, first and last name, university, CMCBox, Venmo username, and password. Our API then takes that information sends a confirmation email to the specified email address. Once the user clicks the link in the email, their account is verified and they can start using the platform.

##### Sample code in Javascript

```javascript
const data = {
    firstName,
    lastName,
    emailAddress,
    university,
    venmoUsername,
    CMCBox,
    password,
};

fetch('/api/auth/signup', {
    headers: {
        'Accept': 'application/json',
        'Content-Type': 'application/json'
    },
    method: 'POST',
    body: data,
}).then((response) => {
  // Do something with the response...
  // Typically you would parse this as JSON.
});
```

#### Login
To log in a user with an existing account using our API, you must provide an email address and a password. Our API then takes that information and validates it, returning a standard HTTP response with a valid JWT (Javascript Web Token) if the user has been authenticated successfully.

##### Sample code in Javascript

```javascript
const data = { emailAddress, password };

fetch('/api/auth/login', {
    headers: {
        'Accept': 'application/json',
        'Content-Type': 'application/json'
    },
    method: 'POST',
    body: data,
}).then((response) => {
  // Do something with the response...
  // Typically you would parse this as JSON.
});
```

### Managing Books 

We manage books in our database by storing them in two separate groups. One group are books that were posted to be sold, and one is books that were requested by a user. Using this model, you never have to manually check for matches for your user, we do that **automatically**. We use a **status** system in our database to manage the state of a given book. A status of **_0 means the book is available to purchase_**, **_1 means the book is currently in a transaction (someone bought the book on the website but we have yet to confirm the condition and book_**), **_2 means the book has been purchased and is in the delivery process_**. Understanding these statuses is key to using our API.

_***Note:**_ Users are able to add books to a cart before 'checking out'. However, when books are in a user's cart, their
status has not been changed. This means that those books are still avaliable to all other users on the site until
someone clicks 'checkout' in their cart. This is also the case with matched books. It is sort of a 'first come first serve'
strategy and it allows us to simply our managment systems.

#### Getting all available books in the database
To get all of the available books (books being sold that are not yet sold yet) you can optionally provide a JWT, and our API will return a JSON array of book objects*. Note that if you don't provide a token, users will see books that they themselves posted, limiting the user experience.

*See our _[schema documentation](https://github.com/BarterOut/schema-docs)_ for more info.
##### Sample code in Javascript

```javascript
fetch('/api/books/getAllBooks', {
    headers: {
        'Accept': 'application/json',
        'Content-Type': 'application/json',
        'Authorization': `Token ${Token}` // optional
    },
    method: 'GET',
}).then((response) => {
  // Do something with the response... (which is an array of books)
  // Typically you would parse this as JSON.
});
```

#### Searching for specific books
To search for specic groups of books, we ask for a single search query, which would typically be
provided by the user. This is passed to the URL under the query parameter. We then search through
our database, matching books on Title, Course Code, and ISBN.

Like any typical `GET` request to the API, you also need to provide a valid user token.

##### Sample code in Javascript
```javascript
fetch('/api/books/search/:query/', {
    headers: {
        'Accept': 'application/json',
        'Content-Type': 'application/json',
        'Authorization': `Token ${Token}`
    },
    method: 'GET',
}).then((response) => {
  // Do something with the response... (which is an array of books)
  // Typically you would parse this as JSON.
});
```


#### Posting a Book to be Sold
To post a book that you want to sell to the database, you need to provide some key information.
This book will be listed to all users on the homepage, besides the user that posted it.
This includes:

- { **name**: String } _REQUIRED_
- { **edition**: Number } _REQUIRED_
- { **course**: String } _REQUIRED_
- { **price**: Number } _REQUIRED_
- { **condition**: String } _REQUIRED_
- { **date**: Object } _REQUIRED_
- { **Comments**: String }
- { **ISBN**: String }

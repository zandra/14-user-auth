# Walkthrough

Hello and welcome to the User and Passport documents, a deep dive into our application's user creation and login authentication. Refer to the [Application Directory](#application-directory) to see the file structure.

- [Setup and Configuration](#backend-setup-and-configuration)
  - [User Schema](#user-schema)
  - [Passport Configuration](#passport-configuration)
  - [Middleware](#middleware)
    - [Sidebar Sessions](#sidebar-sessions)
  - [Login Status](#login-status)
- [Routing and client-side javascript](#routing-and-client-side-javascript)
- [Application Directory](#application-directory)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>

## Backend Setup and Configuration

### User Schema

Our User schema is defined in the models directory using sequelize. (Although the focus of this walkthrough is our User and authentication configuration, you can refer to our [database documentation](https://sequelize.org/master/manual/model-basics.html) for a deep dive into defining the sequelize model.)

```js
// models/user.js
const User = sequelize.define("User", {
  email: {
    type: DataTypes.STRING,
    allowNull: false,
    unique: true,
    validate: {
      isEmail: true
    }
  },
  password: {
    type: DataTypes.STRING,
    allowNull: false
  }
});
```

Notice that a unique email and password is **required** to create a user record in our database. To securely store and retrieve the user password we use the `bcryptjs` package which is required at the top of our file.

We are able to securely store the user's password by adding a hook before creation and calling Bycrypt's `hashSync()` method. Notice that we first pass in the user's password, then bcrypt's hash `bcrypt.genSaltSync(10)`. The generated hash is then stored in our user's DB as the user is created.

```js
User.addHook("beforeCreate", function(user) {
  user.password = bcrypt.hashSync(user.password, bcrypt.genSaltSync(10), null);
});
return User;
```

And finally we augment our User model to use the bcrypt `hashSync()` method. This will check if the unhashed password entered by the user can be compared to the hashed password stored in our database.

```js
User.prototype.validPassword = function(password) {
  return bcrypt.compareSync(password, this.password);
};
```

#### [Full user schema file can be found here](./models/user.js)

### Passport Configuration

Using passport in our express app is simple as requiring it in our main configuration file located at the directory root: `server.js`. All our the actual configuration for passport's authentication strategy in defined in our config directory.

Passport has multiple configuration options to support many types of authorization needs. Our application's passport configuration is defined within our `config/middleware` directory. Notice that we must require our app's passport strategy at the top of our file; to support our apps username/password authentication, we can require passports `LocalStrategy` module after requiring passport.

```js
// config/passport.js
const passport = require("passport");
const LocalStrategy = require("passport-local").Strategy;
```

The purpose of a verify callback is to find the user that possesses a set of credentials.

The verify callback for local authentication accepts username and password arguments, which are submitted to the application via a login form.

Notice that we can optionally configure the user's sign in as "email" in an object passed to passport.

```js
passport.use(new LocalStrategy(
  {
    usernameField: "email"
  },
```

The request authentication is configured in a function passed to Passport. Here we pass our sign in credentials (email & password) as the first two parameters. In addition to the Passport library, we also need to require our models, as our configuration will check the database for the user's credentials.

```js
const db = require('./models');
/// pass in function with sign in credentials and verify callback
function(email, password, done) {
  db.User.findOne({
    where: {
      email: email
     }
    }).then(function(dbUser) {
        return done(null, dbUser);
    });
  }
```

After parsing the credentials contained in the request, Passport will invoke the verify callback with the credentials as arguments for authentication. The third parameter `done` is the callback used by our middleware (passport) to verify the user as authenticated.

We also return `done` (passing in `false`) to define the circumstances in which the user in not authenticated.

```js
if (!dbUser) {
  return done(null, false, {
    message: "Incorrect email."
  });
} else if (!dbUser.validPassword(password)) {
  return done(null, false, {
    message: "Incorrect password."
  });
}
```

Here if we find no user with the given email or if the user's password is incorrect, then passport will consider the authentication as failed and return.

#### [Full passport configuration can be found here](./config/passport.js)

[Understand serialize deserialize](https://stackoverflow.com/questions/27637609/understanding-passport-serialize-deserialize)

### User Session

In order for our application to persist login sessions, we can configure express to use sessions. Following standard practice in javascript application structure, we define our middleware in the main application file located at our project root: `server.js`. Note here that we use `session()` before initializing passport and using `passport.session()`. This ensures that the login session is restored in the correct order.

```js
//server.js
// Tell express to use session first
app.use(
  session({
    secret: "keyboard cat",
    resave: true,
    saveUninitialized: true
  })
);
// Then we initialize passport and configure passport.session()
app.use(passport.initialize());
app.use(passport.session());
```

#### Sidebar on Sessions

[Sessions](http://www.passportjs.org/docs/username-password/)

In a typical web application, the credentials used to authenticate a user will only be transmitted during the login request. If authentication succeeds, a session will be established and maintained via a cookie set in the user's browser.

Each subsequent request will not contain credentials, but rather the unique cookie that identifies the session. In order to support login sessions, Passport will serialize and deserialize user instances to and from the session.

### Login Status

The final piece to our user authentication setup is handled by the `isAuthenticated` module. This will check if the user is logged in and call `next()` if so; if not, we redirect the user to the login page.

We will look more at this middleware below in the require this middleware in the route file that defines will be used route to

```js
//isAuthenticated.js
if (req.user) {
  return next();
}

return res.redirect("/");
```

## Routing and client-side javascript

###

---

# Application Directory

```

├── dot files (.env, .git, .gitignore)
│
├── config
│   ├── passport.js
│   └── middleware
│         └── isAuthenticated.js
├── db
│   └── schema.sql
│
├── models
│    ├── index.js
│    └── user.js
│
├── node_modules
│ 
├── public
│    ├── stylesheets
│    │   └── style.css
│    ├──js
│    │  ├── login.js
│    │  ├── members.js
│    │  └── signup.js
│    │
│    ├── login.html
│    ├── members.html
│    └── signup.html
│  
├── routes
│    ├── api-routes.js
│    └── html-routes.js
│
├── package.json
│
│
├── server.js
│
└── README.md

```

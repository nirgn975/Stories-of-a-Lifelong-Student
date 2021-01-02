---
title: "Simple Twitter - Chapter 4: Login (Server Side)"
subtitle: ""
date: 2020-11-11T09:00:00+03:00
lastmod: 2020-11-11T09:00:00+03:00
draft: false
author: "Nir Galon"
authorLink: "https://nir.galon.io"
description: ""

tags: ["twitter", "development", "github", "node.js", "expressjs", "ngrx", "jwt", "token", "controller", "model", "router", "mongodb", "seed", "auth", "tests"]
categories: ["development"]

hiddenFromHomePage: false
hiddenFromSearch: false

featuredImage: "/posts/2020/chapter-4-simple-twitter/simple-twitter-cover.webp"
featuredImagePreview: "/posts/2020/chapter-4-simple-twitter/simple-twitter-cover.webp"

toc:
  enable: true
math:
  enable: false
lightgallery: true
license: ""
---

From here on out the posts will be shorter, we'll focus on a single topic in each post and it will probably be on the server or the client and not both of them in the same post.

In this chapter we'll build all of the user related stuff including the auth endpoint so we can login, and the auth middleware so we can protect our endpoints we don't want to make public (only to logged in users). When we create a new model in our server, you'll see we'll also create a dummy data for it - to populate the database with some data so we can use it for tests and also to see some data in the client (in the next chapter).

So, let's head over to our [server repo](https://github.com/nirgn975/simple-twitter-server) and create a new issue named _"User entity"_. This issue will layout all of our tasks:

```markdown
- [ ] User model.
- [ ] User routes.
- [ ] User controllers.
- [ ] User dummy data.
- [ ] Auth endpoints.
- [ ] Auth middleware.
- [ ] User tests for controllers.
- [ ] Run GitHub action CI
```

If you want to follow along with me, it's [issue #5 in my repo](https://github.com/nirgn975/simple-twitter-server/issues/5).

&nbsp;

## 1. Create user model

First thing first we're in Typescript, and we want to have an interface to the `user` model (interfaces help us enforce certain properties on an object).

So let's create a file name `userModel.ts`.

```bash
$ mkdir -p "src/api/user" && touch "src/api/user/userModel.ts"
```

And then import `bcrypt` (we'll need it for the password encryption) and some mongoose stuff.

```typescript
import * as bcrypt from "bcrypt";
import { NextFunction } from "express";
import { Document, Schema, model } from "mongoose";
```

From where we `import` the `bcrypt` package? we didn't install it yet. If you're like me and using the [atom-typescript](https://github.com/TypeStrong/atom-typescript) package, then the `import` line of `bcrypt` will highlight in error. And the error will be _"Cannot find module 'bcrypt' or its corresponding type declarations."_. So, it's time to install the [bcrypt](https://www.npmjs.com/package/bcrypt) package.

```bash
$ npm install --save bcrypt
```

Now we're ready to create the interface. Based on [section 5.1](/2020/07/chapter-2-simple-twitter/#51-users-collection) on chapter 2 we already know what fields we need in the user model.

```typescript
export interface IUserModel extends Document {
  firstName: string;
  lastName: string;
  username: string;
  email: string;
  profileImage: string;
  active: boolean;
  password: string;
  createdAt: Date;
  country: string;
  website: string;
  birthday: Date;
}
```

After the interface is created we can create the user schema with mongoose `Schema` - so we can enforce some properties on the Database level. A mongoose schema just takes a simple object with all the properties.

```typescript
const schema = new Schema({
  firstName: {
    type: String,
  },
  lastName: {
    type: String,
  },
  username: {
    type: String,
    required: true,
    unique: true,
  },
  email: {
    type: String,
    required: true,
    unique: true,
  },
  profileImage: {
    type: String,
  },
  active: {
    type: Boolean,
    required: true,
    default: true,
  },
  password: {
    type: String,
    required: true,
    select: false,
  },
  country: {
    type: String,
  },
  website: {
    type: String,
  },
  birthday: {
    type: Date,
  },
}, { timestamps: true });
```

As you can see we didn't create a `createdAt` field, that's because `{ timestamps: true }` as the second argument of the `Schema` will automatically add a `createdAt` and `updatedAt` fields to the MongoDB document.

Now we need to actually create the mongoose model from the schema.

```typescript
export const User = model<IUserModel>("user", schema, "users", true);
```

As you can see we add a field named `password` and it will be a `String` and not be `select` as default - it's mean it'll not come back when we get the document from the MongoDB database as default. If we do want it, we need to explicitly ask for it in the query.

Also, we'll not save the password as plain text in the database, this is not safe if we have a breach and the database is dumped. We need to add a middleware on every `save` to encrypt the password before we save the document and save the encrypted string instead of the plain text password string. So let's create it before we create the model.

```typescript
/**
 * Password hash middleware.
 */
schema.pre<IUserModel>("save", function(next: NextFunction) {
  if (!this.isModified("password")) return next();
  const salt = bcrypt.genSaltSync(10);
  this.password = bcrypt.hashSync(this.password, salt);
  next();
});
```

The last thing we have to do is to create a `comparePassword` method, so we can compare the encrypted password in our database with the password the user will send us in the login (which is a plain text string). `bcrypt` has a function called [compare](https://github.com/kelektiv/node.bcrypt.js#api) that do all the hard work for us (encrypt the plain text password and compare the two).

```typescript
schema.methods.comparePassword = async (candidatePassword: string, userPassword: string): Promise<boolean> => {
  return await bcrypt.compare(candidatePassword, userPassword);
};
```

And let's not forget to add the method to the `IUserModel` interface, so we can call the `comparePassword` and not get a typescript error.

```typescript
comparePassword: (candidatePassword: string) => boolean;
```

That's it, we have all we need for the user model. Let's commit the code to a new branch and open a PR.

```bash
$ git checkout -b 5
$ git add .
$ git commit -m "Add user model code"
$ git push origin 5
```

(This is the last time I'll use the `git` command and open the GitHub PR manually. From now I'll use the [GitHub CLI](https://github.com/cli/cli) since it now officially on version `1.x.x`, so if you want to follow a long and do exactly what I do install the new GitHub CLI).

Now that we have a commit in the new branch, I can open a new Pull Request and you can follow my commits [here](https://github.com/nirgn975/simple-twitter-server/pull/6).

&nbsp;

## 2. Create user routes

Our Model is ready, but we don't do nothing with it yet. It's time to create some new routes for our `user` entity, and _"register"_ them in the main `app` routes. And as before, we can check our [section 4.1](/2020/07/chapter-2-simple-twitter/#41-users) on chapter 2 to see what routes we need to create.

Like always we need to create the file first

```bash
touch src/api/user/userRoutes.ts
```

And then what we need to do is to `import` the express `Router` and create a `Router`. Then, at the end of the file, let's `export` that `Router` so we can `import` it later in the `app` routes - to register that router (with it's routes).

```typescript
import { Router } from "express";

const router: Router = Router();

export default router;
```

Now, we don't have the controllers yet (the function that will run when we hit each of the endpoints - routes), but let's pretend we already created them and write the controllers names in the right endpoint. We'll add them between the `router` variable to the `export`.

```typescript
router.param("username", usernameParam);

router.route("/:username")
  .get(getUserInfo);

router.route("/feed")
  .get(getOwnFeed);

router.route("/reset-password")
  .post(resetOwnPassword);

router.route("/change-password")
  .put(changeOwnPassword);

router.route("/settings")
  .get(getOwnSettings);

router.route("/change-settings")
  .put(changeOwnSettings);

router.route("/")
  .post(createUser)
  .put(editMyInfo)
  .delete(deleteMe);
```

The first one is a little different from the rest, it's a `param` route. You can see in the other `route`s we have a url param named `username` (you can see it have a `:` before the actual param name, this is the way we do [url parameters in express](http://expressjs.com/en/api.html#router.route)). So the first one (named `param`) is a way to create a middleware that will run first, in each of the endpoints that have this url parameter (named `username`).

This way, when we get to the actual controller (in `getUserInfo` for example), we **know** that `usernameParam` middleware (just a function) already run. In this middleware we'll get the `user` by it's `username` and attach it to the request, so when we'll be in `getUserInfo` controller (again, also, just a function), we just need to get the user from `request.user` to get the user.

Every other route is just an endpoint, where the HTTP method is the method of the route and we chains them one after the other in the same path. For example the last one (`/`) will be the `/user`, and it'll have a POST endpoint, a PUT endpoint, and a DELETE endpoint (all of them on the same `/user` path).

Notice the way we write the endpoints, from the very detailed one to the least detailed one (without any more words in the URL path or parameters). This is because the express router is a lazy one, it doesn't check all of the routes, it check the routes until one of them can answer the requested one and run it's controller.

So if we'll put the `/` at the top, even if we send request to `user/foo` (_"foo"_ here is the `username` param), we'll run the `/user` controller if we had a GET method (in our case it will not going to happen because we don't have a `/user` GET route, and also we prevent that by order the routes correctly).

Now that we have all the routes in place, we need to import the controllers from `userController` (although we don't have any of them yet, and don't even the file itself, let's pretend there is).

```typescript
import {
  usernameParam, getUserInfo, getOwnFeed, resetOwnPassword, changeOwnPassword, getOwnSettings,
  changeOwnSettings, createUser, editMyInfo, deleteMe,
} from "./userController";
```

The final thing in this section is to import the route at the `app.ts` file.

```typescript
import * as userRoutes from "./api/user/userRoutes";
```

And before the root (`/`) route, in the `configureRoutes` function, let's register the `user` router.

```typescript
this.app.use("/user", userRoutes.default);
```

{{< admonition type=warning title="Warning" open=true >}}
Don't forget to commit the code and push it to GitHub.
{{< /admonition >}}

&nbsp;

## 3. Create user controllers

To start this section let's first create the `userController.ts` file.

```bash
$ touch src/api/user/userController.ts
```

Inside it we need to `import` the relevant express stuff, and the `User` and `IUserModel` (user model interface).

```typescript
import { User, IUserModel } from "./userModel";
import { Request, Response, NextFunction } from "express";
```

Let's just create all the controllers function we route earlier in the `userRoutes` file, and leave them empty. Just so we can have all the placeholders to not get errors and run the project.

```typescript
/**
 * PARAM /:username
 * A middleware that find a user by username and attach it to the request.
 */
export let usernameParam = async (request: NewRequest, response: Response, next: NextFunction, username: String) => {
};

/**
 * GET /user/:username
 * Get a user general information.
 */
export let getUserInfo = async (request: Request, response: Response) => {
};

/**
 * GET /user/feed
 * Get the own user tweets feed.
 */
export let getOwnFeed = async (request: Request, response: Response) => {
};

/**
 * POST /user/reset-password
 * Send a reset password email request to reset own password.
 */
export let resetOwnPassword = async (request: Request, response: Response) => {
};

/**
 * PUT /user/change-password
 * Change user own password.
 */
export let changeOwnPassword = async (request: Request, response: Response) => {
};

/**
 * GET /user/settings
 * Get user own settings.
 */
export let getOwnSettings = async (request: NewRequest, response: Response) => {
};

/**
 * PUT /user/change-settings
 * Change user own settings.
 */
export let changeOwnSettings = async (request: Request, response: Response) => {
};

/**
 * POST /user
 * Create a new user.
 */
export let createUser = async (request: Request, response: Response) => {
};

/**
 * PUT /user
 * Edit user own general info.
 */
export let editMyInfo = async (request: Request, response: Response) => {
};

/**
 * DELETE /user
 * Delete user own account.
 */
export let deleteMe = async (request: Request, response: Response) => {
};
```

Now we can run our project

```bash
$ npm start
```

![The project is running again](/posts/2020/chapter-4-simple-twitter/project-is-running-again.webp "The project is running again")

Now we can just keep it running in the background while we develop our controllers one by one. Let's start with `usernameParam` because it's a middleware for a lot of the other controllers, so we need to nail it first.

What we need to do in `usernameParam` is to search a user with the given username, and attach it to the `request` object. If there is no user with this `username` we should return an `404` error.

To attach the user to the request we first need to create a new request interface (with the `user` field). We'll do it right after the `import`s.

```typescript
export interface NewRequest extends Request {
   user: IUserModel;
}
```

Now, we'll change the `request` type to use the `newRequest`, and write the code to add the user to the `request` and then call `next` to finish the middleware function and move on to the next controller function.

```typescript
/**
 * PARAM /:username
 * A middleware that find a user by username and attach it to the request.
 */
export let usernameParam = async (request: NewRequest, response: Response, next: NextFunction, username: string) => {
  const user = await User.findOne({ username });
  if (!user) {
    return response.status(404).json({
      _message: `No user with the given username: ${username}`,
      user: user,
    });
  }

  request.user = user;
  next();
};
```

After we finish with that middleware we can start tackle the controller that use that middleware, the `getUserInfo` function. This controller should get all the user information, based on the given `username`, and just return it. And we already have the `user` in the `request` (which we should change to `newRequest` type), so it's easy - we just need to return it.

```typescript
/**
 * GET /user/:username
 * Get a user general information.
 */
export let getUserInfo = async (request: NewRequest, response: Response) => {
  return response.json(request.user);
};
```

We don't even need to handle any errors, because we already did it in the middleware. Next, we'll skip `getOwnFeed` and `resetOwnPassword` controllers, because they require couple more packages and a bit more complicated, so we'll tackle them later.

So, we can do the `changeOwnPassword` controller. Again, we already have the `user` on the `request`, because this endpoint will be protected with `token` (it's means that the user will have to authenticate first, get a `token` from our server, and send it with the request to that endpoint, with this `token` we can identify the `user` and attach it to the `request`, like we did in the `usernameParam`).

Our frontend will have 2 input fields to confirm the password (the user will have to write it twice), so we need to check that they're match (although we're going to check it on the frontend we need to check it on the backend too, because there are way to change the frontend and send us other requests, so we always need to check all the things again in the server, for security reasons).

After that, we're going to check that the password has a minimum length of 8 and maximum of 30, this is just something we want to enforce on our users, this is some best practices (to make the password long), so if we'll have a breach in the future and our database will be dumped, our users passwords not only encrypted, they're long enough so it will be hard to brute force them.

When those checks are passed we can update the password and return some success message with the new user information.

```typescript
/**
 * PUT /user/change-password
 * Change user own password.
 */
export let changeOwnPassword = async (request: NewRequest, response: Response) => {
  const user =  await User.findById(request.user._id).select("+password");

  // Check that the password and confirmPassword are match.
  if (request.body.password !== request.body.confirmPassword) {
    return response.status(400).json({
      _message: "Cannot update password. Passwords are not matched.",
      user: {},
    });
  }

  // Check that they are in the right length.
  if (request.body.password.length < 8 || request.body.password.length > 30) {
    return response.status(400).json({
      _message: "Password should be more then 8 characters and less the 30.",
      user: {},
    });
  }

  // Update and save the new password.
  user.password = request.body.password;
  const savedCustomer = await user.save();

  return response.json({
    _message: "User password successfully updated!",
    user: savedCustomer,
  });
};
```

The next controllers are the `getOwnSettings` and `changeOwnSettings`, we don't have yet any settings fields, so we'll skip them for the time being. And we get to the `createUser` one. In this endpoint we just create a new `user` entity from the request body data (if we're missing a field that is required `mongoose` schema will take care of that and will throw appropriate error message, we just need to return it).

```typescript
/**
 * POST /user
 * Create a new user.
 */
export let createUser = async (request: Request, response: Response) => {
  const newUser = new User(request.body);

  try {
    const savedUser = await newUser.save();
    return response.json({
      _message: "User successfully created!",
      user: savedUser,
    });
  } catch (error) {
    return response.status(400).json({
      _message: error.message,
      user: {},
    });
  }
};
```

For the `editMyInfo` controller we first need to `import` the `lodash` package (we already installed it in an earlier chapter).

```typescript
import * as _ from "lodash";
```

Before we'll merge the new fields from the `request body` we want to check that the `_id` we received in the `body` is the same as the `user` we got from the `token` (so the user don't try to edit some other users data). After that we can use the [merge function from the lodash library](https://lodash.com/docs/4.17.15#merge) to merge the new fields to the `user` ones.

All is left to do is to save the `user` (with it's new fields), and return it with a success message.

```typescript
/**
 * PUT /user
 * Edit user own general info.
 */
export let editMyInfo = async (request: NewRequest, response: Response) => {
  const user = request.user;
  const update = request.body;

  // Check that the _id are matched.
  if (request.body._id && request.body._id != request.user._id) {
    return response.status(400).json({
      _message: "User can not updated! id is not matched.",
      user: {},
    });
  }

  _.merge(user, update);
  const savedCustomer = await user.save();

  response.json({
    _message: "User successfully updated!",
    user: savedCustomer,
  });
};
```

To delete the user, we just need to use mongoose `remove` document on the `user` attached to the request (because a user only delete it's own account so we identify his account by the token attached to the request).

```typescript
/**
 * DELETE /user
 * Delete user own account.
 */
export let deleteMe = async (request: NewRequest, response: Response) => {
  const deleteUser = await request.user.remove();
  return response.json({
    _message: "User successfully deleted!",
    user: deleteUser,
  });
};
```

And that's it, for now, we're done with all the user controllers we can code with the current code (we only have the `userModel` for now).

&nbsp;

## 4. Create user dummy data file & load dummy data in seed script

The next thing we need to do is to add some dummy data to the user collection and load it in the `seed` script. So, let's create a `users.ts` file in a new `dummyData` directory in `util` path.

```bash
$ mkdir -p "src/util/dummyData" && touch "src/util/dummyData/users.ts"
```

Now all we need is an array with couple of users with some diversification in the fields and to export that array so we can later import it in the `seed.ts` file.

```typescript
const users = [
  { firstName: "Nir", lastName: "Galon", username: "nirgn", email: "nir@example.com",  active: true, password: "12345", country: "israel", website: "https://nir.galon.io" },
  { firstName: "Adi", username: "adi", email: "adi@example.com", active: false, password: "123456", birthday: new Date(Date.now() - (24 * 60 * 60 * 1000)) },
];

export default users;
```

In the `seed` file, we first need to import the `User` model and the `users` variable from the `dummyData/users.ts` file.

```typescript
import users from "./dummyData/users";
import { User } from "../api/user/userModel";
```

We already have a `createUsers` method in the `Seed` class, all we need to do is to loop over the `users` array and create a `User` entity from each and every one of them, and then save it. Then we wrap the `map` with a `Promise.all` so we'll return the actual users documents that mongoose created and return just after all of them was created.

```typescript
/**
 * Create fixtures for users.
 *
 * @class Seed
 * @method createUsers
 * @return void
 */
private createUsers() {
  return Promise.all(users.map(user => {
    return new User(user).save();
  }));
}
```

And the last thing we want to do is to add the number of users that was created to the `console.log`, so we can keep track on how much data was seed to the database (and what was seeded).

```typescript
console.log(chalk.yellow(`Seeded DB with ${users.length} users`));
```

Finally lets not forget to commit and push everything.

&nbsp;

## 5. Create auth endpoints

something..

&nbsp;

## 6. Middleware of auth for relevant endpoints

something..

&nbsp;

## 7. Test the user endpoints

something..

&nbsp;

## 8. Summary

something..

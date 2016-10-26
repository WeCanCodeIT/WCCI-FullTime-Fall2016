# Deploy

## Overview

Create an MVC web application and deploy it.

## Tasks

### Required Tasks

- [ ] Yak Shaving
  - [ ] Create your pair's GitHub organization
  - [ ] Create an MVC project with Individual User Authentication enabled
- [ ] Model
  - [ ] Create code-first class(es) for your model
  - [ ] Create seed data so you can quickly test your model
  - [ ] Enable migrations so you can easily deploy it
- [ ] Controllers/Views
  - [ ] Create appropriate controllers/views
  - [ ] Limit actions based upon whether or not a user is logged-in
- [ ] Deploy
  - [ ] Thoroughly test your application on both group members' machines
  - [ ] Deploy the app to Azure on both group members' accounts

### Stretch Tasks

- [ ] Integrate OAuth to allow users to sign up/sign in with Google, Facebook, or Twitter
- [ ] Extend the functionality of your application
- [ ] Make it look ✨AWESOME✨


## Details

You and your partner will be building and deploying an app from the ground up. The app will involve user authentication and multiple model classes.

- **Chatter**: Chatter is a "microblogging" service that allows users to post "chats": short messages (<150 characters). Users can "follow" other users, and when they visit the site, they see the most recent "chats" from all the users they follow.

You will need to create classes for your model. Create them using code-first to simplify deployment.

You should also create some seed data, so that you can easily test your app.

Once you are confident about your model being able to be used for the MVP of your app, create controller(s) and scaffolded views.

Next, you'll need to limit access to certain views to only logged-in users.


## Stretch Tasks

Make the app better! Allow users to log in using their existing Google/Facebook/Twitter accounts. Make your app look cool. Extend the functionality (but keep MVP in mind).

## Hints

Target your MVP first! Focus on getting the required features together first, and build from there.

Specific MVC hints:

### Changing your Model

Once you've enabled migrations, any time you change your model, you need to run `Add-Migration` and `Update-Database` to "commit" the changes to your database. `Add-Migration` takes a single parameter - the migration name. You can think of this similarly to a commit message explaining what has changed.

### Updating Seed Information

You can run `Update-Database` to re-run the `Seed` method in `Configuration.cs`. This is useful if you're trying to load different starting data!

### Dealing with Users

The `ApplicationUser` class (defined in `IdentityModels.cs`) is the preferred way to access user information. You can add properties (including navigation properties) by putting an extra line in that class.

#### Getting the Currently Logged-In User

You need to check a few things to determine if a user is currently logged-in and, if so, to look up information about the logged-in `ApplicationUser`. Here's a Stack Overflow Q/A about this:

http://stackoverflow.com/questions/35906059/asp-mvc5-identity-how-to-get-current-applicationuser-and-use-this-user-to-que


### Followers

In the Chatter project, one of the challenges is keeping track of "followers". You will need to create/modify the `OnModelCreating` method of the `ApplicationDbContext` class (in `IdentityModels.cs`).

This Stack Overflow question may shed some light:

http://stackoverflow.com/questions/31499265/followers-schema-in-entity-framework

  * [Submission Link](https://goo.gl/forms/klAY4tUOb1Kw9xdq1)*

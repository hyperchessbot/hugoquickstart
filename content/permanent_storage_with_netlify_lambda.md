---
title: "Permanent strage with Netlify lambda"
date: "2021-10-15T18:19:58+01:00"
draft: true
---

## Minimal draft that shows the idea

https://github.com/hyperbotauthor/netlikit

## Full blown implementation in actual app

https://github.com/hyperchessbot/openingtrainer

## Motivation

I was looking for a way of storing the state of my opening trainer ( https://openingtrainer.netlify.app/ ). Being a serverless static site, this requires calling some API. Of course you can set up a server for this API, but it is an overkill, since storing the data is a short API call, that can be easily done with a lambda function ( which is essentially a very short running server, on the order of a few seconds lifetime ). Netlify recently added lambda functions, which are implemented as actual AWS lambda functions behind the scene, however integration with serving a static site, simplicity of setup and the presence of a CLI for development makes Netlify lambda the natural choice.

## Choise for development environment ( Repl.it )

For development GitPod would be the most powerful option, however harsh monthly hours quotas ( even for some paid subscriptions ) may put you in a situation where you cannot open your app any longer till the end of month, so this is a no go. There is a new option Code Sandbox, but we need more sophistication that it offers. So we are left with Repl.it. This is between GitPod and Code Sandbox in complexity. Its editor is a bit simplistic ( cannot turn off auto save, no undo, only one file can be open etc. ), but putting that aside Repl.it is a for always free and available, fairly capable development environment. Recently they added secrets ( abandoning the .env solution for the good ), so configuring the app with auth tokens is easy peasy.

## Setting up Netlify app

Netlify CLI lets you set up your app ( https://docs.netlify.com/cli/get-started/#installation ), however this is intended for a desktop environment, where the CLI can open a browser window, save files to user root ( \~ in linux ). In Repl.it these are not possible. So while you can set up the app ( by opening manually the offered link ), the setup will be lost when the container stops. So either you create the app with the CLI and later provide a permanent credential as a secret, or simply create the app using the Netlify web UI the usual way. Once you have the app, you generate a Netlify access token ( https://app.netlify.com/user/applications/personal ) and store it as `NETLIFY_AUTH_TOKEN` secret. This ensures that you can run a development server and make a test build locally.

## Adding lambda functions

You add lambda functions through `netlify.toml`.

```
[build]
  functions = "netlify/functions"
```

The paradigmatic place of functions is the `netlify/functions` folder. Each function is a Javascript module that exports a handler function.

The handler functions take an `event` and and execution `context` and should return a HTTP response:

```javascript
exports.handler = async function(event, context) {
    return {
      statusCode: 200,
      body: "an example content"
    }
}
```

The request meta data is in `event`. To test whether you are dealing with a POST request, you can do

```javascript
if(event.httpMethod === "POST"){
    // handle POST request here
}
```

The functions are then served at `/.netlify/functions/`, so if your function lives in file `test`, then you can call it at `/.netlify/functions/test`.

## Development server

You can run a dev server with Netlify CLI. This will serve up both the site and the functions. However if you are working with a Nuxt.js app in Repl.it, then you can only have one server running. So if you run the Netlify dev server, then the Nuxt.js site has to be built to dist so that you can serve it up. This is cumbersome, so the recommended approach is to create a minimal working HTML, that can test your function, and when the function is properly working, then create the full blown GUI in Nuxt and publish it for build at Netlify ( it may very well not build in Repl.it due to memory and processing quota ).

If your Nuxt site is simple and you insist on a local function test with the built dist, then to not mix the Netlify and the Nuxt site, you may consider creating a Nuxt site in a subfolder ( say `nuxtsite` ) and build it there. The `netlify.toml` then may end up being

```toml
[build]
  command = "# no build command"
  functions = "netlify/functions"
  publish = "/nuxtsite/dist"
```

## Ups and downs

### Secrets taking effect

#### Repl.it

If you add a secret in Repl.it, it won't take effect until you don't restart the container.

#### Netlify

On netlify this is even more porblematic, because if you set a secret for you app, then you need the next build for it to take effect. So you may end up making a dummy commit, that triggers the update.

### Netlify dev server not serving page ( missing function )

You can get strange missing function response even if you do everything right. Sometimes it is just restarting the container and repeating the dev build may cure this.

## Always reflect secrets both locally and remotely

Is is best when you add a secret locally, do the same remotely, so that you don't get strange errros with your deployed site. It is easy to forget about this and be surprised why the build works locally and not remotely. Also as said keep in mind that you have to make a commit after setting the remote secret.

## Choice for permanent storage GitHub vs. database

### Setup complexity and time

Setting up a database can be very complicated. For example setting up a MongoDb cluster is some job, with a lot of steps and also very time consuming. Setting up a GitHub repo is a few clicks, it even offers a ReadMe and a LICENSE out of the box.

### Startup cost

Starting up a database can take time, and in a lambda function time is the question of life and death. On the other hand GitHub API has no startup cost.

### Complexity of API

Database APIs can be complex, while you can update a file in a GitHub repo out of thin air with a single REST request.

### Storage limit

Databases have storage limits, while in GitHub you can store virtually unlimited data. As you never want to clone the storage repo, size is not a problem.

### Review of stored data

Reviewing what is actually stored in a database requires tools. While in GitHub you can browse everything as files, even the history is there in commits.

### API for Javascript

All common databases have their Javascript APIs, but Octokit, the one for GitHub is especially detailted and well documented.

## Authentication

As the opening trainer works from lichess opening explorer and implements lichess variants, it is straighforward to use the lichess identity for storage authentication. If you use a basic token without special scopes, then it is safe to store the token in local storage. So you generate a token at lichess, copy it into the static app's local storage, then from then on you can store your app state under your lichess username at GitHub. Once you have the blob, you can move your state to an other device at the cost of looking up your token in the old device or simply generating a new token of you don't have your hand on the old device. GitHub is a guarantee that your valuable work will be there forever.

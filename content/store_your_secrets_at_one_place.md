---
title: "Store your secrets at one place"
date: "2021-10-17T10:19:58+01:00"
draft: true
---

## Motivation

Have you run into the problem of having to set up secrets for an online IDE? Access tokens are impossible to remember, also they are only shown once upon creation at your providers and never again. You can open your other project at that online IDE and copy the secrets manually. However this has startup cost, you may be only able to run one container at once, so you have twice the startup cost, not to mention the hassle of manual copying.

Would not it be nice if all your secrets were stored at one secure place, and you could import them into your dev environment at the cost of a password that you CAN remember.

## Implementation of secrets store

This task is ideally suited for a Netlify lambda function. You need not set up a server, because the operation of downloading secrets can run very quickly.

So we need a Netlify lambda function that you POST with your password and in return get a bash script that exports your secrets. You source this script and there you go.

## Set up your Netlify app

You can actualize this project to your needs

https://github.com/hyperbotauthor/netlifyauth

The core function that handles the POST request is

```javascript
exports.handler = async function(event, context) {
  if(event.httpMethod === "POST"){
    const request = JSON.parse(event.body)
    if(request.password === process.env.PASSWORD){      
      const exports = Object.keys(process.env).filter(key => key.match(/_TOKEN$/)).map(key => `export ${key}=${process.env[key]}`).join("\n") + "\n"
      return {
        statusCode: 200,
        body: JSON.stringify({
          status: "ok",          
          exports: exports
        })
      }
    }else return {
      statusCode: 401,
      body: JSON.stringify({
        status: "unauthorized, wrong password"
      })
    }
  } else {
    return {
      statusCode: 200,
      body: "you should use a POST request"
    }
  }    
}
```

## Installation of login

First add your password both to your Netlify lambda app, and to the online IDE project as a secret. Then run the login script https://github.com/hyperbotauthor/fetchstore/blob/main/getlogin.sh with

```
source <(curl -s https://raw.githubusercontent.com/hyperbotauthor/fetchstore/main/getlogin.sh)
```

This will run

```bash
rm login.js
rm login.sh
wget https://raw.githubusercontent.com/hyperbotauthor/fetchstore/main/login.js
wget https://raw.githubusercontent.com/hyperbotauthor/fetchstore/main/login.sh
yarn add --dev node-fetch@2.6.5
```

where `login.js` is

```javascript
const fetch = require("node-fetch")

async function test(){
  //const url = "https://netlifyauth.hyperbotauthor.repl.co/.netlify/functions/netlifyindex"
  const url = "https://mynetlifyauth.netlify.app/.netlify/functions/netlifyindex"  
  const password = process.env.PASSWORD
  console.log("logging in with", password)
  const response = await fetch(url, {
    method: "POST",
    body: JSON.stringify({password})
  })
  const blob = await response.json()
  if(blob.status === "ok"){    
    require('fs').writeFileSync("exports.sh", blob.exports)
    console.log("exports written")
  }else{
    console.log(blob.status)
  }  
}

test()
```

Of course you need to actualize this piece of Javascript code with your own Netlify lambda app host ( replace `mynetlifyauth` with your app name ).

and `login.sh` is

```bash
node login.js
cat exports.sh
. exports.sh
rm exports.sh
```

## Login

Once you installed the login script, the login becomes that simple

```
. login.sh
```

The script will print what environment variables are set.
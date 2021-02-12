---
title: "Create your netlify Hugo blog"
date: "2021-02-12T10:19:58+01:00"
draft: true
---

# Create your Hugo blog and host it on netlify

## What is Hugo?

Hugo is a static site generator, that lets you create your site from markdown content, ideally suited to create a blog. It comes with several templates, that let you customize your experience.

## What is netlify?

Netlify is a static web site hosting platform, very business like at that. It offers several features that let you add dynamics to your static site, including forms, serverless functions, etc. We are only going to use its grass root static hosting capability.

https://www.netlify.com/

## Installing Hugo

We begin with the official quick start guide

https://gohugo.io/getting-started/quick-start/

Immediatly on step one we bump into the problem of how to install Hugo. The quick start guide directs us to Hugo installation page

https://gohugo.io/getting-started/installing

Before we get lost in the forest of all the binary releases, Homebrews, Chocolateys, Scoops and apt-get installs, we are going to do it in the fundamental, platform independent way.

### Prerequisites

We need need `git` and the `Go` language. You can object, how having to install two things to install a third one is simpler than installing the third one right away, well these former tools are more fundamental than Hugo, they are more likely to have their installer for your platform and you can use them for other purposes, it is even possible that you already have them installed.

#### Git

https://git-scm.com/book/en/v2/Getting-Started-Installing-Git

#### Go

https://golang.org/doc/install

### Cloning and installing Hugo from source

Open a console window, cd into a directory outside your `GOPATH` ( to find out what your `GOPATH` is, in the console window type something like `set GOPATH` or `echo $GOPATH` depending on your system ), then

```bash
git clone https://github.com/gohugoio/hugo.git
cd hugo
go install
```

Sit back, this will take a while. The Go compiler will download all the dependencies and build Hugo for your platform.

When ready, type

```
hugo version
```

to check your installation.

## Create your site

```bash
hugo new site myblog
cd myblog
```

## Add a theme

### Download a theme

Initialize your blog folder as a git repository and add the `ananke` theme as a submodule.

```bash
git init
git submodule add https://github.com/budparr/gohugo-theme-ananke.git themes/ananke
```

### Add the theme to your config

Add the line

```
theme = "ananke"
```

to your `config.toml` . Change the base url to `/` and edit the title of the blog.

Your `config.toml` by now should look somthing like

```
baseURL = "/"
languageCode = "en-us"
title = "My Blog"
theme = "ananke"
```

## Create and edit your first post

```bash
hugo new posts/my-first-post.md
```

Edit the post in your favorite text editor.

## Start Hugo development server

```bash
hugo server -D
```

then visit [http://localhost:1313/](http://localhost:1313/) in your browser

# Deploying on netlify

## Sign up

Sign up to netlify if you have not already at https://app.netlify.com/signup .

## Build configuration

The docs and examples available on the net get you close to deploying on netlify, but not quite there. At the time of the writing of this post, this configuration will do:

```toml
[build]
publish = "public"
command = "hugo -D"

[context.production.environment]
HUGO_VERSION = "0.80.0"
HUGO_ENV = "production"
HUGO_ENABLEGITINFO = "true"
```

Save this file as `netlify.toml` in the root of your project ( `myblog` folder in our case ).

## Commit and push your site to git

Create a git repo at GitHub https://github.com/, GitLab https://gitlab.com/ or Bitbucket https://bitbucket.org/.

Visit the repo, copy its link and add it as a remote

```
git remote add origin <your repo link>
```

or

```
git remote add origin <your repo link>.git
```

Then create a commit and push ( you may want to change `master` to `main` depending on what default branch was created for you by `git init` )

```
git add .
git commit -m "Initial commit"

git push --set-upstream origin master
# or
git push --set-upstream origin main
```

## Deploy

Create a new site from git at https://app.netlify.com/start .

Don't worry about your deploy settings, they will be taken care of by your `netlify.toml`.

If everything goes well, upon finishing the required steps your site will be built and you can you can open it for viewing. The site will have an auto generated name, you can change this in Site settings / Site information / Change site name.


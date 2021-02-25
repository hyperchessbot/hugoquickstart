---
title: "Hyper Bot - Your way to a 2600 rated lichess bot"
date: 2021-02-24T06:49:32+01:00
draft: true
---

# What is *Hyper Bot* ?

In short [Hyper Bot](https://github.com/hyperchessbot/hyperbot) is a Javascript program that can play on [lichess](https://lichess.org) as a [bot](https://lichess.org/api#tag/Bot).

It is designed for easy deployment on [Heroku](https://www.heroku.com/), which is a cloud application platform.

Once deployed to Heroku, your bot will accept challenges and play games without having to keep a browser page or a console window open in your computer.

# Why *Hyper Bot* ?

## Self learning book

Hyper Bot can play from a MongoDb hosted book, automatically updated with its own games on every server startup.

## Endgame tablebases

Hyper Bot comes with 3-4-5 piece szyzgy endgame tablebases, which make its handling of endgames superficial. Even bots running on strong hardware may be helpless mating with KNB vs K, but not Hyper Bot, it will find the shortest mate in such a position without hesitation.

## Easy installation and setup

Hyper Bot can be set up without any local installation. Forget about having to install Python, deal with virtual envs, copying config.yml.default.

Every installation step is done online. You only need to set two config variables to configure your bot and have it up and running.

## Free cloud hosting

Hyper Bot is designed to be run on a free Heroku account. You can have such an account for the cost of an email address.

A free Heroku account will have an 550 hours running limit per month and will sleep after cca. half an hour of idleness. Hyper Bot accounts for this by having a mechanism that keeps the bot alive from early morning till late night server time.

## Rich configuration options

As you already learned, Hyper Bot initial configuration is very simple, but you have a wide range of options to configure your bot. This is done through setting environment variables ( called config vars on Heroku ). The ReadMe lists and explains all the configuration options.

# How do I make *Hyper Bot* strong?

By default you will get a bot that plays strong chess, however to compete with the strongest bots default settings are not enough any more, you have to fine tune your bot with additional settings.

## The quick way

To make Hyper Bot really strong, you have to sign up to additional services. You need Heroku CLI and a MongoDb account. If you feel this is a hassle and want a strong bot quickly, Hyper Bot offers you some simple settings that will make your bot play strong.

### Using *Stockfish 13*

By default Hyper Bot uses *Stockfish 12*. This is for backward compatibility. To change this you need the `USE_STOCKFISH_13` setting.

```bash
USE_STOCKFISH_13=true
```

### Threads

Running on a single thread the engine cannot make use of multiple CPUs. To allow harnessing all the computing power of your system, you need to set uci option `Threads` to something more than the default `1`. Natural setting for this variable is the number of physical CPUs of your computer. However on Heroku this works a litle bit different, for a free account the value `4` has proved to be optimal.

```bash
ENGINE_THREADS=4
```

### Hash

Increasing the size of the engine hash table using the `Hash` uci option allows the engine to cache in its search results. Again due to Heroku free account memory limits the value `128` can be considered a good compromise.

```bash
ENGINE_HASH=128
```

### Pondering

By default the engine will think on its own time. For stronger play you have to allow the engine to think on opponent time, and this can be done with the `ALLOW_PONDER` setting.

```bash
ALLOW_PONDER=true
```

### Polyglot book

Hyper Bot comes with a polyglot format book compiled from CCRL > 3300 rated games. To allow using this book you need the `USE_POLYGLOT` setting.

```bash
USE_POLYGLOT=true
```

### Skip fen

Hyper Bot determines the position's fen from the game move list supplied by lichess. This takes some time. It is only strictly necessary for book lookups, so if you are content with the home page not showing positions beyond book depth, you can use the `SKIP_FEN` setting.

```bash
SKIP_FEN=true
```

### Incremental update

Hyper Bot determines the position's fen from the game move list supplied by lichess from scratch. This can be imporved upon for variant standard. To allow incremental update with only the latest move, you can use the `INCREMENTAL_UPDATE` setting.

```bash
INCREMENTAL_UPDATE=true
```

### Disable logging

Hyper Bot can show its currently played live game and the evaluation in its home page and logs in the browser debugger window. This requires using additional resources, if you are after performance, you will want to disable this using the `DISABLE_LOGS` setting.

```bash
DISABLE_LOG=true
```

# The hard part 1 - Setting up and using a *MongoDb book*

## Creating a MongoDb account

Refer to this guide [Create a MongoDb account](/createmongodb) .

Also set `MONGODB_URI` config var as explained there.

## Setting up your MongoDb book

We will need Mongo version 2, so set `MONGO_VERSION` config var accordingly:

```bash
MONGO_VERSION=2
```

We want the bot to use the book, so we need the `USE_MONGO_BOOK` setting:

```bash
USE_MONGO_BOOK=true
```

We will set the book depth to 40 plies using the `BOOK_DEPTH` setting:

```bash
BOOK_DEPTH=40
```

Finally delete any config var that makes the bot use some other book, for example if you have set `USE_BOOK` to `true`, delete this config var, or set it to false. The same applies to use `USE_POLYGLOT`.

```bash
USE_BOOK=false
USE_POLYGLOT=false
```

However if you use the antichess solution, it is ok to leave it there:

```bash
USE_SOLUTION=true
```

### PGN url

#### *Seeding the book*

Hyper Bot can build a book from a *PGN url*. A *PGN url* is a web url that points to a downloadable multi game PGN file. For a self learning book this will be set automaticlly to the lichess API endpoint that lets you download your latest games in PGN format. However building a self learning book from scratch takes a long time, so your are better off seeding the book from some high quality game base. Natural choice is using [TCEC games by month](https://ccrl.chessdom.com/ccrl/4040/games.html#by_month) . It is recommended that you use the latest games, at the time of the writing of this post the latest completed month can be downloaded from https://ccrl.chessdom.com/ccrl/4040/games-by-month/2021-01.bare.[10557].pgn.7z . Notice that we used the `bare` link, this is because we dont't need the comments. Including comments just unnecesarily wastes resoucres, so don't do that. Also note that the file has a `.7z` extension, which means it is a zipped file. In general only a plain PGN file is allowed, but this particular `7z` format is recognized by the bot, so you don't need to unzip and upload the file to the web.

```bash
PGN_URL=https://ccrl.chessdom.com/ccrl/4040/games-by-month/2021-01.bare.[10557].pgn.7z
```

For seeding the book set `MAX_GAMES` to some large number, so that all the games in the link are built at once ( on every server startup only `MAX_GAMES` number of games are built, to allow skipping old, already built games ). Also make sure to keep alive the bot while building the book.

```bash
MAX_GAMES=100000
```

When everyhing is in place sit back and watch the Heroku logs while your book is being built.

#### *Update the book with fresh games*

Delete `PGN_URL` from your Heroku config vars to allow using the bot's own games, and set `MAX_GAMES` to the maximum expected number of daily games played by your bot ( assuming it is started up at least once a day ).

```bash
MAX_GAMES=200
```

# The hard part 2 - Using *syzygy tablebases*

By default your bot is built the convential way on Heroku, using the `heroku` stack and the `Node.js` build pack. However this method is limited in the size of files that can be uploaded. To accomodate *syzygy tablebases* you need the `container` stack and a pre built image that contains the tablebases. At the time of writing the post it does not seem possible to set the stack using the Heroku UI. So you need `Heroku CLI` for this.

## Installing Heroku CLI

https://devcenter.heroku.com/articles/heroku-cli#download-and-install


## Setting stack to container

Open a console ( terminal / command prompt ) window ( if you don't know how to do this on your system, find your way based on a search like this https://www.google.com/search?q=open+terminal+command+prompt+window ).

Open a browser window with your app's Heroku login. This should be the active window when you do the following.

In the console type

```bash
heroku login
```

In the browser popup enable login.

When you are logged into Heroku CLI, in the console type ( change *yourappname* to the actual name of your app )

```bash
heroku stack:set container --app yourappname
```

![](https://i.imgur.com/vWROnFj.png)

### Disabling *syzygy tablebases* when using the `container` stack

By default using a `container` stack involves using the tablebases. If you don't like this, use the `DISABLE_SYZYGY` setting:

```bash
DISABLE_SYZYGY=true
```

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

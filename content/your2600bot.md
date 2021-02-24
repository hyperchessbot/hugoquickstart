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

As you already Hyper Bot initial configuration is very simple, but you have a wide range of options to configure your bot. This is done through setting environment variables ( called config vars on Heroku ). The ReadMe lists and explains all the configuration options.
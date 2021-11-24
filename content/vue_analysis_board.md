---
title: "Vue Analysis Board"
date: "2021-11-24T10:19:58+01:00"
draft: true
---

## [Vue Analysis Board](https://vueanalysisboard.netlify.app)

Visit the above link, log in with lichess and enjoy improved analysis experience.

## Motivation - Shortcomings of lichess analysis

Lighess offers plenty of tools to analyze your games. There is analysis board, that has both lichess book explorer and engine analysis. In addition you can have your games analyzed and annotated by fishnet without having to use your own computer resorces. What more can an analysis tool offer?

Well, even lichess analysis has limitations. Storing evaluations in analysis board is limited to near root moves to spare storage. Multipv is limited to a maximum of 5. The number of games you can analyze per day is limited. The analysis is at some reasonable, yet shallow depth, and sidelines or alternatives are not analyzed in detail. Also in the explorer book you cannot have your subjective annotations for moves, to indiciate which move you prefer in a position, and which you deem bad or losing.

## What added value does Vue Analysis Board offer?

Vue Analysis Board offers remedy to these shortcomings.

### Annotate moves

You can annotate moves on a scale of 0 to 10. The moves will appear color coded, good moves in green, experimental moves in blue and bad moves in red.

![](https://imgur.com/DmznNN2.png)

### Analysis storage

You can analyze arbitrary positions up to multipv 10. This analysis will be stored irrespective of depth.

### Share board screenshots

Vue Analysis Board offers easy sharing of board screenshots. You can upload a screenshot directly to GitHub or download it to your computer. Analysis arrows will be part of the screenshot. You get a markdown link with the moves leading to the positon, so that you can easily import it in a lichess blog post ( forum posts curently not allow GitHub shares ).

### Organizing your games

Switching between your games is made easier. Your last 100 games are loaded and shown in a separate tab. Games are color coded according to result, lost games in red jump out, signalling analysis is necessary. Games against lower rated players are more pale so that you can concentrate on your high profile games. The board remembers your last analyzed game, so when you open it again, you can start from where you finished analyzing your games. In the analysis tab, there is a direct control ( blue << and >> ) that loads your previous or next game. This is already several clicks less than having to switch between lichess pages to get your next game to analyze.

![](https://imgur.com/p0Ma5A6.png)

### Study

While it is not a study sharing tool, you can comment every position and save your variations under an arbitrary name, so in essence you can create a study. However this will be only available within Vue Analysis Board for viewing.

## Markdown editor

Lichess blog feature is really nice. However the editor has problems. If you are after markdown, then every time you open the editor it switches back to wyswyg ( what you see what you get ). Even worse if you accidentally enter markdown or link in wyswyg mode, it is escaped.

The app comes with a markdown editor that allows you to write blog posts. You can save your drafts under dedicated names. It is organized in text blocks that you can drag and drop so that structuring of your text is easy. You have an approximate Html preview ( by approximate I mean that only the basic markdown is implemented, emphasis, link and image, but not nested emphasis, table or more complicated features ). Code highlighting is supported. Every document can have a title and an abstract to reflect lichess blog post structure ( you can even have notes, that won't be rendered to your document ). The below image shows this very blog post edited in Vue Analysis Board markdown editor.

![](https://imgur.com/63zyBlj.png)
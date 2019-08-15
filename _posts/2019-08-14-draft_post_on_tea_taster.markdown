---
layout: post
title:      "Tea Tastes Sinatra App"
date:       2019-08-14 19:39:30 -0400
permalink:  draft_post_on_tea_taster
---


My Sinatra student project app is [_Tea Tastes_, "The Place to Think Tea"](https://github.com/aparkening/tea_tastes). I wanted to create a place to write down and share thoughts about teas in a structured way, revolving around models of users, notes, and shops. Along the way, I expanded the app to include a tea model, and then things really got crazy!

![](https://github.com/aparkening/tea_tastes/blob/master/public/assets/tea-tastes-screenshot.png?raw=true)
 
## App Features
- Anyone can view tea tasting notes, teas, tea shops, and users.
- Authenticated users can create, edit, and delete their own tea notes, as well as any teas and shops.
- Notes are related to users and teas, and teas are related to shops, so it's easy to view all teas in a shop, all notes in a shop, all teas from a specific user, etc.
- Teas are browse-able by additional attributes, such as category, region, and country.

## How

After the vague initial concept, I clarified the design by writing out the functionality from initial page structure down to individual display screens. That helped me determine which routes, models, migrations, and views to build. Check out the detailed development process document in the [Tea Tastes Repository](https://github.com/aparkening/tea_tastes/blob/master/development_process.md). 

Armed with the outline, I built the User, Note, and Shop models, and then wrote the corresponding migrations to test out the database. Once my data was set, I build the first application controller, focusing on User activities (signing up, logging, in, etc.). And after some routes were in place, I built some ERB views templates, iterating between routes and templates until everything worked. I used the same process to get the Notes and Shops routes ship-shape, and then... I made things exponentially harder!

## Better, Stronger, Ugh

The basic interface worked: anyone could view notes and shops, and authenticated users could CRUD all the things appropriately. Notes contained references to teas and where to buy them, but those references were pretty willy-nilly. If Jim and Wanda wrote notes about the same tea, there was no way to know that. A tea model, that's what was missing!

So I mapped out the new model, thinking that Tea would mostly replace Note in how it related to Shop and User; it would probably take a day, tops. And instead it completely changed how all the models related to one another! Notes now belonged to Tea, Shop now had many Teas, and User could have many Teas, through Notes. My brain was going in circles, so wrote out a quick shorthand to keep next to me as I was building and modifying all my routes:

**Old Models**
> - User has many notes
> - Shop has and belongs to many notes
> - Note belongs to user
> - Note has and belongs to many shops

**New Models**
> **User:**
> - Has many Notes
> - Has many Teas, through Notes
> 
> **Note:**
> - Belongs to User
> - Belongs to Tea
> 
> **Tea:**
> - Belongs to Shop
> - Has many Notes
> - Has many Users, through Notes
> 
> **Shop:**
> - Has many Teas
> - Has many Notes, through Teas

The good news was that each model was simpler than before Tea came to town. The bad news was that Notes now needed Tea, Tea needed Shop, and I needed to rebuild most of my creation and edit forms to account for the relationships. 

## Lessons Learned

### Planning is Helpful

In the end, it took _many_ days to account for the new relationship in my models, migrations, routes, and templates. I think the project took much longer than if I had included Tea from the beginning. The software industry often holds agile development above all else, the idea that we should build first, modify later. But in this case, a more comprehensive initial plan would have been more efficient and less stressful.

### Complex can be Cleaner

With Tea in the mix, the app interaction is much cleaner and clearer. It's much easier to understand how notes, shops, and teas interact with one another than when notes were adrift and could have multiple shops attached.

### Committing Often can be Painful

A key takeaway from my last student project was to slow down, to make one change at a time. I mostly stuck to that idea this time around, but I noticed that it took 5 minutes of git adding and committing for every 20 minutes of coding. I kept losing my train of thought for what to do next when breaking to commit my changes. And when I spotted a bug that impacted multiple files, I struggled with whether to commit all of the files at once or one at a time. I'm glad for the constant backup for my work, but I still have plenty of room to improve my git workflow!

### More Time for Style

I focused intently on functionality, my routes, forms, templates, etc. And when I looked up, I had a single day to style everything! I opted to use Bootstrap for quick elegance, it it was tough to both learn and incorporate its features in a couple hours! 

## Done!
This was another fun project. I loved building and interacting with a fully-featured app through my web browser! I learned a ton about database interaction, Sinatra error handling and messaging, and all the manual work involved in making a dynamic app. A++ would do again.

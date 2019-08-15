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
- Notes are related to users and teas, and teas are related to shops, so it's easy to view all teas in a shop, all notes on a tea, all notes from a specific user, etc.
- Teas are browse-able by additional attributes, such as category, region, and country.

## How

After deciding on a concept, I clarified the design by writing out a [detailed list of functionality](https://github.com/aparkening/tea_tastes/blob/master/development_process.md), from initial page structure down to individual display screens. That helped me determine which routes, models, migrations, and views to build.

Armed with my outline, I built the User, Note, and Shop models, and then wrote the corresponding migrations to test out the database. Once my data was set, I build the first application controller, focusing on User activities (signing up, logging, in, etc.). And after some routes were in place, I built some ERB views templates, iterating between routes and templates until everything worked. I used the same process to get the Notes and Shops routes ship-shape, and then... I made things exponentially harder!

## Better, Stronger, Ugh

The basic interface worked: anyone could view notes and shops, and authenticated users could CRUD all the things appropriately. Notes contained references to teas and where to buy them, but those references were pretty willy-nilly. If Jim and Wanda each wrote notes about the same tea, there was nothing to connect the two. A tea model, that's what was missing!

I mapped out the new model, thinking that Tea would mostly replace Note in how it related to Shop and User; I assumed it would probably take a day, tops. Instead it completely changed how all the models related to one another! Notes now belonged to Tea, Shop now had many Teas, and User could be related to many Teas through their Notes. My brain was going in circles, so wrote out a quick shorthand to keep next to me as I was building and modifying all my routes:

**Old Model**
> - User has many notes
> - Shop has and belongs to many notes
> - Note belongs to user
> - Note has and belongs to many shops

**New Model**
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

The good news was that, overall, each model was simpler than before Tea came to town. The bad news was that there were new dependencies: Notes now needed Tea, and Tea needed Shop, so I needed to rebuild most of my creation and edit forms to account for the new relationships. 

## Lessons Learned

### Planning is Helpful

In the end, it took _many_ days to account for the new relationship in my models, migrations, routes, and templates. I think the project took much longer than if I had included Tea from the beginning. The software industry often celebrates agile development, where we build something simple first and add complexity later. But in this case, a more thorough initial plan would have been more efficient and less stressful to build.

### Complex can be Cleaner

With Tea in the mix, the app interaction is much cleaner and clearer. It's much easier to understand how Notes, Shops, and Teas interact with one another than when Notes were adrift and could have multiple Shops attached.

### Committing Often can be Painful

A key takeaway from my first student project was to slow down and make only one change at a time. I mostly stuck to that idea this time around, but I noticed that I was spending 5 minutes git adding and committing for every 20 minutes of coding. I kept losing my train of thought for what to do next when pausing to commit my changes. When I spotted a bug that impacted multiple files, I struggled with whether to commit all of the files at once or one at a time. I'm glad for the constant backup of my work, but I still have plenty of room to improve my git workflow!

### More Time for Style

I focused initially on functionality, my routes, forms, templates, etc. When I looked up, I had only a single day left to style everything! I opted to use Bootstrap for simplicity, but it was a lot to both learn and incorporate its features in only a few hours! 

## Done!
This was another fun project. I loved building and interacting with a fully-featured app through my web browser! I learned a ton about database interaction, Sinatra error handling and messaging, and all the manual work involved in making a dynamic app. I drank approximately 4 gallons of tea while building this app, mostly assam and darjeeling in the mornings, with lighter oolongs in the afternoon. They have all been dutifully recorded in my database.

---
layout: post
title:      "Recipe Costs JavaScript and Rails App"
date:       2019-12-18 01:12:22 +0000
permalink:  recipe_costs_javascript_and_rails_app
---


Professional kitchens need to know their recipe costs to profitably price their food. My [JavaScript](https://github.com/aparkening/recipe_costs_frontend) and [Rails](https://github.com/aparkening/recipe_costs_api) student project, _Recipe Costs_, makes it easy for chefs to figure out the cost-per-serving of their recipes. 
 
## App Features
- Recipe costs are calculated per recipe and per serving. 
- Ingredient conversions are automatic. Recipe amounts entered as cups will  seamlessly take advantage of costs entered as liters or gallons, for example.
- App-wide ingredients provide fast recipe creation, and users can modify and add new ingredients.
- All recipes are browsable.
## Users and Dependencies
I decided to build this app on top of my previous Rails student project, thinking it would provide more time to perfect the JavaScript portion of the app.

My first step was to rebuild the app as a Rails API. While I was at it, I transitioned the database from SQLite to Postgres, so the backend app could be deployed on Heroku. So far so good.

The original app required user accounts for recipes and customized ingredients, so I moved on to user authentication. I spent a couple of promising days with Rails' built-in session cookies, but I wasn't able to find great resources to overcome my errors. I decided to follow my cohort lead's demonstration video for implementing JSON Web Tokens with Devise. Unfortunately, my own models were more complex than the demonstration, so I spent several more days hunting down errors on my own. After hitting a wall of unknowns, I got some initial success by pairing with my cohort lead, but then ran into an authentication wall that my cohort lead couldn't help with. With one of my two project weeks gone, it was time to make the hard call: pare down my app to focus on the MVP.

## MVP for You and Me
Going back to the planning stage allowed me to focust quite a bit, ending up with great functionality using only five models and three controllers. Incrementally updating my models, migrations, and controllers was tougher than starting from scratch, so I simply created the new svelt Rails API  without logic associated with users, recipe ownership, and custom user ingredients. And it worked! I felt so much triumph when the controllers successfully rendered JSON that could be fetched with ease! 

With a healthy Rails API under my belt, I was _finally_ able to jump into the JavaScript work. The frontend work was fairly straightforward, so the good news is that I was actually able to finish the MVP project within the deadline. The bad news is that I had _so little_ time to explore the JavaScript! Using my previous project as a starting point ended up completely backfiring!

## Get Help Early and Often
I was initially very excited about showing off an app that had many bells and whistles revolving around users, very much above and beyond the actual project requirements. And when I hit snags, I exhausted all my other options before asking for help. In hindsight, asking for help much earlier would have allowed me to learn more, have more fun, and complete a better overall project. Lesson learned!

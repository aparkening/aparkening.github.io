---
layout: post
title:      "Recipe Costs JavaScript and Rails App"
date:       2019-12-17 20:12:23 -0500
permalink:  recipe_costs_javascript_and_rails_app
---


Professional kitchens need to know their recipe costs to profitably price their food. My [JavaScript](https://github.com/aparkening/recipe_costs_frontend) and [Rails](https://github.com/aparkening/recipe_costs_api) student project, _Recipe Costs_, makes it easy to determine both the full cost and cost per serving of each recipe.
 
## App Features
- Recipe cost is automatically calculated once ingredients are added. If servings are entered, cost per serving is also calculated.
- Ingredient conversions are automatic. Recipe amounts entered as cups will seamlessly take advantage of costs entered as liters or gallons, for example.
- App-wide ingredients provide fast recipe creation. Users can create, modify, and delete ingredients.
- Recipes can be easily created, modified, and deleted.
- Search bar makes it simple to find recipes by name.

## Users and Dependencies
I decided to build this app on top of my [previous Rails student project](https://github.com/aparkening/recipe_costs), thinking it would provide more time to perfect the JavaScript portion of the app.

My first step was to rebuild the app as a Rails API. While I was at it, I transitioned the database from SQLite to Postgres, so the backend app could be deployed on Heroku. So far so good.

The original app required user accounts for recipes and customized ingredients, so I moved on to user authentication. I spent a couple promising days with Rails' built-in session cookies, but I wasn't able to find great resources to fix my errors. So I decided to follow my cohort lead's demonstration video that instead authenticated with JSON Web Tokens and Devise. Unfortunately, my models were more complex than the demonstration, so I spent several more days hunting down new Devise and JWT errors. After hitting a wall of unknowns, I had some initial success by pairing with my cohort lead. But as the errors became more complex, involving even more pairing time, I felt the stress of only accomplishing a fraction of the backend work in one of my two project weeks. It was time to make the hard call: continue down this road without yet starting the JavaScript portion of the project, or finish a minimum viable product (MVP) Rails backend and move on to JavaScript.

## MVP for You and Me
I returned to the planning stage, determining functionality that would solve the project requirements without authentication. While initially feeling like a failure, this planning forced me to focus, arriving at elegant solutions with only five data models and three controllers. And instead of incrementally updating my models, migrations, and controllers, I was able to build a tighter Rails API project from scratch, without any cruft associated with users, recipe ownership, or custom user ingredients. In very little time, it worked! I felt so much triumph when the controllers successfully rendered useful JSON! 

With a minimal Rails API under my belt, I was _finally_ able to jump into the JavaScript work. The frontend work was fairly straightforward at that point, so I was able to finish the MVP project within the deadline. But! With less than a week of coding time, I had very little time to explore the JavaScript. Using my previous project as a starting point completely backfired!

## Lessons Learned
I was initially excited to show off an app with many bells and whistles around users; very much above and beyond the actual project requirements. The final product ended up smaller and tighter, hitting all the the project requirements with room to expand. Along the way, I learned several important lessons:
1. Stick with code that solves the problem. _Then_ add bells and whistles if time permits.
2. Ask for help sooner. Solving problems on my own is great, but exhausting all other options before asking for help wastes needless time.
3. It's okay to refocus. It's healthy to stop going down a path that isn't working, and helpful to take what I learned to move down a new path.

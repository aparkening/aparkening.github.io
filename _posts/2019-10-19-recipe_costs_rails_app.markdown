---
layout: post
title:      "Recipe Costs Rails App"
date:       2019-10-19 16:08:22 -0400
permalink:  recipe_costs_rails_app
---

Professional kitchens need to know their recipe costs to profitably price their food. My Rails student project, [_Recipe Costs_](https://github.com/aparkening/recipe_costs), makes it easy for chefs to figure out the cost-per-serving of their recipes.
 
## App Features
- Users can log in directly or through their Google account.
- Authenticated users can create, read, update, and delete their own recipes and ingredients.
- Recipe costs are calculated per recipe and per serving. 
- Ingredient conversions are automatic. Recipe amounts entered as cups will  seamlessly take advantage of costs entered as liters or gallons, for example.
- App-wide ingredients and default costs provide faster recipe creation, and users can customize ingredient prices for their own recipes.
- Users can browse their full recipe list or recipes by ingredient.

## Models
After working out the concept, I mapped the data models... and kept at it for many days, trying to sort out what data belonged to a user, how each recipe related to a user and ingredients, etc. The six models that store key data were pretty straightforward: *User, Recipe, Ingredient, UserIngredientCost*, and *WeightVolumeConversions*. And the initial modeled relationships between *Recipe* and *Ingredient* (*RecipeIngredient*) weren't too tricky to figure out, either.

While starting out as a standard join only containing the *Recipe* and *Ingredient* `id`s, I realized that *RecipeIngredient* could also model the specific ingredient amounts and units needed by each recipe. By adding columns for `ingredient_amount` and `ingredient_unit`, kept the join while reducing the need for yet another join table. I created a *super join*!

Smack in the middle of my super join high, however, I hit a wall. Models for app-wide *Ingredient* and user-specific *UserIngredientCost* are lovely, but how was the app going to know which of these ingredients to feed into each recipe?

### Tables Tables Tables
My first thought was to have *Recipe* only relate to *UserIngredientCost*; not *Ingredient*. That would mean that each `user_ingredient` table would start out as a duplicate of the entire `ingredient` table. Which might be fine for five users, but what about 1,000, or 100,000? And what happens when a new item is added to the `ingredient` table? Would I need to sync `ingredient` and `user_ingredient` every day? That seemed like a lot of unnecessary duplicate data and hassle -- there had to be a better way!

I wanted to keep app-wide *Ingredient* data and *UserIngredientCost* data separate, so I noodled this over with my cohort lead. An idea emerged: what if the solution didn't have to be in the database? What if another *object* could operate between *Ingredient* and *UserIngredientCost*, providing a recipe with *UserIngredientCost* data when it existed, but otherwise providing *Ingredient* data? That would mean much smaller `user_ingredient` tables while giving users access to the latest app-wide *Ingredient* data.

### The Ingredient Glue
And it worked! The new *CombinedIngredient* object just needed data from the *RecipeIngredient* super join to find everything needed from *User*, *UserIngredientCost*, and *Ingredient*. And it combined all the right data per instance, so *CombinedIngredient* was always pulling the latest data from all tables. 

![](https://github.com/aparkening/recipe_costs/blob/master/public/images/recipe-costs-data-models.png?raw=true)

### Even Better
This freed me up to tackle the conversion dilemma: how would I normalize ingredient costs across units, like pounds, kilograms, cups, and milliliters? While some items, like molasses, have very specific volume and weight conversions, it didn't make sense to store all possible conversions in the `weight_volume_conversion` table -- that would be so much work to populate and maintain!

But wait. I already had a trusty object that was accessing the data I needed. *CombinedIngredient* was already pulling recipe amounts, ingredients, and costs. So I made it run the weight and volume conversions at the same time, returning both the correct ingredient data and the total ingredient cost. Boom, more work with less effort!

## New Thinking
*CombinedIngredient* solved a lot of my problems, providing maximum flexibility for data that really didn't belong to any single model. In moving away from a solution with tons of duplication, I learned a new way to approach data and logic in Rails. Now my app returns the correct cost for 12oz of flour used in my chocolate chip cookie recipe ($0.32!), even though I purchased flour at $21 per 50lb bag.

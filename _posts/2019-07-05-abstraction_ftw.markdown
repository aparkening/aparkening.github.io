---
layout: post
title:      "Abstraction FTW"
date:       2019-07-05 17:37:40 -0400
permalink:  abstraction_ftw
---


I've been writing increasingly complicated code during my time at Flatiron. My classes and methods can do more, but I also notice that the code feels more brittle. My code does what I ask it to do, but I can't reuse it to do similar tasks elsewhere. 

The most useful thing I've learned to address this has been separating each individual task into its own reusable chunk. While I sometimes resist the process because it takes more initial work, abstracting my tasks always makes it easier to reuse and update my code, saving my future-self oodles of time.

## One Class to Rule Them All
The way I had been approaching complicated code was to combine everything into a single wrapper, moving sequentially through each task. For example, if I was trying to make dinner, I could represent it with a Dinner class:

```
Class Dinner
	Decide what to eat
	Procure ingredients
	Determine task order to serve dishes at the right time (simultaneously, in courses, etc.)
	Gather ingredients
	Measure ingredients
	Cut/process ingredients
	Combine or cook until done
		Add ingredients until done
	Divide items into portions
	Transfer portions to dishes
	Serve dishes to consumers
	Clean preparation tools
	Clean tableware
	Put away tools
	Put away tableware
end
```

This approach will get food on the table, yay! But the overall process isn't flexible:
* What if I want to clean our preparation tools while dinner is cooking?
* What about marinating or precooking any items ahead of time?
* How do I add drinks, locations, or other variables to the mix? 

And then I need to do all this again tomorrow? Ugh, this is way too complicated!

## Abstract, Then Assemble
Instead, I can divide these tasks into chunks that can be mixed and reused by multiple Meal instances:

```
Class Meal 
	Plan
	Procure
	for each stored component part
		Prepare
		Cook
		Serve
	end
	Clean
end

Class Plan
	Determine amount of consumers
	Determine amount of processing people (cooks, cleaners, etc.)
	Identify component parts (courses, starches, vegetables, etc.)
	Scale component parts based on amount of consumers and processors
	Mark component order of operations
	Mark ingredients and tools on hand
	Mark external ingredients and tools to procure
	Determine procurement locations (garden, store, neighbor, etc.)
	Mark procurement locations for each item
end

Class Procure
	For each marked location, procure external ingredient and tool item
end

etc.

```

Each meal is now much simpler! I can:
* Clean a few dishes in the middle by adding another instance of Clean before Serve. 
* Introduce marinades and drinks by adding them to Plan, and then automatically take advantage of Procure, Prepare, Cook, Serve, and Clean.
* Divide our meal labor more efficiently. Cleaners don't need to Cook, cooks don't need to Procure, etc.

And beyond each specific meal, abstracting my steps means I improve the flexibility of _all_ meals. Now I can:
* Call Meal any time I need it, rather than just dinner time.
* Inherit Meal into instances of Lunch, Elevensies, etc. if specific meals have individual needs.
* Save effort by thinking about mulitple meals. Since Plan stores its results, I can both Plan and Procure for multiple Meals at once. 

Abstracting tasks isn't always easy, but the results are so much more versatile!

## Try It Yourself
If you've been writing giant single-class programs and want to try abstracting more, here are two questions I ask to determine whether a task might be be abstractable:

* Could this functionality be used again later? Preparing lunch uses the same steps as preparing dinner, for example.
* Do I repeat this step multiple times throughout the process? Cleaning the cutting board between each type of ingredient, for example.

As a bonus, once you've abstracted into smaller chunks, it's much easier to expand later. Smaller chunks are easier to understand, and there are fewer ripple effects when you make changes. When it's time to update the `Serve` class to pour soup into a bowl (not a plate!), you'll be glad you can make a discrete simple change to that class instead of the entire `Dinner`.

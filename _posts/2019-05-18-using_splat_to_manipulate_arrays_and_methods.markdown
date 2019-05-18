---
layout: post
title:      "Using Splat (*) to Manipulate Arrays and Methods"
date:       2019-05-18 14:07:26 -0400
permalink:  using_splat_to_manipulate_arrays_and_methods
---

The splat operator (`*`) is the most flexible Ruby tool I've come across. Not sure how many inputs your method will receive? Want a flexible shortcut to create, join, or break up arrays? What about a handy way to pass arguments to methods? Splat can help.

## Array manipulation
Fundamentally, the splat operator manipulates arrays. It builds, flattens, and separates arrays into component parts with minimal fuss.

### Construction and conversion
Splat converts strings, numbers, floats, and hashes into arrays, either directly or through variables.

```
a = *"The barn"
#=> ["The barn"]
```

```
a = *1,2,3
or 
a = *(1..3)
#=> [1, 2, 3]
```

```
f = *1.1, 2.2, 3.3
a = *f
#=> [1.1, 2.2, 3.3]
```

```
s = "How much hay is enough?"
a = *s
#=> ["How much hay is enough?"]
```

```
a = *{:Rabbits => 2, :Pigs => 3, :Hay => 0}
#=> [[:Rabbits, 2], [:Pigs, 3], [:Hay, 0]]
```

Interestingly, splat can also convert `nil` into an empty array. Something we can't do when calling `Array.new(nil)`, for example.
```
a = *nil
#=> []
```

As you may have guessed, splat treats existing arrays as pass-through arguments. 
```
a = *["pitch fork", "watering bucket"]
#=> ["pitch fork", "watering bucket"]
```

### Flattening
Outside of creating new arrays, splats quickly flatten arrays.

```
a = *["ducks", "geese"], *["pigs", "cows"]
or 
a = [*["ducks", "geese"], *["pigs", "cows"]]
#=> ["ducks", "geese", "pigs", "cows"]
```

```
poultry = *"ducks", "geese"
porcine_bovine = *"pigs", "cows"
a = *poultry, *porcine_bovine
#=> ["ducks", "geese", "pigs", "cows"]
```

### Destructuring
One of my favorite splat uses is breaking down arrays into component parts, otherwise known as destructuring.

We know array elements can be assigned with commas:
```
small, medium, large = ["bantam chicken", "cayuga duck", "weeder goose"] 
#=> small = "bantam chicken"
#=> medium = "cayuga duck"
#=> large = "weeder goose"
```

Splat can assign these values implicitly, in many positions:

```
small, *large = ["bantam chicken", "cayuga duck", "weeder goose", "pig"] 
#=> small = "bantam chicken"
#=> large = ["cayuga duck", "weeder goose", "pig"]
```

```
*various, large = ["bantam chicken", "cayuga duck", "weeder goose", "pig"] 
#=> various = ["bantam chicken","cayuga duck", "weeder goose"]
#=> large = "pig"
```

```
small, *mediums, large = ["bantam chicken", "cayuga duck", "weeder goose", "pig"]
#=> small = "bantam chicken"
#=> mediums = ["cayuga duck", "weeder goose"]
#=> large = "pig"
```


## Flexible method definitions
Knowing that splats manipulate arrays, we see their most common use in method definitions, allowing methods to take an unspecified number of arguments through a single term.

```
def farm_chores(who, action, *animals)
  animals.each.with_index(2) { |animal, index| puts "#{who} will #{action} the #{index} #{animal}"}
end

farm_chores("Farmer Brown", "feed", "ducks", "geese", "chickens", "pigs", "lambs")
#=> Farmer Brown will feed the 2 ducks
#=> Farmer Brown will feed the 3 geese
#=> Farmer Brown will feed the 4 chickens
#=> Farmer Brown will feed the 5 pigs
#=> Farmer Brown will feed the 6 lambs
```

The method above takes in a `who` argument, an `action` argument, and then as many `animal` arguments as we can muster. In the background, `*animals` is bundling all the arguments that aren't `who` and `action` into an array, which is handy for iterating through an ever-growing list of chores.

### Position agnostic
The splat argument can be placed in any position within the method definition. While the method above has an ending splat argument, the splat can also be the first or second argument. 

The return values below are exactly the same as the the initial `#farm_chores` method above. We simply adjust the method definition and call to use the new argument order.

```
def farm_chores(*animals, who, action)
  animals.each.with_index(2) { |animal, index| puts "#{who} will #{action} the #{index} #{animal}"}
end
farm_chores("ducks", "geese", "chickens", "pigs", "lambs", "Farmer Brown", "feed")

def farm_chores(who, *animals, action)
  animals.each.with_index(2) { |animal, index| puts "#{who} will #{action} the #{index} #{animal}"}
end
farm_chores("Farmer Brown", "ducks", "geese", "chickens", "pigs", "lambs", "feed")
```

### Nested input
Splat input automatically accounts for nested arguments, as well. Passing an explicit array into the same `#farm_chores` method doesn't break anything; it simply returns the nested argument as an array.

```
farm_chores("Farmer Brown", "ducks", "geese", "chickens", ["chicks", "baby bunnies"], "feed")
#=> Farmer Brown will feed the 2 ducks
#=> Farmer Brown will feed the 3 geese
#=> Farmer Brown will feed the 4 chickens
#=> Farmer Brown will feed the 5 ["chicks", "baby bunnies"]
```

### Excluded input
Beyond bundling important arguments, splat can separate out messy arguments that would otherwise throw a method call error. 

If our method below might be called with many unknowns, and we only care about `who` and `action`, splat can help us gracefully focus on the key arguments without explicitly specifying the rest.

```
def farm_chores(who, action, *unused)
  puts "#{who} will #{action} the animals every day."
end

farm_chores("Farmer Brown", "feed", "ducks", "geese", "chickens", "pigs", "lambs", "barns", "land", "water")
#=> Farmer Brown will feed the animals every day.
```


## Flexible method calls
On the other side of defining methods, splat can also call a method using a single term that represents multiple arguments. 

If we specify four arguments in a `#two_chores` method definition, splat allows us to call the method with only three arguments, automatically feeding the `animal` arguments.

```
def two_chores(who, action, animal_one, animal_two)
  puts "#{who} will #{action} the #{animal_one} and #{animal_two}"
end

*animals = "pigs", "lambs"
two_chores("Farmer Brown", "feed", *animals)
#=> Farmer Brown will feed the pigs and lambs
```

Even if we explicitly define `animals` as an array, splat will split the arguments for us, resulting in the same return values.
```
animals = ["pigs", "lambs"]
two_chores("Farmer Brown", "feed", *animals)
#=> Farmer Brown will feed the pigs and lambs
```


## Better code
The splat operator offers Rubyists a shortcut for manipulating arrays and methods. Ultimately, splat allows us to build more resilient, flexible functionality with less code, a worthy goal for any programmer.


### Sources
Thanks to the people below for sharing their splat discoveries.
- https://endofline.wordpress.com/2011/01/21/the-strange-ruby-splat/
- https://alexcastano.com/everything-about-ruby-splats/
- https://medium.freecodecamp.org/rubys-splat-and-double-splat-operators-ceb753329a78
- https://www.honeybadger.io/blog/ruby-splat-array-manipulation-destructuring/

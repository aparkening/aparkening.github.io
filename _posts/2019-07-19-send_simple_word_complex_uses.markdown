---
layout: post
title:      "Send: Simple Word, Complex Uses"
date:       2019-07-19 16:55:36 -0400
permalink:  send_simple_word_complex_uses
---


While delving deeper into Ruby object-oriented programming, I've wanted my methods and classes to do more work with less explicit code. Creating three classes that have 80% code overlap is difficult to maintain and feels clunky, so I've moved a lot of my code into sharable modules and inheritable classes. And to really make this sharable and inheritable code most flexible, I've turned to metaprogramming. 

[Rubymonk](http://rubymonk.com/learning/books/2-metaprogramming-ruby/chapters/32-introduction-to-metaprogramming/lessons/75-being-meta) has a helpful definition to get us started:
> Metaprogramming is the act of writing code that operates on code rather than on data. This involves inspecting and modifying a program as it runs using constructs exposed by the language.

When I'm trying to build code that will work across many situations, and when my inputs and/or outputs will have unknowns, I use metaprogramming to gracefully handle the unknowns.

## Hello Send
`Send` has been a handy metaprogramming tool I've started to rely on. I was introduced to `send` as a way to initialize an object when the exact arguments weren't known. 

Instead of this (manual initialization with known arguments):
```
def initialize(sender, receiver, amount)
	@sender, @receiver,@amount = sender, receiver, amount
end
```

We use this (dynamic initialization with unknown arguments):
```
def initialize(attributes)
	attributes.each {|key, value| self.send(("#{key}="), value)}
end
```

I thought that `send` was such an elegant solution for managing unknown attributes that I wanted to go deeper. Where else could `send` save me time and effort?

## So Many Conditionals
When referencing a class with multiple methods, it's been easy to turn to conditional structures like `if/else` and `case` to execute the appropriate code. If I want to build a wall using a miter saw, for example, I can use `case` to determine when to make various cuts.

```
class MiterSaw
  def crosscut
    puts "Cutting perpendicular to the grain!"
  end
  
  def bevel(a, b)
    puts "Making that sweet angle along the thickness of the wood! Moving from point #{a} to point #{b}."
  end
  
  def miter(a, b)
    puts "Woo, cutting an angle along the length of the wood. Moving from point #{a} to point #{b}."
  end
end

class Wall
  def initialize(mitersaw)
    @mitersaw = mitersaw
  end
  
  def cut(type, *arguments)
    case type
    when 'crosscut'
      @mitersaw.crosscut
    when 'bevel'
      @mitersaw.bevel(*arguments)
    when 'miter'
      @mitersaw.miter(*arguments)
    else
      raise NoMethodError.new(type)
    end
  end
end

north_wall = Wall.new(MiterSaw.new)
north_wall.cut("crosscut")
north_wall.cut("miter", [1, 2], [4, 6])
```

This gets the job done; my north wall can get several saw cuts. But to access more `MiterSaw` methods that rip along the grain or make a sliding compound cut, for example, I need to update _both_ the `Wall` class and the `MiterSaw` class. That gets messy quickly when I scale the program to have a crew building many houses at once. 

## My Hero
It turns out that `send` can help by introducing metaprogramming to my explicit `case` statement.

```
class Wall
  def initialize(mitersaw)
    @mitersaw = mitersaw
  end
  
  def cut(type, *arguments)
    @mitersaw.send(type, *arguments)
  end
end
```

Boom, I can replace all my conditionals with a single `send`! Not only do I have less code to maintain, but I have more flexibility to dynamically modify `MiterSaw` with minimal fuss. Now the `MiterSaw` class can be updated or extended with more cut types without touching the `Wall` class or any of its objects!

## Time-Out, What Even Is Going On Here
`@mitersaw.send(type, *arguments)` looks simple, so what's actually happening?

1. `Send` is an instance method available to all objects. In this case, it's acting on the `@mitersaw` object. 
2. The `type` argument that we're feeding to `send` is actually _the name of another method_. `Send` takes in method names as arguments. 
3. `*arguments` are the additional arguments that `#type` needs to execute various `MiterSaw` methods, such as `#bevel`.

We're _sending_ another method name (`type`) to our `@mitersaw` object. 

And our object doesn't need to know the method we're sending before the program runs, so we can change `MiterSaw` right up until we run an instance of `Wall`.

## Send is Great
I'm always looking for ways to make my code more abstract and robust, with less words and work. `Send` has been a great tool to move me along that path.


### More places to learn about `send`:
- [http://ruby-metaprogramming.rubylearning.com/html/ruby_metaprogramming_2.html](http://ruby-metaprogramming.rubylearning.com/html/ruby_metaprogramming_2.html)
- [http://rubymonk.com/learning/books/2-metaprogramming-ruby/chapters/32-introduction-to-metaprogramming/lessons/75-being-meta](http://rubymonk.com/learning/books/2-metaprogramming-ruby/chapters/32-introduction-to-metaprogramming/lessons/75-being-meta)
- [https://stackoverflow.com/questions/3337285/what-does-send-do-in-ruby](https://stackoverflow.com/questions/3337285/what-does-send-do-in-ruby)
- https://ruby-doc.org/core-2.6.3/Object.html#method-i-send(https://ruby-doc.org/core-2.6.3/Object.html#method-i-send)
- https://iamchrissmith.io/ruby-send-method-exploration(https://iamchrissmith.io/ruby-send-method-exploration)

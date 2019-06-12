---
layout: post
title:      "Tea Shopper CLI Gem"
date:       2019-06-11 22:01:37 -0400
permalink:  tea_shopper_cli_gem
---

My command line interface (CLI) Ruby gem student project, Tea Shopper, scrapes tea data from the web and allows users to compare teas by name, price per ounce, and tea shop. When the user chooses a tea, it displays specific details, such as purchase URL, flavors, region, and description. Check out the [short demonstration video](https://www.loom.com/share/5d3cc369d7c243d4af5e665206b39a75).

![](https://github.com/aparkening/tea_shopper/tree/master/assets/Tea_Shopper-welcome.png" alt="Tea Shopper welcome screen)

## Why

I love tea! Access to truly amazing teas has become much easier thanks to the web, but it takes a long time to sift through individual teas at each shop to find my next great tea. I wanted a command line tool to present me with concise lists of teas pulled from multiple web shops, sorted by various criteria. And, since teas are often sold in different amounts at different stores, I also wanted to see a per ounce price that would allow an apples-to-apples comparison. Then I got more excited: what about exploring teas not just by tea type (green, oolong, etc.), but also region, flavor notes, and shop? 

## How

After imagining how cool this tool could be, I wrote out the functionality from initial menus down to individual tea display screens. That helped me realize what methods I'd be able to reuse (such as alphabetically sorted tea lists) and how many levels of menu would be needed. Check out the full outline in the "Plan gem, imagine interface" section of the [Tea Shopper Process.md](https://github.com/aparkening/tea_shopper/blob/master/PROCESS.md). 

Once I had an outline of screens and needs, I started building the code, using Bundler to build the initial gem file tree and necessary files and Github to host the repository. I stubbed out the `run` method in the CLI class, using static output and pseudocode to display a close approximation of the screens my program would dynamically build. Once I liked the static output, I coded one full user path, moving from the main menu to the tea types menu right through to the tea detail view, which also required scraping a tea website: Song Tea & Ceramics. I recorded a [video code-along](https://www.loom.com/share/51df30210f2f4a3ea01cb19490e1c0f1) while building the tea detail view, where I talked through my thought process and showed the many, many times I coded a small bit and then immediately tested it. (This is also where I learned that I should build a console method to make the testing process easier!) 

## Challenges and Sadness

I thought it would be smooth sailing once a full user path was complete. Since I already had the scraped data, the Tea object methods, and the menu methods in place, I thought I could simply reuse that code with different criteria. But no: I ran into a scaling issue.

Initially, I thought I'd take the same approach to region and flavor navigation that I took with tea type. When coding the tea type path, I noticed that Song Tea & Ceramic's web structure didn't allow me to pull tea type, region, _and_ flavor data from a single index scrape. I could scrape any one of those, but the only way to get everything was to scrape the index _and_ all the detail pages at initial load time. This was reasonable when stubbing out the initial path, but I soon realized: adding more tea websites to this tool while preloading all index and detail pages would cause initial load times to skyrocket! 

So I decided to refactor, providing Tea Shopper with just the initial index scrape at load time, giving the user enough data to narrow their criteria and allowing the program to scrape a much smaller subset of detail pages, such as just the oolong teas.

But the lingering problem remained. If I separated the index and detail page scrapings, how would I get the needed region and flavor data to allow additional tea exploration paths? I tried a complicated solution that involved inserting default region data into my tea objects and then replacing it with scraped data later, but I eventually came to a tough decision: it wasn't worth keeping region and flavor navigation. It made more sense to build a rock solid interface for a single path (tea type) than to introduce additional complexity with more potential holes and bugs. Similarly, sorting teas by shop wouldn't be useful until more tea websites were added, so I reduced complexity further by removing that sorting mechanism. Tea Shopper would only categorize teas by type.

After those big decisions, I iterated on core functionality, working out display details and bugs, and refactoring code that felt clunky.

## Lessons Learned

### Console FTW

This project has helped cement the usefulness of the console for object oriented debugging. Ruby object oriented code has felt much more difficult to test than procedural Ruby because I always need to build objects before getting to the root of any logic bugs involving those objects. In this project, I began manually assigning objects, like `assam = Tea.new`. But I quickly learned that the console do these things for me if I wrote some helper methods. 

All of a sudden, I could type `test_build` and have access to my full `Tea.all` array that was populated by the latest scraped data! 

```
# Test initial Tea object build 
def test_build
  tea_array = TeaShopper::SongScraper.scrape_teas
  TeaShopper::Tea.create_from_collection(tea_array)
  TeaShopper::Tea.all
end

# Add all attributes to array
def test_attributes
  TeaShopper::Tea.all.each do |tea|
    attributes = TeaShopper::SongScraper.scrape_profile_page(tea.url)
    tea.add_tea_attributes(attributes)
  end
  TeaShopper::Tea.all
end
```

Or I could reload my console and hit the latest detail view by typing
`reload!` and `test_detail`, which was especially useful when I was trying to tie together `Tea` objects with scraped data towards the beginning of the project.

```
# Reload console to hit latest version of classes
def reload!
  load 'bin/console' 
end

# Display specific detail page
def test_detail
  tea = TeaShopper::Tea.all.find{|obj| obj.name == "Snow Jasmine"}
  TeaShopper::CLI.new.display_tea(tea)
end
```

### ALL THE THINGS

Another thing I learned during this project was to slow. down. In the first couple days of coding, I was so excited to build this cool vision that I wrote multiple methods and classes at the same time, resulting in bundled git commits and bundled bugs. I didn't feel comfortable rolling back commits because they included functionality that I didn't want to lose, so I had to manually go through each line of code across my classes and methods to recreate and solve the issues. After correcting that mess, I resolved to take smaller steps, setting 25-minute timers for focused functionality, and to commit incremental changes often. There were still commits with multiple changes, but both my commit log and my overall progress were much steadier, with fewer errors and frustrations.

## The End

Overall, this was a really fun project. I loved getting to build and see something that came out of my own imagination! I loved watching the evolution of the code itself, moving from messy drafts to more polished, elegant methods. And I loved establishing my own coding style, continuing with strategies that were working, ditching strategies that were difficult to maintain, and getting deeper into the whys behind some of the coding methodologies that we've already learned at Flatiron. I'm excited to make Tea Shopper even better in the future!

---
layout: post
title:      "Tea Taster Sinatra App"
date:       2019-08-14 19:39:30 -0400
permalink:  draft_post_on_tea_taster
---


I set out to make an app that could record thoughts about teas. To do that, I decided to also make models for the teas the notes were about, as well as the shops that sold the teas. Here's how I went about it:


## 1. Plan project, imagine interface

Tea Tastes is a place to record impressions of teas. Features:
- Authenticated users can log in to create, edit, and delete teas, notes, and shops.
- Notes are be linked to teas and shops for easy browsing.
- Unauthenticated users can view teas, notes, and shops.

### Page Structure

- Home
- Sign up
- Log in
- User home (user-specific list of notes)
	- Edit user
- Display all notes
	- Create new note
	- Display note
	- Edit note
- Display all teas
	- Create new tea
		- Create tea shop
	- Display tea
	- Edit tea
- Display all shops
	- Edit shop
	- All shops

#### Main page (index.html)

- Summarize product: words and image(s) for what project is and why you'd use it
- Display login and signup links

#### Sign Up and Log In

- Form 
	- Required username and password

#### Authenticated User Home

- Link to add note
- List of user-specific notes
	- Can sort by date and tea name
- Short list of notes from others

#### Create New Note

- Form with error messages
- Form includes fields for tea shop name and URL

#### Display Note

- Note content
- Edit and delete links
- Success message upon creation
- Delete link

#### Display User

- Profile information
- List of user notes

#### Display Shop

- Profile information, including URL
- List of notes for teas from shop
- Delete link if authenticated user and no related notes

#### Edit Note

- Pre-filled form
- Delete link
- Error and success messages

#### Edit User Profile

- Pre-filled form
- Error and success messages

#### Edit Shop

- Pre-filled form
- Error and success messages

#### Delete Note

- Only if note author

#### Delete Shop

- Only if authenticated user and no related notes


## 2. Outline project structure

### Model Structure

- User has many notes
- User has many teas through notes
- Tea belongs to shop
- Tea has many notes
- Tea has many users through notes
- Shop has many teas
- Shop has many notes through teas
- Note belongs to user
- Note belongs to tea

### File Structure

```
- app:
│   └── controllers:
│   │   └── application_controller.rb
│   │   └── notes_controller.rb
│   │   └── sessions_controller.rb
│   │   └── shops_controller.rb
│   │   └── users_controller.rb
│   └── models:
│   │   └── note.rb
│   │   └── note_shop.rb
│   │   └── shop.rb
│   │   └── user.rb
│   │   └── concerns:
│   │   │   └── slugify.rb
│   └── views:
│   │   └── index.erb
│   │   └── layout.erb
│   │   └── helpers:
│   │   │   └── index.erb
│   │   └── notes:
│   │   │   └── new.erb
│   │   │   └── edit.erb
│   │   │   └── show.erb
│   │   └── shops:     
│   │   │   └── edit.erb
│   │   │   └── show.erb
│   │   └── users:
│   │   │   └── index.erb
│   │   │   └── new.erb
│   │   │   └── edit.erb
│   │   │   └── show.erb
- config:
│   └── environment.rb
- db:
│   └── development.sqlite 
│   └── migrate:
│   │   └── create_users
│   │   └── create_notes
│   │   └── create_shops
│   │   └── create_notes_shops
│   └── seeds.rb
│   └── schema.rb
- public:
│   └── stylesheets:
│   │   └── styles.css
- spec:
│   └── model_spec.rb
│   └── note_spec.rb
│   └── shop_spec.rb
│   └── user_spec.rb
- config.ru
- Gemfile
- Rakefile
- README.md
- PROCESS.md
```


## 3. Start with application controller interface

- Make configuration (views, sessions, etc.)
- Create base route(s) (index)
- Make helper methods


## 4. Code remaining routes and adjust models

1. Sessions 
2. Users
3. Notes
4. Shops
5. Record code-along screencast
6. [New] Add Tea model and adjust relationships


## 5. Write data and tests

1. Write seed data.
2. [Removed] Create Rspec tests.


## 6. Tweak program for user flow

- Update models to include Tea
- Sanitize user input
- Clarify navigation, links, and instructions
- Improve error handling and edge case responses

## 7. Improve styling and production quality

- Adjust frontend interface to be clear, responsive, and beautiful.
- Refactor backend code to consolidate methods and separate concerns
- Clean code for production


## 8. Update Readme and Code of Conduct

Model Readme on pattern in https://gist.github.com/PurpleBooth/109311bb0361f32d87a2.


## 9. Record demonstration w/ narration


## 10. Write blog post about process


## 11. Submit Flatiron checklist


## Stretch goals:

1. Make Responsive interface

2. Suggest new teas based on preferences. Scrape tea websites for suggestions table.
   #### Dobra
   1. 8 index pages: https://www.dobratea.com/
   - div#second-menu ul#menu-second-nav-header li (don't get accessories)
   - Get second page if not "showing all" results: https://www.dobratea.com/category/green/page/2/
   2. Detail pages: https://www.dobratea.com/product/assam-brahmaputra/

   #### Smith - https://www.smithtea.com/collections/all-tea?sort_by=title-ascending

3. Incorporate `tea_shopper` gem for tea shop scraping and object population. Add API to gem for non-CLI interaction.

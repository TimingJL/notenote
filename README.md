# Learning by Doing 

![Ubuntu version](https://img.shields.io/badge/Ubuntu-16.04%20LTS-orange.svg)
![Rails version](https://img.shields.io/badge/Rails-v5.0.0-blue.svg)
![Ruby version](https://img.shields.io/badge/Ruby-v2.3.1p112-red.svg)

Mackenzie Child's video really inspired me. So I decided to follow all of his rails video tutorial to learn how to build a web app. Through the video, I would try to build the web app by my self and record the courses step by step in text to facilitate the review.


# Project 11: How To Build A Online Notebook App With Rails          
This time we built a online notebook application. We have users, and a user have the ability to create, edit, delete a note. But they can only see notes that they created. And we routed the root of the application based on whether the user is signed-in or not. So if they're not signed-in, they see the marketing site. And if they're signed-in, they see the dashboard with all the notes.


https://mackenziechild.me/12-in-12/11/        


### Highlights of this course
1. Users
2. Notes
3. Conditional Routes
4. HAML
5. Custom Styling


# Create A Online Notebook
```console
$ rails new notenote
```

Chage directory to the pin_board. Under `Gemfile`, add `gem 'therubyracer'`, save and run `bundle install`.      

Note: 
Because there is no Javascript interpreter for Rails on Ubuntu Operation System, we have to install `Node.js` or `therubyracer` to get the Javascript interpreter.

```console
$ bundle install
```

Then run the `rails server` and go to `http://localhost:3000` to make sure everything is correct.       

# Rubygems Install
Before we create the view file, let's install a few gems for our application.       
In `Gemfile`
```
gem 'devise'
gem 'simple_form', github: 'kesha-antonov/simple_form', branch: 'rails-5-0'
gem 'haml', '~> 4.0.5'
```
Save that, run `bundle install` and restart our server.


# Welcome Page
We're gonna have users and a user can only be able to see their own notes.       
Let's start out by creating our welcome controller first.   

```console
$ rails g controller Welcome index
```

In `config/routes.rb`, let's setup the root of the application.
```ruby
Rails.application.routes.draw do
  get 'welcome/index'

  root 'welcome#index'
end
```
![image](https://github.com/TimingJL/notenote/blob/master/pic/welcome_controller.jpeg)



Let's go to `app/views/welcome` and rename `index.html.erb` to `index.html.haml`.
```haml
%h1 Welcome#index
%p Find me in app/views/welcome/index.html.erb
```



# CRUD
Next we wanna to is create our note. A user have the ability to add notes, edit, and delete notes as well.      

```console
$ rails g model Note title:string content:text
$ rake db:migrate


$ rails g controller Notes
```

We add the rule of routes:            
In `config/routes.rb`
```ruby
Rails.application.routes.draw do
  get 'welcome/index'

  root 'welcome#index'

  resources :notes
end
```



And in `app/controllers/notes_controller.rb`
```ruby
class NotesController < ApplicationController
	def index
	end

	def show
	end

	def new
	end

	def create
	end

	def edit
	end

	def update
	end

	def destroy
	end

	private

	def find_note
	end

	def note_params
	end
end
```
(13:10)
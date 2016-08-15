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

To install simple_form, we should run the generator:
```console
$ rails g simple_form:install
```


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
	before_action :find_note, only: [:show, :edit, :update, :destroy]
	
	def index
		@notes = Note.all.order("created_at DESC")
	end

	def show
	end

	def new
		@note = Note.new
	end

	def create
		@note = Note.new(note_params)

		if @note.save
			redirect_to @note
		else
			render 'new'
		end
	end

	def edit
	end

	def update
	end

	def destroy
	end

	private

	def find_note
		@note = Note.find(params[:id])
	end

	def note_params
		params.require(:note).permit(:title, :content)
	end
end
```


So under `app/views/notes`, let's create some new files:
	1. new.html.haml
	2. _form.html.haml
	3. show.html.haml
	4. index.html.haml
	5. edit.html.haml

In `app/views/notes/new.html.haml`
```haml
%h1 Add a New Note

= render 'form'
```


Let's add our form in `app/views/notes/_form.html.haml`
```haml
= simple_form_for @note do |f|
	= f.input :title
	= f.input :content
	= f.button :submit
```
![image](https://github.com/TimingJL/notenote/blob/master/pic/add_note.jpeg)


In `app/views/notes/show.html.haml`
```haml
%h1= @note.title
%p= simple_format @note.content

= link_to 'All Notes',  notes_path
```

And in `app/views/notes/index.html.haml`, let's loop through all of the notes here.
```haml
- @notes.each do |note|
	%h2= link_to note.title, note
	%p= time_ago_in_words(note.created_at)

= link_to 'New Note', new_note_path
```
![image](https://github.com/TimingJL/notenote/blob/master/pic/loop_through.jpeg)


### Edit & Destroy

Inside of `app/controllers/notes_controller.rb`
```ruby
def update
	if @note.update(note_params)
		redirect_to @note
	else
		render 'edit'
	end
end

def destroy
	@note.destroy
	redirect_to notes_path
end
```


In `app/views/notes/show.html.haml`
```haml
%h1= @note.title
%p= simple_format @note.content

= link_to 'All Notes',  notes_path
= link_to 'Edit',  edit_note_path(@note)
= link_to 'Delete',  note_path(@note), method: :delete, data: { confirm: 'Are you sure?' }
```

And in `app/views/notes/edit.html.haml`
```haml
%h1 Edit Your Note

= render 'form'

= link_to 'Cancel', note_path
```


# Devise

After we install Devise and add it to our Gemfile, we need to run the generator:
```console
$ rails g devise:install


===============================================================================

Some setup you must do manually if you haven't yet:

  1. Ensure you have defined default url options in your environments files. Here
     is an example of default_url_options appropriate for a development environment
     in config/environments/development.rb:

       config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }

     In production, :host should be set to the actual host of your application.

  2. Ensure you have defined root_url to *something* in your config/routes.rb.
     For example:

       root to: "home#index"

  3. Ensure you have flash messages in app/views/layouts/application.html.erb.
     For example:

       <p class="notice"><%= notice %></p>
       <p class="alert"><%= alert %></p>

  4. You can copy Devise views (for customization) to your app by running:

       rails g devise:views

===============================================================================
```

Let's paste `config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }` in `config/environments/development.rb`.     

And rename the `application.html.erb` to `application.html.haml` under `app/views/layouts`
```haml
!!!
%html
  %head
    %meta{:content => "text/html; charset=UTF-8", "http-equiv" => "Content-Type"}/
    %title Notenote
    = csrf_meta_tags
    = stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track': 'reload'
    = javascript_include_tag 'application', 'data-turbolinks-track': 'reload'
  %body
    %p.notice= notice
    %p.alert= alert
    = yield
```

and do 
```console
$ rails g devise:views
```


Next thing we need to do is generate a devise model.
```console
$ rails g devise User
$ rake db:migrate
```

We may go to `http://localhost:3000/users/sign_up` to make sure everything is correct.
![image](https://github.com/TimingJL/notenote/blob/master/pic/sign_up.jpeg)


Next, we want to make sure when we create a new note that is from the current user.        
So what we need to do is run the migration to add the user model to our note's table.
```console
$ rails g migration add_user_id_to_notes user_id:integer
$ rake db:migrate
```


Then let's pop into the rails console, we can see the `user_id` is `nil`. Which means user_id is added into the column of the table.
```console
$ rails console

> @note = Note.first

  Note Load (0.6ms)  SELECT  "notes".* FROM "notes" ORDER BY "notes"."id" ASC LIMIT ?  [["LIMIT", 1]]             
=> #<Note id: 1, title: "Books I want to read", content: "Hooked by Nir Eyal",          
created_at: "2016-08-15 05:46:27", updated_at: "2016-08-15 05:46:27", user_id: nil>
```

Next thing we want to do is add association between our user model and our note model. And then we want to build out the note from our user.        

In `app/models/user.rb`
```ruby
class User < ApplicationRecord
  # Include default devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable and :omniauthable
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :trackable, :validatable
  has_many :notes
end
```

In `app/models/note.rb`
```ruby
class Note < ApplicationRecord
	belongs_to :user
end
```



Then we need to tweak how the note are created.           
In `app/controllers/notes_controller.rb`
```ruby
def new
	@note = current_user.notes.build
end

def create
	@note = current_user.notes.build(note_params)

	if @note.save
		redirect_to @note
	else
		render 'new'
	end
end
```

Now, we go to `http://localhost:3000/notes/new` to create a new note:
```console
$ rails c

> @note = Note.last

  Note Load (0.5ms)  SELECT  "notes".* FROM "notes" ORDER BY "notes"."id" DESC LIMIT ?  [["LIMIT", 1]]           
=> #<Note id: 4, title: "This note should have a User", content: "asdf asdf asdf asdf",           
created_at: "2016-08-15 08:50:16", updated_at: "2016-08-15 08:50:16", user_id: 1>
```

We can see the `user_id` is `1`. That is good.


### Change index to show only current users posts
Now, anybody can see all the notes which is created on the index page. But we only want to build the show what belongs to that current user.    
So in `app/controllers/notes_controller.rb`, we tweak
```ruby
def index
	@notes = Note.all.order("created_at DESC")
end
```

to

```ruby
def index
	@notes = Note.where(user_id: current_user)
end
```


To be continued...



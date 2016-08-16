# Learning by Doing 

![Ubuntu version](https://img.shields.io/badge/Ubuntu-16.04%20LTS-orange.svg)
![Rails version](https://img.shields.io/badge/Rails-v5.0.0-blue.svg)
![Ruby version](https://img.shields.io/badge/Ruby-v2.3.1p112-red.svg)

Mackenzie Child's video really inspired me. So I decided to follow all of his rails video tutorial to learn how to build a web app. Through the video, I would try to build the web app by my self and record the courses step by step in text to facilitate the review.


# Project 11: How To Build A Online Notebook App With Rails          
This time we built a online notebook application. We have users, and a user have the ability to create, edit, delete a note. But they can only see notes that they created. And we routed the root of the application based on whether the user is signed-in or not. So if they're not signed-in, they see the marketing site. And if they're signed-in, they see the dashboard with all the notes.


https://mackenziechild.me/12-in-12/11/        


![image](https://github.com/TimingJL/notenote/blob/master/pic/welcome_cdn.jpeg)
![image](https://github.com/TimingJL/notenote/blob/master/pic/note.jpeg)
![image](https://github.com/TimingJL/notenote/blob/master/pic/show_page.jpeg)


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


### Change index to show only current users notes
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


# Conditional Routes

In `config/routes.rb`
```ruby
Rails.application.routes.draw do
  devise_for :users
  get 'welcome/index'
  resources :notes

  authenticated :user do
  	root 'notes#index', as: 'authenticated_root'
  end

  root 'welcome#index'
end
```


# Styling

First, in `app/assets/stylesheets`, we rename the `application.css` to `application.css.scss` and remove the ` *= require_tree .`.
```scss
/*
 *= require_self
 */

@import 'normalize';
@import "global.css.scss";
@import "notes.css.scss";
@import "welcome.css.scss";
```

And then let's create those file under `app/assets/stylesheets`:
	1. global.css.scss
	2. normalize.css.scss
	3. notes.css.scss
	4. welcome.css.scss



In `app/assets/stylesheets/normalize.css.scss`   

https://necolas.github.io/normalize.css/           


```scss
/*! normalize.css v3.0.2 | MIT License | git.io/normalize */

/**
 * 1. Set default font family to sans-serif.
 * 2. Prevent iOS text size adjust after orientation change, without disabling
 *    user zoom.
 */

html {
  font-family: sans-serif; /* 1 */
  -ms-text-size-adjust: 100%; /* 2 */
  -webkit-text-size-adjust: 100%; /* 2 */
}

/**
 * Remove default margin.
 */

body {
  margin: 0;
}

/* HTML5 display definitions
   ========================================================================== */

/**
 * Correct `block` display not defined for any HTML5 element in IE 8/9.
 * Correct `block` display not defined for `details` or `summary` in IE 10/11
 * and Firefox.
 * Correct `block` display not defined for `main` in IE 11.
 */

article,
aside,
details,
figcaption,
figure,
footer,
header,
hgroup,
main,
menu,
nav,
section,
summary {
  display: block;
}

/**
 * 1. Correct `inline-block` display not defined in IE 8/9.
 * 2. Normalize vertical alignment of `progress` in Chrome, Firefox, and Opera.
 */

audio,
canvas,
progress,
video {
  display: inline-block; /* 1 */
  vertical-align: baseline; /* 2 */
}

/**
 * Prevent modern browsers from displaying `audio` without controls.
 * Remove excess height in iOS 5 devices.
 */

audio:not([controls]) {
  display: none;
  height: 0;
}

/**
 * Address `[hidden]` styling not present in IE 8/9/10.
 * Hide the `template` element in IE 8/9/11, Safari, and Firefox < 22.
 */

[hidden],
template {
  display: none;
}

/* Links
   ========================================================================== */

/**
 * Remove the gray background color from active links in IE 10.
 */

a {
  background-color: transparent;
}

/**
 * Improve readability when focused and also mouse hovered in all browsers.
 */

a:active,
a:hover {
  outline: 0;
}

/* Text-level semantics
   ========================================================================== */

/**
 * Address styling not present in IE 8/9/10/11, Safari, and Chrome.
 */

abbr[title] {
  border-bottom: 1px dotted;
}

/**
 * Address style set to `bolder` in Firefox 4+, Safari, and Chrome.
 */

b,
strong {
  font-weight: bold;
}

/**
 * Address styling not present in Safari and Chrome.
 */

dfn {
  font-style: italic;
}

/**
 * Address variable `h1` font-size and margin within `section` and `article`
 * contexts in Firefox 4+, Safari, and Chrome.
 */

h1 {
  font-size: 2em;
  margin: 0.67em 0;
}

/**
 * Address styling not present in IE 8/9.
 */

mark {
  background: #ff0;
  color: #000;
}

/**
 * Address inconsistent and variable font size in all browsers.
 */

small {
  font-size: 80%;
}

/**
 * Prevent `sub` and `sup` affecting `line-height` in all browsers.
 */

sub,
sup {
  font-size: 75%;
  line-height: 0;
  position: relative;
  vertical-align: baseline;
}

sup {
  top: -0.5em;
}

sub {
  bottom: -0.25em;
}

/* Embedded content
   ========================================================================== */

/**
 * Remove border when inside `a` element in IE 8/9/10.
 */

img {
  border: 0;
}

/**
 * Correct overflow not hidden in IE 9/10/11.
 */

svg:not(:root) {
  overflow: hidden;
}

/* Grouping content
   ========================================================================== */

/**
 * Address margin not present in IE 8/9 and Safari.
 */

figure {
  margin: 1em 40px;
}

/**
 * Address differences between Firefox and other browsers.
 */

hr {
  -moz-box-sizing: content-box;
  box-sizing: content-box;
  height: 0;
}

/**
 * Contain overflow in all browsers.
 */

pre {
  overflow: auto;
}

/**
 * Address odd `em`-unit font size rendering in all browsers.
 */

code,
kbd,
pre,
samp {
  font-family: monospace, monospace;
  font-size: 1em;
}

/* Forms
   ========================================================================== */

/**
 * Known limitation: by default, Chrome and Safari on OS X allow very limited
 * styling of `select`, unless a `border` property is set.
 */

/**
 * 1. Correct color not being inherited.
 *    Known issue: affects color of disabled elements.
 * 2. Correct font properties not being inherited.
 * 3. Address margins set differently in Firefox 4+, Safari, and Chrome.
 */

button,
input,
optgroup,
select,
textarea {
  color: inherit; /* 1 */
  font: inherit; /* 2 */
  margin: 0; /* 3 */
}

/**
 * Address `overflow` set to `hidden` in IE 8/9/10/11.
 */

button {
  overflow: visible;
}

/**
 * Address inconsistent `text-transform` inheritance for `button` and `select`.
 * All other form control elements do not inherit `text-transform` values.
 * Correct `button` style inheritance in Firefox, IE 8/9/10/11, and Opera.
 * Correct `select` style inheritance in Firefox.
 */

button,
select {
  text-transform: none;
}

/**
 * 1. Avoid the WebKit bug in Android 4.0.* where (2) destroys native `audio`
 *    and `video` controls.
 * 2. Correct inability to style clickable `input` types in iOS.
 * 3. Improve usability and consistency of cursor style between image-type
 *    `input` and others.
 */

button,
html input[type="button"], /* 1 */
input[type="reset"],
input[type="submit"] {
  -webkit-appearance: button; /* 2 */
  cursor: pointer; /* 3 */
}

/**
 * Re-set default cursor for disabled elements.
 */

button[disabled],
html input[disabled] {
  cursor: default;
}

/**
 * Remove inner padding and border in Firefox 4+.
 */

button::-moz-focus-inner,
input::-moz-focus-inner {
  border: 0;
  padding: 0;
}

/**
 * Address Firefox 4+ setting `line-height` on `input` using `!important` in
 * the UA stylesheet.
 */

input {
  line-height: normal;
}

/**
 * It's recommended that you don't attempt to style these elements.
 * Firefox's implementation doesn't respect box-sizing, padding, or width.
 *
 * 1. Address box sizing set to `content-box` in IE 8/9/10.
 * 2. Remove excess padding in IE 8/9/10.
 */

input[type="checkbox"],
input[type="radio"] {
  box-sizing: border-box; /* 1 */
  padding: 0; /* 2 */
}

/**
 * Fix the cursor style for Chrome's increment/decrement buttons. For certain
 * `font-size` values of the `input`, it causes the cursor style of the
 * decrement button to change from `default` to `text`.
 */

input[type="number"]::-webkit-inner-spin-button,
input[type="number"]::-webkit-outer-spin-button {
  height: auto;
}

/**
 * 1. Address `appearance` set to `searchfield` in Safari and Chrome.
 * 2. Address `box-sizing` set to `border-box` in Safari and Chrome
 *    (include `-moz` to future-proof).
 */

input[type="search"] {
  -webkit-appearance: textfield; /* 1 */
  -moz-box-sizing: content-box;
  -webkit-box-sizing: content-box; /* 2 */
  box-sizing: content-box;
}

/**
 * Remove inner padding and search cancel button in Safari and Chrome on OS X.
 * Safari (but not Chrome) clips the cancel button when the search input has
 * padding (and `textfield` appearance).
 */

input[type="search"]::-webkit-search-cancel-button,
input[type="search"]::-webkit-search-decoration {
  -webkit-appearance: none;
}

/**
 * Define consistent border, margin, and padding.
 */

fieldset {
  border: 1px solid #c0c0c0;
  margin: 0 2px;
  padding: 0.35em 0.625em 0.75em;
}

/**
 * 1. Correct `color` not being inherited in IE 8/9/10/11.
 * 2. Remove padding so people aren't caught out if they zero out fieldsets.
 */

legend {
  border: 0; /* 1 */
  padding: 0; /* 2 */
}

/**
 * Remove default vertical scrollbar in IE 8/9/10/11.
 */

textarea {
  overflow: auto;
}

/**
 * Don't inherit the `font-weight` (applied by a rule above).
 * NOTE: the default cannot safely be changed in Chrome and Safari on OS X.
 */

optgroup {
  font-weight: bold;
}

/* Tables
   ========================================================================== */

/**
 * Remove most spacing between table cells.
 */

table {
  border-collapse: collapse;
  border-spacing: 0;
}

td,
th {
  padding: 0;
}
```


In `app/assets/stylesheets/global.css.scss`
```scss
$dark_gray: #2B2E2E;
$light_gray: #C0C1C1;
$green: #33CD99;

@mixin bold_font {
	font-family: "Brandon Grotesque","Helvetica Neue", Helvetica, sans-serif;
	font-weight: bold;
	text-transform: uppercase;
}

* {
	box-sizing: border-box;
	-webkit-font-smoothing: antialiased;
}

body {
	padding-top: 4.5rem;
	font-family: "Open Sans", "Helvetica Neue", Helvetica, sans-serif;
}

.wrapper {
	width: 80%;
	margin: 0 auto;
}

.wrapper_with_padding {
	width: 80%;
	margin: 0 auto;
	padding: 3.5rem 0;
}

.clearfix:before, .clearfix:after {
	content: " ";
	display: table;
}

.clearfix:after {
	clear: both;
}

button, .button {
	border: none;
	outline: none;
	background: $green;
	padding: 1rem 3.5rem;
	border-radius: 2rem;
	a {
		color: white;
		text-decoration: none;
	}
}

.button {
	color: white;
	margin: 2rem 0;
}

a {
	color: $green;
	text-decoration: none;
	&:hover {
		color: $dark_gray;
	}
}

header {
	background: $dark_gray;
	position: fixed;
	top: 0;
	width: 100%;
	padding: 1.75rem 0;
	border-bottom: 7px solid $green;
	.header_inner {
		width: 90%;
		margin: 0 auto;
		#logo {
			display: inline-block;
			width: 10rem;
			float: left;
		}
		nav {
			float: right;
			a {
				text-decoration: none;
				color: white;
				font-size: 1.1rem;
				line-height: 1.5;
				margin-left: 1rem;
				&:hover {
					color: $green;
				}
			}
		}
	}
}

footer {
	background: $dark_gray;
	width: 100%;
	padding: 2rem 0;
	color: white;
	text-align: center;
	@include bold_font;
	p {
		margin: 0;
		font-size: .85rem;
	}
}

input[type="text"], input[type="email"], input[type="password"], textarea {
	width: 100%;
	margin-bottom: 1rem;
	border: 1px solid $light_gray;
	border-radius: .4rem;
}

input[type="text"], input[type="email"], input[type="password"] {
	height: 40px;
}

textarea {
	min-height: 200px;
}
```


In `app/assets/stylesheets/notes.css.scss`
```scss
#notes {
	.note {
		background: $dark_gray;
		width: 30%;
		min-height: 150px;
		margin: 1rem 1%;
		padding: 1.5rem 2.5%;
		float: left;
		border-radius: .4rem;
		position: relative;
		.title {
			@include bold_font;
			color: white;
			margin: 0;
			letter-spacing: 1px;
		}
		.date {
			position: absolute;
			bottom: .25rem;
			left: 8.5%;
			color: $green;
			font-size: .85rem;
		}
	}
}

#note_show {
	.title {
		@include bold_font;
		font-size: 2.5rem;
		margin-bottom: 1.25rem;
		padding-bottom: 1rem;
		border-bottom: 1px solid #E9E9E9;
	}
	p {
		font-size: 1.375rem;
		line-height: 1.75;
	}
	.buttons {
		margin-top: 3rem;
		a.button {
			padding: .5rem 2rem;
			border-radius: 2rem;
			text-decoration: none;
			color: $green;
			border: 5px solid #E9E9E9;
			@include bold_font;
			background: transparent;
		}
	}
}
```


In `app/assets/stylesheets/welcome.css.scss`
```scss
#banner {
	background-image: url(image_path("welcome_banner_image.jpg"));
	background-size: cover;
	background-repeat: no-repeat;
	background-position: top center;
	width: 100%;
	min-height: 500px;
	display: table;
	.banner_content {
		display: table-cell;
		vertical-align: middle;
		text-align: center;
		h1 {
			@include bold_font;
			color: white;
			font-size: 3.5rem;
			margin: 0;
		}
		p {
			font-size: 1.5rem;
			margin: 0 0 2.5rem 0;
			color: rgba(250, 250, 250, 0.5);
		}
	}
}

#testimonial {
	text-align: center;
	font-size: 1.5rem;
	color: #afb0b0;
	padding: 3.5rem 0;
	.quote {
		margin: 0 0 .5rem 0;
	}
	.name {
		@include bold_font;
		color: $dark_gray;
		margin: 0;
		font-size: .85rem;
	}
}

#callouts {
	background-image: url(image_path("callout_background.jpg"));
	background-size: cover;
	background-repeat: no-repeat;
	background-position: top center;
	width: 100%;
	min-height: 400px;
	display: table;
	.callout_inner {
		display: table-cell;
		vertical-align: middle;
	}
	.callout {
		text-align: center;
		width: 33.3333%;
		padding: 1.5rem 2.5%;
		float: left;
		h2 {
			@include bold_font;
			color: white;
			margin-bottom: 0;
		}
		p {
			color: rgba(250, 250, 250, 0.4);
		}
	}
}

#bottom_cta {
	text-align: center;
	padding: 7.5rem 0;
	h2 {
		@include bold_font;
		color: $dark_gray;
		font-size: 3rem;
		margin: 0;
	}
	p {
			font-size: 1.25rem;
			margin: 0 0 2.5rem 0;
			color: #a7a8a8;
		}
}
```


### Structure
In `app/views/layouts/application.html.haml`
```haml
!!!
%html
%head
	%title NoteNote | Online Notebook Application
	= stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track' => true
	= javascript_include_tag 'application', 'data-turbolinks-track' => true
	= csrf_meta_tags
%body
	%header
		.header_inner
			= link_to image_tag( 'logo.svg'), root_path, id: "logo"
			%nav
				- if user_signed_in?
					= link_to "New Note", new_note_path
					= link_to "Sign Out", destroy_user_session_path, method: :delete
				- else
					= link_to "Log In", new_user_session_path
	%p.notice= notice
	%p.alert= alert

	= yield
```
![image](https://github.com/TimingJL/notenote/blob/master/pic/structure.jpeg)


In `app/views/welcome/index.html.haml`, we want to add a banner
```haml
#banner
	.banner_content
		%h1 NoteNote
		%p Your online notebook. Never forget an idea again.
		%p= link_to "Sign Up", new_user_registration_path, class: "button"

#testimonial
	.wrapper
		%p.quote "The greatest notebook application. Period."
		%p.name - Timing.JL
#callouts
	.callout_inner
		.wrapper
			.callout
				%h2 Notes
				%p Viral Echo Park Intelligentsia tattooed, craft beer organic authentic polaroid tousled mlkshk church-key. Fanny pack Banksy vegan  authentic Helvetica.

			.callout
				%h2 Rocket
				%p Viral Echo Park Intelligentsia tattooed, craft beer organic authentic polaroid tousled mlkshk church-key. Fanny pack Banksy vegan  authentic Helvetica.

			.callout
				%h2 Lighting
				%p Viral Echo Park Intelligentsia tattooed, craft beer organic authentic polaroid tousled mlkshk church-key. Fanny pack Banksy vegan  authentic Helvetica.
#bottom_cta
	.wrapper
		%h2 For Realz!
		%p I want you to sign up... If you don’t. I will find you!
		= link_to "Click Meee!!!", new_user_registration_path, class: 'button'

%footer
	%p Timing.JL
```
![image](https://github.com/TimingJL/notenote/blob/master/pic/welcome.jpeg)



### Style How the Note Show Up
Next, let's style how the note show up. It's already style in css. 
So in `app/views/notes/index.html.haml`
```haml
.wrapper_with_padding
	#notes.clearfix
		- unless @notes.blank?
			- @notes.each do |note|
				%a{ href: (url_for [note])}
					.note
						%p.title= note.title
						%p.date= time_ago_in_words(note.created_at)
		- else
			%h2 Add a Note
			%p It appears you haven't created any notes yet... Lets fix that. Why don't you go ahead and create a new note.
			= link_to "New Note", new_note_path, class: 'button'
```
![image](https://github.com/TimingJL/notenote/blob/master/pic/note.jpeg)


### Style the Show Page

Now, let's do the show page.
In `app/views/notes/show.html.haml`
```haml
.wrapper_with_padding
	#note_show
		%h1.title= @note.title
		%p= simple_format(@note.content)

		.buttons
			= link_to "Edit", edit_note_path(@note), class: "button"
			= link_to "Delete", note_path(@note), method: :delete, data: { confirm: "Are you sure?" }, class: "button"
```
![image](https://github.com/TimingJL/notenote/blob/master/pic/show_page.jpeg)


### Style Form

In `app/views/notes/new.html.haml`
```haml
.wrapper_with_padding
	%h1 Add A Note

	= render 'form'
```

In `app/views/notes/edit.html.haml`
```haml
.wrapper_with_padding
	%h1 Edit Note

	= render 'form'

	= link_to "Cancel", note_path
```


In `app/views/notes/_form.html.haml`
```haml
= simple_form_for @note do |f|
	= f.input :title
	= f.input :content
	= f.button :submit, class: 'button'
```


In `app/views/devise/registrations/new.html.haml`
```haml
.wrapper_with_padding
  %h2 Sign up
  = simple_form_for(resource, as: resource_name, url: registration_path(resource_name)) do |f|
    = f.error_notification
    .form-inputs
      = f.input :email, required: true, autofocus: true
      = f.input :password, required: true, hint: ("#{@minimum_password_length} characters minimum" if @validatable)
      = f.input :password_confirmation, required: true
    .form-actions
      = f.button :submit, "Sign up", class: 'button'
```


In `app/views/devise/registrations/edit.html.haml`
```haml
%h2
  Edit #{resource_name.to_s.humanize}
= simple_form_for(resource, as: resource_name, url: registration_path(resource_name), html: { method: :put }) do |f|
  = f.error_notification
  .form-inputs
    = f.input :email, required: true, autofocus: true
    - if devise_mapping.confirmable? && resource.pending_reconfirmation?
      %p
        Currently waiting confirmation for: #{resource.unconfirmed_email}
    = f.input :password, autocomplete: "off", hint: "leave it blank if you don't want to change it", required: false
    = f.input :password_confirmation, required: false
    = f.input :current_password, hint: "we need your current password to confirm your changes", required: true
  .form-actions
    = f.button :submit, "Update"
%h3 Cancel my account
%p
  Unhappy? #{link_to "Cancel my account", registration_path(resource_name), data: { confirm: "Are you sure?" }, method: :delete}
= link_to "Back", :back
```


In `app/views/devise/sessions/edit.html.haml`
```haml
.wrapper_with_padding
  %h2 Log in
  = simple_form_for(resource, as: resource_name, url: session_path(resource_name)) do |f|
    .form-inputs
      = f.input :email, required: false, autofocus: true
      = f.input :password, required: false
      = f.input :remember_me, as: :boolean if devise_mapping.rememberable?
    .form-actions
      = f.button :submit, "Log in", class: 'button'
  = render "devise/shared/links"
```


### Using Font Awesome

http://fontawesome.io/get-started/            

We add this to `app/views/layouts/application.html.haml`
```html
%link{:href => "path/to/font-awesome/css/font-awesome.min.css", :rel => "stylesheet"}/
```

like

```haml
!!!
%html
%head
    %title NoteNote | Online Notebook Application
    = stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track' => true
    = javascript_include_tag 'application', 'data-turbolinks-track' => true
    %link{:href => "//maxcdn.bootstrapcdn.com/font-awesome/4.3.0/css/font-awesome.min.css", :rel => "stylesheet"}/
    = csrf_meta_tags
%body
    %header
        .header_inner
            = link_to image_tag( 'logo.svg'), root_path, id: "logo"
            %nav
                - if user_signed_in?
                    = link_to "New Note", new_note_path
                    = link_to "Sign Out", destroy_user_session_path, method: :delete
                - else
                    = link_to "Log In", new_user_session_path
    %p.notice= notice
    %p.alert= alert
    = yield
```





Then in `app/views/welcome/index.html.haml`
```haml
#banner
	.banner_content
		%h1 NoteNote
		%p Your online notebook. Never forget an idea again.
		%p= link_to "Sign Up", new_user_registration_path, class: "button"

#testimonial
	.wrapper
		%p.quote "The greatest notebook application. Period."
		%p.name - Timing.JL
#callouts
	.callout_inner
		.wrapper
			.callout
				%i.fa.fa-pencil{"aria-hidden" => "true"}
				%h2 Notes
				%p Viral Echo Park Intelligentsia tattooed, craft beer organic authentic polaroid tousled mlkshk church-key. Fanny pack Banksy vegan  authentic Helvetica.

			.callout
				%i.fa.fa-rocket{"aria-hidden" => "true"}
				%h2 Rocket
				%p Viral Echo Park Intelligentsia tattooed, craft beer organic authentic polaroid tousled mlkshk church-key. Fanny pack Banksy vegan  authentic Helvetica.

			.callout
				%i.fa.fa-bolt{"aria-hidden" => "true"}
				%h2 Lighting
				%p Viral Echo Park Intelligentsia tattooed, craft beer organic authentic polaroid tousled mlkshk church-key. Fanny pack Banksy vegan  authentic Helvetica.
#bottom_cta
	.wrapper
		%h2 For Realz!
		%p I want you to sign up... If you don’t. I will find you!
		= link_to "Click Meee!!!", new_user_registration_path, class: 'button'

%footer
	%p Timing.JL
```
![image](https://github.com/TimingJL/notenote/blob/master/pic/welcome_cdn.jpeg)

Finished!



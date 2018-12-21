---
layout: post
title:      "Resource Summary For Simple Rails Features"
date:       2018-12-21 07:04:33 +0000
permalink:  resource_summary_for_simple_rails_features
---


## I made my first rails app last week, and here are some resources and strategies I used to make it work

### Google Authentication

- [Google OAuth gem](https://github.com/zquestz/omniauth-google-oauth2)

```ruby
# config/initializers/omniauth.rb

Rails.application.config.middleware.use OmniAuth::Builder do
    provider :google_oauth2, ENV['GOOGLE_KEY'], ENV['GOOGLE_SECRET']
end
```
- You can add a `google_uid` column and a `google_refresh_token` to the `user` table if you need a unique property with which to query a user, or you want to use more advanced features of Google's API (and allow an offline refresh of the token). Because I have a uniqueness validation of a user's `email` and I am only using Google for authentication, I didn't need those columns but added them anyway just in case I had a design change in the future.
- In my schema, I made a `user`'s `email` validate uniqueness so that logging in with Google could create a whole new account, or find an account with that email and update a `google_uid` property to it without altering the password. 
  - To make this method foolproof, it would be important to have a user verify their email if the account is made traditionally (ie without OAuth). 
- You need to have an action dedicated to handling the OAuth callback. 
  - This action needs to be associated with the callback route you specify in the Google API page (the link to that page is on the GitHub of the Google OAuth gem linked above.)
  - My custom route for the callback that I defined in my `routes.rb` file was: `get 'auth/google_oauth2/callback', to: 'sessions#googleAuth'`
  - My callback action in my `SessionsController` utilized a custom class method written in the `User` model, to which it passed the return from the OAuth process, like so:

```ruby
#  SessionController#googleAuth
    access_token = request.env["omniauth.auth"]
    user = User.from_google(access_token)

#  User.rb

    def self.from_google(auth)
        refresh_token = auth.credentials.refresh_token
        if (found_user = User.find_by(email: auth.info.email))
            found_user.google_uid = auth.credentials.token
            found_user.google_refresh_token = refresh_token if refresh_token.present?
            return found_user
        else
            new_user = User.new do |u|
                u.email = auth.info.email
                u.name = auth.info.name
                u.google_uid = auth.credentials.token
                u.google_refresh_token = refresh_token if refresh_token.present?
                rand_password = RandomPasswordStrategy.random_password
                u.password = rand_password
                u.password_confirmation = rand_password
            end
            return new_user
        end
        
    end
```

- From there, you just check to see if the user is valid, and log them in if it is, and otherwise display an error.
- The `RandomPasswordStrategy` is just a class I made that uses the `sysrandom/securerandom` method `SecureRandom.hex(64)` combined with random ordering of required symbols and numbers (as required by the password policy) to create a complex, valid random password.

### Authorization Methods, Error Pages

- I put a lot of private methods in my `ApplicationController` to abstract away authorization throughout my app, making it easier to authenticate a user in the correct manner and display an appropriate message to them if there was an issue. 
- I wanted logged out users to be able to see most of the app, and then restrict other parts of it to only users with accounts. There are some aspects that only specific users have access to, such as editing your own posts, etc. 
- Some pages will show differently based on whether or not the user is logged in. To accomplish this simply, I added two methods to the `ApplicationController` so it is available to all Controllers (which inherit from `ApplicationController`), as well as adding them as `helper_methods` to make them available in the Views if necessary:

```ruby
#  application_controller.rb

helper_method :logged_in?
helper_method :current_user


private

def current_user
    User.find_by(id: session[:user_id])
end

def logged_in?
    !current_user.nil?
end
```

- These are primarily used as decision-makers on what is shown on certain pages, or building blocks to more advanced authorization methods. 
- If an unauthorized user tries to access a page you don't want them to, it is appropriate to show a 403 error page. I accomplished this by creating a `public/403.html.erb` file, and calling it via another method in `ApplicationController`

```ruby
# application_controller.rb

def not_authorized(msg = "You are not authorized to view that page")
    flash[:danger] = msg
    render(:file => File.join(Rails.root, 'public/403.html.erb'), :status => 403, :layout => false)
end
```

- Which can accept a custom flash message as a parameter, and which serves as a good building block for other methods, like this:

```ruby
# application_controller.rb

def authorize(user=nil)
    if user.nil?
        not_authorized('<a href="/login">Login</a> or <a href="/signup">Signup</a> to view this page!') unless logged_in?
        !!current_user
    else
        if user == current_user
            not_authorized
            false
        else
            true
        end
    end
end
```
- If the above is called without a user parameter, it will only display a 403 error page if the user is not logged in. If it is called with a user, the 403 will be called unless that specific user is logged in. It also returns a boolean to indicate success or failure. If you need to use `current_user` in the controller after calling `authorize`, you will get an error if the authorization fails. An easy way to fix this is to use an `if-else` block with the authorization check rather than using it as a standalone function so that any code which calls current_user will not be run unless authorization succeeds. For this to work, a boolean (or at least truthy and falsy) value must be returned. If the authorization fails, it will return false and then once the controller action is complete it will render the 403 error page. 

### Following Users, Sending Messages, Reacting to Posts, Scope Methods

For some relationships, ActiveRecord relations aren't as simple as `has_many :classes, through: :user_classes` or something like that. 3 Situations particularly I had to get more creative to make work:

#### Following other Users:
   - Many to many relationships, ie a user is following many users and is followed by many users.
   - The answer to the problem is just a simple join table, but it's just slightly different than a standard join table, as it joins a table to itself, and requires a few extra words in the ActiveRecord class methods. The join table I used was: 

```ruby
# schema.rb

create_table "following_users", force: :cascade do |t|
    t.integer "following_id"
    t.integer "follower_id"
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
end
```
   - And then to make the User model work correctly:

```ruby
# user.rb

has_many :following_users, foreign_key: "follower_id", dependent: :destroy
has_many :follower_users, class_name: "FollowingUser", foreign_key: "following_id", dependent: :destroy
has_many :followers, class_name: "User", through: :follower_users, foreign_key: "follower_id"
has_many :following, class_name: "User", through: :following_users, foreign_key: "following_id"
```
#### Messaging
- This was pretty similar to implementing following/followers, except the model `Message` is the join table, and is also the model that we are interested in, so it is simply just:

```ruby 
# schema.rb

create_table "messages", force: :cascade do |t|
    t.integer "sender_id"
    t.integer "reciever_id"
    t.text "content"
    t.boolean "viewed", default: false
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
end
```

```ruby
# user.rb

has_many :sent_messages, :class_name => "Message", :foreign_key => "sender_id", dependent: :destroy
has_many :recieved_messages, :class_name => "Message", :foreign_key => "reciever_id", dependent: :destroy
```

#### Reacting to Posts

   - There were a few solutions to this problem I considered, but I decided to make a table `ReactionType` which has a many-to-many polymorphic relationship to anything that is `reactable`, in my case `Topic`s and `Post`s. `Reactions` were the join tables between the `reactables` and the `ReactionTypes`. I did this instead of `Reaction` holding the reaction type via a string in order to minimize errors (ie spelling errors), and allow for flexibility, for example, easy expansion of allowed `ReactionTypes` in the future. 
     - Doing it this way made `Reactions` a 3-way join table between `User`, `reactables`, and `ReactionType`
     - `ReactionType`s have to be seeded to the types you want to be available to the user. I had `like`, `dislike`, `genius`, and `report`.

```ruby
# schema.rb

  create_table "reaction_types", force: :cascade do |t|
    t.string "name"
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
  end

  create_table "reactions", force: :cascade do |t|
    t.integer "user_id"
    t.string "reactable_type"
    t.bigint "reactable_id"
    t.integer "reaction_type_id"
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
    t.index ["reactable_type", "reactable_id"], name: "index_reactions_on_reactable_type_and_reactable_id"
  end

```

```ruby
# reaction.rb

  belongs_to :user
  belongs_to :reaction_type
  belongs_to :reactable, polymorphic: true
```

```ruby
# post.rb

has_many :reactions, as: :reactable, dependent: :destroy
has_many :reaction_types, through: :reactions

#topic.rb

has_many :reactions, as: :reactable, dependent: :destroy
has_many :reaction_types, through: :reactions
```

   - In the same fashion using polymorphic relationships, you can create tags that can be attached to `taggables`, and post replies to things which are `postable`.

#### Advanced ActiveRecord class methods 
   - When I was trying to make the correct ActiveRecord class methods to retrieve specific "statistics" for a user dashboard or to find popular topics for my homepage, I found that the best resource for me was to see other advanced and specific class methods and use those examples as a reference. I will put some of mine here in the hopes that others can use it as a reference.
     - To create a "spotlight topic" of the day for my homepage - take the most positively reacted topic in the past 24 hours:

```ruby
# topic.rb

def self.trending_today
    Topic.joins(:reaction_types).where(reaction_types: {name: ['like', 'genius']}, classroom_id: nil).where(reactions: { created_at: ((Time.now - 24 * 3600)..Time.now)}).group('id').order(Arel.sql('count(reaction_type_id) DESC')).limit(1)
end
```

- Find the most reacted `Topic` by `User` and `reaction_type`:

```ruby
#topic.rb

def self.most_liked_by_user(user, limit=5)
    most_reacted_type_by_user(user, 'like', limit, false)
end

private

def self.most_reacted_type_by_user(user, type, limit, count)
    if count
        Topic.joins(:reaction_types).where(reaction_types: {name: type}, user_id: user.id).group('id').order(Arel.sql('count(reaction_type_id) DESC')).limit(limit).count
    else
        Topic.joins(:reaction_types).where(reaction_types: {name: type}, user_id: user.id).group('id').order(Arel.sql('count(reaction_type_id) DESC')).limit(limit)
    end
end

#user.rb

def most_liked_topics(limit=5)
    Topic.most_liked_by_user(self, limit)
end
```

### Setting up Bootstrap
- [This post](https://medium.freecodecamp.org/add-bootstrap-to-your-ruby-on-rails-project-8d76d70d0e3b) walks through how to set up your rails app with bootstrap in just a few minutes.
- There is a complication with bootstrap that won't be ideal right 'out of the box' -> while rails will give your `form_for` fields the `fields_with_errors` class automatically when there are issues with the form inputs, Bootstrap will not respond to it. Instead, it responds to the class `is-invalid`.
  - [This blog](https://jasoncharnes.com/bootstrap-4-rails-fields-with-errors/) shows a working good solution to make form errors work well with rails. It just includes adding a file and code to your `config/initializers` folder, and basically, all it does is change the default `field-with-errors` class to `is-invalid` instead, which is what bootstrap uses to indicate a field which was filled out incorrectly.

### Rendering Markdown 
- I was between the two libraries `redcarpet` and `kramdown`. I ended up choosing [kamdown](https://github.com/gettalong/kramdown) because:
  - It is more recently committed and seems to be maintained more
  - Its popularity is growing while `redcarpet` is decreasing
  - It allows integration with `MathJax` which allows LaTex rendering.
- Getting basic integration was relatively simple. It got a little more complicated to achieve the following 2 goals:
  - Syntax highlighting for code
  - Being able to sanitize the user's input while allowing all of the Markdown features to implement correctly.

- I decided to use [rouge](https://github.com/jneen/rouge) for syntax highlighting. I'll show you the code below to get it working with Kramdown, but the hard piece of information to find was how to get the proper CSS files that make the highlighting happen. Here's a good [stack overflow answer](https://stackoverflow.com/questions/43905103/kramdown-rouge-doesnt-highlight-syntax) that's hard to find. Basically, once you tell `kramdown` that you want to use `rouge` (and you have installed both the `kramdown gem` and the `rouge gem`), you can run `rougify help style` in your terminal, and you can see all of the custom CSS files you can add to your `app/assets/stylesheets` path. To get the raw CSS, type `rougify style` followed by one of the allowed styles. For example: `rougify style colorful`. The CSS will be printed in your terminal. You could also run   
  
```
rougify style colorful > ./app/assets/stylesheets/rouge_style.css
```

- Now, it's relatively easy to render markdown. I made a helper method to make it extra simple:  


```ruby
# application_helper.rb

def render_markdown(markdown)
    render 'components/content', markdown: Kramdown::Document.new(markdown, parse_block_html: true, syntax_highlighter: :rouge, syntax_highlighter_opts: {line_numbers: false}).to_html
end

```
```erb

<!-- views/components/_content.html.erb -->

<%= sanitize_markdown markdown %>

``` 
- Custom sanitize method:
  - As you can see here, I am not using `raw` or `.html_safe` to render this HTML data because ultimately it *came from a user and cannot be trusted*. However, the safe `sanitize` method disallows certain things I want to be rendered, such as tables. You can get proper sanitation AND your desired HTML tags by manually appending to the whitelisted tags allowed through `sanitize` by doing something like what is shown below.
```ruby
# application_helper.rb

def sanitize_markdown(content)
    sanitize(content, tags:  Loofah::HTML5::WhiteList::ALLOWED_ELEMENTS_WITH_LIBXML2.to_a + %w(table th td tr span), attibutes: Loofah::HTML5::WhiteList::ALLOWED_ATTRIBUTES + %w( style ))
end
```

### General tips
- Set up the app with PostgreSQL
  - [Here's a blog post](https://dev.to/micahshute/setting-up-windows-subsytem-for-linux-3b7n) I wrote about setting up WSL, but there's a section at the bottom that covers using pgAdmin and setting up a Rails app with PostgreSQL
- Save all secret keys as environment variables. I used [Figaro](https://github.com/laserlemon/figaro) to make it easy.
- Avoid the N+1 problem
  - This occurs when you get a collection of models, of which you want to query and show nested models. If you iterate over your models `n` times to do this, you are making `n+1` database calls which can be quite expensive with large amounts of data, especially when requesting over a network. This is easily fixed by using the `includes` method when making your initial query. [Check this site out](https://guides.rubyonrails.org/active_record_querying.html) and go to the section called "Solution to N + 1 queries problem" for more info. 
- Clean up the database automatically when objects are destroyed
  - When you appropriately add 
  ```ruby 
  dependent: :destroy
  ``` 
  to your ActiveRecord class methods (examples can be seen in code snippets above), you are telling ActiveRecord to destroy these relations upon the destruction of the model. As you can see above, I implemented this for `reactions` within `post.rb`. This tells rails that when a `post` is destroyed, all user `reactions` to it should also be destroyed. This prevents disconnected likes and dislikes, etc, from floating around in your database for no reason. Also, note this should be a one-sided relationship. You would not want a `post` to be destroyed if a user destroys their one reaction to it.  

- Adding custom files and classes
  - There are going to be things you want your app to do that are outside of MVC. This means you are going to want to add files outside of the standard `model`, `view` and `controller` directories. Here are some resources with advice on how to do this:
    - [A StackOverflow answer](https://stackoverflow.com/questions/15260984/guidelines-for-where-to-put-classes-in-rails-apps-that-dont-fit-anywhere)
    - [A GitHub gist post](https://gist.github.com/maxim/6503591)
- Personally, I used a `lib` directory in my `app` directory because it is eagerly loaded in production and lazy loaded in development. I did not need to alter any configuration or environment files for the classes within files in that directory were referenced.

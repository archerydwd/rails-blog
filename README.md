# Ruby on Rails Blog Application

Please note this is not a tutorial, I have wrote it in that style so you can follow along. If you get into trouble (like I did) try the mailing list or just google it. You will find that you will actually learn more from researching it and getting into tight spots. ;) 

I built this app with the Ruby on Rails framework to be used as part of a series of applications that I will be 
performing tests on. This is a Ruby on Rails version of the Chicago Boss blog application: https://github.com/archerydwd/cb_blog & the Flask version is here: https://github.com/archerydwd/flask_blog

I am going to be performing tests on this app using some load testing tools such as Gatling & Tsung. 

Once I have tested this application and the other verisons of it, I will publish the results, which can then be used as a benchmark for others when trying to choose a framework. 
You can build this app using a framework of your choosing and then follow the testing mechanisms that I will describe and then compare the results against my benchmark to get an indication of performance levels of your chosen framework.

==
###Installing Ruby and Rails
==

At time of writing this the Ruby version was: 2.2.0 and the rails version was: 4.2.0.

Use RVM, the Ruby Version Manager: https://rvm.io, to install Ruby and manage your Rails versions.
If you are using osx then there is a system version of Ruby, which will more than likely be out of data. Use RVM to install Ruby and the “system Ruby” will remain on your system but the RVM version will take precedence when calls are made to Ruby.

```
\curl -L https://get.rvm.io | bash -s stable --ruby
```

* If you are using a mac and your user name includes a space, you need to change it to remove the space. Directions on apple support site: http://support.apple.com/en-us/HT201548.

* If you have any problems, check that your xcode version is up to date and make sure to open it and accept the licence too. Also you need to install the xcode command line tools: 

```
xcode-select --install
```


If you have RVM installed you can update it and then install Ruby.
```
rvm get stable --autolibs=enable
rvm install ruby
rvm --default use ruby-2.2.0
```

Use the following to update the gem manager:

```
gem update --system
```

To install the latest version of Ruby on Rails and check the version afterwards:

```
gem install rails
rails -v
```

**Create the blog app**
```
rails new ror_blog
cd ror_blog
```

==
###Building the application
==

**Starting the development server**

To start the dev server:
```
rails server
```

To stop the development server:
```
ctrl + c
```

**Create articles**

In rails we can use something called generators to generate controllers with actions. To generate our articles controller with the methods index, show, new, destroy, edit, update:
```
rails generate controller articles index show new destroy edit update
```

The above creates a few things for us, the following is the output:
```
create  app/controllers/articles_controller.rb
route  get 'articles/update'
route  get 'articles/edit'
route  get 'articles/destroy'
route  get 'articles/create'
route  get 'articles/show'
route  get 'articles/index'
route  get 'articles/new'
invoke  erb
create    app/views/articles
create    app/views/articles/index.html.erb
create    app/views/articles/show.html.erb
create    app/views/articles/create.html.erb
create    app/views/articles/destroy.html.erb
create    app/views/articles/edit.html.erb
create    app/views/articles/update.html.erb
create    app/views/articles/new.html.erb
invoke  test_unit
create    test/controllers/articles_controller_test.rb
invoke  helper
create    app/helpers/articles_helper.rb
invoke    test_unit
create      test/helpers/articles_helper_test.rb
invoke  assets
invoke    coffee
create      app/assets/javascripts/articles.js.coffee
invoke    scss
create      app/assets/stylesheets/articles.css.scss
```

For the purposes of this we are only concerned with:
* create app/controllers/articles_controller.rb
* route  get 'articles/update'
* route  get 'articles/edit'
* route  get 'articles/destroy'
* route  get 'articles/create'
* route  get 'articles/show'
* route  get 'articles/index'
* * route  get 'articles/new'
* create app/views/articles/ all of them

the controller holds actions which map to our templates (views) and the requesting url is matched with what is in the route file (config/routes.rb) and this tells rails which action to call, and in which controller.
For the above, we actually only need the views: edit.html.erb, index.html.erb, new.html.erb and show.html.erb.
So we can remove the rest of them:
```
rm app/views/articles/destroy.html.erb
rm app/views/articles/update.html.erb
rm app/views/articles/create.html.erb
```
To make app/views/articles/index.html.erb appear as our root page, eg. when we go to http://localhost:3000/. We need to edit: config/routes.rb and add the following:
```
root 'articles#index'
```

**Update the views**

Next we must update the index view: app/views/articles/index.html.erb. This is done using HTML and ERB (Embedded Ruby). View files always end in .html.erb.
```
<h1>Listing articles</h1>
<%= link_to 'New article', new_article_path %>
<table>
  <tr>
    <th>Title</th>
    <th>Text</th>
    <th colspan="3"></th>
  </tr>

  <% @articles.each do |article| %>
    <tr>
      <td><%= article.title %></td>
      <td><%= article.text %></td>
      <td><%= link_to 'Show', article_path(article) %></td>
      <td><%= link_to 'Edit', edit_article_path(article) %></td>
      <td><%= link_to 'Destroy', article_path(article), method: :delete, data: { confirm: 'Are you sure?' } %></td>
    </tr>
  <% end %>
</table>
```

We can then update: app/views/articles/show.html.erb
```
<p>
  <strong>Title:</strong>
  <%= @article.title %>
</p>
<p>
  <strong>Text:</strong>
  <%= @article.text %>
</p>
<h2>Comments</h2>
<%= render @article.comments %>
<h2>Add a comment:</h2>
<%= render "comments/form" %>

<%= link_to 'Back', articles_path %>
| <%= link_to 'Edit', edit_article_path(@article) %>
```

In the above, you will notice something about comments, include this and we will have a look later at them.
Next we will update: app/views/articles/new.html.erb

```
<h1>New article</h1>
<%= render 'form' %>
<%= link_to 'Back', articles_path %>
```
Short and sweet, here we are including the file: app/views/articles/_form.html.erb which is a reusable element that we will now create:

>touch app/views/articles/_form.html.erb

Then edit it using your text editor and enter the following:

```
<%= form_for @article do |f| %>
  <% if @article.errors.any? %>
    <div id="error_explanation">
      <h2><%= pluralize(@article.errors.count, "error") %>
        Prohibited this article from being saved:</h2>
      <ul>
        <% @article.errors.full_messages.each do |msg| %>
          <li><%= msg %></li>
        <% end %>
      </ul>
    </div>
  <% end %>
  <p>
    <%= f.label :title %><br>
    <%= f.text_field :title %>
  </p>
  <p>
    <%= f.label :text %><br>
    <%= f.text_area :text %>
  </p>
  <p>
    <%= f.submit %>
  </p>
<% end %>
```

Now edit: app/views/articles/edit.html.erb
```
<h1>Edit article</h1>
<%= render 'form' %>
<%= link_to 'Back', articles_path %>
```
Another short and sweet file.
Ok that's all of our views for articles, we will now move onto the controller.

**The articles controller**

The controller for articles was created above with out generator command, so lets now edit: app/controllers/articles_controller.rb

```
class ArticlesController < ApplicationController

  def destroy
    @article = Article.find(params[:id])
    @article.destroy
    redirect_to articles_path
  end

  def edit
    @article = Article.find(params[:id])
  end

  def update
    @article = Article.find(params[:id])
    if @article.update(article_params)
      redirect_to @article
    else
      render 'edit'
    end
  end

  def new
    @article = Article.new
  end

  def index
    @articles = Article.all
  end

  def show
    @article = Article.find(params[:id])
  end

  def create
    @article = Article.new(article_params)
    if @article.save
      redirect_to @article
    else
      render 'new'
    end
  end

  private
  def article_params
    params.require(:article).permit(:title, :text)
  end

  private
  def article_params
    params.require(:article).permit(:title, :text)
  end
end
```

The above has lots of methods, The thing to note for now is the method names corrospond to our view names. This helps when the controller method index gets called, rails knows that the index.html.erb file needs to be displayed with this.

**Creating the articles model**

Now we need a model to complete the M in our MVC. To do this just enter the following:

```
rails generate model Article title:string text:text
```

This creates the following file: app/models/article.rb:

```
class Article < ActiveRecord::Base
end
```

We now want to add a validation to check for empty titles and a min length of 5 characters: 

```
class Article < ActiveRecord::Base
  validates :title, presence: true, length: { minimum: 5 }
end
```

The above generator also created: db/migrate/20150203151229_create_articles.rb

```
class CreateArticles < ActiveRecord::Migration
  def change
    create_table :articles do |t|
      t.string :title
      t.text :text

      t.timestamps
    end
  end
end
```

This is used for storing data in the database, in the form we wanted.

**Creating the comments**

for this, we will need a controller, model and 2 views which are just include files. Lets start with the model:

```
rails generate model Comment commenter:string body:text article:references
```

The above has one difference to the article model generator, it includes a article:references field. This just means that we will create a comment model that will hold references to articles.

If you now look at: app/models/comment.rb you will see a line that says belongs_to :article this is the way rails links active records, so comments know to what article they belong.

```
class Comment < ActiveRecord::Base
  belongs_to :article
end
```

We now need to add 'has_many :comments dependent: :destroy' to: app/models/article.rb

```
class Article < ActiveRecord::Base
  has_many :comments, dependent: :destroy
  validates :title, presence: true, length: { minimum: 5 }
end
```

This means that each article has many comments. If an article is deleted, destroy its connected comments.

Now we need to add comments to: config/routes.rb as a nested resource within articles:

```
resources :articles do
    resources :comments
  end
```

**Create the comments controller**

Now we need to create a controller, to do this we will again use the generator:

```
rails generate controller Comments
```

Update this controller: app/controllers/comments_contoller.rb

```
class CommentsController < ApplicationController

  def destroy
    @article = Article.find(params[:article_id])
    @comment = @article.comments.find(params[:id])
    @comment.destroy
    redirect_to article_path(@article)
  end

  def create
    @article = Article.find(params[:article_id])
    @comment = @article.comments.create(comment_params)
    redirect_to article_path(@article)
  end

  private
    def comment_params
      params.require(:comment).permit(:commenter, :body)
    end

end
```

**Create the forms for the comments**

As we seen earlier, the articles/show.html.erb mentions comments in two render parts, Lets now create them.

>touch app/views/comments/_comment.html.erb

>touch app/views/comments/_form.html.erb

Then edit: app/views/comments/_comment.html.erb

```
<p>
  <strong>Commenter:</strong>
  <%= comment.commenter %>
</p>
<p>
  <strong>Comment:</strong>
  <%= comment.body %>
</p>
<p>
  <%= link_to 'Destroy Comment', [comment.article, comment], method: :delete, data: { confirm: 'Are you sure?' } %>
</p>
```

This will display all comments for a certain article being viewed. It will also give us the option to delete a comment.

Now edit: app/views/comments/_form.html.erb

```
<%= form_for([@article, @article.comments.build]) do |f| %>
  <p>
    <%= f.label :commenter %><br>
    <%= f.text_field :commenter %>
  </p>
  <p>
    <%= f.label :body %><br>
    <%= f.text_area :body %>
  </p>
  <p>
    <%= f.submit %>
  </p>
<% end %>
```

This is just a form to create new comments.

**Migrations**

Every time that we created a model, we also created a migration for that model. This is responsible for creating the database for storing that certain model. To run the migrations we have so far:

```
rake db:migrate
```

This will create the tables associated with the models in the database. For this application, we don't perform tests on the response time from the database, so this is as far as I will go in explaining migrations. If you want to learn more on them, I have a much more detailed application called 'ror_sakila', which uses a version of the mysql sakila database.

==
###Getting Production Ready
==

The following changes must be made in order to have the application in a production environment ready state.

**Add A Secret Key**

We need to add a secret key for production mode, this can be done by changing directory to the railsblog and running the following command:

```
rake secret
```

If this does not work, then please run:

```
bundle exec rake secret
```

The above command will produce a secret key. Copy this key and place it in as a production key in: config/secrets.yml

```
development:
  secret_key_base: 05215a3d1a1dd8648f7c6ee3c333778e5c99cb9115fdf313f1ee79b276008dd32d36817dc60bded89d3eb7d679797096758ae2667c08ff756e234851016296ed

test:
  secret_key_base: 5b3e49d01fe19e183c7c5d4a758352071cf581d1192f9c3c137f99e8c383436f3cc99893bf1fb34a8140c17cfdebfce67e44ea64ba25c85cdee8cb72d5a9512f

production:
  secret_key_base: 5ec6914064c1f8ebf3fe3e0ccb29351240db088138e68462f2067582066a3e6bba48792b41ec22193567893f5638a564f9c44a6bdceeb9083ba6b6f74ae0eba8
```

In the normal use case this secret key should be placed in an environment variable and read in here. For us this is not necessary as we are not deploying a release of the application.

**Change The Production Database**

In the normal development use case you should have a seperate database for development, testing and production. In our example we can use the development database for production as we are not building a product for general release but rather for testing purposes. In saying that we now need to rename the production database from production to development as per the following: config/database.yml:

```
# SQLite version 3.x
#   gem install sqlite3
#
#   Ensure the SQLite 3 gem is defined in your Gemfile
#   gem 'sqlite3'
#
default: &default
  adapter: sqlite3
  pool: 5
  timeout: 5000

development:
  <<: *default
  database: db/development.sqlite3

# Warning: The database defined as "test" will be erased and
# re-generated from your development database when you run "rake".
# Do not set this db to the same as development or production.
test:
  <<: *default
  database: db/test.sqlite3

production:
  <<: *default
  database: db/development.sqlite3
```

**Add a new route**

As we will be testing this application along with other versions of it, I have added a route that will match with the articles/new route to make it easier to deploy the same test across the different versions of the app. Edit config/routes.rb and add the following:

```
match 'articles/create' => 'articles#new', via: :get
```

**How To Run In Production Mode**

To start the app in the production mode issue the command:

```
rails s -e production
```

**Problem**

If you get a problem when you run the app and try to delete a comment or an article. Then you need to do the following:

```
vim Gemfile
```

uncomment the line that includes 'therubyracer'
and then:

```
bundle install
```

==
###The End
==

Thats all there is to it. 
Thanks for reading and hopefully you learned something. :)

Darren.
















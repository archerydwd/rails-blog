# Ruby on Rails Blog Application

I built this app with the Ruby on Rails framework to be used as part of a series of applications that I will be 
performing tests on. This is a Ruby on Rails version of the Chicago Boss blog application: https://github.com/archerydwd/boss-blog.

I am going to be performing tests on this app using some load testing tools such as Tsung, J-Meter and Basho bench. 

Once I have tested this application and the Chicago Boss verison of it, I will publish the results, which can then be used as a benchmark for others to use when trying to choose a framework. 
You can build this app using a framework of your choosing and then follow the testing mechanisms that I will describe and then compare the results against my benchmark to get an indication of performance levels of your chosen framework.

###Installing Ruby and Rails

At time of writing this the Ruby version was: 2.2.0 and the rails version was: 4.2.0
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
rails new blog
cd blog
```

###Building the application

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

  http_basic_authenticate_with name: "yourname", password: "secret", except: [:index, :show]

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

The above has lots of methods, The thing to note for now is the method names corrospond to our view names. This helps when the controller method index gets called, rails knows that the index.html.erb file needs to be displayed with this. The athentication line at the top, provides very basic authentication and except allows them files to be accessed without login.

**Creating the articles model**

Now we need a model to complete the M in our MVC. To do this just enter the following:

```
rails generate model article title:string text:text
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

======






**The end**

Thats all there is to it. This was my first self made (without a tutorial) application with Chicago Boss, so if you notice
any problems or enhancements, please drop me a message. 

Thanks for reading and hopefully you learned something. :)

Darren.
















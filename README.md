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
make app PROJECT=blog
cd ../blog
```

###Building the application

**Create the article model**

First we will need an article model, this file lives at: src/model/article.erl and you should 
insert the following into it:
```
-module(article, [Id, ArticleTitle, ArticleText]).
-compile(export_all).
```
We need to explain some things here. 

* The name of the file should be non-plural and in -module the name should be the exact same (not including .erl extension of course)
* The attribute list is the [Id, ArticleTitle, ArticleText] part and this should always start with Id, which sets Chicago Boss to auto generate the id. The names should all use camel case (even though in the database it will be article_title).
* -compile(export_all) is put in to export all functions available to the article model.

**Create the article controller**

The second thing we will need to do is to create the controller associated with the article. This file will live at: 
src/controller/blog_articles_controller.erl
```
-module(blog_articles_controller, [Req]).
-compile(export_all).   
index('GET', []) ->
    Articles = boss_db:find(article, []),
    {ok, [{articles, Articles}]}.
```
Again in the above we are placing the name of the file in the module part. Please note the structure for the name of this 
file is: APPNAME_MODELNAME_CONTROLLER.ERL so for our case the appname is blog, the modelname is articles and then controller: 
blog_articles_controller.erl also note that the modelname is plural here.

This controller gets all the articles and provides them in a list to the index template that we will now create.
please note the name of the actions in this controller have to be the same as the template they are calling.
eg: index will get us the index.html template.

**Display all articles template**

The next thing we will need to do is create the template for displaying the articles, eg an index page. This file 
lives at: src/view/articles/index.html you will need to create the directory src/view/articles, please note it is also plural.
>mkdir src/view/articles
```
<html>
  <body>
    <h1>Listing articles</h1>
    <table>
      <tr>
        <th>Title</th>
        <th>Text</th>
      </tr>
      {% for article in articles %}
        <tr>
          <td>{{ article.article_title }}</td>
          <td>{{ article.article_text }}</td>
        </tr>
      {% endfor %}
    </table>
  <body>
</html>
```

The Chicago Boss templating uses Django's templating, Documentation can be found here: http://www.chicagoboss.org/api-view.html

**Associating comments with articles**

We first must create the comment model: src/model/comment.erl
```
-module(comment, [Id, Commenter, Body, ArticleId]).
-compile(export_all).
-belongs_to(article).
```
The above says that each comment should belong to 1 article.

To follow this up we must add the following -has line to the src/model/article.erl file.
```
-module(article, [Id, ArticleTitle, ArticleText]).
-compile(export_All).

-has({comments, many}).
```

While the server is running, navigate to localhost:8001/doc/article and in that you should see comments/0 and comments/1 methods have been made available. Also navigate to localhost:8001/doc/comment and you will see that article/0 and article/1 methods have been made available. If these methods are not here, it means that you have made a mistake in the above 2 code boxes.

**Display individual articles**

The first thing we will do is create a link to the individual articles in the src/view/articles/index.html file, by just adding the following:
```
...
  <tr>
    <th>Title</th>
    <th>Text</th>
    <th colspan="1"></th>
  </tr>
  {% for article in articles %}
    <tr>
      <td>{{ article.article_title }}</td>
      <td>{{ article.article_text }}</td>
      <td><a href="/articles/show/{{ article.id }}">Show</a></td>
...
```

Then in the src/controller/blog_articles_controller.erl file we need to add the following action:
```
...
show('GET', [ArticleId]) ->
   Article = boss_db:find(ArticleId),
   {ok, [{article, Article}]}.

```

We now need the src/view/articles/show.html file:
```
<html>
  <body>
    <p>
      <strong>Title:</strong>
      {{ article.article_title }}
    </p>
    <p>
      <strong>Text:</strong>
      {{ article.article_text }}
    </p>
    <a href="/articles/index">Back</a>
  </body>
</html>
```

Now we need a way to create new articles so we can then create new comments on them.

**Creating the articles**

We need to add a link to the create template from the index template (src/view/articles/index.html)
```
<html>
  <body>
    <h1>Listing articles</h1>
    <a href="{% url action="create" %}">New article</a>
    <table>
      <tr>
...
```

The action should be the same name that you have in your controller and the same name of the create template.

now we will add the create template to src/view/articles/create.html
```
<html>
  <body>
    <h1>New article</h1>
    {% if errors %}
      <ol>
        {% for error in errors %}
          <li><font color=red>{{ error }}</font>
        {% endfor %}
      </ol>
    {% endif %}
    
    <form method="post">
      <p>
        Title:<br>
        <input name="article_title" value="{{ article.article_title|default_if_none:'' }}"/>
      </p>
      <p>
        Text:<br>
        <textarea name="article_text" value="{{ article.article_text|default_if_none:'' }}"></textarea>
      </p>
      <p>
        <input type="submit" value="Create Article"/>
      </p>
    </form>
    <a href="{% url action="index" %}">Back</a>
  </body>
</html>
```

And finally we will add the create action to the src/controller/blog_articles_controller.erl
```
...
create('GET', []) -> ok;
create('POST', []) -> Article = article:new(id, Req:post_param("article_title"), Req:post_param("article_text")),
  case Article:save() of
    {ok, SavedArticle} -> {redirect, "/articles/show/"++SavedArticle:id()};
    {error, Errors} -> {ok, [{errors, Errors}, {article, Article}]}
  end.
```

The above create action has 2 versions, one is for displaying the create page, and one is for processing the 
posted data from the create page. It extracts the information by use of Req:post_param() and then creates a 
new article with the article:new() method and then saves it with the :save() method. If it saves ok, then a 
redirect occurs to view that article otherwise we get an error because the validation_tests in the model have 
failed, again this is explained in the adding validation section.

Now navigate to: http://localhost:8001/articles/create in your browser and you should see a form for 
creating the articles, submit a few. You should see them displayed individually and they should 
display in the list of articles on the index page.



**Starting the development server**

To start the dev server:
```
./init-dev.sh
```

oh and to stop the server:
```
ctrl + c
```

**Deleting an article**

To delete an article we need to update the src/view/articles/index.html page to include a link to 
delete as per the following:
```
...
    <th>Text</th>
    
    <th colspan="2"></th>
  
  </tr>
  {% for article in articles %}
    <tr>
      <td>{{ article.article_title }}</td>
      <td>{{ article.article_text }}</td>
      <td><a href="/articles/show/{{ article.id }}">Show</a></td>
      
      <td><a href="/articles/delete/{{ article.id }}">Destroy</a></td>
      
    </tr>
  {% endfor %}
...
```

We now need to add the delete action to the controller. src/controller/blog_articles_controller.erl
```
...
delete('GET', [ArticleId]) ->
  boss_db:delete(ArticleId),
  {redirect, [{action, "index"}]}.
```

**Adding validation**

In the creating new articles section above we seen the following code in the view:
```
{% if errors %}
  <ol>
    {% for error in errors %}
      <li><font color=red>{{ error }}</font>
       {% endfor %}
  </ol>
{% endif %}
```
This code basically means if there are any reported errors, print them out, so where did these errors come from? well if we look at the controller:
```
create('GET', []) -> ok;
create('POST', []) -> Article = article:new(id, Req:post_param("article_title"), Req:post_param("article_text")),
  case Article:save() of
    {ok, SavedArticle} -> {redirect, "/articles/show/"++SavedArticle:id()};
    {error, Errors} -> {ok, [{errors, Errors}, {article, Article}]}
  end.
```
We see the create method either performs a redirect to display the saved article, or if there was an error it pushes those errors back instead. So now you are asking: Where is the new article being checked for errors? The answer to this is: It is not currently being checked and we now need to add the code to check for errors.

To add validation that only allows article titles to be longer than 5 characters, in the src/model/article.erl add the following code:
```
-module(article, [Id, ArticleTitle, ArticleText]).
-compile(export_All).
-has({comments, many}).

validation_tests() ->
 [{fun() -> length(ArticleTitle) > 0 end, "Title can't be blank"},
  {fun() -> length(ArticleTitle) >= 5 end, "Title is too short (minimum is 5 characters)"}].
```
Then if you navigate to localhost:8001/article/create and enter a title that is less than 5 characters long, it will produce and display an error. This will also happen if you don’t enter a title, you will get both errors. So what is happening here? The controller is trying to save an article, but because we have declared this validation_tests method in the model, they must execute first, if an error is found, the controller then passes these errors back to the view, which then displays them for your viewing convenience.

**Updating articles**

To add the update functionality we need to create a new file in: src/view/article/update.html and insert the following into it:
```
<html>
  <body>
    <form method="post">
      <p>
        Title:<br>
        <input name="article_title" value="{{ article.article_title }}"/>
      </p>
      <p>
        Text:<br>
        <textarea name="article_text">{{ article.article_text }}</textarea>
      </p>
      <p>
        <input type="submit" value="Update Article"/>
      </p>
    </form>
    <a href="{% url action="index" %}">Back</a>
  </body>
</html>
```
We need to add a link to the src/view/articles/show.html file so we can edit the article:
```
...
  {{ article.article_text }}
</p>
<a href="/articles/index">Back</a> | <
a href=”/articles/update/{{ article.id }}”>Edit</a>
```

Now we also need to add a link into the src/view/articles/index.html file:
```
...
  <tr>
    <td>{{ article.article_title }}</td>
    <td>{{ article.article_text }}</td>
    <td><a href="/articles/show/{{ article.id }}">Show</a></td>
    <td><a href="/articles/update/{{ article.id }}">Edit</a></td>
    <td><a href="/articles/delete/{{ article.id }}">Destroy</a></td>
...
```

In the above we just added a link that calls the update action in the controller with the id of the article.
Now to get the update to work we need to add an update action to the controller at: src/controller/blog_articles_controller.erl and insert this:
```
...
update('GET', [ArticleId]) -> Article = boss_db:find(ArticleId), {ok, [{article, Article}]};
update('POST', [ArticleId]) ->
   Article = boss_db:find(ArticleId),
   EditedArticle = Article:set([{article_title, Req:post_param("article_title")},
                                {article_text,Req:post_param("article_text")}]),
   EditedArticle:save(),
   {redirect, [{action, "index"}]}.
```
This action has 2 clauses, 1 which gives back that article and 1 which accepts a form submission and updates the information of that article.

**Creating comments**

*Issue:*

With this I had a problem, due to the nature of chicago boss each action maps to a view, and due to the comments 
being created in the src/articles/show.html view, when I submitted the form, it is automatically going to the 
controller: src/controller/blog_articles_controller.erl and hitting the show action, Instead of going to the 
controller: src/controller/blog_comments_controller.erl and hitting the create method.

*Fix:*

The solution that I arrived at for this issue was to create a new route that mapped to a create action in the 
comments controller, this routing file lives at: priv/blog.routes and it will be populated with a few examples. 
If you change the first example under the Formats: section to the following:
```
% Formats:
{"/articles/comments/create", [{controller, "comments"}, {action, "create"}]}.
```
this says that when the url: /article/comments/create is entered, map it to the controller: comments and the 
action: create.

In order to display the comments and get this url input we need to edit the src/view/article/show.html file as per the following:
```
<html>
  <body>
    <p>
      <strong>Title:</strong>
      {{ article.article_title }}
    </p>
    <p>
      <strong>Text:</strong>
      {{ article.article_text }}
    </p>
    <h2>Comments</h2>
    {% for comment in article.comments %}
      <p>
        <strong>Commenter:</strong>
        {{ comment.commenter }}
      </p>
      <p>
        <strong>Comment:</strong>
        {{ comment.body }}
      </p>
    {% endfor %}
    <h2>Add a comment:</h2>
    <form method="post" action="{% url action="comments/create" %}">
      <input type="hidden" name="id" value="{{ article.id }}" />
      <p>
        Commenter:<br>
        <input name="commenter" value="{{ comment.commenter|default_if_none:'' }}"/>
      </p>
      <p>
        Body:<br>
        <textarea name="body" value="{{ comment.body|default_if_none:'' }}"></textarea>
      </p>
      <p>
        <input type="submit" value="Create Comment"/>
      </p>
    </form>
    <a href="/articles/index">Back</a> | 
    <a href="/articles/update/{{ article.id }}”>Edit/a> 
  </body>
</html>
```

The part for displaying the comments makes use of the article.comments method which returns all comments that belong to this article. We can then loop through them and display them. The part that is responsible for getting the correct url is the url action=”comments/create” attribute in the form head.

Now we need a controller for the comment and an action called create in it, this file lives at: src/controller/blog_comments_controller.erl, insert the following into it:
```
-module(blog_comments_controller, [Req]).
-compile(export_all).

create('POST', []) -> Comment = comment:new(id, Req:post_param("commenter"), Req:post_param("body"), Req:post_param("id")),
  Comment:save(),
  {redirect, [{controller, "articles"},{action, "show"},{article_id, Comment:article_id()}]}.
```
We should now be able to add comments and see them while viewing an individual article.

**Deleting comments**

We need to edit the src/view/articles/show.html file to include a link for deleting comments:
```
...
    <strong>Comment:</strong>
    {{ comment.body }}
  </p>
  <a href="/comment/delete/{{ comment.id }}">Delete Comment</a>
{% endfor %}
...
```

The above form submits the comment’s id and article_id values, which are needed to delete the comment and 
redirect back to show this article again.

We need to insert the following into the src/controller/blog_comments_controller.erl file:
```
...
delete('GET', [CommentId]) ->
  Comment = boss_db:find(CommentId),
  ArticleId = Comment:article_id(),
  boss_db:delete(CommentId),
  {redirect, [
    {controller, "article"},
    {action, "show"},
    {article_id, ArticleId}
  ]}.
```
In this, we are getting the comment by using the comment id and then we are getting the article id from the 
comment, Deleting the comment and then redirecting to show. 

So thats all there is to deleting comments, go to http://localhost:8001/articles/index and select an article 
and add a few comments and then try to delete some of them.

**Routes**

Edit the priv/blog.routes file and input:
```
% Front page
{"/", [{controller, "articles"}, {action, "index"}]}.
```
This makes http://localhost:8001/ redirect to http://localhost:8001/articles/index

**The end**

Thats all there is to it. This was my first self made (without a tutorial) application with Chicago Boss, so if you notice
any problems or enhancements, please drop me a message. 

Thanks for reading and hopefully you learned something. :)

Darren.
















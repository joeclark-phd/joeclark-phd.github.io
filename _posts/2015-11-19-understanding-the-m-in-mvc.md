---
title: Understanding the "M" in MVC&colon; a database nerd tries to learn how SQLAlchemy ORM fits in with Flask
---

Having cut my teeth as a web developer in the bad old days of spaghetti code (when PHP was the innovative new thing!), I came back to it after several years away and have discovered with delight the new species of web framework they call MVC---model, view controller.

My favorite so far is [Flask](https://palletsprojects.com/p/flask/), a Python-based microframework.  Unlike the leading Python framework ([Django](https://www.djangoproject.com/)), it's a minimalistic framework that doesn't make many decisions for you.  I like to go slow and figure things out on my own, so that's perfect for me.  A simple Flask application is structured like so:

{% highlight python linenos %}
    from flask import Flask
    app = Flask(__name__)

    @app.route("/")
    def hello():
        return "<h1>Hello World!<h1>"

    @app.route('/user/<name>')
    def user(name):
        return "<h1>Hello, %s!</h1>" % name

    if __name__ == "__main__":
        app.run()
{% endhighlight %}

What's great about this is that instead of having PHP or some other kind of code threaded into your front-end HTML code, and spread across as many pages as you have in your application, all of your business logic is neatly organized in a Python code file separate from display logic.  In the example, each `@app.route()` decorator specifies a URL pattern and the function below it defines the logic.  (Instead of the crude `return` instructions in the example, you would generally call a template file full of HTML to format and style the output.)

## The trouble I have with the "M"

I have a great handle on the **controller** (the code file that handles the business logic) and on the **views** (the HTML and CSS templates).  The "M" for **model** is what I don't yet understand.  Hopefully, by the time I finish this blog post, I'll have worked it out.

Basically, as a database nerd who really cares about good SQL and likes to build clever logic into the database with stored procedures, triggers, recursive queries and so on, I expected to set up "routes" in Flask like this (from a project done by my students):

{% highlight python %}
    @app.route('/users')
    def users():
        #do query
        db.execute("SELECT * FROM users ORDER BY timestamp")
        rows = db.cur.fetchall()
        #pass result to template
        return render_template("users.html", rows=rows)
{% endhighlight %}

Simple, right?  A URL maps to a function, the function does some operations on the database (in this case a SELECT), and the results are fed into an HTML template which gets rendered as a response to the user.

But that's not how the pros do it.  What you're advised to do is to use an object-relational mapper (ORM) like [SQLAlchemy ORM](https://flask-sqlalchemy.palletsprojects.com/en/2.x/).  This means declaring your "database" "tables" as a kind of Python objects, trust SQLAlchemy to manage the database for you, and calling SQL-like functions on the objects in order to use data in your controller.  A "table" declaration might look like this:

{% highlight python %}
    class User(db.Model):
        __tablename__ = 'users'
        id = db.Column(db.Integer, primary_key=True)
        username = db.Column(db.String(80), unique=True)
        email = db.Column(db.String(120), unique=True)
        def __init__(self, username, email):
            self.username = username
            self.email = email
{% endhighlight %}

You create the table in the database with something like this:

{% highlight python %}
    db.create_all()
{% endhighlight %}

And you might use it in a view like so:

{% highlight python %}
    @app.route('/user/<username>')
    def show_user_profile(username):
        user = User.query.filter_by(username=username).first_or_404()
        return render_template('profile.html', user=user)
{% endhighlight %}

There's something here I must be missing.  I see how this might be a convenience for somebody who knows Python but doesn't know SQL, but if you care about your data (and I do) you're giving up lots of control over table creation, keys, indexes, and other optimizations.  You're losing the ability to use views, triggers, and stored procedures to put logic into the application.  And it looks like the Python application is going to be making changes to the database *structure* without asking or informing you about how it does that.  Other than the ability to write queries in something that looks like Python, what's the benefit?

## Figuring it out

A few days ago, when I wrote the above, I had to assume I was wrong about some of this, because "models" are deemed a core component of the MVC concept, and lots of Flask developers swear by the SQLAlchemy ORM.  The purpose of this blog post was to try to understand what this is all about, and to decide whether someone who *cares* about his database would hand over control to an ORM.  In order to begin the learning process, I dove into Miguel Grinberg's Flask tutorial ([in book form](http://amzn.to/1MmCdzY)) and, suspending my inhibitions, worked through all of the examples despite the pull of what must no doubt be a millennia-old caveman instinct to just write raw SQL.  This turned up a few clues to the popularity of the ORM approach, as follow in roughly the order I discovered them.

### First clue: Interchangeable databases

In Grinberg's tutorial, you use a SQLite database for development and testing, then switch to PostgreSQL when you deploy your app to Heroku.  It's pretty cool that you don't have to change any of the code to do this---all SQLAlchemy needs is a database URL from the DATABASE_URL environment variable, and it looks at the prefix (e.g., `postgres://`) to determine what sort of database it'll be working with.  Or you can give it a default (say, SQLite) database for the development environment and let it use Postgres or whatever in staging and production, with a line in your configuration file like this:

{% highlight python %}
    SQLALCHEMY_DATABASE_URI = os.environ.get('DATABASE_URL') or \
        'sqlite:///' + os.path.join(basedir, 'data-test.sqlite')
{% endhighlight %}


I like this feature a lot, but in order to take advantage of it, obviously I would have to limit myself to features that are common to all the databases---and forego using Postgres's unique features like "recursive" queries.

### Second clue: 'flask-migrate' wows me with version control

Database version control is a tricky problem (worth a blog post of its own) because the database can't be torn down and re-created every time its structure needs to change.  Because the data must persist, [database refactoring](http://amzn.to/1HG86S2) is something like re-designing a ship while it's out to sea.  What you'd probably do is create an initial script to create the first version of the database, then for each change create a new script to update the schema, version control these scripts, and remember to run the scripts in order upon deploying a new instance of the application.  If you want the ability to go backwards, to regress to earlier versions, it's even more complicated because you'd need "downgrade" scripts as well.

Automating this would be amazing but seems quite difficult.  However, in Grinberg's tutorial, he demonstrates a utility called [Alembic](https://bitbucket.org/zzzeek/alembic) with SQLAlchemy that does exactly that---automatically generate "upgrade" and "downgrade" scripts as the all-Python description of his data model changes.  It also features a command-line "upgrade" that seems to know where you are in the sequence of scripts (so it doesn't re-run old ones) and automatically applies the upgrade(s) necessary to bring your database up to the current version of the code.  

In a few lines of code, the database version control problem is solved!  Also, because it is Python and not SQL, the upgrade and downgrade scripts can be used for any type of database.  This could be a great convenience if you are developing initially for an open-source software stack and eventually want to port your application to, oh, Oracle or I don't know what.

There are some caveats, though.  First, it's easy to mess it up.  I found that I occasionally used the `drop_all()` and `create_all()` commands while developing the application and testing, and these caused some quirks.  Alembic wouldn't create an upgrade script if by running `create_all()` I had already created the new tables.  (It would say to itself, "the database matches the code, therefore nothing needs to be updated".)  Also, I understand there are some kinds of schema changes it can't detect, and moreover, I still see that I would have to forego using any of PostgreSQL's unique features to take advantage of this database-agnostic tool.

Which makes me wonder---are there any alternatives that are specific to Postgres, and can give me the benefit of "migrations" without the trade off?

### Third clue (and first that really pertains to database abstraction): the "backref" in 1:M relationships

I found a podcast in which Mike Bayer, creator of SQLAlchemy, was [interviewed in April 2015](http://talkpython.fm/episodes/show/5/sqlalchemy-and-data-access-in-python) (cf. around 44:00).  He points to one very useful mapping that relates to one-to-many relationships between tables.  A 1:M relationship in the database is really only seen from the M:1 side: it's the "child" entity that holds a foreign key reference to the "parent".  However, in object-oriented programming, we typically want to reference the child objects *from* the parent object.  We want to see an author's blog posts just as often as we want to see a blog post's author.

By representing the Author and the Post "models" as Python classes, a  `.posts` attribute can be added to the Author class which looks like a database field but actually returns the results of a separate query.  What's more, SQLAlchemy allows us to do this without writing a lot of extra code.  We simply assign the "backref" argument in the method that creates a foreign key relationship.  When I saw this, I thought it was pretty slick.

### More clues, and the discovery of SQLAlchemy "Core"

A number of other convenient little functions and methods worked their way into the application that show the benefit of having an object "wrapper" around a database entity.  Methods like `get_or_404()` simplify error handling when a user requests a non-existent resource.  Custom "set" methods like `change_email()` can be created to perform validation tasks like sending a confirmation e-mail.  Add-ons like [Flask-Login](https://github.com/maxcountryman/flask-login/) expect certain methods like `is_authenticated()` and `get_id()` and make it easy to implement user accounts.

By the time I finished the tutorial, I saw that creating object models for my entities could be a very powerful tool for organizing code that operates on the data.  The User entity in my tutorial app, for example, has numerous methods for everything from checking permissions to sending account confirmation e-mails and outputting data about the user as JSON.  Putting all of that in a "models.py" package makes my controllers much cleaner.

However, as I researched SQLAlchemy more, I discovered that there are in fact two major parts of it: the ORM, and the underlying layer called SQLAlchemy Core.  I could take advantage of SQLAlchemy core, which does not automatically map objects to tables, or auto-generate SQL, without having to use the ORM.  Furthermore I could simply use psycopg2, the Python-Postgres connector, and not use any part of SQLAlchemy at all with this concept of an object to wrap my database operations around a particular entity.  So the question has changed:

## Given that I'm going to embrace "Model", do I implement it as ORM, use an engine like SQLAlchemy Core, or write my own?

Here's what the ORM layer seems to give me: it allows me to define a class with a few attributes that SQLAlchemy interprets as columns, and which Alembic will allow me to automatically create as database tables.  The first few lines of my Post model, for example, show the definition of a table with a foreign key to the Users table, a "backref" to the Comments table, and some other columns:

{% highlight python %}
    class Post(db.Model):
        __tablename__ = "posts"
        id = db.Column(db.Integer, primary_key=True)
        body = db.Column(db.Text)
        timestamp = db.Column(db.DateTime, index=True, \
         default=datetime.utcnow)
        author_id = db.Column(db.Integer, db.ForeignKey("users.id"))
        body_html = db.Column(db.Text)
        # comments
        comments = db.relationship("Comment", backref="post",\
         lazy="dynamic")
{% endhighlight %}

One of the nice things that I can do with this object is make a change to a row in the database by simply assigning a new value to an attribute of a Post object.  That reduces the amount of SQL I would have to write for "get" and "set" methods.  However, it takes away my control of data definition (DDL) and, presumably, prevents me from using a view in place of a table.  It would be somewhat tedious but not extremely so, to write my own getters and setters in SQL.  Also, character for character, the ORM approach really doesn't seem to make the code any shorter.  Consider [this model-definition code](https://github.com/joeclark-phd/flaskula/blob/master/app/models.py) from my tutorial app.

### The SQLAlchemy core as an option

SQLAlchemy Core seems to provide this option.  I could write my own plain-text queries (if I didn't care about cross-database compatibility) *or* use the core's SQL-like query syntax to maintain database-agnosticism.  This could be a very attractive middle ground.  SQLAlchemy Core still does things like simplifying database connection and managing a connection pool when multiple concurrent users are online.  I believe that what I'd lose with this approach is the slick version-control functionality of Alembic.  [**Update: No, I wouldn't.  [Mike Bayer informs me via Twitter](https://twitter.com/zzzeek/status/668149034431987712) that Alembic works on the core.**]

Compared to "rolling my own", this seems like the smart way to go.  Not only does it automate things like opening and closing of connections, it also allows me to take advantage of a mix of higher-level and lower-level abstractions as I need them.  Since it also allows me to bypass abstraction entirely and go directly to raw SQL when I need to, I can't see any advantage of writing my own database layer with plain psycopg2.

## Three big questions

In sum, I have seen the benefit of creating "model" objects to hold a variety of methods and helper-functions that correspond to database entities, and I have seen some benefits of the ORM: notably, that by giving up the freedom to use database-specific features, you can get automated database version control.  But what should I use in my own app development?  I've boiled it down to three questions for myself:

1. Do I want to use Postgres-only features?
  - And am I willing to trade database agnosticism and version control for them?
2. Do I want to put logic into the database (views, triggers, stored procedures)?
3. Can I find substitutes for the features I'd give up by using Core, such as automated version control?

On balance, it seems like if I'm developing a simple CRUD application with no complex relationships between entities, and I want to get it up and running quickly with version control and continuous integration, ORM does make sense.  I love database optimization, but it's a craft that takes time, and may not be necessary for every project.  For more complicated systems, I find that I want to put logic into the database such as views and triggers.  The reasoning for that is to prevent code duplication and inconsistency with other, future applications that access the same data.  A trigger, for example, can be thought of as a special type of DDL.

However, in the newer world of [API-first development]({% post_url 2015-09-18-5-reasons-to-try-api-first-development %}), it may make more sense to have my web app itself expose an API which future applications would build on.  If that's the case, then the models I implement in Python would define in one place the logic that all database users are going to need.  This is a weak argument, perhaps.

In my next project, I think, I'll use SQLAlchemy Core without the highest-level abstractions of the ORM, in order to better gauge the difference.  I'm curious to see exactly how tedious it gets writing pure SQL; frankly, I don't think it could be worse that writing the Python equivalent.  However, I can say that as a result of this experience I do appreciate the role of Models and of ORM, even though I'm not personally ready to give up the unique features of my favorite relational database. 
Welcome to Flask-WhooshAlchemy!
===============================

[![image](https://circleci.com/gh/dhamaniasad/Flask-WhooshAlchemy/tree/master.svg?style=shield)](https://circleci.com/gh/dhamaniasad/Flask-WhooshAlchemy/tree/master) [![image](https://img.shields.io/pypi/v/Flask-WhooshAlchemy-Redux.svg)](https://pypi.python.org/pypi/Flask-WhooshAlchemy-Redux)

Flask-WhooshAlchemy is a Flask extension that integrates the text-search functionality of [Whoosh](https://bitbucket.org/mchaput/whoosh/wiki/Home) with the ORM of [SQLAlchemy](http://www.sqlalchemy.org/) for use in [Flask](http://flask.pocoo.org/) applications.

Source code and issue tracking at [GitHub](https://github.com/dhamaniasad/Flask-WhooshAlchemy).

View the official docs [here](https://Flask-WhooshAlchemy.readthedocs.org/).

Install
-------

    pip install flask_whooshalchemy_redux

Or:

    git clone https://github.com/dhamaniasad/Flask-WhooshAlchemy.git

Quickstart
----------

Let's set up the environment and create our model:

```python
import flask.ext.whooshalchemy as whooshalchemy

# set the location for the whoosh index
app.config['WHOOSH_BASE'] = 'path/to/whoosh/base'


class BlogPost(db.Model):
  __tablename__ = 'blogpost'
  __searchable__ = ['title', 'content']  # these fields will be indexed by whoosh
  __analyzer__ = SimpleAnalyzer()        # configure analyzer; defaults to
                                         # StemmingAnalyzer if not specified

  id = app.db.Column(app.db.Integer, primary_key=True)
  title = app.db.Column(app.db.Unicode)  # Indexed fields are either String,
  content = app.db.Column(app.db.Text)   # Unicode, or Text
  created = db.Column(db.DateTime, default=datetime.datetime.utcnow)

whooshalchemy.whoosh_index(app, BlogPost)
```

Only two steps to get started:

1)  Set the `WHOOSH_BASE` to the path for the whoosh index. If not set, it will default to a directory called 'whoosh\_index' in the directory from which the application is run.
2)  Add a `__searchable__` field to the model which specifies the fields (as `str` s) to be indexed .

Let's create a post:

```python
db.session.add(
    BlogPost(title='My cool title', content='This is the first post.')
); db.session.commit()
```

After the session is committed, our new `BlogPost` is indexed. Similarly, if the post is deleted, it will be removed from the Whoosh index.

Text Searching
--------------

To execute a simple search:

```python
results = BlogPost.query.whoosh_search('cool')
```

This will return all `BlogPost` instances in which at least one indexed
field (i.e., 'title' or 'content') is a text match to the query. Results
are ranked according to their relevance score, with the best match
appearing first when iterating. The result of this call is a (subclass
of) sqlalchemy.orm.query.Query object, so you can chain other SQL
operations. For example:

```python
two_days_ago = datetime.date.today() - datetime.timedelta(2)
recent_matches = BlogPost.query.whoosh_search('first').filter(
    BlogPost.created >= two_days_ago)
```

Or, in alternative (likely slower) order:

```python
recent_matches = BlogPost.query.filter(
    BlogPost.created >= two_days_ago).whoosh_search('first')
```

We can limit results:

```python
# get 2 best results:
results = BlogPost.query.whoosh_search('cool', limit=2)
```

By default, the search is executed on all of the indexed fields as an OR
conjunction. For example, if a model has 'title' and 'content' indicated
as `__searchable__`, a query will be checked against both fields,
returning any instance whose title or content are a content match for
the query. To specify particular fields to be checked, populate the
`fields` parameter with the desired fields:

```python
results = BlogPost.query.whoosh_search('cool', fields=('title',))
```

By default, results will only be returned if they contain all of the
query terms (AND). To switch to an OR grouping, set the `or_` parameter
to `True`:

```python
results = BlogPost.query.whoosh_search('cool', or_=True)
```

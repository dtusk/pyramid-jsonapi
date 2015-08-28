# pyramid-jsonapi

Utilities for creating JSON-API apis from a database using the sqlAlchemy ORM and pyramid framework.

# Installation

Pretty basic right now: copy the directory jsonapi/ into your PYTHONPATH or into your project.

# Quick preview

If you are happy with the defaults, you can get away with the following additions to the standard pyramid alchemy scaffold's top level `__init__.py`:

```python
import jsonapi
# Or 'from . import jsonapi' if you copied jsonapi directly into your project.

from . import models # Your models module.

# In the main function:
  renderer = jsonapi.JSONAPIFromSqlAlchemyRenderer()
  # JSONAPIFromSqlAlchemyRenderer inherits pyramid.renderers.JSON
  # so you can add JSON adapters in the same way:
  # renderer.add_adapter(datetime.date, datetime_adapter)
  config.add_renderer('jsonapi', renderer)
  jsonapi.create_jsonapi_using_magic_and_pixie_dust(models)
  # Make sure we scan the *jsonapi* package.
  config.scan(package=jsonapi)
```

You should now have a working JSON-API.

Start the server:

```bash
$ pserv your_project/development.ini
```

Using the rather lovely [httpie](https://github.com/jkbrzt/httpie) to test:

```bash
$ http http://localhost:6543/people
```
```
HTTP/1.1 200 OK
Content-Length: 1387
Content-Type: application/vnd.api+json; charset=UTF-8
Date: Fri, 28 Aug 2015 20:22:46 GMT
Server: waitress

{
  "data": [
  {
    "attributes": {
      "name": "alice"
    },
    "id": "2",
    "links": {
      "self": "http://localhost:6543/people/2"
    },
    "relationships": {
    ...
  }
  ...
  ]
}
```

See `test_project/test_project/__init__.py` for a fully working `__init__` file.

You don't need a views.py unless you have some other routes and views.

Yes, there really is a method called "create_api_using_magic_and_pixie_dust". No, you don't *have* to call it that. If you are feeling more sensible you can use the synonym `create_jsonapi()`.

# Usage

## Auto-creating a JSON-API

More or less the same as the quick preview above. Spelled out in a bit more detail:

1. Create Some Models in the usual style:

    ```python

    class Person(Base):
      __tablename__ = 'people'
      id = Column(BigInteger, primary_key=True, autoincrement=True)
      name = Column(Text)
      blogs = relationship('Blog', backref='owner')
      posts = relationship('Post', backref='author')
    ```
  * Some options can be set which will be used by the default query functions
    and renderer. They should be set in a dictionary stored in the `__jsonapi__`
    attribute of a model class:

    ```python
    Person.__jsonapi__ = {
      'options': {
        'default_limit': 2
      }
    }
    ```

1. Create the API end points from the model:

  ```python
  jsonapi.create_jsonapi_using_magic_and_pixie_dust(models)
  ```

1. Use the jsonapi renderer for renderer type 'jsonapi':

  ```python
  renderer = jsonapi.JSONAPIFromSqlAlchemyRenderer()
  config.add_renderer('jsonapi', renderer)
  ```

1. Use pyramid/venusian to scan the jsonapi package:

  ```python
  config.scan(package=jsonapi)
  ```

### Auto-create Assumptions

1. Your model classes all inherit from a base class returned by sqlalchemy's `declarative-base()`.
1. Each model has a primary_key called 'id'.
1. You are happy to give your collection end-points the same name as the corresponding database table.
1. You are happy to expose every model class defined in `models` as a resource collection.
1. You have defined any relationships to exposed via the API using `sqlalchemy.orm.relationship()` (or `backref()`).
1. You are happy to expose any so defined relationship via a relationship URL.

### Customising the Generated API

There are two optional parameters to `create_api_using_magic_and_pixie_dust()`:
`links_callback` and `meta_callback`. Both of these can be supplied with callback functions. They are the same ones described in [create_resource](#create_resource).
## Creating Individual JSON-API Resources from Models

### <a name="create_resource"></a>Resource Creation On-the-fly: `create_resource()`

#### `create_resource()` Parameters

* `links_callback` and `meta_callback`:

  These are functions which should return dictionaries.

  In the case of links, the returned links dictionary will be merged (via
  `dict.update()`) with the default one generated within the jsonapi renderer
  (which will certainly contain `self`, possibly some pagination links).

  There is no default meta dictionary created within the renderer, though there
  is a default implementation of `meta_callback`: `jsonapi.std_meta()`, which
  adds some information to meta. If you pass your own `meta_callback` and you'd
  like to retain the values provided by `std_meta()` you will need to call
  `jsonapi.std_meta()` yourself and `update()` your own dictionary with the
  results.


### Decorating Class Definitions

Modifying the tree
==================

Beautiful Soup's main strength is in searching the parse tree, but you
can also modify the tree and write your changes as a new HTML or XML
document.

Changing tag names and attributes
---------------------------------

I covered this earlier, in :ref:`kind_of_obj attributes`, but it bears repeating. You
can rename a tag, change the values of its attributes, add new
attributes, and delete attributes::

 soup = BeautifulSoup('<b class="boldest">Extremely bold</b>')
 tag = soup.b

 tag.name = "blockquote"
 tag['class'] = 'verybold'
 tag['id'] = 1
 tag
 # <blockquote class="verybold" id="1">Extremely bold</blockquote>

 del tag['class']
 del tag['id']
 tag
 # <blockquote>Extremely bold</blockquote>


Modifying ``.string``
---------------------

If you set a tag's ``.string`` attribute, the tag's contents are
replaced with the string you give::

  markup = '<a href="http://example.com/">I linked to <i>example.com</i></a>'
  soup = BeautifulSoup(markup)

  tag = soup.a
  tag.string = "New link text."
  tag
  # <a href="http://example.com/">New link text.</a>

Be careful: if the tag contained other tags, they and all their
contents will be destroyed.

``append()``
------------

You can add to a tag's contents with ``Tag.append()``. It works just
like calling ``.append()`` on a Python list::

   soup = BeautifulSoup("<a>Foo</a>")
   soup.a.append("Bar")

   soup
   # <html><head></head><body><a>FooBar</a></body></html>
   soup.a.contents
   # [u'Foo', u'Bar']

``BeautifulSoup.new_string()`` and ``.new_tag()``
-------------------------------------------------

If you need to add a string to a document, no problem--you can pass a
Python string in to ``append()``, or you can call the factory method
``BeautifulSoup.new_string()``::

   soup = BeautifulSoup("<b></b>")
   tag = soup.b
   tag.append("Hello")
   new_string = soup.new_string(" there")
   tag.append(new_string)
   tag
   # <b>Hello there.</b>
   tag.contents
   # [u'Hello', u' there']

If you want to create a comment or some other subclass of
``NavigableString``, pass that class as the second argument to
``new_string()``::

   from bs4 import Comment
   new_comment = soup.new_string("Nice to see you.", Comment)
   tag.append(new_comment)
   tag
   # <b>Hello there<!--Nice to see you.--></b>
   tag.contents
   # [u'Hello', u' there', u'Nice to see you.']

(This is a new feature in Beautiful Soup 4.2.1.)

What if you need to create a whole new tag?  The best solution is to
call the factory method ``BeautifulSoup.new_tag()``::

   soup = BeautifulSoup("<b></b>")
   original_tag = soup.b

   new_tag = soup.new_tag("a", href="http://www.example.com")
   original_tag.append(new_tag)
   original_tag
   # <b><a href="http://www.example.com"></a></b>

   new_tag.string = "Link text."
   original_tag
   # <b><a href="http://www.example.com">Link text.</a></b>

Only the first argument, the tag name, is required.

``insert()``
------------

``Tag.insert()`` is just like ``Tag.append()``, except the new element
doesn't necessarily go at the end of its parent's
``.contents``. It'll be inserted at whatever numeric position you
say. It works just like ``.insert()`` on a Python list::

  markup = '<a href="http://example.com/">I linked to <i>example.com</i></a>'
  soup = BeautifulSoup(markup)
  tag = soup.a

  tag.insert(1, "but did not endorse ")
  tag
  # <a href="http://example.com/">I linked to but did not endorse <i>example.com</i></a>
  tag.contents
  # [u'I linked to ', u'but did not endorse', <i>example.com</i>]

``insert_before()`` and ``insert_after()``
------------------------------------------

The ``insert_before()`` method inserts a tag or string immediately
before something else in the parse tree::

   soup = BeautifulSoup("<b>stop</b>")
   tag = soup.new_tag("i")
   tag.string = "Don't"
   soup.b.string.insert_before(tag)
   soup.b
   # <b><i>Don't</i>stop</b>

The ``insert_after()`` method moves a tag or string so that it
immediately follows something else in the parse tree::

   soup.b.i.insert_after(soup.new_string(" ever "))
   soup.b
   # <b><i>Don't</i> ever stop</b>
   soup.b.contents
   # [<i>Don't</i>, u' ever ', u'stop']

``clear()``
-----------

``Tag.clear()`` removes the contents of a tag::

  markup = '<a href="http://example.com/">I linked to <i>example.com</i></a>'
  soup = BeautifulSoup(markup)
  tag = soup.a

  tag.clear()
  tag
  # <a href="http://example.com/"></a>

``extract()``
-------------

``PageElement.extract()`` removes a tag or string from the tree. It
returns the tag or string that was extracted::

  markup = '<a href="http://example.com/">I linked to <i>example.com</i></a>'
  soup = BeautifulSoup(markup)
  a_tag = soup.a

  i_tag = soup.i.extract()

  a_tag
  # <a href="http://example.com/">I linked to</a>

  i_tag
  # <i>example.com</i>

  print(i_tag.parent)
  None

At this point you effectively have two parse trees: one rooted at the
``BeautifulSoup`` object you used to parse the document, and one rooted
at the tag that was extracted. You can go on to call ``extract`` on
a child of the element you extracted::

  my_string = i_tag.string.extract()
  my_string
  # u'example.com'

  print(my_string.parent)
  # None
  i_tag
  # <i></i>


``decompose()``
---------------

``Tag.decompose()`` removes a tag from the tree, then `completely
destroys it and its contents`::

  markup = '<a href="http://example.com/">I linked to <i>example.com</i></a>'
  soup = BeautifulSoup(markup)
  a_tag = soup.a

  soup.i.decompose()

  a_tag
  # <a href="http://example.com/">I linked to</a>


.. _replace_with:

``replace_with()``
------------------

``PageElement.replace_with()`` removes a tag or string from the tree,
and replaces it with the tag or string of your choice::

  markup = '<a href="http://example.com/">I linked to <i>example.com</i></a>'
  soup = BeautifulSoup(markup)
  a_tag = soup.a

  new_tag = soup.new_tag("b")
  new_tag.string = "example.net"
  a_tag.i.replace_with(new_tag)

  a_tag
  # <a href="http://example.com/">I linked to <b>example.net</b></a>

``replace_with()`` returns the tag or string that was replaced, so
that you can examine it or add it back to another part of the tree.

``wrap()``
----------

``PageElement.wrap()`` wraps an element in the tag you specify. It
returns the new wrapper::

 soup = BeautifulSoup("<p>I wish I was bold.</p>")
 soup.p.string.wrap(soup.new_tag("b"))
 # <b>I wish I was bold.</b>

 soup.p.wrap(soup.new_tag("div")
 # <div><p><b>I wish I was bold.</b></p></div>

This method is new in Beautiful Soup 4.0.5.

``unwrap()``
---------------------------

``Tag.unwrap()`` is the opposite of ``wrap()``. It replaces a tag with
whatever's inside that tag. It's good for stripping out markup::

  markup = '<a href="http://example.com/">I linked to <i>example.com</i></a>'
  soup = BeautifulSoup(markup)
  a_tag = soup.a

  a_tag.i.unwrap()
  a_tag
  # <a href="http://example.com/">I linked to example.com</a>

Like ``replace_with()``, ``unwrap()`` returns the tag
that was replaced.
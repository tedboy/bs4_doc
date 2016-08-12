.. _searching_the_tree:

Searching the tree
==================

Beautiful Soup defines a lot of methods for searching the parse tree,
but they're all very similar. I'm going to spend a lot of time explaining
the two most popular methods: ``find()`` and ``find_all()``. The other
methods take almost exactly the same arguments, so I'll just cover
them briefly.

Once again, I'll be using the "three sisters" document as an example::

 html_doc = """
 <html><head><title>The Dormouse's story</title></head>

 <p class="title"><b>The Dormouse's story</b></p>

 <p class="story">Once upon a time there were three little sisters; and their names were
 <a href="http://example.com/elsie" class="sister" id="link1">Elsie</a>,
 <a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
 <a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
 and they lived at the bottom of a well.</p>

 <p class="story">...</p>
 """

 from bs4 import BeautifulSoup
 soup = BeautifulSoup(html_doc)

By passing in a filter to an argument like ``find_all()``, you can
zoom in on the parts of the document you're interested in.

Kinds of filters
----------------

Before talking in detail about ``find_all()`` and similar methods, I
want to show examples of different filters you can pass into these
methods. These filters show up again and again, throughout the
search API. You can use them to filter based on a tag's name,
on its attributes, on the text of a string, or on some combination of
these.

.. _a string:

A string
^^^^^^^^

The simplest filter is a string. Pass a string to a search method and
Beautiful Soup will perform a match against that exact string. This
code finds all the <b> tags in the document::

 soup.find_all('b')
 # [<b>The Dormouse's story</b>]

If you pass in a byte string, Beautiful Soup will assume the string is
encoded as UTF-8. You can avoid this by passing in a Unicode string instead.

.. _a regular expression:

A regular expression
^^^^^^^^^^^^^^^^^^^^

If you pass in a regular expression object, Beautiful Soup will filter
against that regular expression using its ``match()`` method. This code
finds all the tags whose names start with the letter "b"; in this
case, the <body> tag and the <b> tag::

 import re
 for tag in soup.find_all(re.compile("^b")):
     print(tag.name)
 # body
 # b

This code finds all the tags whose names contain the letter 't'::

 for tag in soup.find_all(re.compile("t")):
     print(tag.name)
 # html
 # title

.. _a list:

A list
^^^^^^

If you pass in a list, Beautiful Soup will allow a string match
against `any` item in that list. This code finds all the <a> tags
`and` all the <b> tags::

 soup.find_all(["a", "b"])
 # [<b>The Dormouse's story</b>,
 #  <a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
 #  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
 #  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

.. _the value True:

``True``
^^^^^^^^

The value ``True`` matches everything it can. This code finds `all`
the tags in the document, but none of the text strings::

 for tag in soup.find_all(True):
     print(tag.name)
 # html
 # head
 # title
 # body
 # p
 # b
 # p
 # a
 # a
 # a
 # p

.. a function:

A function
^^^^^^^^^^

If none of the other matches work for you, define a function that
takes an element as its only argument. The function should return
``True`` if the argument matches, and ``False`` otherwise.

Here's a function that returns ``True`` if a tag defines the "class"
attribute but doesn't define the "id" attribute::

 def has_class_but_no_id(tag):
     return tag.has_attr('class') and not tag.has_attr('id')

Pass this function into ``find_all()`` and you'll pick up all the <p>
tags::

 soup.find_all(has_class_but_no_id)
 # [<p class="title"><b>The Dormouse's story</b></p>,
 #  <p class="story">Once upon a time there were...</p>,
 #  <p class="story">...</p>]

This function only picks up the <p> tags. It doesn't pick up the <a>
tags, because those tags define both "class" and "id". It doesn't pick
up tags like <html> and <title>, because those tags don't define
"class".

Here's a function that returns ``True`` if a tag is surrounded by
string objects::

 from bs4 import NavigableString
 def surrounded_by_strings(tag):
     return (isinstance(tag.next_element, NavigableString)
             and isinstance(tag.previous_element, NavigableString))

 for tag in soup.find_all(surrounded_by_strings):
     print tag.name
 # p
 # a
 # a
 # a
 # p

Now we're ready to look at the search methods in detail.

``find_all()``
--------------

Signature: find_all(:ref:`name <name>`, :ref:`attrs <attrs>`, :ref:`recursive
<recursive>`, :ref:`text <text>`, :ref:`limit <limit>`, :ref:`**kwargs <kwargs>`)

The ``find_all()`` method looks through a tag's descendants and
retrieves `all` descendants that match your filters. I gave several
examples in `Kinds of filters`_, but here are a few more::

 soup.find_all("title")
 # [<title>The Dormouse's story</title>]

 soup.find_all("p", "title")
 # [<p class="title"><b>The Dormouse's story</b></p>]

 soup.find_all("a")
 # [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
 #  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
 #  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

 soup.find_all(id="link2")
 # [<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>]

 import re
 soup.find(text=re.compile("sisters"))
 # u'Once upon a time there were three little sisters; and their names were\n'

Some of these should look familiar, but others are new. What does it
mean to pass in a value for ``text``, or ``id``? Why does
``find_all("p", "title")`` find a <p> tag with the CSS class "title"?
Let's look at the arguments to ``find_all()``.

.. _name:

The ``name`` argument
^^^^^^^^^^^^^^^^^^^^^

Pass in a value for ``name`` and you'll tell Beautiful Soup to only
consider tags with certain names. Text strings will be ignored, as
will tags whose names that don't match.

This is the simplest usage::

 soup.find_all("title")
 # [<title>The Dormouse's story</title>]

Recall from `Kinds of filters`_ that the value to ``name`` can be `a
string`_, `a regular expression`_, `a list`_, `a function`_, or `the value
True`_.

.. _kwargs:

The keyword arguments
^^^^^^^^^^^^^^^^^^^^^

Any argument that's not recognized will be turned into a filter on one
of a tag's attributes. If you pass in a value for an argument called ``id``,
Beautiful Soup will filter against each tag's 'id' attribute::

 soup.find_all(id='link2')
 # [<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>]

If you pass in a value for ``href``, Beautiful Soup will filter
against each tag's 'href' attribute::

 soup.find_all(href=re.compile("elsie"))
 # [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>]

You can filter an attribute based on `a string`_, `a regular
expression`_, `a list`_, `a function`_, or `the value True`_.

This code finds all tags that have an ``id`` attribute, regardless of
what the value is::

 soup.find_all(id=True)
 # [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
 #  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
 #  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

You can filter multiple attributes at once by passing in more than one
keyword argument::

 soup.find_all(href=re.compile("elsie"), id='link1')
 # [<a class="sister" href="http://example.com/elsie" id="link1">three</a>]

Some attributes, like the data-* attributes in HTML 5, have names that
can't be used as the names of keyword arguments::

 data_soup = BeautifulSoup('<div data-foo="value">foo!</div>')
 data_soup.find_all(data-foo="value")
 # SyntaxError: keyword can't be an expression

You can use these attributes in searches by putting them into a
dictionary and passing the dictionary into ``find_all()`` as the
``attrs`` argument::

 data_soup.find_all(attrs={"data-foo": "value"})
 # [<div data-foo="value">foo!</div>]

.. _attrs:

Searching by CSS class
^^^^^^^^^^^^^^^^^^^^^^

It's very useful to search for a tag that has a certain CSS class, but
the name of the CSS attribute, "class", is a reserved word in
Python. Using ``class`` as a keyword argument will give you a syntax
error. As of Beautiful Soup 4.1.2, you can search by CSS class using
the keyword argument ``class_``::

 soup.find_all("a", class_="sister")
 # [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
 #  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
 #  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

As with any keyword argument, you can pass ``class_`` a string, a regular
expression, a function, or ``True``::

 soup.find_all(class_=re.compile("itl"))
 # [<p class="title"><b>The Dormouse's story</b></p>]

 def has_six_characters(css_class):
     return css_class is not None and len(css_class) == 6

 soup.find_all(class_=has_six_characters)
 # [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
 #  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
 #  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

:ref:`Remember <multivalue>` that a single tag can have multiple
values for its "class" attribute. When you search for a tag that
matches a certain CSS class, you're matching against `any` of its CSS
classes::

 css_soup = BeautifulSoup('<p class="body strikeout"></p>')
 css_soup.find_all("p", class_="strikeout")
 # [<p class="body strikeout"></p>]

 css_soup.find_all("p", class_="body")
 # [<p class="body strikeout"></p>]

You can also search for the exact string value of the ``class`` attribute::

 css_soup.find_all("p", class_="body strikeout")
 # [<p class="body strikeout"></p>]

But searching for variants of the string value won't work::

 css_soup.find_all("p", class_="strikeout body")
 # []

If you want to search for tags that match two or more CSS classes, you
should use a CSS selector::

 css_soup.select("p.strikeout.body")
 # [<p class="body strikeout"></p>]

In older versions of Beautiful Soup, which don't have the ``class_``
shortcut, you can use the ``attrs`` trick mentioned above. Create a
dictionary whose value for "class" is the string (or regular
expression, or whatever) you want to search for::

 soup.find_all("a", attrs={"class": "sister"})
 # [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
 #  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
 #  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

.. _text:

The ``text`` argument
^^^^^^^^^^^^^^^^^^^^^

With ``text`` you can search for strings instead of tags. As with
``name`` and the keyword arguments, you can pass in `a string`_, `a
regular expression`_, `a list`_, `a function`_, or `the value True`_.
Here are some examples::

 soup.find_all(text="Elsie")
 # [u'Elsie']

 soup.find_all(text=["Tillie", "Elsie", "Lacie"])
 # [u'Elsie', u'Lacie', u'Tillie']

 soup.find_all(text=re.compile("Dormouse"))
 [u"The Dormouse's story", u"The Dormouse's story"]

 def is_the_only_string_within_a_tag(s):
     """Return True if this string is the only child of its parent tag."""
     return (s == s.parent.string)

 soup.find_all(text=is_the_only_string_within_a_tag)
 # [u"The Dormouse's story", u"The Dormouse's story", u'Elsie', u'Lacie', u'Tillie', u'...']

Although ``text`` is for finding strings, you can combine it with
arguments that find tags: Beautiful Soup will find all tags whose
``.string`` matches your value for ``text``. This code finds the <a>
tags whose ``.string`` is "Elsie"::

 soup.find_all("a", text="Elsie")
 # [<a href="http://example.com/elsie" class="sister" id="link1">Elsie</a>]

.. _limit:

The ``limit`` argument
^^^^^^^^^^^^^^^^^^^^^^

``find_all()`` returns all the tags and strings that match your
filters. This can take a while if the document is large. If you don't
need `all` the results, you can pass in a number for ``limit``. This
works just like the LIMIT keyword in SQL. It tells Beautiful Soup to
stop gathering results after it's found a certain number.

There are three links in the "three sisters" document, but this code
only finds the first two::

 soup.find_all("a", limit=2)
 # [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
 #  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>]

.. _recursive:

The ``recursive`` argument
^^^^^^^^^^^^^^^^^^^^^^^^^^

If you call ``mytag.find_all()``, Beautiful Soup will examine all the
descendants of ``mytag``: its children, its children's children, and
so on. If you only want Beautiful Soup to consider direct children,
you can pass in ``recursive=False``. See the difference here::

 soup.html.find_all("title")
 # [<title>The Dormouse's story</title>]

 soup.html.find_all("title", recursive=False)
 # []

Here's that part of the document::

 <html>
  <head>
   <title>
    The Dormouse's story
   </title>
  </head>
 ...

The <title> tag is beneath the <html> tag, but it's not `directly`
beneath the <html> tag: the <head> tag is in the way. Beautiful Soup
finds the <title> tag when it's allowed to look at all descendants of
the <html> tag, but when ``recursive=False`` restricts it to the
<html> tag's immediate children, it finds nothing.

Beautiful Soup offers a lot of tree-searching methods (covered below),
and they mostly take the same arguments as ``find_all()``: ``name``,
``attrs``, ``text``, ``limit``, and the keyword arguments. But the
``recursive`` argument is different: ``find_all()`` and ``find()`` are
the only methods that support it. Passing ``recursive=False`` into a
method like ``find_parents()`` wouldn't be very useful.

Calling a tag is like calling ``find_all()``
--------------------------------------------

Because ``find_all()`` is the most popular method in the Beautiful
Soup search API, you can use a shortcut for it. If you treat the
``BeautifulSoup`` object or a ``Tag`` object as though it were a
function, then it's the same as calling ``find_all()`` on that
object. These two lines of code are equivalent::

 soup.find_all("a")
 soup("a")

These two lines are also equivalent::

 soup.title.find_all(text=True)
 soup.title(text=True)

``find()``
----------

Signature: find(:ref:`name <name>`, :ref:`attrs <attrs>`, :ref:`recursive
<recursive>`, :ref:`text <text>`, :ref:`**kwargs <kwargs>`)

The ``find_all()`` method scans the entire document looking for
results, but sometimes you only want to find one result. If you know a
document only has one <body> tag, it's a waste of time to scan the
entire document looking for more. Rather than passing in ``limit=1``
every time you call ``find_all``, you can use the ``find()``
method. These two lines of code are `nearly` equivalent::

 soup.find_all('title', limit=1)
 # [<title>The Dormouse's story</title>]

 soup.find('title')
 # <title>The Dormouse's story</title>

The only difference is that ``find_all()`` returns a list containing
the single result, and ``find()`` just returns the result.

If ``find_all()`` can't find anything, it returns an empty list. If
``find()`` can't find anything, it returns ``None``::

 print(soup.find("nosuchtag"))
 # None

Remember the ``soup.head.title`` trick from :ref:`navtree using tag names`? That trick works by repeatedly calling ``find()``::

 soup.head.title
 # <title>The Dormouse's story</title>

 soup.find("head").find("title")
 # <title>The Dormouse's story</title>

``find_parents()`` and ``find_parent()``
----------------------------------------

Signature: find_parents(:ref:`name <name>`, :ref:`attrs <attrs>`, :ref:`text <text>`, :ref:`limit <limit>`, :ref:`**kwargs <kwargs>`)

Signature: find_parent(:ref:`name <name>`, :ref:`attrs <attrs>`, :ref:`text <text>`, :ref:`**kwargs <kwargs>`)

I spent a lot of time above covering ``find_all()`` and
``find()``. The Beautiful Soup API defines ten other methods for
searching the tree, but don't be afraid. Five of these methods are
basically the same as ``find_all()``, and the other five are basically
the same as ``find()``. The only differences are in what parts of the
tree they search.

First let's consider ``find_parents()`` and
``find_parent()``. Remember that ``find_all()`` and ``find()`` work
their way down the tree, looking at tag's descendants. These methods
do the opposite: they work their way `up` the tree, looking at a tag's
(or a string's) parents. Let's try them out, starting from a string
buried deep in the "three daughters" document::

  a_string = soup.find(text="Lacie")
  a_string
  # u'Lacie'

  a_string.find_parents("a")
  # [<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>]

  a_string.find_parent("p")
  # <p class="story">Once upon a time there were three little sisters; and their names were
  #  <a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
  #  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a> and
  #  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>;
  #  and they lived at the bottom of a well.</p>

  a_string.find_parents("p", class="title")
  # []

One of the three <a> tags is the direct parent of the string in
question, so our search finds it. One of the three <p> tags is an
indirect parent of the string, and our search finds that as
well. There's a <p> tag with the CSS class "title" `somewhere` in the
document, but it's not one of this string's parents, so we can't find
it with ``find_parents()``.

You may have made the connection between ``find_parent()`` and
``find_parents()``, and the :ref:`.parent` and :ref:`.parents` attributes
mentioned earlier. The connection is very strong. These search methods
actually use ``.parents`` to iterate over all the parents, and check
each one against the provided filter to see if it matches.

``find_next_siblings()`` and ``find_next_sibling()``
----------------------------------------------------

Signature: find_next_siblings(:ref:`name <name>`, :ref:`attrs <attrs>`, :ref:`text <text>`, :ref:`limit <limit>`, :ref:`**kwargs <kwargs>`)

Signature: find_next_sibling(:ref:`name <name>`, :ref:`attrs <attrs>`, :ref:`text <text>`, :ref:`**kwargs <kwargs>`)

These methods use :ref:`.next_siblings <sibling-generators>` to
iterate over the rest of an element's siblings in the tree. The
``find_next_siblings()`` method returns all the siblings that match,
and ``find_next_sibling()`` only returns the first one::

 first_link = soup.a
 first_link
 # <a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>

 first_link.find_next_siblings("a")
 # [<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
 #  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

 first_story_paragraph = soup.find("p", "story")
 first_story_paragraph.find_next_sibling("p")
 # <p class="story">...</p>

``find_previous_siblings()`` and ``find_previous_sibling()``
------------------------------------------------------------

Signature: find_previous_siblings(:ref:`name <name>`, :ref:`attrs <attrs>`, :ref:`text <text>`, :ref:`limit <limit>`, :ref:`**kwargs <kwargs>`)

Signature: find_previous_sibling(:ref:`name <name>`, :ref:`attrs <attrs>`, :ref:`text <text>`, :ref:`**kwargs <kwargs>`)

These methods use :ref:`.previous_siblings <sibling-generators>` to iterate over an element's
siblings that precede it in the tree. The ``find_previous_siblings()``
method returns all the siblings that match, and
``find_previous_sibling()`` only returns the first one::

 last_link = soup.find("a", id="link3")
 last_link
 # <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>

 last_link.find_previous_siblings("a")
 # [<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
 #  <a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>]

 first_story_paragraph = soup.find("p", "story")
 first_story_paragraph.find_previous_sibling("p")
 # <p class="title"><b>The Dormouse's story</b></p>


``find_all_next()`` and ``find_next()``
---------------------------------------

Signature: find_all_next(:ref:`name <name>`, :ref:`attrs <attrs>`, :ref:`text <text>`, :ref:`limit <limit>`, :ref:`**kwargs <kwargs>`)

Signature: find_next(:ref:`name <name>`, :ref:`attrs <attrs>`, :ref:`text <text>`, :ref:`**kwargs <kwargs>`)

These methods use :ref:`.next_elements <element-generators>` to
iterate over whatever tags and strings that come after it in the
document. The ``find_all_next()`` method returns all matches, and
``find_next()`` only returns the first match::

 first_link = soup.a
 first_link
 # <a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>

 first_link.find_all_next(text=True)
 # [u'Elsie', u',\n', u'Lacie', u' and\n', u'Tillie',
 #  u';\nand they lived at the bottom of a well.', u'\n\n', u'...', u'\n']

 first_link.find_next("p")
 # <p class="story">...</p>

In the first example, the string "Elsie" showed up, even though it was
contained within the <a> tag we started from. In the second example,
the last <p> tag in the document showed up, even though it's not in
the same part of the tree as the <a> tag we started from. For these
methods, all that matters is that an element match the filter, and
show up later in the document than the starting element.

``find_all_previous()`` and ``find_previous()``
-----------------------------------------------

Signature: find_all_previous(:ref:`name <name>`, :ref:`attrs <attrs>`, :ref:`text <text>`, :ref:`limit <limit>`, :ref:`**kwargs <kwargs>`)

Signature: find_previous(:ref:`name <name>`, :ref:`attrs <attrs>`, :ref:`text <text>`, :ref:`**kwargs <kwargs>`)

These methods use :ref:`.previous_elements <element-generators>` to
iterate over the tags and strings that came before it in the
document. The ``find_all_previous()`` method returns all matches, and
``find_previous()`` only returns the first match::

 first_link = soup.a
 first_link
 # <a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>

 first_link.find_all_previous("p")
 # [<p class="story">Once upon a time there were three little sisters; ...</p>,
 #  <p class="title"><b>The Dormouse's story</b></p>]

 first_link.find_previous("title")
 # <title>The Dormouse's story</title>

The call to ``find_all_previous("p")`` found the first paragraph in
the document (the one with class="title"), but it also finds the
second paragraph, the <p> tag that contains the <a> tag we started
with. This shouldn't be too surprising: we're looking at all the tags
that show up earlier in the document than the one we started with. A
<p> tag that contains an <a> tag must have shown up before the <a>
tag it contains.

CSS selectors
-------------

Beautiful Soup supports the most commonly-used `CSS selectors
<http://www.w3.org/TR/CSS2/selector.html>`_. Just pass a string into
the ``.select()`` method of a ``Tag`` object or the ``BeautifulSoup``
object itself.

You can find tags::

 soup.select("title")
 # [<title>The Dormouse's story</title>]

 soup.select("p nth-of-type(3)")
 # [<p class="story">...</p>]

Find tags beneath other tags::

 soup.select("body a")
 # [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
 #  <a class="sister" href="http://example.com/lacie"  id="link2">Lacie</a>,
 #  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

 soup.select("html head title")
 # [<title>The Dormouse's story</title>]

Find tags `directly` beneath other tags::

 soup.select("head > title")
 # [<title>The Dormouse's story</title>]

 soup.select("p > a")
 # [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
 #  <a class="sister" href="http://example.com/lacie"  id="link2">Lacie</a>,
 #  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

 soup.select("p > a:nth-of-type(2)")
 # [<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>]

 soup.select("p > #link1")
 # [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>]

 soup.select("body > a")
 # []

Find the siblings of tags::

 soup.select("#link1 ~ .sister")
 # [<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
 #  <a class="sister" href="http://example.com/tillie"  id="link3">Tillie</a>]

 soup.select("#link1 + .sister")
 # [<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>]

Find tags by CSS class::

 soup.select(".sister")
 # [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
 #  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
 #  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

 soup.select("[class~=sister]")
 # [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
 #  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
 #  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

Find tags by ID::

 soup.select("#link1")
 # [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>]

 soup.select("a#link2")
 # [<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>]

Test for the existence of an attribute::

 soup.select('a[href]')
 # [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
 #  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
 #  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

Find tags by attribute value::

 soup.select('a[href="http://example.com/elsie"]')
 # [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>]

 soup.select('a[href^="http://example.com/"]')
 # [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
 #  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
 #  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

 soup.select('a[href$="tillie"]')
 # [<a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

 soup.select('a[href*=".com/el"]')
 # [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>]

Match language codes::

 multilingual_markup = """
  <p lang="en">Hello</p>
  <p lang="en-us">Howdy, y'all</p>
  <p lang="en-gb">Pip-pip, old fruit</p>
  <p lang="fr">Bonjour mes amis</p>
 """
 multilingual_soup = BeautifulSoup(multilingual_markup)
 multilingual_soup.select('p[lang|=en]')
 # [<p lang="en">Hello</p>,
 #  <p lang="en-us">Howdy, y'all</p>,
 #  <p lang="en-gb">Pip-pip, old fruit</p>]

This is a convenience for users who know the CSS selector syntax. You
can do all this stuff with the Beautiful Soup API. And if CSS
selectors are all you need, you might as well use lxml directly,
because it's faster. But this lets you `combine` simple CSS selectors
with the Beautiful Soup API.
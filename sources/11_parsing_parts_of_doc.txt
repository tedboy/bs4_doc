.. _parse_doc_part:

Parsing only part of a document
===============================

Let's say you want to use Beautiful Soup look at a document's <a>
tags. It's a waste of time and memory to parse the entire document and
then go over it again looking for <a> tags. It would be much faster to
ignore everything that wasn't an <a> tag in the first place. The
``SoupStrainer`` class allows you to choose which parts of an incoming
document are parsed. You just create a ``SoupStrainer`` and pass it in
to the ``BeautifulSoup`` constructor as the ``parse_only`` argument.

(Note that *this feature won't work if you're using the html5lib parser*.
If you use html5lib, the whole document will be parsed, no
matter what. This is because html5lib constantly rearranges the parse
tree as it works, and if some part of the document didn't actually
make it into the parse tree, it'll crash. To avoid confusion, in the
examples below I'll be forcing Beautiful Soup to use Python's
built-in parser.)

``SoupStrainer``
----------------

The ``SoupStrainer`` class takes the same arguments as a typical
method from :ref:`searching_the_tree`: :ref:`name <name>`, :ref:`attrs
<attrs>`, :ref:`text <text>`, and :ref:`**kwargs <kwargs>`. Here are
three ``SoupStrainer`` objects::

 from bs4 import SoupStrainer

 only_a_tags = SoupStrainer("a")

 only_tags_with_id_link2 = SoupStrainer(id="link2")

 def is_short_string(string):
     return len(string) < 10

 only_short_strings = SoupStrainer(text=is_short_string)

I'm going to bring back the "three sisters" document one more time,
and we'll see what the document looks like when it's parsed with these
three ``SoupStrainer`` objects::

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

 print(BeautifulSoup(html_doc, "html.parser", parse_only=only_a_tags).prettify())
 # <a class="sister" href="http://example.com/elsie" id="link1">
 #  Elsie
 # </a>
 # <a class="sister" href="http://example.com/lacie" id="link2">
 #  Lacie
 # </a>
 # <a class="sister" href="http://example.com/tillie" id="link3">
 #  Tillie
 # </a>

 print(BeautifulSoup(html_doc, "html.parser", parse_only=only_tags_with_id_link2).prettify())
 # <a class="sister" href="http://example.com/lacie" id="link2">
 #  Lacie
 # </a>

 print(BeautifulSoup(html_doc, "html.parser", parse_only=only_short_strings).prettify())
 # Elsie
 # ,
 # Lacie
 # and
 # Tillie
 # ...
 #

You can also pass a ``SoupStrainer`` into any of the methods covered
in :ref:`searching_the_tree`. This probably isn't terribly useful, but I
thought I'd mention it::

 soup = BeautifulSoup(html_doc)
 soup.find_all(only_short_strings)
 # [u'\n\n', u'\n\n', u'Elsie', u',\n', u'Lacie', u' and\n', u'Tillie',
 #  u'\n\n', u'...', u'\n']
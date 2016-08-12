Specifying the parser to use
============================

If you just need to parse some HTML, you can dump the markup into the
``BeautifulSoup`` constructor, and it'll probably be fine. Beautiful
Soup will pick a parser for you and parse the data. But there are a
few additional arguments you can pass in to the constructor to change
which parser is used.

The first argument to the ``BeautifulSoup`` constructor is a string or
an open filehandle--the markup you want parsed. The second argument is
`how` you'd like the markup parsed.

If you don't specify anything, you'll get the best HTML parser that's
installed. Beautiful Soup ranks lxml's parser as being the best, then
html5lib's, then Python's built-in parser. You can override this by
specifying one of the following:

* What type of markup you want to parse. Currently supported are
  "html", "xml", and "html5".

* The name of the parser library you want to use. Currently supported
  options are "lxml", "html5lib", and "html.parser" (Python's
  built-in HTML parser).

The section :ref:`parser-installation` contrasts the supported parsers.

If you don't have an appropriate parser installed, Beautiful Soup will
ignore your request and pick a different parser. Right now, the only
supported XML parser is lxml. If you don't have lxml installed, asking
for an XML parser won't give you one, and asking for "lxml" won't work
either.

.. _parser_differences:

Differences between parsers
---------------------------

Beautiful Soup presents the same interface to a number of different
parsers, but each parser is different. Different parsers will create
different parse trees from the same document. The biggest differences
are between the HTML parsers and the XML parsers. Here's a short
document, parsed as HTML::

 BeautifulSoup("<a><b /></a>")
 # <html><head></head><body><a><b></b></a></body></html>

Since an empty <b /> tag is not valid HTML, the parser turns it into a
<b></b> tag pair.

Here's the same document parsed as XML (running this requires that you
have lxml installed). Note that the empty <b /> tag is left alone, and
that the document is given an XML declaration instead of being put
into an <html> tag.::

 BeautifulSoup("<a><b /></a>", "xml")
 # <?xml version="1.0" encoding="utf-8"?>
 # <a><b/></a>

There are also differences between HTML parsers. If you give Beautiful
Soup a perfectly-formed HTML document, these differences won't
matter. One parser will be faster than another, but they'll all give
you a data structure that looks exactly like the original HTML
document.

But if the document is not perfectly-formed, different parsers will
give different results. Here's a short, invalid document parsed using
lxml's HTML parser. Note that the dangling </p> tag is simply
ignored::

 BeautifulSoup("<a></p>", "lxml")
 # <html><body><a></a></body></html>

Here's the same document parsed using html5lib::

 BeautifulSoup("<a></p>", "html5lib")
 # <html><head></head><body><a><p></p></a></body></html>

Instead of ignoring the dangling </p> tag, html5lib pairs it with an
opening <p> tag. This parser also adds an empty <head> tag to the
document.

Here's the same document parsed with Python's built-in HTML
parser::

 BeautifulSoup("<a></p>", "html.parser")
 # <a></a>

Like html5lib, this parser ignores the closing </p> tag. Unlike
html5lib, this parser makes no attempt to create a well-formed HTML
document by adding a <body> tag. Unlike lxml, it doesn't even bother
to add an <html> tag.

Since the document "<a></p>" is invalid, none of these techniques is
the "correct" way to handle it. The html5lib parser uses techniques
that are part of the HTML5 standard, so it has the best claim on being
the "correct" way, but all three techniques are legitimate.

Differences between parsers can affect your script. If you're planning
on distributing your script to other people, or running it on multiple
machines, you should specify a parser in the ``BeautifulSoup``
constructor. That will reduce the chances that your users parse a
document differently from the way you parse it.
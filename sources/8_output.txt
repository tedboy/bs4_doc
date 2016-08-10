Output
======

.. _.prettyprinting:

Pretty-printing
---------------

The ``prettify()`` method will turn a Beautiful Soup parse tree into a
nicely formatted Unicode string, with each HTML/XML tag on its own line::

  markup = '<a href="http://example.com/">I linked to <i>example.com</i></a>'
  soup = BeautifulSoup(markup)
  soup.prettify()
  # '<html>\n <head>\n </head>\n <body>\n  <a href="http://example.com/">\n...'

  print(soup.prettify())
  # <html>
  #  <head>
  #  </head>
  #  <body>
  #   <a href="http://example.com/">
  #    I linked to
  #    <i>
  #     example.com
  #    </i>
  #   </a>
  #  </body>
  # </html>

You can call ``prettify()`` on the top-level ``BeautifulSoup`` object,
or on any of its ``Tag`` objects::

  print(soup.a.prettify())
  # <a href="http://example.com/">
  #  I linked to
  #  <i>
  #   example.com
  #  </i>
  # </a>

Non-pretty printing
-------------------

If you just want a string, with no fancy formatting, you can call
``unicode()`` or ``str()`` on a ``BeautifulSoup`` object, or a ``Tag``
within it::

 str(soup)
 # '<html><head></head><body><a href="http://example.com/">I linked to <i>example.com</i></a></body></html>'

 unicode(soup.a)
 # u'<a href="http://example.com/">I linked to <i>example.com</i></a>'

The ``str()`` function returns a string encoded in UTF-8. See
:ref:`encodings` for other options.

You can also call ``encode()`` to get a bytestring, and ``decode()``
to get Unicode.

.. _output_formatters:

Output formatters
-----------------

If you give Beautiful Soup a document that contains HTML entities like
"&lquot;", they'll be converted to Unicode characters::

 soup = BeautifulSoup("&ldquo;Dammit!&rdquo; he said.")
 unicode(soup)
 # u'<html><head></head><body>\u201cDammit!\u201d he said.</body></html>'

If you then convert the document to a string, the Unicode characters
will be encoded as UTF-8. You won't get the HTML entities back::

 str(soup)
 # '<html><head></head><body>\xe2\x80\x9cDammit!\xe2\x80\x9d he said.</body></html>'

By default, the only characters that are escaped upon output are bare
ampersands and angle brackets. These get turned into "&amp;", "&lt;",
and "&gt;", so that Beautiful Soup doesn't inadvertently generate
invalid HTML or XML::

 soup = BeautifulSoup("<p>The law firm of Dewey, Cheatem, & Howe</p>")
 soup.p
 # <p>The law firm of Dewey, Cheatem, &amp; Howe</p>

 soup = BeautifulSoup('<a href="http://example.com/?foo=val1&bar=val2">A link</a>')
 soup.a
 # <a href="http://example.com/?foo=val1&amp;bar=val2">A link</a>

You can change this behavior by providing a value for the
``formatter`` argument to ``prettify()``, ``encode()``, or
``decode()``. Beautiful Soup recognizes four possible values for
``formatter``.

The default is ``formatter="minimal"``. Strings will only be processed
enough to ensure that Beautiful Soup generates valid HTML/XML::

 french = "<p>Il a dit &lt;&lt;Sacr&eacute; bleu!&gt;&gt;</p>"
 soup = BeautifulSoup(french)
 print(soup.prettify(formatter="minimal"))
 # <html>
 #  <body>
 #   <p>
 #    Il a dit &lt;&lt;Sacré bleu!&gt;&gt;
 #   </p>
 #  </body>
 # </html>

If you pass in ``formatter="html"``, Beautiful Soup will convert
Unicode characters to HTML entities whenever possible::

 print(soup.prettify(formatter="html"))
 # <html>
 #  <body>
 #   <p>
 #    Il a dit &lt;&lt;Sacr&eacute; bleu!&gt;&gt;
 #   </p>
 #  </body>
 # </html>

If you pass in ``formatter=None``, Beautiful Soup will not modify
strings at all on output. This is the fastest option, but it may lead
to Beautiful Soup generating invalid HTML/XML, as in these examples::

 print(soup.prettify(formatter=None))
 # <html>
 #  <body>
 #   <p>
 #    Il a dit <<Sacré bleu!>>
 #   </p>
 #  </body>
 # </html>

 link_soup = BeautifulSoup('<a href="http://example.com/?foo=val1&bar=val2">A link</a>')
 print(link_soup.a.encode(formatter=None))
 # <a href="http://example.com/?foo=val1&bar=val2">A link</a>

Finally, if you pass in a function for ``formatter``, Beautiful Soup
will call that function once for every string and attribute value in
the document. You can do whatever you want in this function. Here's a
formatter that converts strings to uppercase and does absolutely
nothing else::

 def uppercase(str):
     return str.upper()

 print(soup.prettify(formatter=uppercase))
 # <html>
 #  <body>
 #   <p>
 #    IL A DIT <<SACRÉ BLEU!>>
 #   </p>
 #  </body>
 # </html>

 print(link_soup.a.prettify(formatter=uppercase))
 # <a href="HTTP://EXAMPLE.COM/?FOO=VAL1&BAR=VAL2">
 #  A LINK
 # </a>

If you're writing your own function, you should know about the
``EntitySubstitution`` class in the ``bs4.dammit`` module. This class
implements Beautiful Soup's standard formatters as class methods: the
"html" formatter is ``EntitySubstitution.substitute_html``, and the
"minimal" formatter is ``EntitySubstitution.substitute_xml``. You can
use these functions to simulate ``formatter=html`` or
``formatter==minimal``, but then do something extra.

Here's an example that replaces Unicode characters with HTML entities
whenever possible, but `also` converts all strings to uppercase::

 from bs4.dammit import EntitySubstitution
 def uppercase_and_substitute_html_entities(str):
     return EntitySubstitution.substitute_html(str.upper())

 print(soup.prettify(formatter=uppercase_and_substitute_html_entities))
 # <html>
 #  <body>
 #   <p>
 #    IL A DIT &lt;&lt;SACR&Eacute; BLEU!&gt;&gt;
 #   </p>
 #  </body>
 # </html>

One last caveat: if you create a ``CData`` object, the text inside
that object is always presented `exactly as it appears, with no
formatting`. Beautiful Soup will call the formatter method, just in
case you've written a custom method that counts all the strings in the
document or something, but it will ignore the return value::

 from bs4.element import CData
 soup = BeautifulSoup("<a></a>")
 soup.a.string = CData("one < three")
 print(soup.a.prettify(formatter="xml"))
 # <a>
 #  <![CDATA[one < three]]>
 # </a>


``get_text()``
--------------

If you only want the text part of a document or tag, you can use the
``get_text()`` method. It returns all the text in a document or
beneath a tag, as a single Unicode string::

  markup = '<a href="http://example.com/">\nI linked to <i>example.com</i>\n</a>'
  soup = BeautifulSoup(markup)

  soup.get_text()
  u'\nI linked to example.com\n'
  soup.i.get_text()
  u'example.com'

You can specify a string to be used to join the bits of text
together::

 # soup.get_text("|")
 u'\nI linked to |example.com|\n'

You can tell Beautiful Soup to strip whitespace from the beginning and
end of each bit of text::

 # soup.get_text("|", strip=True)
 u'I linked to|example.com'

But at that point you might want to use the :ref:`.stripped_strings <string-generators>`
generator instead, and process the text yourself::

 [text for text in soup.stripped_strings]
 # [u'I linked to', u'example.com']
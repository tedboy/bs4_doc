.. _encodings:

Encodings
=========

Any HTML or XML document is written in a specific encoding like ASCII
or UTF-8.  But when you load that document into Beautiful Soup, you'll
discover it's been converted to Unicode::

 markup = "<h1>Sacr\xc3\xa9 bleu!</h1>"
 soup = BeautifulSoup(markup)
 soup.h1
 # <h1>Sacré bleu!</h1>
 soup.h1.string
 # u'Sacr\xe9 bleu!'

It's not magic. (That sure would be nice.) Beautiful Soup uses a
sub-library called `Unicode, Dammit`_ to detect a document's encoding
and convert it to Unicode. The autodetected encoding is available as
the ``.original_encoding`` attribute of the ``BeautifulSoup`` object::

 soup.original_encoding
 'utf-8'

Unicode, Dammit guesses correctly most of the time, but sometimes it
makes mistakes. Sometimes it guesses correctly, but only after a
byte-by-byte search of the document that takes a very long time. If
you happen to know a document's encoding ahead of time, you can avoid
mistakes and delays by passing it to the ``BeautifulSoup`` constructor
as ``from_encoding``.

Here's a document written in ISO-8859-8. The document is so short that
Unicode, Dammit can't get a good lock on it, and misidentifies it as
ISO-8859-7::

 markup = b"<h1>\xed\xe5\xec\xf9</h1>"
 soup = BeautifulSoup(markup)
 soup.h1
 <h1>νεμω</h1>
 soup.original_encoding
 'ISO-8859-7'

We can fix this by passing in the correct ``from_encoding``::

 soup = BeautifulSoup(markup, from_encoding="iso-8859-8")
 soup.h1
 <h1>םולש</h1>
 soup.original_encoding
 'iso8859-8'

In rare cases (usually when a UTF-8 document contains text written in
a completely different encoding), the only way to get Unicode may be
to replace some characters with the special Unicode character
"REPLACEMENT CHARACTER" (U+FFFD, �). If Unicode, Dammit needs to do
this, it will set the ``.contains_replacement_characters`` attribute
to ``True`` on the ``UnicodeDammit`` or ``BeautifulSoup`` object. This
lets you know that the Unicode representation is not an exact
representation of the original--some data was lost. If a document
contains �, but ``.contains_replacement_characters`` is ``False``,
you'll know that the � was there originally (as it is in this
paragraph) and doesn't stand in for missing data.

Output encoding
---------------

When you write out a document from Beautiful Soup, you get a UTF-8
document, even if the document wasn't in UTF-8 to begin with. Here's a
document written in the Latin-1 encoding::

 markup = b'''
  <html>
   <head>
    <meta content="text/html; charset=ISO-Latin-1" http-equiv="Content-type" />
   </head>
   <body>
    <p>Sacr\xe9 bleu!</p>
   </body>
  </html>
 '''

 soup = BeautifulSoup(markup)
 print(soup.prettify())
 # <html>
 #  <head>
 #   <meta content="text/html; charset=utf-8" http-equiv="Content-type" />
 #  </head>
 #  <body>
 #   <p>
 #    Sacré bleu!
 #   </p>
 #  </body>
 # </html>

Note that the <meta> tag has been rewritten to reflect the fact that
the document is now in UTF-8.

If you don't want UTF-8, you can pass an encoding into ``prettify()``::

 print(soup.prettify("latin-1"))
 # <html>
 #  <head>
 #   <meta content="text/html; charset=latin-1" http-equiv="Content-type" />
 # ...

You can also call encode() on the ``BeautifulSoup`` object, or any
element in the soup, just as if it were a Python string::

 soup.p.encode("latin-1")
 # '<p>Sacr\xe9 bleu!</p>'

 soup.p.encode("utf-8")
 # '<p>Sacr\xc3\xa9 bleu!</p>'

Any characters that can't be represented in your chosen encoding will
be converted into numeric XML entity references. Here's a document
that includes the Unicode character SNOWMAN::

 markup = u"<b>\N{SNOWMAN}</b>"
 snowman_soup = BeautifulSoup(markup)
 tag = snowman_soup.b

The SNOWMAN character can be part of a UTF-8 document (it looks like
☃), but there's no representation for that character in ISO-Latin-1 or
ASCII, so it's converted into "&#9731" for those encodings::

 print(tag.encode("utf-8"))
 # <b>☃</b>

 print tag.encode("latin-1")
 # <b>&#9731;</b>

 print tag.encode("ascii")
 # <b>&#9731;</b>

.. _unicode damn:

Unicode, Dammit
---------------

You can use Unicode, Dammit without using Beautiful Soup. It's useful
whenever you have data in an unknown encoding and you just want it to
become Unicode::

 from bs4 import UnicodeDammit
 dammit = UnicodeDammit("Sacr\xc3\xa9 bleu!")
 print(dammit.unicode_markup)
 # Sacré bleu!
 dammit.original_encoding
 # 'utf-8'

Unicode, Dammit's guesses will get a lot more accurate if you install
the ``chardet`` or ``cchardet`` Python libraries. The more data you
give Unicode, Dammit, the more accurately it will guess. If you have
your own suspicions as to what the encoding might be, you can pass
them in as a list::

 dammit = UnicodeDammit("Sacr\xe9 bleu!", ["latin-1", "iso-8859-1"])
 print(dammit.unicode_markup)
 # Sacré bleu!
 dammit.original_encoding
 # 'latin-1'

Unicode, Dammit has two special features that Beautiful Soup doesn't
use.

Smart quotes
^^^^^^^^^^^^

You can use Unicode, Dammit to convert Microsoft smart quotes to HTML or XML
entities::

 markup = b"<p>I just \x93love\x94 Microsoft Word\x92s smart quotes</p>"

 UnicodeDammit(markup, ["windows-1252"], smart_quotes_to="html").unicode_markup
 # u'<p>I just &ldquo;love&rdquo; Microsoft Word&rsquo;s smart quotes</p>'

 UnicodeDammit(markup, ["windows-1252"], smart_quotes_to="xml").unicode_markup
 # u'<p>I just &#x201C;love&#x201D; Microsoft Word&#x2019;s smart quotes</p>'

You can also convert Microsoft smart quotes to ASCII quotes::

 UnicodeDammit(markup, ["windows-1252"], smart_quotes_to="ascii").unicode_markup
 # u'<p>I just "love" Microsoft Word\'s smart quotes</p>'

Hopefully you'll find this feature useful, but Beautiful Soup doesn't
use it. Beautiful Soup prefers the default behavior, which is to
convert Microsoft smart quotes to Unicode characters along with
everything else::

 UnicodeDammit(markup, ["windows-1252"]).unicode_markup
 # u'<p>I just \u201clove\u201d Microsoft Word\u2019s smart quotes</p>'

Inconsistent encodings
^^^^^^^^^^^^^^^^^^^^^^

Sometimes a document is mostly in UTF-8, but contains Windows-1252
characters such as (again) Microsoft smart quotes. This can happen
when a website includes data from multiple sources. You can use
``UnicodeDammit.detwingle()`` to turn such a document into pure
UTF-8. Here's a simple example::

 snowmen = (u"\N{SNOWMAN}" * 3)
 quote = (u"\N{LEFT DOUBLE QUOTATION MARK}I like snowmen!\N{RIGHT DOUBLE QUOTATION MARK}")
 doc = snowmen.encode("utf8") + quote.encode("windows_1252")

This document is a mess. The snowmen are in UTF-8 and the quotes are
in Windows-1252. You can display the snowmen or the quotes, but not
both::

 print(doc)
 # ☃☃☃�I like snowmen!�

 print(doc.decode("windows-1252"))
 # â˜ƒâ˜ƒâ˜ƒ“I like snowmen!”

Decoding the document as UTF-8 raises a ``UnicodeDecodeError``, and
decoding it as Windows-1252 gives you gibberish. Fortunately,
``UnicodeDammit.detwingle()`` will convert the string to pure UTF-8,
allowing you to decode it to Unicode and display the snowmen and quote
marks simultaneously::

 new_doc = UnicodeDammit.detwingle(doc)
 print(new_doc.decode("utf8"))
 # ☃☃☃“I like snowmen!”

``UnicodeDammit.detwingle()`` only knows how to handle Windows-1252
embedded in UTF-8 (or vice versa, I suppose), but this is the most
common case.

Note that you must know to call ``UnicodeDammit.detwingle()`` on your
data before passing it into ``BeautifulSoup`` or the ``UnicodeDammit``
constructor. Beautiful Soup assumes that a document has a single
encoding, whatever it might be. If you pass it a document that
contains both UTF-8 and Windows-1252, it's likely to think the whole
document is Windows-1252, and the document will come out looking like
` â˜ƒâ˜ƒâ˜ƒ“I like snowmen!”`.

``UnicodeDammit.detwingle()`` is new in Beautiful Soup 4.1.0.
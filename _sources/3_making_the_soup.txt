Making the soup
===============

To parse a document, pass it into the ``BeautifulSoup``
constructor. You can pass in a string or an open filehandle::

 from bs4 import BeautifulSoup

 soup = BeautifulSoup(open("index.html"))

 soup = BeautifulSoup("<html>data</html>")

First, the document is converted to Unicode, and HTML entities are
converted to Unicode characters::

 BeautifulSoup("Sacr&eacute; bleu!")
 <html><head></head><body>Sacré bleu!</body></html>

Beautiful Soup then parses the document using the best available
parser. It will use an HTML parser unless you specifically tell it to
use an XML parser. (See :ref:`parsing-xml`.)
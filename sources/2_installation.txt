.. _installation:

Installing Beautiful Soup
=========================

If you're using a recent version of Debian or Ubuntu Linux, you can
install Beautiful Soup with the system package manager:

:kbd:`$ apt-get install python-bs4`

Beautiful Soup 4 is published through PyPi, so if you can't install it
with the system packager, you can install it with ``easy_install`` or
``pip``. The package name is ``beautifulsoup4``, and the same package
works on Python 2 and Python 3.

:kbd:`$ easy_install beautifulsoup4`

:kbd:`$ pip install beautifulsoup4`

(The ``BeautifulSoup`` package is probably `not` what you want. That's
the previous major release, `Beautiful Soup 3
<http://www.crummy.com/software/BeautifulSoup/bs3/documentation.html>`_. 
Lots of software uses
BS3, so it's still available, but if you're writing new code you
should install ``beautifulsoup4``.)

If you don't have ``easy_install`` or ``pip`` installed, you can
`download the Beautiful Soup 4 source tarball
<http://www.crummy.com/software/BeautifulSoup/download/4.x/>`_ and
install it with ``setup.py``.

:kbd:`$ python setup.py install`

If all else fails, the license for Beautiful Soup allows you to
package the entire library with your application. You can download the
tarball, copy its ``bs4`` directory into your application's codebase,
and use Beautiful Soup without installing it at all.

I use Python 2.7 and Python 3.2 to develop Beautiful Soup, but it
should work with other recent versions.

Problems after installation
---------------------------

Beautiful Soup is packaged as Python 2 code. When you install it for
use with Python 3, it's automatically converted to Python 3 code. If
you don't install the package, the code won't be converted. There have
also been reports on Windows machines of the wrong version being
installed.

If you get the ``ImportError`` "No module named HTMLParser", your
problem is that you're running the Python 2 version of the code under
Python 3.

If you get the ``ImportError`` "No module named html.parser", your
problem is that you're running the Python 3 version of the code under
Python 2.

In both cases, your best bet is to completely remove the Beautiful
Soup installation from your system (including any directory created
when you unzipped the tarball) and try the installation again.

If you get the ``SyntaxError`` "Invalid syntax" on the line
``ROOT_TAG_NAME = u'[document]'``, you need to convert the Python 2
code to Python 3. You can do this either by installing the package:

:kbd:`$ python3 setup.py install`

or by manually running Python's ``2to3`` conversion script on the
``bs4`` directory:

:kbd:`$ 2to3-3.2 -w bs4`

.. _parser-installation:

Installing a parser
-------------------

Beautiful Soup supports the HTML parser included in Python's standard
library, but it also supports a number of third-party Python parsers.
One is the `lxml parser <http://lxml.de/>`_. Depending on your setup,
you might install lxml with one of these commands:

:kbd:`$ apt-get install python-lxml`

:kbd:`$ easy_install lxml`

:kbd:`$ pip install lxml`

Another alternative is the pure-Python `html5lib parser
<http://code.google.com/p/html5lib/>`_, which parses HTML the way a
web browser does. Depending on your setup, you might install html5lib
with one of these commands:

:kbd:`$ apt-get install python-html5lib`

:kbd:`$ easy_install html5lib`

:kbd:`$ pip install html5lib`

This table summarizes the advantages and disadvantages of each parser library:

+----------------------+--------------------------------------------+--------------------------------+--------------------------+
| Parser               | Typical usage                              | Advantages                     | Disadvantages            |
+----------------------+--------------------------------------------+--------------------------------+--------------------------+
| Python's html.parser | ``BeautifulSoup(markup, "html.parser")``   | * Batteries included           | * Not very lenient       |
|                      |                                            | * Decent speed                 |   (before Python 2.7.3   |
|                      |                                            | * Lenient (as of Python 2.7.3  |   or 3.2.2)              |
|                      |                                            |   and 3.2.)                    |                          |
+----------------------+--------------------------------------------+--------------------------------+--------------------------+
| lxml's HTML parser   | ``BeautifulSoup(markup, "lxml")``          | * Very fast                    | * External C dependency  |
|                      |                                            | * Lenient                      |                          |
+----------------------+--------------------------------------------+--------------------------------+--------------------------+
| lxml's XML parser    | ``BeautifulSoup(markup, ["lxml", "xml"])`` | * Very fast                    | * External C dependency  |
|                      | ``BeautifulSoup(markup, "xml")``           | * The only currently supported |                          |
|                      |                                            |   XML parser                   |                          |
+----------------------+--------------------------------------------+--------------------------------+--------------------------+
| html5lib             | ``BeautifulSoup(markup, "html5lib")``      | * Extremely lenient            | * Very slow              |
|                      |                                            | * Parses pages the same way a  | * External Python        |
|                      |                                            |   web browser does             |   dependency             |
|                      |                                            | * Creates valid HTML5          |                          |
+----------------------+--------------------------------------------+--------------------------------+--------------------------+

If you can, I recommend you install and use lxml for speed. If you're
using a version of Python 2 earlier than 2.7.3, or a version of Python
3 earlier than 3.2.2, it's `essential` that you install lxml or
html5lib--Python's built-in HTML parser is just not very good in older
versions.

Note that if a document is invalid, different parsers will generate
different Beautiful Soup trees for it. See :ref:`parser_differences` for details.
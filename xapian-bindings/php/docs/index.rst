PHP bindings for Xapian
***********************

The PHP bindings for Xapian are packaged in the ``xapian``
extension.  The PHP API provided by this extension largely follows Xapian's C++
API.  This document lists the differences and additions.

As of Xapian version 1.3.1, these bindings require at least PHP 5.3.
(if you really need support for PHP 5.0, 5.1, or 5.2, Xapian 1.2
supports these).

PHP strings, arrays, etc., are converted automatically to and from the
corresponding C++ types in the bindings, so generally you can pass arguments as
you would expect.  One thing to be aware of though is that SWIG implements
dispatch functions for overloaded methods based on the types of the parameters,
so you can't always pass in a string containing a number (e.g.
``"42"``) where a number is expected as you usually can in PHP.
You need to
explicitly convert to the type required - e.g. use ``(int)`` to
convert to an integer, ``(string)`` to string, ``(double)``
to a floating point number.

With version 1.2.6 and later, you can subclass Xapian classes in PHP and
virtual methods defined in PHP are called from C++ in the way you'd expect.

PHP has a lot of reserved words of various sorts, which sadly clash with common
method names.  Because of this ``empty()`` methods of various
container-like classes are wrapped as ``is_empty()`` for PHP
and the ``clone()`` method of the ``XapianWeight``
class and subclasses is wrapped as ``clone_object()``.

The ``examples`` subdirectory contains examples showing how to use the
PHP bindings based on the simple examples from ``xapian-examples``:
`simpleindex.php5 <examples/simpleindex.php5>`_,
`simplesearch.php5 <examples/simplesearch.php5>`_,
`simpleexpand.php5 <examples/simpleexpand.php5>`_,
`simplematchdecider.php5 <examples/simplematchdecider.php5>`_.

Note that these examples are written to work with the command line (CLI)
version of the PHP interpreter, not through a webserver.  Xapian's PHP
bindings may of course also be used under CGI, Apache's modphp, ISAPI,
etc.

Installation
============

Assuming you have a suitable version of PHP installed, running
configure will automatically enable the PHP bindings, and
``make install`` will install the extension shared library in
the location reported by ``php-config --extension-dir``.

Check that php.ini has a line like ``extension_dir = "<location reported by php-config --extension-dir>"``.


Then add this line to php.ini: ``extension = xapian.so`` (or
whatever the library is called - not all UNIX systems use ``.so``
as the extension, and MS Windows uses ``.dll``).

If you're using PHP as a webserver module (e.g. mod_php with Apache), you
may need to restart the webserver for this change to take effect.

Alternatively, you can get scripts which use Xapian to explicitly load it.
This approach is useful if you don't have root access and so can't make
changes to php.ini.  The simplest set up is to copy ``xapian.so`` into
the same directory as your PHP script, and then add the following line to the
start of your PHP scripts which use Xapian: ``dl('xapian.so');``

You can put ``xapian.so`` elsewhere (and it's probably better to)
but note that ``dl()`` requires a **relative** path so you
might have to use something insane-looking like:
``dl('../../../../usr/lib/php5/20051025/xapian.so');``

You also need to add ``include&nbsp;"xapian.php"``
to your PHP scripts which use Xapian in order to get the PHP class wrappers.

Exceptions
##########

Exceptions thrown by Xapian are translated into PHP Exception objects
which are thrown into the PHP script.

Object orientation
##################

These PHP bindings use a PHP object oriented style.

To construct an object, use
``$object = new XapianClassName(...);``.  Objects are destroyed
when they go out of scope - to explicitly destroy an object you can use
``unset($object);`` or ``$object = Null;``

You invoke a method on an object using ``$object->method_name()``.

Unicode Support
###############

The Xapian::Stem, Xapian::QueryParser, and
Xapian::TermGenerator classes all assume text is in UTF-8.  If you want
to index strings in a different encoding, use the PHP `iconv function <http://php.net/iconv>`_ to convert them to UTF-8 before passing them to Xapian, and when reading values back from Xapian.

Iterators
#########

All iterators support ``next()`` and ``equals()`` methods
to move through and test iterators (as for all language bindings).
MSetIterator and ESetIterator also support ``prev()``.

In C++, trying to dereference or advance an iterator which is at its end point
is undefined behaviour and should be avoided, and currently the PHP wrappers
around the Xapian iterator classes inherit these semantics.  The plan is to
wrap these PHP iterators (see `ticket#520 <http://trac.xapian.org/ticket/520>`_).


Iterator dereferencing
######################

C++ iterators are often dereferenced to get information, eg
``(*it)``. With PHP these are all mapped to named methods, as
follows:

+------------------+----------------------+
| Iterator         | Dereferencing method |
+==================+======================+
| PositionIterator |    ``get_termpos()`` |
+------------------+----------------------+
| PostingIterator  |  ``get_docid()``     |
+------------------+----------------------+
| TermIterator     |     ``get_term()``   |
+------------------+----------------------+
| ValueIterator    |     ``get_value()``  |
+------------------+----------------------+
| MSetIterator     |     ``get_docid()``  |
+------------------+----------------------+
| ESetIterator     |     ``get_term()``   |
+------------------+----------------------+

Other methods, such as ``MSetIterator::get_document()``, are
available unchanged.

MSet
####

MSet objects have some additional methods to simplify access (these
work using the C++ array dereferencing):

+-----------------------------------+----------------------------------------+
| Method name                       |            Explanation                 |
+===================================+========================================+
| ``get_hit(index)``                |   returns MSetIterator at index        |
+-----------------------------------+----------------------------------------+
|``get_document_percentage(index)`` | ``convert_to_percent(get_hit(index))`` |
+-----------------------------------+----------------------------------------+
| ``get_document(index)``           | ``get_hit(index)->get_document()``     |
+-----------------------------------+----------------------------------------+
| ``get_docid(index)``              | ``get_hit(index)->get_docid()``        |
+-----------------------------------+----------------------------------------+


Database Factory Functions
##########################


- ``Xapian::Auto::open_stub(file)`` is wrapped as ``Xapian::auto_open_stub(file)`` (now deprecated)
- ``Xapian::Brass::open()`` is wrapped as ``Xapian::brass_open()`` (now deprecated)
- ``Xapian::Chert::open()`` is wrapped as ``Xapian::chert_open()`` (now deprecated)
- ``Xapian::InMemory::open()`` is wrapped as ``Xapian::inmemory_open()``
- ``Xapian::Remote::open(...)`` is wrapped as ``Xapian::remote_open(...)`` (both the TCP and "program" versions are wrapped - the SWIG wrapper checks the parameter list to decide which to call).
- ``Xapian::Remote::open_writable(...)`` is wrapped as ``Xapian::remote_open_writable(...)`` (both the TCP and "program" versions are wrapped - the SWIG wrapper checks the parameter list to decide which to call).

Constants
#########

Constants are wrapped as ``const`` members of the appropriate class.
So ``Xapian::DB_CREATE_OR_OPEN`` is available as
``Xapian::DB_CREATE_OR_OPEN``, ``Xapian::Query::OP_OR`` is
available as ``XapianQuery::OP_OR``, and so on.

Functions
#########

Non-class functions are wrapped in the natural way, so the C++
function ``Xapian::version_string`` is wrapped under the same
name in PHP.

Query
#####

In C++ there's a Xapian::Query constructor which takes a query operator and
start/end iterators specifying a number of terms or queries, plus an optional
parameter.  In PHP, this is wrapped to accept an array listing the terms
and/or queries (you can specify a mixture of terms and queries if you wish)
For example:

::

   $subq = new XapianQuery(XapianQuery::OP_AND, "hello", "world");
   $q = new XapianQuery(XapianQuery::OP_AND, array($subq, "foo", new XapianQuery("bar", 2)));



MatchAll and MatchNothing
-------------------------

In Xapian 1.3.0 and later, these are wrapped as static methods
``XapianQuery::MatchAll()`` and ``XapianQuery::MatchNothing()``.

If you want to be compatible with earlier versions, you can continue to use
``new XapianQuery('')`` for MatchAll and
``new XapianQuery()`` for MatchNothing.


Enquire
#######

There is an additional method ``get_matching_terms()`` which takes
an MSetIterator and returns a list of terms in the current query which
match the document given by that iterator.  You may find this
more convenient than using the TermIterator directly.
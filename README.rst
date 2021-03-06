=============================================
smart_open -- utils for streaming large files
=============================================

|Travis|_
|Downloads|_
|License|_

.. |Travis| image:: https://img.shields.io/travis/piskvorky/smart_open/master.svg
.. |Downloads| image:: https://img.shields.io/pypi/dm/smart_open.svg
.. |License| image:: https://img.shields.io/pypi/l/smart_open.svg
.. _Travis: https://travis-ci.org/piskvorky/smart_open
.. _Downloads: https://pypi.python.org/pypi/smart_open
.. _License: https://github.com/piskvorky/smart_open/blob/master/LICENSE

What?
=====

``smart_open`` is a Python 2 & Python 3 library for **efficient streaming of very large files** from/to S3, HDFS or local (compressed) files.
It is well tested (using `moto <https://github.com/spulec/moto>`_), well documented and sports a simple, Pythonic API:

.. code-block:: python

  >>> # stream lines from an S3 object
  >>> for line in smart_open.smart_open('s3://mybucket/mykey.txt'):
  ...    print line

  >>> # can use context managers too:
  >>> with smart_open.smart_open('s3://mybucket/mykey.txt') as fin:
  ...     for line in fin:
  ...         print line
  ...     fin.seek(0)  # seek to the beginning
  ...     print fin.read(1000)  # read 1000 bytes

  >>> # stream from HDFS
  >>> for line in smart_open.smart_open('hdfs://user/hadoop/my_file.txt'):
  ...    print line

  >>> # stream content *into* S3 (write mode):
  >>> with smart_open.smart_open('s3://mybucket/mykey.txt', 'wb') as fout:
  ...     for line in ['first line', 'second line', 'third line']:
  ...          fout.write(line + '\n')

  >>> # stream from/to local compressed files:
  >>> for line in smart_open.smart_open('./foo.txt.gz'):
  ...    print line

  >>> with smart_open.smart_open('/home/radim/foo.txt.bz2', 'wb') as fout:
  ...    fout.write("some content\n")

Since going over all (or select) keys in an S3 bucket is a very common operation,
there's also an extra method ``smart_open.s3_iter_bucket()`` that does this efficiently,
**processing the bucket keys in parallel** (using multiprocessing):

.. code-block:: python

  >>> # get all JSON files under "mybucket/foo/"
  >>> bucket = boto.connect_s3().get_bucket('mybucket')
  >>> for key, content in s3_iter_bucket(bucket, prefix='foo/', accept_key=lambda key: key.endswith('.json')):
  ...     print key, len(content)

For more info (S3 credentials in URI, minimum S3 part size...) and full method signatures, check out the API docs:

.. code-block:: python

  >>> import smart_open
  >>> help(smart_open.smart_open_lib)

Why?
----

Working with large S3 files using Amazon's default Python library, `boto <http://docs.pythonboto.org/en/latest/>`_, is a pain. Its ``key.set_contents_from_string()`` and ``key.get_contents_as_string()`` methods only work for small files (loaded in RAM, no streaming).
There are nasty hidden gotchas when using ``boto``'s multipart upload functionality, and a lot of boilerplate.

``smart_open`` shields you from that. It builds on boto but offers a cleaner API. The result is less code for you to write and fewer bugs to make.

Installation
------------

The module has no dependencies beyond Python >= 2.6 (or Python >= 3.3) and ``boto``::

    pip install smart_open

Or, if you prefer to install from the `source tar.gz <http://pypi.python.org/pypi/smart_open>`_::

    python setup.py test  # run unit tests
    python setup.py install

To run the unit tests (optional), you'll also need to install `mock <https://pypi.python.org/pypi/mock>`_ and `moto <https://github.com/spulec/moto>`_ (``pip install mock moto``). The tests are also run automatically with `Travis CI <https://travis-ci.org/piskvorky/smart_open>`_ on every commit push & pull request.

Todo
----

``smart_open`` is an ongoing effort. Suggestions, pull request and improvements welcome!

On the roadmap:

* improve ``smart_open`` support for HDFS (streaming from/to Hadoop File System)
* better documentation for the default ``file://`` scheme

Comments, bug reports
---------------------

``smart_open`` lives on `github <https://github.com/piskvorky/smart_open>`_. You can file
issues or pull requests there.

----------------

``smart_open`` is open source software released under the `MIT license <https://github.com/piskvorky/smart_open/blob/master/LICENSE>`_.
Copyright (c) 2015-now `Radim Řehůřek <http://radimrehurek.com>`_.

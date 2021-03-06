========
tinyflow
========

Experiments in data flow programming.

.. image:: https://travis-ci.org/geowurster/tinyflow.svg?branch=master
    :target: https://travis-ci.org/geowurster/tinyflow?branch=master

.. image:: https://coveralls.io/repos/geowurster/tinyflow/badge.svg?branch=master
    :target: https://coveralls.io/r/geowurster/tinyflow?branch=master

After some experimentation, Apache Beam's Python SDK got the API right.
Use that instead.


Word Count
==========

Grab the 5 most common words in ``LICENSE.txt``

Dataflow-style:

.. code-block:: python

    from tinyflow.pipeline import Pipeline
    import tinyflow.transform as t


    p = Pipeline() \
        | "Split line into words" >> t.Map(lambda x: x.lower().split()) \
        | "Create stream of words" >> t.Wrap(it.chain.from_iterable) \
        | "Remove empty lines" >> t.Filter(bool) \
        | "Count words and grab top 5" >> t.Wrap(lambda x: Counter(x).most_common(5)) \
        | "Sort by frequency desc" >> t.Sort(lambda x: x[1], reverse=True)

    with open('LICENSE.txt') as f:
        results = dict(p(f))


MapReduce-style:

.. code-block:: python

    from tinyflow.pipeline import Pipeline
    import tinyflow.transform as t

    p = Pipeline() \
        | "Split lines into words" >> t.Map(lambda x: x.lower().split()) \
        | "Create a stream of words" >> t.Wrap(it.chain.from_iterable) \
        | "Create a key/val pair" >> t.Map(lambda x: (x, 1)) \
        | "Filter to optimize sort" >> t.Filter(lambda x: x[1] > 1) \
        | "Compute word frequency" >> t.ReduceByKey(op.iadd) \
        | "Sort by frequency desc" >> t.Sort(lambda x: x[1]) \
        | "Grab top 10" >> t.Wrap(lambda x: it.islice(x, 5))

    with open('LICENSE.txt') as f:
        results = dict(p(f))


Using only Python's builtins:

.. code-block:: python

    from collections import Counter
    import itertools as it

    with open('LICENSE.txt') as f:
        lines = (line.lower().split() for line in f)
        words = it.chain.from_iterable(lines)
        count = Counter(words)
        results = dict(count.most_common(10))


Roadmap
=======

``async/await`` probably.


Developing
==========

.. code-block:: console

    $ git clone https://github.com/geowurster/tinyflow.git
    $ cd tinyflow
    $ pip install -e .\[all\]
    $ py.test --cov tinyflow --cov-report term-missing


License
=======

See ``LICENSE.txt``


Changelog
=========

See ``CHANGES.md``

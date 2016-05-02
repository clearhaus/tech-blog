Doubly-augmented interval trees
===============================

Firstly, an interval tree is, well, a tree of intervals. But, it's a `binary
search tree`_. And, a "traditional" `augmented interval tree`_ is where the BST
is sorted by "minimal endpoint" and you augment each node `x` with "maximal
right endpoint in subtree rooted in `x`".

.. _`binary search tree`: https://en.wikipedia.org/wiki/Binary_search_tree
.. _`augmented interval tree`: https://en.wikipedia.org/wiki/Interval_tree#Augmented_tree


Introducing what I hereby name "doubly-augmented interval trees": Just add also
`min`; say, augment each node `x` with "maximal right endpoint in subtree rooted
in `x`" and "minimal left endpoint in subtree rooted in `x`".


What is it good for
-------------------

... absolutely nothing^W something.

Given

* ``intervals``: an array of intervals, and
* ``q``: an interval you query for,

it is good for finding all intervals in ``intervals`` that has a non-empty
intersection with ``q``.


Algorithms
----------

Warning: Pseudo-Ruby code to be seen!

Building the tree
~~~~~~~~~~~~~~~~~

First, generate the tree. We never update the BST, so we can just perfectly
balance it by ``intervals.length/2``, recursively.

While building the tree, before you build a given node, find ``min`` by
``[left, self.range, right].map(&:min).min``; vice versa for ``max``.

Searching
~~~~~~~~~

As a helper, monkey patch ``Range`` with ``#intersect?(other)`` like ``self.min
<= other.max && self.max >= other.min``.

First, should I go left? (Yes, do this before considering the current node---to
get the output sorted.) Well, only if ``not left.max < q.min``. Recurse.

Secondly, should I add the current node's range, ``r``? Well, only if
``r.intersect?(q)``.

Lastly, should I go right? Well, only if ``not q.max < right.min``. Recurse.

Add some checks for ``nil`` and you're done.


Performance
-----------

Linear search through the data we have in question would take ~40 ms. The lookup
takes ~0.2 ms, and the memory usage is of the same order of magnitude.


Implementation
--------------

To be found on https://github.com/clearhaus/range-tree.

Snake algorithm
===============

We have this problem of servers which should select a number between, say, `0` and
`999` for each transaction. And, they should be "as unique as possible" and "as
sequential as possible". (Nice spec, eh?)

Our current implementation is along the lines of

* have shared state in a database, "last reserved number is `x`"
* each server reserves a bunch of numbers by moving `x` modulo `1000` and holds
  these as a local cache; when this is "almost empty", a thread will reserve
  some more. And, the servers shares a mutex for reserving.

This is serving us pretty well. The stuff is battle tested by now, but it's way
too hard to write and test such multi-threaded stuff, and more so to debug it. I
would love to see it replaced with something simpler and something that doesn't
rely on a database.

We have been thinking of replacing this complex solution to a crappy problem
with a simpler strategy.

The knowledge we have which isn't utilized in the current solution is that

    If we use the same number "too fast", we'll get an error from some other
    system. (It's pretty cheap and we could in most cases recover from the
    error.)

We could just choose a number (uniform [1]_) randomly and choose a new one if it
was used "too recently". However, the `Birthday Paradox`_ is against us.

.. _`Birthday Paradox`: https://en.wikipedia.org/wiki/Birthday_problem

The strategy we have been thinking about is:

1. Each server chooses a random number in the range.
2. For each use of a number, just increment.
3. When we get the "used too recently" error, choose a new number randomly.

In Ruby, you could imagine something like

.. code-block:: ruby

    @current_number = rand(0..999)

    def next_number
      @current_number = (@current_number + 1) % 1000
    end

    def next_number_because_of_error
      @current_number = rand(0..999)
    end

It's kinda like multi-player snake_ [2]_; the faster you invoke ``next_number``
the faster you're moving, but the speed of the tail is dependant only on time.
[3]_

.. _snake: https://en.wikipedia.org/wiki/Snake_%28video_game%29

It certainly improves the "as sequential as possible", but what about the
Birthday Paradox? Let's go to the extremes:

* If there was only one server, there would be no Birthday Paradox involved.
* If there was as many servers as transactions, the Birthday Paradox is "the
  same".

Non-trivial toy example: 2 servers, range is modulo 10; select 3 numbers.

* Server `1` chooses the first number `m(1)` by random. Collision probability 0.

* Server `2` chooses the first number `n(1)` by random. Collision probability
  `1/10`. If collision happens, chose a new `n(1)` by random (repeat until no
  collision). Thus, assume by now that `m(1) ≠ n(1)`.

* Server `1` (WLOG_) choose a new number `m(2)`.

  a. Using the "random strategy", `P[m(2) ∈ {m(1), n(1)}] = 2/10 = 1/5`.
  b. Using the "snake strategy", `m(2) = m(1) + 1` so `P[m(2) ∈ {m(1), n(1)}] =
     P[m(2) = n(1)] = 1/10`.

.. _WLOG: https://en.wikipedia.org/wiki/Without_loss_of_generality

Intuitively, I'm pretty sure the snake algorithm is always (except for the one
extreme case) better than just random. How much better? Well, it surely depends
on the figures, and I don't care to put it on formulas. For us, the number of
servers are way smaller than the range, I cannot change the number of servers
required, and I cannot change the range.


I'd love to learn about alternative strategies. I feel like this is a research
area that someone has thought about many years ago, and that the snake algorithm
is just one of possibly other standard solutions? What's the research area
called?


.. [1] ... and this will be my last use of "uniform". Right, and "not too
    different from uniform random" is most likely good enough.

.. [2] The snakes are dropped in random positions and all slither to the right
    and the right edge wraps to the left on the next "line". When a tail is
    eaten, the tail-hungry snake is dropped in a new random position.

.. [3] Would it be equivalent if there was just edible items all over and they
    reappear instantly when a tail has left?

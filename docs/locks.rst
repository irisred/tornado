``tornado.locks`` -- Synchronization primitives
===============================================

.. versionadded:: 4.2

Coordinate coroutines with synchronization primitives analogous to those the
standard library provides to threads.

*(Note that these primitives are not actually thread-safe and cannot be used in
place of those from the standard library--they are meant to coordinate Tornado
coroutines in a single-threaded app, not to protect shared objects in a
multithreaded app.)*

.. automodule:: tornado.locks

   Condition
   ---------
   .. autoclass:: Condition
    :members:

    With a `Condition`, coroutines can wait to be notified by other coroutines:

    .. testcode::

        from tornado import ioloop, gen, locks


        io_loop = ioloop.IOLoop.current()
        condition = locks.Condition()


        @gen.coroutine
        def waiter():
            print("I'll wait right here")
            yield condition.wait()  # Yield a Future.
            print("I'm done waiting")


        @gen.coroutine
        def notifier():
            print("About to notify")
            condition.notify()
            print("Done notifying")


        @gen.coroutine
        def runner():
            # Yield two Futures; wait for waiter() and notifier() to finish.
            yield [waiter(), notifier()]

        io_loop.run_sync(runner)

    .. testoutput::

        I'll wait right here
        About to notify
        Done notifying
        I'm done waiting

    `wait` takes an optional ``timeout`` argument, which is either an absolute
    timestamp::

        io_loop = ioloop.IOLoop.current()

        # Wait up to 1 second for a notification.
        yield condition.wait(deadline=io_loop.time() + 1)

    ...or a `datetime.timedelta` for a deadline relative to the current time::

        # Wait up to 1 second.
        yield condition.wait(deadline=datetime.timedelta(seconds=1))

    The method raises `tornado.gen.TimeoutError` if there's no notification
    before the deadline.

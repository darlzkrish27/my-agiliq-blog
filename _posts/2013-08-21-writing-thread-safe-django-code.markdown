---
layout: post
title:  "Writing thread-safe django - get_or_create"
date:   2013-08-21 19:34:24+05:30
tags: threads
author: Javed
---
In this blog post, we'll discuss thread-safety, why it's important and how to
write thread-safe django code, especially for bulk operations like management
commands. We'll take a simple example - get or create.

Thread-safety:
--------------

Thread-safety means that our code can be run in multiple threads and behave as
expected. The reason that code can be unsafe with regard to threads is because
we'll be manipulating shared memory (e.g. database) from the threads and there's
a chance of a race-condition which will produce unexpected results.

To avoid this, we have the option of using read-write locks, transactions etc.

We'll look at some simple examples and try to understand these options.

The usual way:
--------------

Let's consider a management command that syncs data from another source (e.g. API,
remote database etc.. The correct way to do this would be to use the built-in
django utility - `get_or_create`:

**Update**: Updated the command to run each arg in a thread

    class MyThread(Thread):

        def __init__(self, my_id):
            super(MyThread, self).__init__(name=my_id)
            self.my_id = my_id

        def run(self):
            instance, created = MyModel.objects.get_or_create(my_id=my_id)
            print '%s %s' % (instance.id, created)
            instance.delete()
            return


    class Command(BaseCommand):
        args = '<my_id my_id ...>'
        help = 'Get or create instace of mymodel with my_id'

        def handle(self, *args, **options):
            for my_id in args:
                thread = MyThread(my_id=my_id)
                thread.start()


In this command, we'll be using the command line arg my_id get a MyModel
instance if it exists, else, we create one.

Note: we'll be discarding the object for simplicity.

Now, this management command (let's call it sync_myapp) is thread-safe because
we're just using the built-in `get_or_create` to do our database call.

To test this, run the command with same arg repeated multiple times:

    $> python manage.py sync_myapp 1 1 1 1 1

This will create five threads which will simultaneously try to get or create an instance
with my_id = 1


    $> 1 True
    $> 1 False
    $> 1 False
    $> 1 False


Note that all the threads are successful, even though they were working on the
same database at the same time.

Now, even though get_or_create is the way to go, we might need to customize a
few things which are outside the scope of `get_or_create`. For example, let's
say we need to do something special just before creating a new instance.

The problem:
------------

Let's assume our code was:

    def run(self):
        created = False
        try:
            instance = MyModel.objects.get(my_id=my_id)
        except MyModel.DoesNotExist:
            something_special()
            instance = MyModel.objects.create(my_id=my_id)
            created = True
        instance.delete()
        return

If we try to test it with the above command, you'll probably get:

    Exception in thread 1:
    Traceback (most recent call last):
    ....
    IntegrityError: duplicate key value violates unique constraint "myapp_mymodel_my_id_key"
    DETAIL:  Key (my_id)=(1) already exists.

This indicates that the try except block isn't thread-safe.

Let's try to fix this problem.

**Update**: I've removed the section dealing with locks since it's only useful when
dealing with shared memory in python processes, it's not applicable to
databases.

Using database transactions:
----------------------------

We use django's built-in `transactions` module as follows:

**Update**: We only need to wrap the create command in a transaction

    def run(self):
        created = False
        try:
            instance = MyModel.objects.get(my_id=self.my_id)
        except MyModel.DoesNotExist:
                try:
                    something_special()
                    with transaction.commit_on_success():
                        instance = MyModel.objects.create(my_id=self.my_id)
                    created = True
                except IntegrityError:
                    instance = MyModel.objects.get(my_id=self.my_id)
        print '%s %s' % (instance.id, created)
        instance.delete()
        return

Now, the transaction will be committed only if there's no error within the
context block, so we can be sure that only one thread gets the go-ahead for
`create` call.

In addition to the context manager, django also has options for using savepoints,
manually commits, rollbacks etc.

[https://docs.djangoproject.com/en/1.5/topics/db/transactions](https://docs.djangoproject.com/en/1.5/topics/db/transactions)

Caveats:
--------

If you're using MySQL, refer to this open issue on problem with `get_or_create`:

[https://code.djangoproject.com/ticket/13906](https://code.djangoproject.com/ticket/13906)

Conclusion:
-----------

Using database transactions, we can avoid data integrity issues and write
thread-safe code which can be run in parallel without any
issues.



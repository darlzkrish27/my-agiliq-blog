---
layout: post
title:  "The missing documentation for django.utils.datastructures"
date:   2012-11-01 00:00:00
author:   thejaswi
tags:   internals
---

::: {.note}
::: {.admonition-title}
Note
:::

`django.utils.datastructures` is intentionally not documented by the
django core devs because it is an internal API and is liable to change
without any notice. This file is not governed by django\'s lenient
backwards-compatible policy. You have been sufficiently warned!
:::

With the note out of the way, let\'s look at the interesting
[datastructures](https://en.wikipedia.org/wiki/Data_structure) in this
file. You may ask why we should learn about those when we shouldn\'t be
using them? Reading code is the best way of learning and this file has
some beautiful code.

`MergeDict` is the first of the datastructures in the file. It provides
a dictionary like interface but can look up from multiple dictionaries
provided during the initialization.

Here\'s an example:

    >>> md = MergeDict({"foo": "bar", "moo": "cow"}, {"abc": "def"})
    >>> md["foo"]
    'bar'
    >>> md["abc"]
    'def'
    >>> md.get("abc")
    'def'
    >>> md["xyz"]
    KeyError:
    >>> md.items()
    [('foo', 'bar'), ('moo', 'cow'), ('abc', 'def')]
    >>> md.keys()
    ['foo', 'moo', 'abc']
    >>> md.values()
    ['bar', 'cow', 'def']

The MergeDict is used within django in attaching values with a form
widget and in `request.REQUEST`.

The built-in dictionary does not maintain the order of the items but the
`SortedDict` is a subclass of the built-in dictionary that maintains the
keys in exactly the same order they were inserted.

Here\'s an example:

    >>> dd = {"foo": "bar", "moo": "cow", "abc": "def"}
    {"abc": "def", "foo": "bar", "moo": "cow"}
    >>> sd = SortedDict((("foo", "bar"), ("moo", "cow"), ("abc", "def")))
    {"foo": "bar", "moo": "cow", "abc": "def"}
    >>> dd["xyz"] = "pqr"
    >>> dd
    {'abc': 'def', 'foo': 'bar', 'moo': 'cow', 'xyz': 'pqr'}
    >>> dd["lmn"] = "ghi"
    >>> dd
    {'abc': 'def', 'foo': 'bar', 'lmn': 'ghi', 'moo': 'cow', 'xyz': 'pqr'}
    >>> sd["xyz"] = "pqr"
    >>> sd
    {'foo': 'bar', 'moo': 'cow', 'abc': 'def', 'xyz': 'pqr'}
    >>> sd["lmn"] = "ghi"
    {'foo': 'bar', 'moo': 'cow', 'abc': 'def', 'xyz': 'pqr', 'lmn': 'ghi'}

The `SortedDict` is fairly widely used inside of django generally to
build a hierarchy (like models and it\'s parents), maintaining the order
of the form fields while iterating etc.

In python 2.7, a new datastructure was introduced that mimics the
SortedDict in the
[collections](http://docs.python.org/2/library/collections.html) module
and is called
[OrderedDict](http://docs.python.org/2/library/collections.html#collections.OrderedDict).

`MultiValueDict` is a dictionary subclass that can handle multiple
values assigned to a key.

Here\'s an example:

    >>> dd = {"abc": "def", "foo": ["bar1", "bar2"]}
    >>> dd["foo"]
    ['bar1', 'bar2']
    >>> mvd = MultiValueDict({"abc": "def", "foo": ["bar1", "bar2"]})
    >>> mvd["foo"]
    'bar2'
    >>> mvd.getlist("foo")
    ['bar1', 'bar2']
    >>> mvd.getlist('blah')
    []
    >>> mvd.getlist('abc')
    'def'
    >>> mvd.setlist('xyz', ['pqr', 'ghi'])
    >>> mvd
    <MultiValueDict: {'xyz': ['pqr', 'ghi'], 'abc': 'def', 'foo': ['foo1', 'foo2']}>
    >>> mvd.appendlist('xyz', 'ijk')
    >>> mvd
    <MultiValueDict: {'xyz': ['pqr', 'ghi', 'ijk'], 'abc': 'def', 'foo': ['foo1', 'foo2']}>
    >>> mvd.update({'xyz': 'lmn'})
    >>> mvd
    <MultiValueDict: {'xyz': ['pqr', 'ghi', 'ijk', 'lmn'], 'abc': 'def', 'foo': ['foo1', 'foo2']}>

The `MultiValueDict` is used in binding data to `request.POST`, the
files to `request.FILES` and in the get parameter parsing.

The `ImmutableList` is an immutable datastructure that raises errors
when it is attempted to be mutated.

Here\'s an example:

    >>> il = ImmutableList(['foo', 'bar', 'abc'])
    >>> il += 'lmn'
    AttributeError: ImmutableList object is immutable.
    >>> il = ImmutableList(['foo', 'bar', 'abc'], warning='Custom warning')
    >>> il[1] = 123
    AttributeError: Custom warning

The `ImmutableList` is used in `request.upload_handlers` to prevent
modification after the `request.POST` or `request.FILES` have been
accessed.

The `DictWrapper` is a subclass of the built-in dictionary that prefixes
the keys. It takes a dictionary, a function and a prefix as arguments.
If a specific key lookup begins with the prefix then the value is passed
through the function before it is returned.

Here\'s an example:

    >>> dw = DictWrapper({'foo': 'bar', 'moo': 'cow'}, lambda x: x, 'abc_')
    >>> dw['foo']
    'bar'
    >>> dw['abc_foo']
    'bar'
    >>> dw['xyz_foo']
    KeyError: 'xyz_foo'
    >>> def post_process_value(value):
    ...     return "The value is " + value
    >>> dw = DictWrapper({'foo': 'bar', 'moo': 'cow'}, post_process_value, 'abc_')
    >>> dw['foo']
    'bar'
    >>> dw['abc_foo']
    'The value is bar'

The `DictWrapper` is used in quoting names for SQL queries with the key
prefix.

Hope you enjoyed learning about these hidden gems and how django works
under the hood but take the note on the top into consideration.

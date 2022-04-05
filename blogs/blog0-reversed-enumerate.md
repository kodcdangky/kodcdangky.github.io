# How to reverse an enumerate in python?
Published: April 5, 2022

## TLDR

This is the best solution I've found as of the last update to this page
```py
for index, elem in zip(reversed(range(len(my_collection))), reversed(my_collection)):
    # Do something with reversed order of index and elements
```

## The blog part
This blog post was prompted after I looked through every top Google results for "python reverse enumerate" for my own use and only found the solutions to all be using deprecated features, most notably `itertools.izip()` and `xrange()`

I am currently on Python 3.10.1

So, have you ever needed to loop through a collection in python in reverse?

The most pythonic way to loop through a collection often recommended is

```py
for elem in my_collection:
    # Do something with the current element
```

or if you also need the index as you are looping through

```py
for index, elem in enumerate(my_collection):
    # Do something with the current element its index
```

So naturally, the most obvious solution to loop through a collection in reverse while still getting both indices and elements should be

```py
for index, elem in reversed(enumerate(my_collection)):
    # Now it should spit out elements and indices in reverse, but will it though?
```

But this won't work. Anyone who has tried this will know that Python will spit on this code and tell you that `enumerate object is not reversible`. To understand why Python does not allow this, I recommend reading about `generator`, which is what `enumerate` is (well, close enough, it's not exactly inheriting from `collections.abc.Generator`, but it behaves like one), but the short answer is that `enumerate()` generates the variables `index` and `elem` as you loop through it, meaning it must start at the beginning and ends at the end, similar to a linked list per se, which also can not present it's final member without going from the beginning.

Oh and don't try

```py
for index, elem in enumerate(reversed(my_collection)):
    # Don't
```

Looking up `enumerate` on the official docs, it says that `enumerate()` is equivalent to

```py
def enumerate(sequence, start=0):
    n = start
    for elem in sequence:
        yield n, elem
        n += 1
```

and as you can see, `enumerate()` always counts up, explaining why putting `reversed()` inside `enumerate()` won't work.

So let's try writing our own. Our `n` will begin at the end of the sequence, which is `len(sequence) - 1`, minus start, similar to `enumerate()`'s n begins at `0 + start`, and we'll `yield n, elem` as we loop the sequence in reverse, and `n` will count down instead of up. And now we have

```py
def reversed_enumerate(sequence, start=0):
    n = len(sequence) - 1 - start
    for elem in reversed(sequence):
        yield n, elem
        n -= 1
```

This looks reasonable enough. `reversed()`, like `enumerate()`, is a lazy operation, meaning that it only presents the next value when asked, and not make the whole reversed sequence then loop through it, which means `reversed_enumerate()`'s performance should be on par with its builtin function counterpart's

But while I was celebrating for having come up with such a genius solution, another crossed my mind. What if I just use `zip()`? Using `zip()` will look a little something like this

```py
for index, elem in zip(reversed(range(len(my_collection))), reversed(my_collection)):
    # Do something
```

This effectively is `enumerate()` in reversed gear, and all the functions used are builtin, with the unwanted outcome of looking really ugly. So let's compare their performance. A basic performace comparing program I can come up with looks something like this

```py
from time import perf_counter

def reversed_enumerate(sequence, start=0):
    n = len(sequence) - 1 - start
    for elem in reversed(sequence):
        yield n, elem
        n -= 1

my_list = [i for i in range(10 ** 7)]

first = perf_counter()
for index, elem in reversed_enumerate(my_list):
    pass
second = perf_counter()
for index, elem in zip(reversed(range(len(my_collection))), reversed(my_collection)):
    pass
third = perf_counter()

print(second - first) # reversed_enumerate()'s time
print(third - second) # ugly-looking-zip()'s time
```

You can copy this program and run it yourself. The result honestly surprises me quite a bit. The `zip()` approach not only wins, but by a huge margin. Maybe it's just my computer, but in all my runs, the `zip()` method only takes roughly 40% of the time it takes for the `reversed_enumerate()` method to finish. In other words, `reversed_enumerate()` consistently takes more than twice as long as `zip()`

So double `reversed()` in a `zip()` takes the cake, apparently. But after all this I realize there's a much simpler approach I haven't tried, which is to just loop through indices in reverse. Laughing at myself, I also realize the potential problem of look up time, especially if the value is looked up multiple times per loop, so onto another test it is

```py
from time import perf_counter

my_list = [i for i in range(10 ** 7)]

first = perf_counter()
for index, elem in zip(reversed(range(len(my_list))), reversed(my_list)):
    for _ in range(10):
        elem
second = perf_counter()
for index in reversed(range(len(my_list))):
    for _ in range(10):
        my_list[index]
third = perf_counter()

print(second - first) # double-reversed()-in-a-zip()'s time
print(third - second) # only reversing index approach's time
```

Same list, slightly different loop. Per loop we will look up the current element 10 times, over 10 million loops we will have done 100 million look-ups, which seems reasonable to me.

On my computer, there's a clear difference between the two, with `zip()` taking the cake once again, needing only about 80% - 85% the time it takes for the other approach to finish. So until I find a better approach, I must crown this `zip()` approach as the best way to technically, loop through a reversed enumerate of a collection.

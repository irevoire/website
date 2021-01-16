+++
title = "Iterators"
tags = ["iterator", "introduction"]
authors = ["Thomas Campistron @ irevoire"]
+++

# Iterators

This blogpost is about Iterators.

We'll see:
1. What is an iterator
2. Why would you want to use one
3. How can you implement one in theory

<!-- more -->

## What is an iterator?

An iterator sometimes called a “Generator”, is a function that returns a value every time it's called and that doesn't take any arguments.
It must also return a stop value to indicate that nothing more will be returned.
This can look basic at first, but it allows us to abstract completely the things we are working on and use a lot of functions defined on the iterators.

Once you have this basic setting we'll consider there are only three types of operations defined on iterators:
1. Create
2. Modify
3. Collect


### You should be able to create an iterator
An iterator can be created in multiple ways and from multiple sources:
You could create an iterator that returns all the values between a range of integer
Or you could iterate over the elements of a data structure:
* The elements of an array, a list, or a HashSet
* he keys or values of a hashmap
* The lines of a String
* It could also be used to iterate on IO operations:
* The lines of a file or stdio
* The packets coming over the network
* And much much more

### You should be able to modify the iterator
Map is one of the most useful functions of iterators.
It's the function that allows you to modify the elements contained in an iterator. It takes your iterator and a function, then it applies the provided function to each element one by one.

After Map most of the function you'll see defined on iterators modify directly the iterator instead of modifying the content of the iterator.
The most common operations are probably:
* [Take n]: shorten your iterator to only the first 'n' elements.
* [skip n]: skip the first 'n' elements.
* [filter fn]: apply the provided function on each element, if it returns true the element is returned, if not it's skipped.

### You should be able to collect your iterator
What I call a collector here is any function that takes an iterator as an argument and does not return an iterator.
For example, if you have an iterator returning the following values: 1, 2, 3.
You could:
* Collected it in a data structure like an array or a HashSet
* Execute code for each element
* Return the first element matching a specific condition
* Sum every element
* Count the number of elements
* and much more

### Once these three operations are defined the usage of iterators should look like that:

TODO picture
data -> into iterator -> modify -> modifiy -> modify -> collect to -> data

## Why would you want to use it?

### It removes a lot of mental weight when reading code
For this part, we are going to see how to write a basic sum from an array in C, and then we are going to see how much we can simplify it by using iterators.

```C
int a[] = {1, 2, 3, 4};
int nb_elements = 4;

int i = 0;
int res = 0;

while (i < nb_elements) {
	res += a[i];
	i++;
}
```

There is a lot of “problems” with this code, and we are going to fix them one by one.
The first thing we are going to do is move to a for loop and see what are the improvement:

```C
int a[] = {1, 2, 3, 4};
int nb_elements = 4;

int res = 0;

for (int i = 0; i < nb_elements; i++) {
	res += a[i];
}
```

Here the big difference is that the "i" variable is now declared and managed on the first line, and can be used in the body of the loop without any boilerplate code. Once you read the three expressions inside of the for you can remove this weight from your head and move on to read the body of the loop.
Also now, once you are out of the loop, the "i" variable comes out of your environment. You can't access it anymore and it's better this way because if you were to use it it would certainly be by error.

But you still have to read carefully the body of the loop:
What happens if "i" is modified inside of the loop
What if your condition is wrong and "i" stop too early or too late
What if you increment it badly

That's why today, in most languages we have iterator based for-loop.
For example in rust we would write:

```rust
let a = [1, 2, 3, 4];
let mut res = 0;

for el in a.iter() {
	res += el;
}
```

Now there is no “i” variable at all to be worried about.
This is concise, easier to write, easier to read, and way more error-proof.

This can look far from what we've seen about iterators but actually, it's not, the `.iter()` method is the “into iterator” part we've seen.
Then there is no modification of the iterator and finally, the “collect” part is the for-loop that will execute any code for each element.

NOTE: we could also write this like that:
```rust
let a = [1, 2, 3, 4];
let mut res = 0;

a.iter().for_each(|el| res += el);
```

Ok great, now if we continue in this direction we could remove the `res` variable and do something like that:

```rust
let a = [1, 2, 3, 4];
let res = a.iter().sum();
```
Now with that little lines and such explicit names, it's hard to miss what is going on.
And same as before, the `.iter()` creates the iterator while the `.sum()` method collects it producing the sum of each elements.

------------

>>> Ok *great*, but in real life, you'll never have such an easy and clean problem

Yes, that's right, that's why now we are going to see how would this code evolve if we complexify the problem step by step.
For the sake of readability, we are going to compare the full iterative solution versus the iterator based for-loop solution since that's what most peoples know anyway.
This will also be an opportunity to learn about the most common methods defined on iterators.

#### What if we want to modify the values in the array before doing the sum

For example, if we wanted to add `1` for each value in the initial array we could write something like that in a non-iterative way:

```rust
let a = [1, 2, 3, 4];
let mut res = 0;

for el in a.iter() {
	res += el + 1;
}
```

Here you can see we are already starting to mix things up, the “collect” part and the “modify” part are mixed in one line.
Or we could've written the for-loop in two lines like that:
```rust
for el in a.iter() {
	let tmp = el + 1;
	res += tmp;
}
```

Now we don't mix things up but we had to declare one useless variable.

----------

And in comparison the full-iterative way look like that:
```rust
let a = [1, 2, 3, 4];

let res = a.iter().map(|el| el + 1).sum();
```

Here we have no temporary variable, the “modification” part is focused only on modifying the elements and nothing else.
And the “collect” part is also really easy to read.

#### What if we don't want to do a sum at the end but something else
We are going to say we wanted to subtract every element instead of adds them.
Since this is not a common operation there is no already defined way to do it.

```rust
let a = [1, 2, 3, 4];
let mut res = 0;

for el in a.iter() {
	res -= el + 1;
}
```

This version doesn't change much since it was using nothing pre-defined.

-----------

The full-iterative version:

```rust
let a = [1, 2, 3, 4];

let res = a.iter().map(|el| el + 1).fold(0, |acc, el| acc - el);
```

Ok now there is some interesting code, the `.fold` method, sometimes called `reduce` or `inject` has this definition in the rust documentation:
>>> `fold()` takes two arguments: an initial value, and a closure with two arguments: an 'accumulator', and an element. The closure returns the value that the accumulator should have for the next iteration.
>>> The initial value is the value the accumulator will have on the first call.

Ok so if we decompose our computation here is what is happening:

| initial values | after map  |   | what the fold is computing  | the result of the fold  |
|----------------|------------|---|-----------------------------|-------------------------|
|        1       |      2     |   |(initial value) 0 - (current value) 2|    -2           |
|        2       |      3     |   |(previous result)   -2 - 3   |            -5           |
|        3       |      4     |   |             -5 - 4          |            -9           |
|        4       |      5     |   |             -9 - 5          |            -14          |

So basically we introduced some kind of “mutability” but unlike the `res` variable of the non-iterative version, it's actually limited to a very small scope and only inside the fold.


This brings us to my last point of why you should use iterator:
If you need to refactorize this code there is a good chance you will be able to just delete or copy and paste a method without having to copy any code defined somewhere else like `res` or even reading all the other method applied to the iterator.


#### Special note for python
Python implements iterator as objects, but bad.
Instead of defining the operation on iterators as methods, they define it as global functions.
Not only there is no point in doing that since you won't use one of these operations for something else than an iterator.
But it also breaks the previous argument because an easy code we've seen before like:

```rust
let a = [1, 2, 3, 4];

let res = a.iter().map(|el| el + 1).sum();
```

Will be transformed into:

```python
a = [1, 2, 3, 4]
res = sum(map(lambda el: el + 1, a))
```

The problem with this way of writing any object code is that the reading order will go from left to right / top to bottom to absolute shit.
This small example for example will be read in this order probably:

```python
sum(map(lambda el: el + 1, a))
-v- -v- --------v--------  v
 4    2          3          1
```

Also now you can't just copy / paste / delete code easily.
And obviously it gets worse with real longer code:

```rust
let res = a
    .iter()
    .map(|el| el + 1)
    .filter(blablabla)
    .scan(lalala)
    .sum();
```

Here is how you would write code too long to stay on one line. Everything we said earlier stays true.
But you just can't split the python code on multi-lines since there is parenthesis everywhere.


### The speed
If the speed of your code really matters, there is a good chance you'll write it in a fast language like C, C++, or rust.
Sadly no one uses iterators in C.
But for the cases of C++ and rust, all the iterator abstraction is supposed to be optimized out and should give you on par performance with a handwritten loop.

---------

Also if you are using any other language you probably won't mind the fact it's a bit slower **and** you'll gain the ability to easily execute your code concurrently or even in parallel.
In this blogpost I won't explain why it's easier to do this on iterator-based code, but just know most of the time it is possible.

And you'll most likely find a library doing everything for you:
- rust: [rayon](https://docs.rs/rayon/1.5.0/rayon/)
- scala: [ParIterable](https://www.scala-lang.org/api/2.12.2/scala/collection/parallel/ParIterable.html)
- TODO: put more links


### When you should **not** use iterators
If you are using iterator it's to keep each function small and depending on nothing from the callee environment.
So if your code is using a lot of variables coming from **outside** the environment or too much mutability just give up and come back to a good old ugly loop because badly written iterator code will probably be harder to read.




## How are iterators implemented in theory?

Now we know what is an iterator we'll see how you could implement one easily.
Keep in mind these implementations are just for fun.
It'll not be efficient and usually, someone already did the work way better than you'll ever do it.


So the first step is to define what is an iterator.
As we said earlier, an iterator is something returning values. Nothing more.

Since there is a lot of ways do to this we'll see some implementations in multiple languages depending on what features the languages implements.
The aim will be to create an `iiter` function that creates an iterator from an array returning each element from the index 0 (or 1 if the language starts at one, looking at you Lua and R) to the first index returning no value.
Then we'll create a `map` and a `sum` function.
And finally, we'll write the previous code with our basic implementation.

### Embedded function
The first language feature we are going to use is “embedded function” or “closure”.
The feature we are looking for is the possibility of capturing the data we are working on.


a lot of great implementations here

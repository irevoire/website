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

After Map most of the function you'll see defined on iterator modify directly the iterator instead of modifying the content of the iterator.
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
```rust
let a = [1, 2, 3, 4];

let mut res = 0;
let mut i = 0;

while i < a.len() {
	res += a[i];
	i += 1;
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

This is way more concise, easier to write and read, and way more error-proof.
Now if we look with what we know of iterators it's not obvious to see what happens because the for-loop hides everything.
First, the for-loop will transform your data structure into an iterator.
Then, no operations are applied to the iterator, and finally, the collector is just the execution of our code on every element.

Now there is no “i” variable at all to be worried about.

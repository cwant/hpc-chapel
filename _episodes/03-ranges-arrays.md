---
title: "Ranges and arrays"
teaching: 60
exercises: 30
questions:
- "What is Chapel and why is it useful?"
objectives:
- "Learn to define and use ranges and arrays."
keypoints:
- "A range is a sequence of integers."
- "An array holds a sequence of values."
- "Chapel arrays can start at any index, not just 0 or 1."
- "You can index arrays with the `[]` brackets."
---

## Ranges and Arrays

A series of integers (1,2,3,4,5, for example), is called a **_range_**. 
Ranges are generated with the `..` operator, and are useful, among other things, to declare **arrays** of variables.

Let's examine what a range looks like (`ranges.chpl` in this example):

```
var example_range = 0..10;
writeln('Our example range was set to: ', example_range);
```
{: .source}
```
chpl ranges.chpl -o ranges
./ranges
```
{: .bash}
```
Our example range was set to: 0..10
```
{: .output}

An **array** is a multidimensional sequence of values. 
Arrays can be any size, and are defined using ranges:
Let's define a 1-dimensional array of the size `example_range` and see what it looks like.
Notice how the size of an array is included with its type.

```
var example_range = 0..10;
writeln('Our example range was set to: ', example_range);

var example_array: [example_range] real;
writeln('Our example array is now: ', example_array);
```
{: .source}

We can reassign the values in our example array the same way we would reassign a variable.
An array can either be set all to a single value, or a sequence of values.

```
var example_range = 0..10;
writeln('Our example range was set to: ', example_range);

var example_array: [example_range] real;
writeln('Our example array is now: ', example_array);

example_array = 5;
writeln('When set to 5: ', example_array);

example_array = 1..11;
writeln('When set to a range: ', example_array);
```
{: .source}
```
chpl ranges.chpl -o ranges
./ranges
```
{: .bash}
```
Our example range was set to: 0..10
Our example array is now: 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0
When set to 5: 5.0 5.0 5.0 5.0 5.0 5.0 5.0 5.0 5.0 5.0 5.0
When set to a range: 1.0 2.0 3.0 4.0 5.0 6.0 7.0 8.0 9.0 10.0 11.0
```
{: .output}

Notice how ranges are "right inclusive", the last number of a range is included in the range. 
This is different from languages like Python where this does not happen.

## Indexing elements

One final thing - we can retrieve and reset specific values of an array using `[]` notation.
Let's try retrieving and setting a specific value in our example so far:

```
var example_range = 0..10;
writeln('Our example range was set to: ', example_range);

var example_array: [example_range] real;
writeln('Our example array is now: ', example_array);

example_array = 5;
writeln('When set to 5: ', example_array);

example_array = 1..11;
writeln('When set to a range: ', example_array);

// retrieve the 5th index
writeln(example_array[5]);

// set index 5 to a new value
example_array[5] = 99999;
writeln(example_array);
```
{: .source}
```
chpl ranges.chpl -o ranges
./ranges
```
{: .bash}
```
Our example range was set to: 0..10
Our example array is now: 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0
When set to 5: 5.0 5.0 5.0 5.0 5.0 5.0 5.0 5.0 5.0 5.0 5.0
When set to a range: 1.0 2.0 3.0 4.0 5.0 6.0 7.0 8.0 9.0 10.0 11.0
6.0
1.0 2.0 3.0 4.0 5.0 99999.0 7.0 8.0 9.0 10.0 11.0
```
{: .output}

One very important thing to note - in this case, index 5 was actually the 6th element.
This was caused by how we set up our array. 
When we defined our array using a range starting at 0, element 5 corresponds to the 6th element.
Unlike most other programming languages, arrays in Chapel do not start at a fixed value - 
they can start at any number depending on how we define them!
For instance, let's redefine example_range to start at 5:

```
var example_range = 5..15;
writeln('Our example range was set to: ', example_range);

var example_array: [example_range] real;
writeln('Our example array is now: ', example_array);

example_array = 5;
writeln('When set to 5: ', example_array);

example_array = 1..11;
writeln('When set to a range: ', example_array);

// retrieve the 5th index
writeln(example_array[5]);

// set index 5 to a new value
example_array[5] = 99999;
writeln(example_array);
```
{: .source}
```
chpl ranges.chpl -o ranges
./ranges
```
{: .bash}
```
Our example range was set to: 5..15
Our example array is now: 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0
When set to 5: 5.0 5.0 5.0 5.0 5.0 5.0 5.0 5.0 5.0 5.0 5.0
When set to a range: 1.0 2.0 3.0 4.0 5.0 6.0 7.0 8.0 9.0 10.0 11.0
1.0
99999.0 2.0 3.0 4.0 5.0 6.0 7.0 8.0 9.0 10.0 11.0
```
{: .output}

## Back to our simulation

Let's define a two dimensional array for use in our simulation:

~~~
// this is our "plate"
var temperature: [0..rows+1, 0..cols+1] real = 25;
~~~
{: .source}

This is a matrix (2D array) with (`rows + 2`) rows and (`cols + 2`) columns of real numbers,
all initialised as 25.0.

The ranges `0..rows+1` and `0..cols+1` used here, not only define the size and shape of the array, 
they stand for the indices with which we could access particular elements of the array
using the `[ , ]` notation.

Since each index of our two-dimensional array is "zero-based"
(has lowest index of zero in both dimensions), some extra thought may be necessary to understand
which datum we are getting when we access our array with the indices. Here are some examples:

* `temperature[0,0]` is the real variable located at the first row and first column of
  the array `temperature`;
* `temperature[3,7]` is the one at the 4th row and 8th column;
* `temperature[2,3..15]` access columns 4 through 16 of the 3rd row of `temperature`;
* `temperature[0..3,4]` corresponds to the first 4 rows on the 5th column of `temperature`.

We are now be ready to expand the base solution of our simulation.
Let's print some information about the initial configuration, compile the code,
and execute it to see if everything is working as expected.

~~~
// Number of rows and columns in matrix 
const rows = 100;
const cols = 100;

// Number of iterations
const num_iterations = 500;

// Row and column of desired position
const x = 50;
const y = 50;

// Smallest difference in temperature that would be accepted before stopping
const min_diff = 0.0001: real;

// Print temperature every print_iterations iterations
const print_iterations = 20: int;

// This is our "plate" of temperature values
var temperature: [0..rows+1, 0..cols+1] real = 25;

// Greatest difference in temperature from one iteration to another
var current_diff: real;

writeln('This simulation will consider a matrix of ', rows, ' by ', cols, ' elements.');
writeln('Temperature at start is: ', temperature[x, y]);
~~~
{: .source}
```
>> chpl base_solution.chpl -o base_solution
>> ./base_solution
```
{: .input}
~~~
This simulation will consider a matrix of 100 by 100 elements.
Temperature at start is: 25.0
~~~
{: .output}

{% include links.md %}

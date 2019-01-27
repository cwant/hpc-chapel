---
title: "Conditional statements"
teaching: 60
exercises: 30
questions:
- "How do I add conditional logic to my code?"
objectives:
- "You can use the `==`, `>`, `>=`, etc. operators to make a comparison that returns true or false."
keypoints:
- "Conditional statements in Chapel are very similar to these in other languages."
---

Chapel, as most *high level programming languages*, has different statements to control the flow of the program or code.  The conditional statements are: the **_if statement_**, and the **_while statement_**.
These statements both rely on comparisons between values. 
Let's try a few comparisons to see how they work (`conditionals.chpl`):

```
writeln(1 == 2);
writeln(1 != 2);
writeln(1 > 2);
writeln(1 >= 2);
writeln(1 < 2);
writeln(1 <= 2);
```
{: .source}
```
chpl conditionals.chpl -o conditionals.o
./conditionals.o
```
{: .bash}
```
false
true
false
false
true
true
```
{: .output}

You can combine comparisons with the `&&` (AND) and `||` (OR) operators.
`&&` only returns `true` if both conditions are true, 
while `||` returns `true` if either condition is true.

```
writeln(1 == 2);
writeln(1 != 2);
writeln(1 > 2);
writeln(1 >= 2);
writeln(1 < 2);
writeln(1 <= 2);
writeln(true && true);
writeln(true && false);
writeln(true || false);
```
{: .source}
```
chpl conditionals.chpl -o conditionals.o
./conditionals.o
```
{: .bash}
```
false
true
false
false
true
true
true
false
true
```
{: .output}

## Control flow

The general syntax of a while statement is: 

```
while (condition) do 
{
  // Instructions to loop over go here
}
```

The code flows as follows: first, the condition is evaluated, and then, if it is satisfied, all the instructions within the curly brackets are executed one by one. This will be repeated over and over again until the condition does not hold anymore.

The main loop in our simulation can be programmed using a while statement like this

~~~
const num_iterations = 500;
const min_diff = 0.0001;
var c = 0;
var current_diff = min_diff;

// A simplified main loop of the simulation
while (c < num_iterations && current_diff >= min_diff) do
{
  c += 1;
  // Actual simulation calculations will go here
}
~~~
{:.source}

What we want is to repeat all the code inside the curly brackets until
the number of iterations is greater than or equal to `num_iterations`, or the
difference of temperature between iterations is less than `min_diff`.

In our case, since `current_diff` doesn't really have
a calculated value yet when we first start our code, a good starting point
is set `current_diff` equal to `min_diff` so it can initially enter the
main loop.

To count iterations we just need to keep adding 1 to the counter variable `c`.
We could do this with `c = c + 1`, or with the compound assignment, `+=`,
as in the code above. To program the rest of the logic inside the curly
brackets, on the other hand, we will need more elaborated instructions. 

Let's focus, first, on printing the temperature every 20 iterations.
To achieve this, we only need to check whether `c` is a multiple of 20,
and in that case, to print the temperature at the desired position.
This is the type of control that an **_if statement_** give us.
The general syntax is: 

```
if (condition)
{
  // Instructions A go here
} 
else {
  // Instructions B go here
}
```

The set of instructions A is executed once if the condition is satisfied;
the set of instructions B is executed otherwise (the `else` part of the
`if` statement is optional). 

So, in our case this would do the trick:

~~~
if (c % 20 == 0)
{
  writeln('Temperature at iteration ', c, ': ', temperature[x, y]);
}
~~~
{:.source}

Note that when only one instruction will be executed, there is no
need to use the curly brackets.

`%` is the modulo operator, it returns the remainder after the division.

For example, `83 % 20` returns **`3`**, because `20` divides `83` four times,
with `3` left over (i.e., `83 = 20 * 4 + 3`)

In our case, `c % 20` returns zero only when `c` is multiple of 20. 

Let's compile and execute our code to see what we get until now

```
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
writeln('We will look at the temperature at the location x = ', x,
        ', y = ', y, '.');
writeln('We will run up to ', num_iterations, ' iterations, or until ',
        'the largest difference in temperature between iterations is ',
        'less than ', min_diff, '.');

writeln('\nTemperature at start is: ', temperature[x, y]);

// This is the main loop of the simulation
var c = 0;
while (c < num_iterations) do
{
  c += 1;
  if (c % print_iterations == 0)
  {
    writeln('Temperature at iteration ', c, ': ', temperature[x, y]);
  }
}
```
{: .source}
```
chpl base_solution.chpl -o base_solution.o
./base_solution.o
```
{: .bash}
```
This simulation will consider a matrix of 100 by 100 elements.
We will look at the temperature at the location x = 50, y = 50.
We will run up to 500 iterations, or until the largest difference in temperature between iterations is less than 0.0001.

Temperature at start is: 25.0
Temperature at iteration 20: 25.0
Temperature at iteration 40: 25.0
Temperature at iteration 60: 25.0
Temperature at iteration 80: 25.0
Temperature at iteration 100: 25.0
Temperature at iteration 120: 25.0
Temperature at iteration 140: 25.0
Temperature at iteration 160: 25.0
Temperature at iteration 180: 25.0
Temperature at iteration 200: 25.0
Temperature at iteration 220: 25.0
Temperature at iteration 240: 25.0
Temperature at iteration 260: 25.0
Temperature at iteration 280: 25.0
Temperature at iteration 300: 25.0
Temperature at iteration 320: 25.0
Temperature at iteration 340: 25.0
Temperature at iteration 360: 25.0
Temperature at iteration 380: 25.0
Temperature at iteration 400: 25.0
Temperature at iteration 420: 25.0
Temperature at iteration 440: 25.0
Temperature at iteration 460: 25.0
Temperature at iteration 480: 25.0
Temperature at iteration 500: 25.0
```
{:.output}

Of course the temperature is always 25.0 at any iteration other than the initial one, as we haven't done any computation yet.

{% include links.md %}

---
title: "Using command-line arguments"
teaching: 60
exercises: 30
questions:
- "How do I use the same program for multiple use-cases?"
objectives:
- "First objective."
keypoints:
- "Config variables accept values from the command line at runtime, without you having to recompile the code."
---

From the last run of our code, we can see that 500 iterations is not enough
to get to a _steady state_ (a state where the difference in temperature
does not vary too much, i.e. `current_diff`<`min_diff`).

Now, if we want to change the number of iterations we would need to
modify `num_iterations` in the code, and compile it again. What if we
want to change the number of rows and columns in our grid to have more
precision, or if we want to see the evolution of the temperature at a
different point (x,y)? The answer would be the same: modify the code
and compile it again!

But there is a better approach than this tedious and inefficient workflow: we can pass desired configuration values to our compiled program when it
is run at the command line.

The Chapel mechanism for this is to use **_config_** variables.
A variable is declared with the `config` keyword (in addition to
`var` or `const`), like this:

~~~
// Number of iterations
config const num_iterations = 500;
~~~
{: .source}

```
>> chpl base_solution.chpl -o base_solution
```
{: .input}

Now it can be initialised with a specific value when executing the code
at the command line, using the syntax:

```
>> ./base_solution --num_iterations=3000
```
{: .input}

~~~
This simulation will consider a matrix of 100 by 100 elements.
We will look at the temperature at the location x = 1, y = 1.
We will run up to 3000 iterations, or until the largest difference in temperature between iterations is less than 0.0001.

Temperature at start is: 25.0
Temperature at iteration 20: 1.48171
Temperature at iteration 40: 0.767179
Temperature at iteration 60: 0.517628
Temperature at iteration 80: 0.390586
.
.
.
Temperature at iteration 2960: 0.00941973
Temperature at iteration 2980: 0.00932673
Temperature at iteration 3000: 0.00923474

Final temperature at x = 1, y = 1 after 3000 iterations is: 0.00923474
The largest difference in temperatures between the last two iterations was: 0.00454617
~~~
{: .output}

We still haven't converged to a steady state!

> ## Exercise
> Also make `print_iterations`, `x`, `y`, `min_diff`, `rows` and `cols`
configurable variables, and test the code simulating different configurations.
>> ## Solution
>> For example, lets use a 300 x 300 grid and observe the evolution
>> of the temperature at the position (50, 50) for 10000 iterations,
>> or until the difference of temperature between iterations is less
>> than 0.002. Also, let's print the temperature every 500 iterations.
>> ~~~
>> ./base_solution --rows=300 --cols=300 --x=50 --y=50 --num_iterations=10000 --min_diff=0.002 --print_iterations=500
>> ~~~
>> {: .input}
>> ~~~
>> This simulation will consider a matrix of 300 by 300 elements.
>> We will look at the temperature at the location x = 50, y = 50.
>> We will run up to 10000 iterations, or until the largest difference in temperature between iterations is less than 0.002.
>> 
>> Temperature at start is: 25.0
>> Temperature at iteration 500: 24.922
>> Temperature at iteration 1000: 23.748
>> Temperature at iteration 1500: 21.7191
>> Temperature at iteration 2000: 19.6298
>> Temperature at iteration 2500: 17.7517
>> Temperature at iteration 3000: 16.1303
>> Temperature at iteration 3500: 14.7441
>> Temperature at iteration 4000: 13.5575
>> Temperature at iteration 4500: 12.536
>> 
>> Final temperature at x = 50, y = 50 after 4502 iterations is: 12.5322
>> The largest difference in temperatures between the last two iterations was: 0.00199999
>> ~~~
>> {: .output}
> {: .solution}
{: .challenge}

At the end of this exercise, or program should look like this:

```
// Number of rows and columns in matrix
config const rows = 100;
config const cols = 100;

// Number of iterations
config const num_iterations = 500;

// Row and column of desired position
config const x = 1;
config const y = 1;

// Smallest difference in temperature that would be accepted before stopping
config const min_diff = 0.0001: real;

// Print temperature every print_iterations iterations
config const print_iterations = 20: int;

// This is our "plate" of temperature values
var temperature: [0..rows+1, 0..cols+1] real = 0.0;

// Set initial condition in interior
temperature[1..rows, 1..cols] = 25.0;

// Temporary storage for newly computed values
var temperature_new: [0..rows+1, 0..cols+1] real;

// Greatest difference in temperature from one iteration to another
var current_diff: real;

writeln('This simulation will consider a matrix of ', rows, ' by ', cols, ' elements.');
writeln('We will look at the temperature at the location x = ', x,
        ', y = ', y, '.');
writeln('We will run up to ', num_iterations, ' iterations, or until ',
        'the largest difference in temperature between iterations is ',
        'less than ', min_diff, '.');

writeln('\nTemperature at start is: ', temperature[x, y]);

current_diff = min_diff;

// This is the main loop of the simulation
var c = 0;
while (c < num_iterations && current_diff >= min_diff) do
{
  // Calculate the new temperatures (temperature_new) using the
  // existing temperatures (temperature)
  for i in 1..rows do
  {
    for j in 1..cols do
    {
      temperature_new[i,j] = (temperature[i-1, j] +
                              temperature[i+1, j] +
                              temperature[i, j-1] +
                              temperature[i, j+1]) / 4;
    }
  }
  var this_diff = 0.0;

  // Update current_diff
  current_diff = 0;
  for i in 1..rows do
  {
    for j in 1..cols do
    {
      this_diff = abs(temperature_new[i, j] - temperature[i, j]);
      if this_diff > current_diff then current_diff = this_diff;
    }
  }
  temperature = temperature_new;

  c += 1;
  if (c % print_iterations == 0)
  {
    writeln('Temperature at iteration ', c, ': ', temperature[x, y]);
  }

}

// Print final information
writeln('\nFinal temperature at x = ', x, ', y = ', y, ' after ', c,
        ' iterations is: ', temperature[x,y]);
writeln('The largest difference in temperatures between ',
        'the last two iterations was: ',
        current_diff, '\n');

```
{: .code}

{% include links.md %}

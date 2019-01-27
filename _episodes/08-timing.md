---
title: "Measuring code performance"
teaching: 60
exercises: 30
questions:
- "How do I know how fast my code is?"
objectives:
- "First objective."
keypoints:
- "Use UNIX `time` command or instrument your Chapel code to measure performance."
---

The code generated after the last exercise of the previous section is
the basic implementation of our simulation. We will use it as a benchmark,
to see how much we can improve the performance when introducing the
parallel programming features of the language in the following lessons. 

But first, we need a quantitative way to measure the performance of our code.
The easiest way to do it is to see how long it takes to finish a simulation.
The UNIX command `time` could be used to this effect

```
>> time ./base_solution --rows=300 --cols=300 --x=50 --y=50 --num_iterations=10000 --min_diff=0.002 --print_iterations=500
```
{: .input}

~~~
This simulation will consider a matrix of 300 by 300 elements.
We will look at the temperature at the location x = 50, y = 50.
We will run up to 10000 iterations, or until the largest difference in temperature between iterations is less than 0.002.

Temperature at start is: 25.0
Temperature at iteration 500: 24.922
Temperature at iteration 1000: 23.748
Temperature at iteration 1500: 21.7191
Temperature at iteration 2000: 19.6298
Temperature at iteration 2500: 17.7517
Temperature at iteration 3000: 16.1303
Temperature at iteration 3500: 14.7441
Temperature at iteration 4000: 13.5575
Temperature at iteration 4500: 12.536

Final temperature at x = 50, y = 50 after 4502 iterations is: 12.5322
The largest difference in temperatures between the last two iterations was: 0.00199999


real    3m9.253s
user    2m59.422s
sys     0m0.035s
~~~
{: .output}

The real time is what interests us. Our code is taking around 3 minutes
from the moment it is called at the command line until it returns. 

Some times, however, it could be useful to take the execution time of
specific parts of the code. This can be achieved by modifying the code to
output the information that we need. This process is called
**_instrumentation of code_**.

An easy way to instrument our code with Chapel is by using the module `Time`.

**_Modules_** in Chapel are libraries of useful functions and methods
that can be used once the module is loaded. To load a module we use
the keyword `use` followed by the name of the module. Once the `Time`
module is loaded we can create a variable of the type `Timer`, and use
the methods `start`, `stop` and `elapsed` to instrument our code.

```
use Time;
var watch: Timer;
watch.start();

// This is the main loop of the simulation
var c = 0;
while (c < num_iterations && curr_diff >= min_diff) do
{
...
}

watch.stop();
```
{: .code}

We can add a timing report to our final output:

```
// Print final information
writeln('\nThe simulation took ', watch.elapsed(), ' seconds');
writeln('\nFinal temperature at x = ', x, ', y = ', y, ' after ', c,
        ' iterations is: ', temperature[x,y]);
writeln('The largest difference in temperatures between ',
        'the last two iterations was: ',
        current_diff, '\n');
```
{: .code}

```
>> chpl base_solution.chpl -o base_solution
>> ./base_solution --rows=300 --cols=300 --x=50 --y=50 --num_iterations=10000 --min_diff=0.002 --print_iterations=500
```
{: .input}

~~~
This simulation will consider a matrix of 300 by 300 elements.
We will look at the temperature at the location x = 50, y = 50.
We will run up to 10000 iterations, or until the largest difference in temperature between iterations is less than 0.002.

Temperature at start is: 25.0
Temperature at iteration 500: 24.922
Temperature at iteration 1000: 23.748
Temperature at iteration 1500: 21.7191
Temperature at iteration 2000: 19.6298
Temperature at iteration 2500: 17.7517
Temperature at iteration 3000: 16.1303
Temperature at iteration 3500: 14.7441
Temperature at iteration 4000: 13.5575
Temperature at iteration 4500: 12.536

The simulation took 178.269 seconds

Final temperature at x = 50, y = 50 after 4502 iterations is: 12.5322
The largest difference in temperatures between the last two iterations was: 0.00199999
~~~
{: .output}

But wait a second! What happened to that `--fast` flag we used to compile code with? Let's time the impact of that flag:

```
>> chpl base_solution.chpl --fast -o base_solution
>> ./base_solution --rows=300 --cols=300 --x=50 --y=50 --num_iterations=10000 --min_diff=0.002 --print_iterations=500
```
{: .input}

~~~
This simulation will consider a matrix of 300 by 300 elements.
We will look at the temperature at the location x = 50, y = 50.
We will run up to 10000 iterations, or until the largest difference in temperature between iterations is less than 0.002.

Temperature at start is: 25.0
Temperature at iteration 500: 24.922
Temperature at iteration 1000: 23.748
Temperature at iteration 1500: 21.7191
Temperature at iteration 2000: 19.6298
Temperature at iteration 2500: 17.7517
Temperature at iteration 3000: 16.1303
Temperature at iteration 3500: 14.7441
Temperature at iteration 4000: 13.5575
Temperature at iteration 4500: 12.536

The simulation took 1.19623 seconds

Final temperature at x = 50, y = 50 after 4502 iterations is: 12.5322
The largest difference in temperatures between the last two iterations was: 0.00199999
~~~
{: .output}

It's quite clear that the `--fast` flag does some serious optimization!

{% include links.md %}

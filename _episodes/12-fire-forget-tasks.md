---
title: "Fire-and-forget tasks"
teaching: 60
exercises: 30
questions:
- "How do execute work in parallel?"
objectives:
- "First objective."
keypoints:
- "Use `begin` or `cobegin` or `coforall` to spawn new tasks."
- "You can run more than one task per core, as the number of cores on a node is limited."
---

## Fire-and-what?

The term "Fire-and-Forget" originates from the military, it refers to
guided missles that do not need to be controlled after they are launched
(they will seek their target without further instructions from the
person launching the missle).

In our context, we will have a main thread that will launch
subtasks in other threads. Once any other subtask has been launched,
the main thread moves on to other work, without waiting for the
completion of any launched subtask.

## The `begin` statement

A Chapel program always start as a single main thread.
You can then start concurrent tasks with the `begin` statement.
A task spawned by the `begin` statement will run in a different thread
while the main thread continues its normal execution.

Consider the following example:
 
~~~
var x = 100;

writeln("This is the main thread starting the first task ...");

// Fire-and-forget this first task
begin
{
  var count = 0;
  while count < 10
  {
    count += 1;
    writeln('Thread 1: ', x + count);
  }
}

writeln("This is the main thread starting the second task ...");

// Fire-and-forget this second task
begin
{
  var count = 0;
  while count < 10
  {
    count += 1;
    writeln('Thread 2: ', x + count);
  }
}

writeln('This is main thread, I am done!');
~~~
{: .source}
 
```
>> chpl begin_example.chpl -o begin_example
>> ./begin_example
```
{: .input}
 
~~~
This is the main thread starting the first task ...
This is the main thread starting the second task ...
This is main thread, I am done!
Thread 1: 101
Thread 1: 102
Thread 2: 101
Thread 2: 102
Thread 1: 103
Thread 2: 103
Thread 1: 104
Thread 1: 105
Thread 1: 106
Thread 1: 107
Thread 2: 104
Thread 1: 108
Thread 1: 109
Thread 1: 110
Thread 2: 105
Thread 2: 106
Thread 2: 107
Thread 2: 108
Thread 2: 109
Thread 2: 110
~~~
{: .output}

As you can see the order of the output is not what we would expected,
and actually it is completely unpredictable. This is a well known effect
of concurrent tasks accessing the same shared resource at the same time
(in this case the screen); the system decides in which order the tasks
could write to the screen. 

> ## Discussion
> What would happen if in the last code we declare `count` in the main thread? 
> What would happen if we try to modify the value of `x` inside a begin statement?
> Discuss your observations.
>> ## Key idea
>> All variables have a **_scope_** in which they can be used.
>> The variables declared inside a concurrent tasks are accessible only
>> by the task. The variables declared in the main task can be read
>> everywhere, but Chapel won't allow two concurrent tasks to try to
>> modify them. 
> {: .solution}
{: .discussion}

## The `cobegin` statement

A slightly more structured way to start concurrent tasks in Chapel
is by using the `cobegin`statement. Here you can start a block of
concurrent tasks, one for each statement inside the curly brackets.
The main difference between the `begin`and `cobegin` statements is
that with the `cobegin`, all the spawned tasks are synchronised at
the end of the statement, i.e. the main thread won't continue it's
execution until all tasks are done. 

~~~
var x = 0;
writeln("This is the main thread, my value of x is ", x);

cobegin
{
  // First statement, thread 1:
  {
    var x = 5;
    writeln("This is task 1, my value of x is ", x);
  }

  // Second statement, thread 2:
  writeln("This is task 2, my value of x is ", x);
}

writeln("This message won't appear until all tasks are done...");
~~~
{: .source}

```
>> chpl cobegin_example.chpl -o cobegin_example
>> ./cobegin_example
```
{: .input}
 
~~~
This is the main thread, my value of x is 0
This is task 2, my value of x is 0
This is task 1, my value of x is 5
This message won't appear until all tasks are done...
~~~
{: .output}

As you may have concluded from the discussion exercise above, the
variables declared inside a task are accessible only by the task,
while those variables declared in the main task are accessible to all tasks. 

## The `coforall` statement

The last, and most useful way to start concurrent/parallel tasks in Chapel,
is the `coforall` loop. This is a combination of the for-loop and the
`cobegin`statements. The general syntax is:

```
coforall index in iterand 
{
  // Instructions (possibly depending on index) go here
}
```

This will start a new task, for each iteration.
Each tasks will then perform all the instructions inside
the curly brackets.
Each task will have a copy of the variable **_index_** with the
corresponding value yielded by the iterand.
This index allows us to _customize_ the set of instructions for
each particular task. 

~~~
var x = 10;
config var num_tasks = 2;

writeln("This is the main task: x = ", x);

coforall task_id in 1..num_tasks do 
{
  // Each task runs this block with it's value of task_id
  var c = task_id + 1;
  writeln("This is task ", task_id,
          ": x + ", task_id, " = ", x + task_id,
          ". My value of c is: ", c);
}

writeln("This message won't appear until all tasks are done...");
~~~
{: .source}

~~~
 >> chpl coforall_example.chpl -o coforall_example
 >> ./coforall_example --num_tasks=5
 ~~~
 {: .input}
 
 ~~~
This is the main task: x = 10
This is task 5: x + 5 = 15. My value of c is: 6
This is task 2: x + 2 = 12. My value of c is: 3
This is task 4: x + 4 = 14. My value of c is: 5
This is task 3: x + 3 = 13. My value of c is: 4
This is task 1: x + 1 = 11. My value of c is: 2
This message won't appear until all tasks are done...
 ~~~
 {: .output}
 
Notice how we are able to customise the instructions inside the `coforall`,
to give different results depending on the task that is executing them.
Also, notice how, once again, the variables declared outside the `coforall`
can be read by all tasks, while the variables declared inside,
are available only to the particular task. 

> ## Exercise 1
> Would it be possible to print all the messages in the right order? Modify the code in the last example as required.
>
> Hint: you can use an array of strings declared in the main task, where all the concurrent tasks could write their messages in the corresponding position. Then, at the end, have the main task printing all elements of the array in order.
>> ## Solution
>> The following code is a possible solution:
>> ~~~
>> var x = 10;
>> config var num_tasks=2;
>> var messages: [1..num_tasks] string;
>>
>> writeln("This is the main task: x = ",x);
>>
>> coforall task_id in 1..num_tasks do
>> {
>>    var c = task_id + 1;
>>    var s = "This is task " + task_id + ": x + " + task_id + 
>>            " = " + (x + task_id) + ". My value of c is: " + c;
>>    messages[task_id] = s;
>> }
>>
>> for i in 1..num_tasks do writeln(messages[i]);
>> writeln("This message won't appear until all tasks are done...");
>> ~~~
>> {: .source}
>>
>> ~~~
>> chpl exercise_coforall.chpl -o exercise_coforall
>> ./exercise_coforall --numoftasks=5
>> ~~~
>> {: .input}
>>
>> ~~~
This is the main task: x = 10
This is task 1: x + 1 = 11. My value of c is: 2
This is task 2: x + 2 = 12. My value of c is: 3
This is task 3: x + 3 = 13. My value of c is: 4
This is task 4: x + 4 = 14. My value of c is: 5
This is task 5: x + 5 = 15. My value of c is: 6
This message won't appear until all tasks are done...
>> ~~~
>> {: .output}
>>
>> Note that `+` is a **_polymorphic_** operand in Chapel. In this case it concatenates `strings` with `integers` (which are transformed to strings). 
> {: .solution}
{: .challenge}

> ## Exercise 2
> Consider the following code (`random_max.chpl`):
> ```
> use Random;
>
> // Declare big array (50 million members!) called buffer
> config const length = 50000000;
> var buffer: [1..length] int;
>
> // Fill array with random numbers
> fillRandom(buffer);
>
> var buffer_max = buffer[1];
>
> // Missing parallel code to find the maximum value in the buffer ...
>
> writeln("The maximum value in the buffer is: ", buffer_max);
> ```
> {: .code}
> Complete the program using parallel methods to find the
> maximum value in the array `buffer`.
>> ## Solution
>> ```
>> use Random;
>>
>> // Declare big array (50 million members!) called buffer
>> config const length = 50000000;
>> var buffer: [1..length] int;
>>
>> // Fill array with random numbers
>> fillRandom(buffer);
>>
>> // Bonus: Sometimes it might be handy to look in the buffer
>> config const print_buffer = false;
>> if (print_buffer) {
>>   writeln('Random buffer: ', buffer);
>> }
>>
>> var buffer_max = buffer[1];
>>
>> // Tasks
>> config const num_tasks = 12;
>>
>> // If num_task doesn't divide length evenly, we'll adjust the final task
>> const task_length = length / num_tasks;
>> const remainder = length % num_tasks;
>>
>> var task_max: [1..num_tasks] int;
>>
>> coforall task_id in 1..num_tasks do
>> {
>>   // This task's chunk of the buffer
>>   var task_first = (task_id - 1) * task_length + 1;
>>   var task_last = task_first + task_length - 1;
>>
>>   // Add the remaining elements to the last task
>>   if (task_id == num_tasks)
>>   {
>>      task_last += remainder;
>>   }
>>
>>   // Initialize
>>   task_max[task_id] = buffer[task_first];
>>
>>   // Loop to find the maximum of this threads chunk of buffer
>>   for i in (task_first+1)..task_last do
>>   {
>>     if (buffer[i] > task_max[task_id]) {
>>       task_max[task_id] = buffer[i];
>>     }
>>   }
>> }
>>
>> // Find the max of the task maximums
>> for i in 1..num_tasks do
>> {
>>   if task_max[i] > buffer_max then buffer_max = task_max[i];
>> }
>>
>> writeln("The maximum value in the buffer is: ", buffer_max);
>> ```
>> {: .code}
>> We use `coforall` to spawn tasks that work concurrently in their own
>> chunk of the array. The trick here is to determine, the amount of
>> elements in the last chunk (because math).
>> Each task obtains the maximum in its chunk, and after the `coforall`
>> is done, the main task obtains the maximum of the array from the
>> maximums of all tasks.  
> {: .solution}
{: .challenge}

> ## Discussion
> Run the code of the last exercise using different number of tasks,
> and different lengths of the array `buffer` to see how the execution
> time changes. For example:
> ~~~
> >> time ./random_max --length=130000000 --num_tasks=1
> ~~~
> {: .input}
>
> Discuss your observations. Is there a limit on how fast the code could run?
{: .discussion}

> ## Try this...
> Substitute the code to find _mymax_ in the last exercise with:
> ~~~
> buffer_max = max reduce buffer;
> ~~~
> {: .source}
> Time the execution of the original code and this new one. How do they compare?
>
>> ## Key idea
>> It is always a good idea to check whether there is _built-in_ functions
>> or methods in the used language, that can do what we want to do as
>> efficiently (or better) than our house-made code.
>>
>> In this case,
>> the `reduce` statement reduces the given array to a single number
>> using the given operation (in this case `max`), and it is parallelized
>> and optimised to have a very good performance. 
> {: .solution}
{: .challenge}


The code in these last exercises somehow _synchronise_ the tasks to
obtain the desired result.
In addition, Chapel has specific mechanisms task synchronisation,
that could help us to achieve fine-grained parallelization. 

{% include links.md %}

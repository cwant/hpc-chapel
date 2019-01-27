---
title: "Synchronising tasks"
teaching: 60
exercises: 30
questions:
- "How should I access my data in parallel?"
objectives:
- "First objective."
keypoints:
- "You can explicitly synchronise tasks with `sync` statement."
- "You can also use sync and atomic variables to synchronise tasks."
---

## The `sync` keyword

The keyword `sync` provides various mechanisms to synchronise tasks in Chapel. 

We can simply use `sync` to force the **parent** task to stop and wait
until it's **spawned child-task** ends:

~~~
var x = 0;
writeln("This is the main thread starting a synchronous task");

sync
{
  begin
  {
    var count = 0;
    while count < 10
    {
      count += 1;
      writeln('Thread 1: ', x + count);
    }
  }
}
writeln("The first task is done...");

writeln("This is the main thread starting an asynchronous task");
begin
{
  var count = 0;
  while count < 10
  {
    count += 1;
    writeln('Thread 2: ', x + count);
  }
}

writeln('This is main thread, I am done...');
~~~
{: .source}

```
>> chpl sync_task.chpl -o sync_task
>> ./sync_task
```
{: .input}

~~~
This is the main thread starting a synchronous task
Thread 1: 1
Thread 1: 2
Thread 1: 3
Thread 1: 4
Thread 1: 5
Thread 1: 6
Thread 1: 7
Thread 1: 8
Thread 1: 9
Thread 1: 10
The first task is done...
This is the main thread starting an asynchronous task
This is main thread, I am done...
Thread 2: 1
Thread 2: 2
Thread 2: 3
Thread 2: 4
Thread 2: 5
Thread 2: 6
Thread 2: 7
Thread 2: 8
Thread 2: 9
Thread 2: 10
~~~
{: .output}

> ## Discussion
> What would happen with the following? 
> ~~~
var x = 0;
begin
{
  // sync inside an asynchronous task???
  sync
  {
    var count = 0;
    while count < 10
    {
      count += 1;
      writeln('Thread 1: ', x + count);
    }
  }
}
writeln("The first task is done...");
> ~~~
> {: .source}
> Discuss your observations. 
{: .discussion}

> ## Exercise 3
> Use `begin` and `sync` statements to reproduce the functionality of `cobegin` in cobegin_example.chpl.
>> ## Solution
>> ~~~
>> var x = 0;
>> writeln("This is the main thread, my value of x is ", x);
>>
>> sync
>> {
>>   begin
>>   {
>>      var x = 5;
>>      writeln("This is task 1, my value of x is ", x);
>>   }
>>   begin writeln("This is task 2, my value of x is ", x);
>> }
>>
>> writeln("This message won't appear until all tasks are done...");
>> ~~~
>> {: .source}
> {: .solution}
{: .challenge}

## The `sync` type qualifier

A more elaborated and powerful use of `sync` is as a type qualifier
for variables. When a variable is declared as `sync`, a state that
can be **full** or **empty** is associated to it.  

To **assign** a new value to a `sync` variable,  its state must be **empty**
(after the assignment operation is completed, the state will be set as
**full**).

On the contrary, to **read** a value from a `sync` variable, its state must
be **full** (after the read operation is completed, the state will be
set as **empty** again).

~~~
// value_we_need is initially 'empty'
var value_we_need: sync int;

writeln("value_we_need is empty");
writeln("This is main task launching a new task ",
        "to calculate value_we_need");

begin {
  var count = 0;

  for i in 1..10 do {
    count += i;
    writeln("This is new task adding ", i);
  }

  // The next line makes value_we_need 'full'
  value_we_need = count;
  writeln("New task finished calculating value_we_need");
}

writeln("This is main task after launching a new task... ",
        "I will wait until it is done");

var value_we_need_out: int;

// We wait until value_we_need is full...
value_we_need_out = value_we_need;

// But now value_we_need is empty again and we can't read it
writeln("value_we_need is ", value_we_need_out);
~~~
{: .source}

```
>> chpl sync_type.chpl -o sync_type
>> ./sync_type
```
{: .input}

~~~
This is main task launching a new task to calculate value_we_need
This is main task after launching a new task... I will wait until it is done
This is new task adding 1
This is new task adding 2
This is new task adding 3
This is new task adding 4
This is new task adding 5
This is new task adding 6
This is new task adding 7
This is new task adding 8
This is new task adding 9
This is new task adding 10
New task finished calculating value_we_need
value_we_need is 55
~~~
{: .output}

Notice that we needed to assign `value_we_need` to something else
(`value_we_need_out`) before we could print it? The toggle between
**full** and **empty** and back again means we have to be carefull.

> ## Discussion
> What would happen if we assign a value to `value_we_need` right before
> launching the new task?
{: .discussion}

## `sync` variable methods

There are a number of methods defined for `sync` variables.
Suppose `x` is a sync variable of a given type, 

```
// **** General methods ****

// Set the state as empty and the value as the default of x's type
x.reset()

// Return true is the state of x is full, false if it is empty
x.isfull()

// **** Blocking read/write methods ****

// Block until the state of x is empty, then assign value to set the state to full 
x.writeEF(value)

// Block until the state of x is full, then assign value and keep state full
x.writeFF(value)

// Block until x is full, then return x's value, and set the state to empty
x.readFE()

// Block until x is full, then return x's value, but leave the state as full
x.readFF()

// **** Non-blocking read/write methods ****

// Assign the value no matter the state of x, and then set the state as full
x.writeXF(value)

// Return the value of x regardless of state. The state will remain unchanged
x.readXX()
```

## The `atomic` type qualifier 

Chapel also implements **_atomic_** operations with variables declared as
`atomic`, and this provides another option to synchronise tasks.

Atomic operations run completely independently of any other thread
or process.
This means that when several tasks try to write to an atomic variable,
only one will succeed at a given moment,
providing implicit synchronisation between them.

There are a number of methods defined for atomic variables,
among them `sub()`, `add()`, `write()`, `read()`, and `waitfor()`.
These are very useful to establish explicit synchronisation between tasks,
as showed in the next example:

~~~
var lock: atomic int;
const num_tasks = 5;

// The main task sets the value of lock to zero
lock.write(0); 

coforall task_id in 1..num_tasks
{
  writeln("Greetings form task ", task_id,
          "... I am waiting for all tasks to say hello");

  // Task task_id says hello and atomically adds 1 to lock
  lock.add(1);

  //Then it waits for lock to be equal to num_tasks
  // (which will happen when all tasks say hello)
  lock.waitFor(num_tasks);

  writeln("Task ", task_id, " is done...");
}
~~~
{: .source}

```
>> chpl atomic_example.chpl -o atomic_example
>> ./atomic_example
```
{: .input}

~~~
Greetings form task 4... I am waiting for all tasks to say hello
Greetings form task 5... I am waiting for all tasks to say hello
Greetings form task 2... I am waiting for all tasks to say hello
Greetings form task 3... I am waiting for all tasks to say hello
Greetings form task 1... I am waiting for all tasks to say hello
Task 1 is done...
Task 5 is done...
Task 2 is done...
Task 3 is done...
Task 4 is done...
~~~
{: .output}

> ## Try this...
> Comment out the line `lock.waitfor(num_tasks)` in the code above to
> clearly observe the effect of the task synchronisation.
{: .challenge}

Finally, with all the material studied so far,
we should be ready to parallelize our code for the simulation of the
heat transfer equation.

{% include links.md %}

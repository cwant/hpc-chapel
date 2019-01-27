---
title: "Procedures for organizing code"
teaching: 15
exercises: 0
questions:
- "How do I write functions?"
objectives:
- "Be able to write our own procedures."
keypoints:
- "Functions in Chapel are called procedures."
- "Procedures can be recursive."
- "Procedures can take a variable number of parameters."
- "Procedures can have default parameter values."
---

Similar to other programming languages, Chapel lets you define your own
functions. These are called
'procedures' in Chapel and have an easy-to-understand syntax:

~~~
proc add_one(n) {
  // n is an input parameter
  return n + 1;
}
writeln(add_one(10));
~~~
{: .source}

Procedures can be recursive:

~~~
proc fibonacci(n: int): int {
  if n <= 1 then return n;
  return fibonacci(n-1) + fibonacci(n-2);
}
writeln(fibonacci(10));
~~~
{: .source}

They can take a variable number of parameters:

~~~
proc max_of(x ...?k) {
  // take a tuple of one type with k elements
  var maximum = x[1];
  for i in 2..k do {
    maximum = if maximum < x[i] then x[i] else maximum;
  return maximum;
}
writeln(max_of(1, -5, 123, 85, -17, 3));
~~~
{: .source}

Procedures can have default parameter values, and keyword parameters:

~~~
proc return_tuple(x: int, y: real = 3.1415926): (int, real) {
  return (x,y);
}
writeln(return_tuple(1));
writeln(return_tuple(x = 2));
writeln(return_tuple(x = -10, y = 10));
// The parameters can be named out of order
writeln(return_tuple(y = -1, x = 3));
~~~
{: .source}

Chapel procedures have many other useful features, however, they are not
essential for learning task and data parallelism, so we refer the
interested readers to the official Chapel documentation.

{% include links.md %}

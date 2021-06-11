---
title: lambda capture clause
date: 2021-06-11 12:03:00 +0200
categories: [c++, basics]
tags: [lambda]     # TAG names should always be lowercase
---
Lambdas have their very own capture clause.  
What is it? How to use it to our advantage? 
Let’s find out!  
Capture clause basically defines which parameters we want to capture by value and which by reference.
So we can use:
[] - empty capture clause to say that we don’t want to capture any outside params.
[&] - to say that we want to capture all outside values we use in lambda by reference by default.
[=] - means we want to make constant copies of the captured values.
[a,b] - we can also pass some parameters explicitly. 
[&a,&b] - or reference to those parameters
[&a, b] - or a combination of both

here is an example:

````c++
#include <iostream>

int main()
{
	int a = 5;
	int b = 7;
//results in compilation error as parameter b is a const copy of b
	std::cout << [a, b] { return b += a; }() << std::endl;
//same here
	std::cout << [=] { return b += a; }() << std::endl;
//b is changed as both a and b are references.
	std::cout << [&] { return b += a; }() << std::endl;
//b is changed again as it is a reference and a is a constant copy.
	std::cout << [a,&b] { return b+= a; }() << std::endl;
}
```

Now that is all good. But sometimes we might not want our copy to be const. It is a copy after all so who cares if it gets changed.
If that is what your heart desires there is a way to make regular copies of passed parameters instead of const ones with a keyword **mutable**

```c++
#include <iostream>

int main()
{
	int a = 5;
	int b = 10;
  std::cout << [a, b]() mutable { return b += a; }() << std::endl;
//prints 15
}
```

impressed? well prepare to be amazed! Let’s consider next example:

```c++
#include <iostream>

int main()
{
	auto func = [count = 0]() mutable { return ++count; };
	for (int i = 0; i < 5; i++) {
		std::cout << func() << ", ";
	}
	std::cout << std::endl;
	//outputs 1,2,3,4,5
}
```

This means that you can actually store some modifiable values inside your lambdas.

Hope you found this article useful.
Until next time.

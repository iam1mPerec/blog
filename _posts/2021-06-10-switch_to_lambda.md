---
title: switch to lambda
date: 2021-06-10 12:03:00 +0200
categories: [c++, basics]
tags: [lambda]     # TAG names should always be lowercase
---

So we all love switch. It is great! You create an enum, make a variable, pass it to switch and get all sorts of different things going.  
Like so:

```c++
#include <iostream>
#include <map>

using namespace std;

enum class eCalculation {
  add,
  subtract,
  multiply,
  divide
};

float calculate(float a, float b, eCalculation calculationType) {
  switch (calculationType) {
    case eCalculation::add:
      return a + b;
    case eCalculation::subtract:
      return a - b;
    case  eCalculation::multiply:
      return a * b;
    case  eCalculation::divide:
      return a / b;
  }
}

int main()
{
  cout << calculate(10, 5, eCalculation::add) << endl;
}
```

simple enough right?

The problem is that the switch only works with integers in c++. What if we need a string, or a char?

Well here is how we can use lambda nested inside a map to solve this:

```c++
#include <iostream>
#include <map>

using namespace std;

int main()
{
  map<char, double(*)(double, double)> operators{
    {'+', [](double a, double b) {return a + b; }},
    {'-', [](double a, double b) {return a - b; }},
    {'*', [](double a, double b) {return a * b; }},
    {'/', [](double a, double b) {return a / b; }}
  };
  cout<<operators['-'](1, 5)<<endl;
}
```
Hope it was informative. There is a lot more to be said about lambda functions,
so I'll try to keep you posted.

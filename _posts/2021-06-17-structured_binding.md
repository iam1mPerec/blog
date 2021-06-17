---
title: structured binding
date: 2021-06-17 8:00:00 +0200
categories: [c++, basics]
tags: [c++]     # TAG names should always be lowercase
---

Binds the specified names to sub objects or elements of the initializer. It’s a great feature of c++17 language.

What does it do and why should you care?

Well to answer this let’s start with an example:

```c++
//let's create a simple map of heroes
//it would contain Heroe's name and current health

std::map<std::string, int> healthMap;
healthMap["Crusader"] = 52;
healthMap["Highwayman"] = 32;
healthMap["Shieldbreaker"] = 30;

//now let's print it out

for (auto [key, value] : healthMap) {
  std::cout << "key: " << key << " value : " << value << std::endl;
}

//prints:
//key: Crusader value : 52
//key: Highwayman value : 32
//key : Shieldbreaker value : 30
```
As you can see we just take a pair and split it into 2 variables. Sure there are other ways to do this, but this one is easy to read and understand (and we can use a bit of that in c++ code). So which bundled structures can we use it with?
Pretty much any:
*Pairs - [first, second]
*Tuples - [first, second, third, forth]
*Custom Objects - [field_1, field_2, field_3]

Now **that** is some powerful staff!

So let’s go through some more examples:

so let’s start with pairs:

```c++
//let’s define a simple division function that returns a division result and a remainder

std::pair<int, int> divide(int a, int b) {
  return { a/b, a%b };
}

```
```c++
//now let’s use this function in int main and print the result:

auto[fraction, remainder] = divide(10, 3);
std::cout << fraction << " : " << remainder << std::endl;
```


for the last example let’s consider an object:

```c++
struct Hero {
  std::string name;
  int hp;
  int lvl;
};
```
now let’s use this Hero struct to create a party and print it to the console:

```c++
std::vector<Hero> party = {
  {"Crusader", 52, 2},
  {"Highwayman", 32, 2},
  {"Shieldbreaker", 30, 1}};

for (auto [name, hp, lvl] : party) {
  std::cout << "Name: " << name;
  std::cout << " Hp: " << hp;
  std::cout << " lvl: " << lvl;
  std::cout << std::endl;
}

//prints:
//Name: Crusader Hp : 52 lvl : 2
//Name : Highwayman Hp : 32 lvl : 2
//Name : Shieldbreaker Hp : 30 lvl : 1
```

Hope that was informative.

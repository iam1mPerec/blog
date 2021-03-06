---
title: observing menu options
date: 2021-06-07 18:03:00 +0200
categories: [olcPixelGameEngine, patterns]
tags: [c++]     # TAG names should always be lowercase
---

So I watched [Javidx9’s](https://github.com/OneLoneCoder) tutorial on creating menus [Programming Menu System](https://www.youtube.com/watch?v=jde1Jq5dF0E&t=2676s)
and I really liked the idea of simplifying Users’ life by adding the ability to add menus quickly like:

```c++ 
menu MENU;
MENU["start"];
MENU["load"];
MENU["load"]["slot 1"]; 
MENU["load"]["slot 2"];
MENU["load"]["slot 3"]; 
MENU["quit"];
```

Now that looks *fancy*! Let’s quickly go over how he achieved this in case you like to read more than you like to watch videos for some reason.

Basically there are 2 containers inside a menu class:

```c++
std::unordered_map <std::string, size_t> m_indexes;
std::vector<menu> m_options;
```

one of them is an unordered_map which has a key of string for option names and a value of size_t for option indexes.
The idea is, that you feed m_indexes a string and it returns you an index by which you can find the menu with that name in m_options.  
Well that is if the menu with such a name exists. If it does not then we just add a new menu item with a given name and give it
a place in our m_options container.

```c++
#include <unordered_map>
#include <vector>
#include <string>

class menu {
  std::string m_name;
  std::unordered_map <std::string, size_t> m_indexes;
  std::vector<menu> m_options;

public:

  menu(std::string name = "root") : m_name(name) {  }

  menu& operator[](const std::string& name) {
    if (m_indexes.count(name) == 0) {
      m_indexes[name] = m_options.size();
      m_options.push_back(menu(name));
    }
    return m_options[m_indexes[name]];
  }
};
```

ok. So now let’s add a few things:

```c++
#pragma once
#include <unordered_map>
#include <vector>
#include <string>

class menu {
  std::string m_name;
  std::unordered_map <std::string, size_t> m_indexes;
  std::vector<menu> m_options;

  int m_id; //would be useful for sending signals
  static int ID; // a unique id for a new option
public:
  //now we to add id to menu constructor:
  menu(int id = 0, std::string name = "root") : m_id(id), m_name(name) {  }

  menu& operator[](const std::string& name) {
    if (m_indexes.count(name) == 0) {
        m_indexes[name] = m_options.size();
        m_options.push_back(menu(++ID, name));
    }
    return m_options[m_indexes[name]];
  }
};

int menu::ID = 0;
```
Now that our menu items have their unique ids we can **do staff and go places**.
So let’s dive into madness.

Say I need to connect these menus to my game.  
Obviously we would need an observer. It should watch for a certain event to occur and then notify our subscriber.  
Here is how I would like it to work:

```c++
menu MENU;
m_menuManager.connect(MENU["option"], eSignals::click,
[&](menu& sender) { this->onOptionClicked(sender); });
```

If this seems confusing, I want to have an observer inside m_menuManager.
then connect a sender **(in this case MENU[“option”])** to signal **(in this case an enum eSignals::click)** to the slot
**(in this case a method onOptionClicked())**;

So menuManager should notify the game that a click event happened and game needs to react by using an onOptionClicked(menu& sender) method.  
onOptionClicked() should contain the sender (it might be useful later on to know who sent the signal)  

So first let’s create an observer:

```c++
#pragma once

#include <map>
#include <functional>

enum class eSignals {
  click,
  check
};

template <typename T>
class observer {
  std::map<eSignals, std::map<int, std::function<void(T&)>>> m_events;
public:
  void connect(int senderID, eSignals signal, std::function<void(T&)> slot);
  void dispatch(T& sender, eSignals signal);
  observer();
};

template <typename T>
observer<T>::observer() {

}

template <typename T>
void observer<T>::dispatch(T& sender, eSignals signal) {
  if (m_events[signal][sender.getID()])      m_events[signal][sender.getID()](sender);
}

template <typename T>
void observer<T>::connect(int senderID, eSignals signal, std::function<void(T&)> slot) {
  m_events[signal][senderID] = slot;
}
```

So let’s quickly go over how it works.

It has a map that holds eSignals enums as keys.  
So if you give it a signal it would return you a map of all sender ids as keys and what they should do whenever this signal is dispatched as a value;

e.g. [eSignals::click] would hold:  
**key:** MENU[“Start”].id **value:** [](){onLoadClicked()}  
**key:** MENU[“Quit”].id  **value:** [](){onQuitClicked()}  
and so on.

This way we can not only define a menu option but also define what the game should do about a signal this menu option sends.

Now all we have left to do is to define the connect method in menuManager class like this:

```c++
const menu& menuManager::connect(const menu& menu, eSignals signal, std::function<void(class menu&)> slot) {
  m_observer.connect(menu.getID(), signal, slot);
  return menu;
}
```
Now whenever menuManager get’s invoked it can do something like this:
```c++
void menuManager::submit() {
  //we check which option is currently selected as menu was clicked
  auto sender = m_menuStack.top()->option(m_currentIndex);
  //if clicked option contains nested menu we proceed to that menu
  //e.g. if [load] is clicked let player 
  // choose between [slot 1], [slot 2] or [slot 3]
  if (m_menuStack.top()->option(m_currentIndex).getItemsCount()) {
    //push selected option to the top
    m_menuStack.push(&m_menuStack.top()->option(m_currentIndex));
    //currently selected item index in this menu is going to be 0
    m_currentIndex = 0;
  }
  //finally dispatch the click action to tell the game that
  //an option was clicked
  m_observer.dispatch(sender, eSignals::click);
}
```

Now we can define both the menu item and how the program should react to its events.

I hope the article was useful and not too confusing.

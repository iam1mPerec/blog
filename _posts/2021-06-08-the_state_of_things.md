---
title: the State of things
date: 2021-06-08 12:03:00 +0200
categories: [olcPixelGameEngine, patterns]
tags: [c++]     # TAG names should always be lowercase
---

Dear Grandma 

So you want to start using 
[olcPixelGameEngine](https://github.com/OneLoneCoder/olcPixelGameEngine) to write your awesome game.
Fair enough. Where would I advise you to start?  
Well how about a state pattern?

So in your game you might have a start menu with options like:  
* start
* load
* quit

very basic staff.

![img-state](/assets/post_assets/state.png)
_State Pattern_

When you press start or load you would not need this menu anymore. All of its logic and menu names are now useless.  
A similar situation would occur if you move between levels. You might need a few details from your previous levels like what
items you now have, what level your hero is and staff like this.   
But those are just elements you need to start creating your new level. You don't want to worry about the things you left behind in your previous level.  
So how do we separate these?

If you have already used olcPixelGameEngine a bit you should know a bit about the Game loop. Game loop has a very basic concept:

* take user input
* update state
* draw frame

Instead of putting all of that into our onUserUpdate method and using switch to figure out which level we are on,
we can just create a GameMode class that would handle user input, update state and even draw frames to the screen.  
Let’s get coding.

As usual I want to first show how it should work:

```c++
class engine : public olc::PixelGameEngine {
    //implementing gameMod ref as a part of the State design pattern
    //gameMode will tell the engine what to draw where and when
    //the engine would handle drawing and keyboard events
    class gameMode* m_mode;
public:
    engine() : m_mode(nullptr) {}
    void begin() {
        if (Construct(900, 600, 1, 1)) {
            //gameMod will handle transfering to another mode
            //that is why it takes engine as a parameter
            m_mode = new GMStartMenu(this);
            Start();
        }
    }
    void setGameMode(class gameMode* newGameMode) {
        //first we check if we already have a game mode
        //and if we do - just delete it. We would not need it anymore at this point
        if (m_mode) {
            delete m_mode;
        }
        //next we set a new game mode that we received
        //please stay tuned to find out where from
        m_mode = newGameMode;
        clearLayers();
        //honestly I did not know what to call this method
        //but DrawFrame basicly draws what needs to be
        //drawn first after new game mode is initialised
        m_mode->DrawFrame(this);
    }
    bool OnUserCreate() override {
        //this is the place where first game mode draws
        //its first frame
        m_mode->DrawFrame(this);
        return true;
    }
    bool OnUserUpdate(float fElapsedTime) override {
        //MARK: - Handle input
        if (GetKey(olc::Key::ENTER).bPressed || GetKey(olc::Key::F).bPressed) m_mode->submit();
        if (GetKey(olc::Key::BACK).bPressed || GetKey(olc::Key::ESCAPE).bPressed || GetKey(olc::Key::E).bPressed) m_mode->quit();
        if (GetKey(olc::Key::UP).bPressed || GetKey(olc::Key::W).bPressed) m_mode->up();
        if (GetKey(olc::Key::DOWN).bPressed || GetKey(olc::Key::S).bPressed) m_mode->down();
        if (GetKey(olc::Key::LEFT).bPressed || GetKey(olc::Key::A).bPressed) m_mode->left();
        if (GetKey(olc::Key::RIGHT).bPressed || GetKey(olc::Key::D).bPressed) m_mode->right();
        //game mods would now which game mods come
        //after them. So if you pressed start in your
        //MainMenu GameMode it will lead you to your
        //Level_1 GameMode or something like this
        //MARK: - Update state
        //MARK: - Draw frame
        //Draw method handles drawing after initial draw (by DrawFrame method)
        m_mode->Draw(this);

        //Here we ask GameMode if we are done with the game yet.
        return m_mode->inProgress();
    }
    ~engine() {
        //since modes are created dynamically
        //always remember to delete them in the end
        //to avoid memory leaks
        delete m_mode;
    }
};
```

so now that we know **what** GameMode should do let’s talk about **how** it should do it:

```c++
class gameMode {
protected:
    //is the game still running? Game mode has an ability to change that
    //at any moment by changing m_inProgress from true to false
    bool m_inProgress;
    //as already mentioned game mode can tell the engine
    //which mode will come after
    //this game mode is done
    engine* m_engine;

public:
    gameMode(engine* engine) :
        m_inProgress(true),
        m_engine(engine) {}
    //does nothing unless redefined by a child game mode
    virtual void DrawFrame() {}
    //needs to be redefined by the child
    //also makes this class abstract so
    //an instance of this class can not be created
    //by users
    virtual void Draw() = 0;
    bool inProgress() const;
    //a bunch of input handlers that can be redefined later
    //or left alone if you want them to do nothing
    virtual void submit() {}
    virtual void quit() {}
    virtual void up() {}
    virtual void down() {}
    virtual void left() {}
    virtual void right() {}
};
```
    
    
Now let us create our first gameMode:

```c++
#pragma once

#include "gameMode.hpp"
#include "menuManager.hpp"
#include "olcPixelGameEngine.h"

class GMStartMenu : public gameMode {
    //creates a basic menu manager that would help us draw menu options
    //if you want to learn more about it
    //I will leave a link to my previous post
    //where I talk about menuManage and signal slot connect pattern
    menuManager m_menuManager;
public:

    GMStartMenu() {
        menu MENU;
        m_menuManager.connect(MENU["start"], eSignals::click, [&](menu& sender) { this->onStartClicked(sender); });
        m_menuManager.connect(MENU["load"], eSignals::click, [&](menu& sender) { this->onLoadClicked(sender); });
        m_menuManager.connect(MENU["quit"], eSignals::click, [&](menu& sender) { this->onQuitClicked(sender); });

        m_menuManager.setMenu(std::move(MENU));
    }

    //here we can draw frames by redefining Draw
    //method from GameMenu class
    void Draw(class engine* engine) override {
        Engine->Clear(olc::BLANK);
        //since in mainMenu GameMode we only have menu
        //there is not much to draw
        //MenuManager handles drawing of the menu
        m_menuManager.draw(Engine);
    }

    //MARK: - slots
//this slots are called when menu options are clicked
    void onStartClicked(class menu& sender) {
        //by creating a new GameMode we are telling
        //game loop that we are ready to delete this
        //mode and move on to the next
        m_engine->setGameMode(new newMode(m_engine));
    }
    void onLoadClicked(class menu& sender) {
        //here is what's cool. Before current GameMode dies
        //it can prepare the new GameMode by
        //giving it parameters. In this case we can
        //load everything we need from the file
        //like what hat our hero was wearing
        //also we can even load a different GM
        //if we saved on say lvl 3, we can create
        //game mode lvl 3 
        auto newMode = new GMLevel_3();
        newMode.setHat(eHats::strawHat);
        m_engine->setGameMode(new GMLevel_1(m_engine));
    }
    void onQuitClicked(class menu& sender) {
        //as previously mentioned just setting this to false
        //will tell our game loop that we want to quit
        m_inProgress = false;
    }

    //here we redefine input handlers
    //notice that we did not redefine left() right()
    //handlers since we don't need them 


//MARK: - User input
    void submit() override;
    void quit() override;
    void up() override;
    void down() override;
};
```

You can read more about MenuManager and **signal-slot-connect** pattern in my previous [post](https://iam1mperec.github.io/blog/posts/options_observer/).

I hope this article was not too complicated.

Sincerely yours

Moonpie

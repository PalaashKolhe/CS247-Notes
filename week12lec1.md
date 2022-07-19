# Week 12 Lecture 1

## Singleton Design Pattern
- is an anti-pattern
- Typically a **bad idea** to use it. It indicates a flawed design

**Problem** it solves: When you only ever want *one* of an object in the entire program. 

This rarely ever happens.

**Intended use case**: When the class manages a piece of hardware we know we will only have one of

**Actual use case**: We want a bunch of global data but don't want it to look like that

### Bad example (Don't do this)
We are writing a game and you think you should only ever have 1 of these `settings` objects. So we use the singleton pattern.

```cpp
class Settings {
    class settingsInpl {
        int w, h;
        ...
    }  

    static unique_ptr<settingsInpl> *p;

   public:

    Settings() {
        if (p.get() == nullptr) {
            p = make_unique<SettingsInpl>();
        }
    }
};

unique_ptr<settingsInpl> Settings::p = nullptr;

class Character {
    ...
   public:
    void draw() {
        Settings s;
        int w = s.getW();
        int h = s.getH();
    }
}

int main() {
    Settings s;
    // first time so Settings p is setup
}
```

### Alternatively
```cpp
class Singleton {
    int w, h;
    ...
    // only one singleton object ever made, and we return data for this object
    static Singleton *instance;
    // then all ctors are private
   public: 
    static getInstance() {
        if (!instance) {
            instance = new Singleton(...);
        }

        return instance;
    }
}

int main() {
    Singleton *p = Singleton::getInstance();
}
```

## Curiously Recurring Template Pattern (CRTP) Design Pattern

**Problem**: You want dynamic binding (i.e. virtual dispatch), but you want it without all the pesky virtual overhead - We want "virtual" dispatch at compiler time. 

- Design pattern doesn't work with polymorphism

### Pros
Shares common interfaces with different implementations of those interfaces.

### Cons
No polymorphism (eg. collection of **abstract base class** pointers)

```cpp
template <typename Derived>
class Base {
   public:
    void doSomething() {
        // interface
        static_cast<Derived *>(this)->doSomethingImpl();
    }
}

class DerivedOne : public Base<DerivedOne> {
   public:
    void doSomethingImpl() {
        ...
    }
}

class DerivedTwo : public Base<DerivedTwo> {
   public:
    void doSomethingImpl() {
        ...
    }
}
```

What if we do something like this?
```cpp
class DerivedThree : public Base<DerivedOne> {
   public:
    void doSomethingImpl() {
        ...
    }
}
```

```cpp
int main() {
    DerivedThree d;
    d.doSomething();
    // what does d.doSomething() do?
}
```

This is an error because `d.doSomething();` ultimately calls `DerivedOne::doSomethingImpl()` on `d`, but `d` is not a `DerivedOne`. This is undefined behaviour. 

**Observation**: Derived classes must instantiate their base class components.

**Solution**: Make all ctors of `Base` private and declare the template param class as a friend!

```cpp
template <typename Derived>
class Base {
    // make all ctors private
   public:
    void doSomething() {
        // interface
        static_cast<Derived *>(this)->doSomethingImpl();
    }
    friend class Derived;
}
```

Now we will get a compilation error since all ctors to `DerivedOne` are private to `DerivedThree`. 

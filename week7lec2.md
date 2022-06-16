# Week 7 Lecture 2

## What to do when things go wrong?

```cpp
Node &operator=(const Node &other) {
    if (this == &other) return *this;

    delete next;
    data = other.data;
    next = other.next ? new Node {*other.next} : nullptr;
    return *this;
}
```

`new` can detect the problem, but it doesn't know what to do about the problem. On the other hand, the client knows what to do, but they can't detect the error. The solution to this is **error handling**.

## Error Handling
- Inherently a non local problem
  
### Solution in C
The functions return a status node, or set a global variable. But it leads to awkward programming since you lose your train of thought.

### Solution in C++
When an error condition arises, the function raises **an exception**. 

By default, execution stops.

**OR**

We write handlers to catch the exceptions and deal with the problem. 

## Back to our topic for what to do when new fails
When new fails, it throws an expressiong of type `std::bad_alloc`. 

***Example***: vectors.
- Out of bounds index
  - `V[i]` such that `i` is out of bounds. This doesn't raise an exception since this access is unchecked. Simply undefined behaviour as it is with arrays. **Undefined Behaviour**.
  - `V.at(i)` is checked, so if `i` is out of range then it raises a `std::out_of_range` exception. 
    - This is fairly costly since we have a check.


## Writing an exception handler (Out of bounds index example)

```cpp
#include<stdexcept>
...
    // statement may raise an exception
    try {
        // out of bounds
        cout << v.at(10000) << endl; 
    } except (out_of_range r) {
        cerr << "Range Error" << r.what() << endl;
    }
```

Now consider **exception thrower**:
```cpp
void f() {
    // the message can be entirely arbitrary
    throw out_of_range{"f"};
}

void g() { f(); }

void h() { g(); }

int main() {
    try {
        h();
    } catch (out_of_range r){
        ...
    }
}
```

**Error handling** is a non-local problem. The problem occurs in `f`, but the function that deals with that error is not `g()` or `h()`, it is `main`. 

***So what happens?***

1. `main` calls `h`
2. `h` calls `g`
3. `g` calls `f`
4. `f` throws.

Control goes back through the call chain (***unwinding the stack***) until a handler is found. In this case all the way back to `main`. `main` handles the exception. 

If there is no handler, then the program terminates.

A handler can do part of the recovery job, i.e. execute some corrective code, or throw another error.

```cpp
try { ... }
except (someErrorType s) {
    ...
    throw someOtherError{...};
}
```

Or rethrow the same error -
```cpp
try { ... }
except (someErrorType s) {
    ...
    // throws the same exception
    throw;
}
```

Why `throw;` and not `throw s;`?

*Soln*: What if you have a derived error? For example, `specialErrorType` is a derived error type from `someErrorType`. Technically it is `someErrorType`, but it isn't correct to return `someErrorType` when the actual class is `specialErrorType`. `throw` is **preferred** over `throw s;`.

## Universal exception
In **C++**, there isn't a top level exception like there is **Java**. So there isn't a guaranteed universal exception that can be used.

A handler can act as a `catch all`
```cpp
try{ ... }
catch(...) { // acc use ... this is real syntax. ... means arbitrary number of params
    throw;
}
```

You can throw anything you want, for eg. `int`. But generally, define your own classes (or use appropriate existing ones) for errors.

```cpp
class BadInput {};
try {
    int n;
    if (!(cin >> n)) throw BadInput{};
} catch (BadInput &) {
    cerr << "Input not well-formed\n";
}
```

**Throwing by value, and catching by reference is more efficient.** (saves time copying)

*Also:*

```cpp
class BaseExc {};

class DerivedExc : public BaseExc {};

void f() {
    DerivedExc d;
    throw d;
}

int main() {
    try { f(); } 
    catch ( BaseException & ) {
        cout << "Base";
    } catch ( DerivedException & ) {
        cout << "Derived";
    }
}
```

Prints base

Since `BaseException` is the parent class, the `catch ( BaseException & )` will always evaluate to true. `catch ( DerivedException & )` never runs.

Consider the following:

```cpp
void h() {
    DerivedExc d;
    BaseExc &b = d;
    throw b;
}

int main() {
    try { h() } { }
    catch(DerivedExc &) {
        cout << "Derived";
    } catch (BaseExc &) {
        cout << "Base";
    }
}
```

Prints base

**Exception handlers** chosen based on the **static** type of the data thrown. i.e. Base ex. (type of ref);

**WARNING: NEVER** let a dtor throw an error!

## Why not to throw an error in a dtor
If an exception is raised, dtors run during stack unwinding. If a dtor throws, then you have an infinite loop. 2 exceptions cannot run at the same time.

If one of these throws, now have 2 active exceptions looking for a handler. The program WILL stop immediately.




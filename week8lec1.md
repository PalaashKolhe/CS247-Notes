# PLMPL Idiom, Exceptions, Exception Safety, RAII, and Unique Pointers

## Pointer to Implementation Idiom
### Header Library visibility
```cpp
// Screen.h
class Screen {
    SDL_Surface *screen;
    int w, h;
    int numRects;
    SDL_Rect **rects;

    public:
        ...
}
```

As a client, we can see all of the fields (`screen`, `w`, `h`, etc.). Doesn't actually matter what these fields are, since we just use the public intergace. However, if these fields change, For eg. rects becomes a vector of pointers), then all client code needs to recompile (Compilation dependencies).

The client doesn't care about the private field details, so we **don't want to recompile** everytime internals are changed. 

## Solution: Implementation Class
We can create an **implementation class** that stores the internal fields, and point to it in the class that is accessible by the client!
- Only change we make is initilaization of objects and other methods
- Refer to fields as `pImpl->FieldName`
- Or we can move methods to the implementation class and call them

```cpp
// ScreenImpl.h
struct ScreenImpl {
    SDL_Surface *screen;
    int w, h;
    int numRects;
    SDL_Rect **rects;
    ...
}
```

```cpp
// Screen.h
class Screen {
    ScreenImpl* pImpl;
    public:
        ...
}
```

```cpp
// Screen.cc
Screen::Screen(....): pImpl{new ScreenImpl(....)} {}

void drawRect(int x, int y, int w, int h, colour c) {
    // before ScreenImp;
    if (numRects == 10) return;
    rect[numRects++] == SDL_Rect(....);
    ...

    // After ScreenImpl
    if (pImpl->numRects == 10) return;
    rect[pImpl->numRects++] = SDL_Rect(....);
    ...
}
```

Now we can make changes to `Screen.cc` and `ScreenImpl.h` without the clients having to recompile each time. 

## Back to exceptions
### Leaky exception throwing
```cpp
int** Foo(int x) {
    int** arr = new int*[x];
    for (int i=0; i<x; i++) {
        arr[i] = new int{i};
        arr[i] = g(arr[i]);
    }
    return arr;
}

int main() {
    int **i = foo(100);
    for (int i=0; i<100; i++) delete x[i];
}
```
If `new` throw (fails) or `g` throws, then we leak memory. If `main` is calling `Foo` then it is fine, but if another function is calling it and it catches it, we've leaked everything we have already allocated.

For example, if we fail at `i=50`, this program leaks 49 pointers.

## Catching the leak (Attempt 1) (Try/Catch)
This can still at most one int. If `new int{i}` doesn't fail, but `g` fails, then we neglect to the delete the `i`'th int. 

```cpp
int** Foo(int x) {
    int** arr = new int*[x];
    for (int i=0; i<x; i++) {
        try {
            arr[i] = new int{i};
            arr[i] = g(*arr[i]);
        } catch(...) {
            for (int j=0; j<i; j++) {
                delete arr[j];
            }
            delete [] arr;
            throw;
        }
    }
    return arr;
}
```

## Catching the leak (Attempt 2) (Double Try/Catch)
We can try to fix attempt by writing 2 try/catches. One for `new` and one for `g`.
- `new` catch: deletes up to `i`
- `g` catch: deletes up to and including `i`

Problem: It's ugly and error prone

## Catching the leak (Attempt 3) (Creating a new class)
Implement a class called `IntPtr` to manage memory. When `emplace)back(new IntPtr())` fails or `g` fails, the stack unwinds. Then `v` unwinds, and its destructor deletes every `IntPtr` object. 

Now there are no leaks, however, it's **not practical** to write a class for every pointer you want to store.

## Catching the leak (Attempt 4) (unique_ptr) 
Use `unique_ptrs`. There is no leak and we don't have to write a new class.

```cpp
#include <memory>

vector<unique_ptr<int>> Foo(int x) {
    vector<unique_ptr<int>> v;
    for (int i=0; i<x; i++) {
        v.emplace_back(unique_ptr<int>{nex int {i}});
        v[i] = g(*v[i]);
    }
    return v;
}
```
## RAII Idiom - Resource Acquisition is Initialization
Some languages have a `finally` clause. This code is **guaranteed to run**, no matter how a function exits. C++ has no such feature. However, C++ guarantees that the stack is unwound and the destructors are run. The destructors of stack allocated objects are our `finally` clause. 

**Definition:** Resources should only ever be acquired through the **initialization of a stack-based object** whose job it is to **manage** it. 

In practical terms, the main priciple of RAII is to give **ownership** of any heap-allocated resource, for example, dynamically allocation memory or system object handles, to a **stack allocated object** whose destructor contains the code to delete or free the resource and also any associated cleanup code. 

## Exception Safe Code
RAII helps us write **exception safe code**. 3 levels of safety offered -

1. **Basic Guarantee** - If exception is thrown or propagated by this function, then the program will be left in a valid but unspecified state.
   1. Valid State - won't leak any resources. No class invariants are violated (imp. for member functions)
   2. Unspecified - not sure of the details of the state (globals changed, things printed, etc.)
2. **Strong Guarantee** - *Basic Guarantee* + the state has not changed (nothing printed, globals left as is)
3. **No Throw Guarantee** - Exception is never thrown

A function can also offer **no safety**. `vector::emplace_back` and `push_back` **both** offer strong guarantee.

## Intro to Unique Ptr
`unique_ptr` is a class in C++ std lib for managing pointers to data. To use it, we `include <memory>`. Each pointer is unique - it believes it is the only holder of that data, so its data **cannot be shared**.

## Smart Pointer Idiom
We want to follow RAII. Raw pointer owndership is confusing and leaks memory if mismanaged. 

```cpp
int main() {
    int *p = new int {10}; // doesn't follow RAII
    if (....) {
        unique_ptr<int> x{p};
    } // when x goes out of scope, the following code is a mem error
    *p = 50; // p is already freed
}
```
Ideally, we shouldn't say `new` at all. Instead we can use `make_unique` to allocate data for us.

```cpp
unique_ptr<int> p = make_unique<int>(10);
// make unique similar to emplace_back
// its paramters are jsut the parameters of the obejct you want to create
```

## Mixing smart pointers with unique pointers
```cpp
void Foo(unique_ptr<int> p) {
    ...
}
unique_ptr<int> q = make_unique<int>(10);
Foo(q); // does not work
```

Doesn't correctly mutate the data, because `p` is a shallow copy. We will instead get 2 unique pointers to the same data, so there will be a **double free**. **Deep copy in this case is wrong** because then changing the value in the functions won't change the value outside the function. This is why `unique_ptr` does not have a copy constructor. 

## .get method
`.get` is a method on `unique_ptr`'s that return the **raw pointer**. We should never delete it or use it after it goes out of scope. The raw pointers life cycle should be less that the life cycle of the `unique_ptr`. 

```cpp
void Foo(int* p) {
    ...
}

unique_ptr<int> q = make_unique<int>(10); 
foo(q.get());
```
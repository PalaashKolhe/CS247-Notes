# Week 2 Lecture 2

## R-values
- Temporaries, typically the result of a function or operator can't return by value
- May noy have memory (eg. may exist only in a register)
- Can't be assigned to
- Variables local to a functions stack frame become r-values when that function returns

## L-values 
- Non temporary, have some lifetime beyond a single expression
- Have memory and thus an address
- Can be assigned, unless they're `const`

### Consider 
```C++
int Foo(int &x) {
  x += 1;
  return x;
}

int main() {
  Foo(5);
  // what does this mean?
  // Not valid, this is a compilation error
}
```

But if this is changed to this
```C++
int bar(int &y) {
  return y+1;
}

int main() {
  cout << bar(5) << endl;
  // still invalid!!
}
```

**The above technically means that I am doing 5=5+1**


We can fix this by instead making bar take in a const reference
```C++
int bar(const int &y) {
  return y+1;
}

int main() {
  cout << bar(5) << endl;
  // NOW VALID AND COMPILES!!!
  // Compiler will make a temporary location for 5, if it didn't already have one, and let y refer to that
}
```

`int&` is an **l-value** reference (a ref to an **l-value**) - if a param is a `const` **l-value** reference - we can validly accept **r-values** as arguments.

`int&&` is a **r-value** reference (a ref to an **r-value**), only **r-value** args.

```cpp
// x and y are nodes that are lists
Node z = x + y;

// the list from operator+ 's stack frame is copied out to the caller stack to a temporary. That temporary is then copied into z, two copies of two lists that are just going to be destroyed anyways. 

// Very inefficient ^^ 
```

Wasteful to spend all this time copying something thats going to be destroyed anyways. So, we can write a new **constructor** and another **overload** for the assignment operator that operate on **r-values** instead.

These are called the *move constructor* and the *move assignment operator*.

**Note: &o means l-value, &&o means r-value**

```cpp
struct Node {
  // as before

  // just take other's data. if it is dying, then why copy it manually? just reassign it

  // MOVE CONSTRUCTOR
  Node(Node &&o) : data{o.data}, next{o.next} {
    // we need to prevent o from dying

    // we set o.next to something other than the list that we need
    o.next = nullptr
  }

  // MOVE ASSIGNMENT OPERATOR (take 1)
  Node &operator=(Node &&o) {
    // delete my next, and then set my next to o's next
    // perform a switcharoo
    delete next;
    next = o.next;
    o.next = nullptr;
    data = o.data;
    return *this;
  }

  // MOVE ASSIGNMENT OPERATOR (take 2)
  Node &operator=(Node &&o) {
    // perform a switcharoo
    // my old data dies along with the automatically called destructor for o
    swap(o);
    return *this;
  }
}
```

**Elision**: Happens when compiler automatically optimizes away the copy/move constructor. To get rid of this, you type in the compile code to not optimize away the operator/constructors.

```cpp
int main() {
  Node *p = createList(10);
  Node *q = createList(5);
  Node z{*p}; // copy ctor
  Node y{*p + *q} // move ctor OR none (compiler optimization), THEN move assign operator
}
```

Copy ctor above runs 10 times\
Move runs once\
MAO runs once

*CONSTRUCTORS* **CAN** be elided, *ASSIGNMENT OPERATORS* **CANNOT** be elided

**Note:** In cases like ours, For eg.
```cpp
  // x and y are nodes
  Node n = x + y;
  // The compiler is allowed to elide these ctors, operator to instead build the return value directly in it's memory!
```

It's allowed to do so even if the constructor has side effects.

## Our Node class is terrible
It has no encapsulation (no private). Our classes have invariants. 

*Invariant* - a statement that must always hold true

Without being able to assume invariants, code can become very hard/impossible to write. In fact, we used an invariant in our class, but because we have no encapsulation, we have no guarantee it actually holds. We assume `next` is always either a valid heap allocated node or the nullptr. Consider:

```cpp
struct Node {
  // as before
  ~Node() {delete next}
}
```

But because we're poorly encapsulated, the client can write code like this

```cpp
int main() {
  Node a{3, nullptr};
  Node b{2, &a};
  Node c{1, &b};
  // Our invariant no longer holds
  // When b and c go out of scope, they will try to delete a and b respectively. They will try to call delete on portions of the stack.
}
```
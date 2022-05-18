# Week 3 Lecture 1

From last class, we found a problem. Our node class relied on the invariant. The next pointer is always either
- a valid heap allocated node
- or, the nullptr


This is necessary because our destructor is 
```cpp
Node::~node() {
  delete next;
}
```

Problem: we can't currently maintain this invariant

```cpp
  Node a{3, nullptr};
  Node b{2, &a};
```

b's destructor tries to delete a, but a is on the stack so it is already deleted. We get a **double free error**.

Solution: Properly ***encapsulate*** our class so that the client *can't* violate our invariant. 

Recall: An encapsulated class should be trated like a black box, an **ADT** where all the client cares about is the abstract data type (in this case the **list**), NOT the implementation. (In this case the *linked list*).

So, we make a new class `list` that the client uses that **encapsulates** the implementation. 

Our node class already has the big 5 
1) copy ctor
2) copy assignment operator (CAO)
3) move ctor
4) move assignment operator (MAO)
5) dtor

So our list class will use that

### Copy constructor
```cpp
class List {
  struct Node; // private nested class
  Node *head;

  Public:
    List(): head{nullptr} {};
    List(const List &o): head{o.head?
      new Node{*o.head} : nullptr} {}
}

```

### Copy assignment Operator
```cpp
List &operator=(const List &o) {
  *head = *o.head;
  return *this;
}
```

### Move constructor
```cpp
List(List &&o) : head {o.head} {
  o.head = nullptr;
}
```

### Move assignment Operator
```cpp
List &operator=(List &&o) {
  *head = std::move(*o.head);
  return *this;

  // std move is a function in the utility header. it forces a value to be treated like an r value

  // could use swap, but using move means we use the MAO we made for node
}
```

### Destructor
```cpp
~List() {
  void cons (int n) {
    head = new Node {n, head};
  }
}
```

Great, our client can create lists how do they use them?

They need to be able to access the *individual* data (each element), so we must provide a means to do so. 

```cpp
Class List {
  // as before

  int &ith(int i) {
    Node *p = head;
    while (i && p) {
      --i;
      p = p->next;
    }
    return p->data; // assume i is in the list
  }
}

int main() {
  List l{};
  l.cons(3);
  l.cons(2);
  l.cons(1);

  for (int i=0; i<3; ++i) {
    ++(l.ith(i));
    cout << l.ith(i) << endl;
  } // works, but run time is O(n^2)
}
```

But now we are looping over the list at O(n^2) because ith is O(n). But we can't just return nodes to our client since that breaks the encapsulation.

We can instead return an object that encapsulates the idea of a ptr to our list. This is called an ***iterator***.

## Iterator Design Pattern
```cpp
class List {
  // as before
  Public: 

    class Iterator {
      Node *p;
      Public:
        int &operator*() {return p->data;}
        Iterator &operator++() {
          if (p) p = p-> next;
          return *this;
        }

        // Note: to implement postfix operator ++, you must include a second token int param Iterator operator++(int) {}

        Iterator operator++(int) {
          Iterator it{pt};
          if (p) p = p->next;
          return it;
        }

        // We don't need postfix though

        Iterator (Node *p) : p{p} {}
        bool operator!=(const Iterator &o) {
          return p != o.p;
        }
    }
}
```

Need to define a function that returns an iterator by value, and a function that returns an iterator named n
```cpp
Iterator begin() {
  return Iterator{head};
}
Iterator end() {
  return Iterator{nullptr};
}
```

Final solution for our client to use
```cpp
int main () {
  List l{};
  l.cons(3);
  l.cons(2);
  l.cons(1);

  for (List::Iterator it=l.begin(), it != l.end(), ++it) {
    cout << *it << endl;
    // So runtime is now O(n)
  }
}
```

Saying `List::Iterator` is cumbersome. The compiler knows what type `List::begin` returns, so why don't we tell the compiler to figure out the type. We use `auto`.

```cpp
  for (auto it=l.begin(), it != l.end(), ++it) {
    cout << *it << endl;
    // auto tells compiler to deduce type based on initialization of a variable
  }
```

Can't always use `auto`, can only use it when the compiler can definitively deduce. But, we shouldn't use auto all the time. The compiler may know what every function returns, but the reader might not know. ***auto reduces readability***. 

*But is is fine for iterators*.

This is the iterator design pattern, it solves the problem of "how do I allow iteration over my container class efficiently without breaking encapsulation?". 

This is the essence of a design pattern. 

**Design Pattern-** If you have a problem like x, then maybe this is a good solution. 

The iterator pattern is so pervasive to C++ programming, that there is even specialized syntax for it. 

```cpp
for (auto it = l.begin(); it != l.end(); ++it) {
  cout << *it << endl;>>>>
}

for (auto x:l) {
  // this is a range based for loop, and x is an int or whatever the container contains.
  cout << x << endl;
}
```

But there is a caveat
```cpp
List l; l.cons(3); l.cons(2); l.cons(1);
for (auto x: l) {++x;}
for (auto x: l) {cout << x << endl;}

// prints 1,2,3. Why????
```

Variable x is a local copy of the value returned by dereferencing each iterator


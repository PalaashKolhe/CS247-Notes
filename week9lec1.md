# Week 9 Lecture 1

## Templates
What is a template? A **template type** is a type that is *parameterized* with a type. For example:
- `vector<int> v;` <int> is the template
- `vector<char> c;` <char> is the template

Example:
```cpp
class List {
    struct Node {
        int data;
        Node *next;
    }
    public: ...
};
```

`Node`'s store `int`'s. What if we want a list that stores other things? We declare list as a **template type**.

```cpp
template <typename T>
class List {
    struct Node {
        T data;
        Node *next;
    }
    Node *head;

    public:
        // iterator
        class Iterator {
            Node *p;
            explicit Iterator (Node *p) : p{p} {}

            public:
                Iterator& operator++() { p = p->next; return *this; }
                bool operator!= (const Iterator &o) { return p = o.p; }
                T& operator*() {return p->data; }       
        }
        void cons (T x) {
            head = new Node{x, head};
        }
}
```

Client Code
```cpp
List<int> l1;
l1.cons(3);
l1.cons(4);
List<char> l2;
l2.cons('a');
l2.cons('&');
List<List<int>> l3;
l3.cons(l1);
for (auto &l : l3) {
    for (auto &n : l) {
        cout << n << " ";
    }
    cout << endl;
}
```

## Drawbacks of Templates
Templated classes **must** have their method implementations in the **header file**. Why? A templated class is really defining a **family** of types. The compiler generates specific forms of this class for each way the template class is specialized. Valid template specializations/instantiations are whatever matches the way your template class uses the type!

```cpp
template<typename T>
class Foo {
    T x;
    public:
        Foo operator+(const Foo &o) {
            return Foo {x + o.x} // T now requires + operator
        }
        T getX() { return x; } // T now requires the copy constructor
}
```
Which types are valid substitutes for T? Anything that can be added or copied.

**Note**: Some classes not copyable! Consider unique pointers for instance. `Foo<unique_ptr<int>> f;`. How does `unique_ptr` prevent us from copying? It declares the relevant methods as `delete`:

```cpp
unique_ptr(const unique_ptr<T> &o) = delete; // no copy ctor
unique_ptr<T> &operator= (const unique_ptr<T>&) = delete; // no copy assignment
```

## What if we ant specialized behaviour of a template for a certain type?
```cpp
template<>
class Foo<int *> {
    int *x;
    public:
        Foo(int *x): x{x} {}
        Foo oeprator+(const Foo<int*> &o) {
            return Foo{new int{*x + *o.x}};
        }
        ~Foo() { delete x; }
}
```

This makes `Foo` add and delete differently when it is initialized with an `int*` type.

**Note:** This is not inheritance. If we didn't write `operator+` in the `int*` specialization, then it would not contain one. Every template specialization is a whole different class.





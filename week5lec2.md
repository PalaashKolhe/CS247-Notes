# Week 5 Lecture 2

The inclusion of virtual functions made our objects 8 bytes (size of a pointer) larger. That is because at the beginning of the first 8 bytes of these objects, the compiler is storing a pointer called the `vpointer` or the **virtual table ptr** . 

It points at the *virtual function table/vtable* for the class that object's type. 

For ex.

```cpp
class A {
    int x;
    public:
        virtual void hello() {
            cout << "I'm an A";
        }
}

class B : public A {
    int y;
    public:
        void hello() {
            cout << "I'm a B";
        }
}

int main() {
    A a1{5};
    A a2{3};
    B b1{1,3};
    B b2{7,8};
}
```

![week5lec2part1.jpeg](1.jpeg)

**Warning:** Never use arrays of objects polymorphically. If we want to use arrays, it needs to be pointers to objects.

For eg.

```cpp
class One {
    int a,b;
    public:
        One(int a, int b) : a{a}, b{b} {}
}

class Two : public One {
    int c;
    public:
        Two(int a, int b, int c) : One{a, b}, c{c} {}
}

void Foo(One arr[]) {
    arr[0] = {7,8};
    arr[1] = {9,10};
}

int main() {
    Two arr[2];
    arr[0] = {1,2,3};
    arr[1] = {4,5,6};
    foo(arr);
}
```

## New hierarchy

```cpp
class Book {
    protected:
        int pages;
        string title, author;

    public:
        Book(int pages, string title, string author) : pages{pages}, title{title}, author{author} {}

        virtual bool isHeavy() {
            return pages > 200;
        }
}

class Comic : public Book {
    string hero;

    public:
        Comic(int pages, string title, string author, string hero) : Book{pages, title, author}, hero{hero} {}

        // this is us telling the compiler that we are trying to override the function in its parent class

        // it catches small errors such as spelling mistakes, even paramters too. It asks compiler to double check for us

        bool isHeavy() override {
            return pages > 50;
        }
}
```

**Quick Note-** The override keyword does not change the behaviour of your program. It jsut asks the compiler to check *"is this a valid override"*. i.e. check that a matching virtual function exists in a parent class. It

It is good practise to use override

Continuing the example from above
```cpp
class TextB : public Book {
    string topic;
    public:
        Text(int pages, string title, string author, string topic) : Book{pages, title, author}, topic{topic} {}

        bool isHeavy() override {
            return pages > 500;
        }
}
```

Now can make a library 

```cpp
void isLibraryHeavy(Book **library, int size) {
    for (int i=0; i<size; ++i) {
        cout << library[i]->getTitle() << "is heavy" << library[i]->isHeavy() << endl;
    }
}


int main() {
    Book **library = new Book*[5];
    library[0] = new Book {453, "Spin", "Robert"};
    library[1] = new Comic {57, "Spider-man", "Stan Lee", "Morbius"};
    library[2] = new TextB {5373, "C++ the language", "old guy", "C++"};
    isLibraryHeavy(library, 3);
}
```

## Copy Assignment Operator

What if do this

```cpp
void copyFirstBook (Book **lib, int size) {
    for (int i=0; i<size; ++i) {
        *lib[i] = *lib[0];
    }
}
```

This invokes the ***Copy Assignment Operator***, but CAO is not virtual. So this calls `Book`'s assignment operator and it only assigns the `Book` component of these objects. So only `pages, title, and author` get copied over. This is **partial assignment**. 





# Week 3 Lecture 2

## Encapsulation Issues
Iterator's ctor is public. The client can make their own iterator, but should only be able to through begin and end. 

### Solution - Make ctor private

```cpp
class List {
    ...
    public {
        class Iterator {
            Iterator (Node * cur) ... {}
            Public:
                friend class List;
                // can go anywhere in class Iterator
        }
    }
}
```
BUT now List can't access the ctor, **sol'n** is to declare List a friend.

## Friend

Through implementing as a friend, List can access Iterator's private members, but not vice versa. 

List is a friend of iterator, but iterator is not a friend of List.

In general, try to have as few friends as possible. It weakens encapsulation.

You can also declare functions as friends. This is particularly useful for two operators that should not be member functions.

Recall: When operators are overloaded as member functions, the first operand is always the object pointed to by `this`. We don't always want the first operand to be the type of our object. 

For example - 
```cpp
class Pair {
    int x, y;
    public:
        Pair(int x, int y) : x{x}, y{y} {}
        istream &operator>>(istream &in) {
            return in >> x >> y;
        }

        ostream &operator<<(ostream &out) {
            out << "(" << x << ", " << y << ")";
        }
}
```

These ops are member functions, the first operand is then a `Pair` so 
```cpp
Pair p{1,2};
cout << p << endl; // THIS IS NOT WHAT OUR FUNCTION DOES
cin >> p; // THIS IS NOT WHAT OUR FUNCTION DOES
```

Our function is defined as 
```cpp
Pair p{1,2};
(p << cout) << endl;
p >> cin;
```

It works, but this is weird

## Operator overloading for cin and cout
**Solution**: Declare a friend

```cpp
// Header File
class Pair {
    ...
    public:
        ...
        // header file
        friend istream &operator>>(istream&, Pair&);
        // output operator is similiar
}
```

```cpp
// Implementation file
istream &operator>> (istream &in, Pair &p) {
    return in >> p.x >> p.y;
}
```

## Separate Compilation
```cpp
// student.h
class Student {
    const int ID;
    std::string name;

    public :
        Student(int id, std::string name);
}
```
```cpp
// class.h
#include "student.h"
class Course {
    int numStudents;
    Student *students;
    int enrolled;

    public:
        Course(int numStudents);
        void enroll(const Student &s);
}
```
```cpp
// student.cc
#include <string>
#include "student.h"
using namespace std;

Student::Student(int id, string name) : id{id}, name{name} {}



```

```cpp
// course.cc
#include <string>
#include "student.h"

Course::Course(int n) : numStudents{n}, students{new *Student[numStudents]}, enrolled{0} {}

void Course::enroll(const student &s) {
    students[enrolled] = new Student{s};
    enrolled++;
}
```

```cpp
// main.cc
int main() {
    Course cs247{140};
    Student s1{0, "Albert"};

    cs247.enroll(s1);
}
```

To compile, lets do `g++ -std=c++14 student.cc course.cc main.cc -o main`. This compiles and links all 3 files and creates a `main` executable.

### What if you only change 1 file?

If a change is made to `student.cc`, now to rebuild the entire program you have to recompile all of the files. This is **wasteful**. Compilation can take a long time.

A better option is to compile files separately into object files (Compiled binary files that are not complete programs). Then, link the final executable.

Eg.\
```g++ -std=c++14 -c main.cc```

The `-c` flag means compile only, don't link. This will create `main.o` here, an object file.

```g++ -std=c++14 -c student.cc```
```g++ -std=c++14 -c course.cc```
```g++ main.o student.o course.o```

If we change a file, we only need to recompile files that depend on it. For eg.

```cpp
// change student.cc only needed
g++ -std=c++14 -c student.cc
g++ main.o student.o course.o
```

For eg. We change student.h, we need to do
```
g++ -std=c++14 -c main.cc
g++ -std=c++14 -c student.cc
g++ -std=c++14 -c course.cc

g++ main.o student.o course.o -o main
```
 However,  this is not user friendly.



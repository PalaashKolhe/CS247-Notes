# Week 4 Lecture 2

Continuing from last class,

```cpp
// student.h
#include "course.h"

class Student {
    const int ID;
    std::string name;
    Course **arr; // courses we're in, capped at 5
    public:
        ...
        void enroll(Course *);
};
```

```cpp
// course.h
#include "student.h"
class Course {
    std::string name;
    Student** enrolled;
    const int capacity;
    int numEnrolled;

    Public:
        Course(std::string name, int capacity);
        void enroll(Student *);
}
```

```cpp
// course.cc

#include "course.h"
#include "student.h"
#include <string>

Course::Course(string name, int maxSize) : capacity{maxSize}, name{name}, enrolled{new Student* [capacity]}, numEnrolled{0} {} // THIS IS BUGGY SINCE new Student* [capacity], CAPACITY IS NOT INITIALIZED YET
```

Regardless of **MIL** order, fields are intialized in the order they were declared in the type. So, `enrolled`'s initialization occurs **before** `capacity`'s. So `new Student*[capacity]` we don't know what capacity is.

Solution:
- initialized enrolled as `new Student* [maxSize]`
- OR
- switch order of declaration in class

Compiler will warn you if your MIL order doesnt match declaration order.

`static` should be used if there is a constant variable for a class. It is defined at the class level, rather than the object of the class level. 




***Problem -*** we try to compile `student.cc` (or `course.cc` or `main.cc`) and we get an error. `student.cc` includes `course.h` AND `student.h`. 

`student.h` includes `course.h`, AND `course.h` includes `student.h`.

This is a **cyclical include**.

But in `student.h`, the *header guard* is defined. So when `student.h` includes `course.h`, the contents of `course.h` are pasted into `student.h`. But, since `student.h`'s header guard flag is already defined, `course.h`'s include of `student.h` comes back empty. So, the pasted copy of `course.h` refers to **type student**, but doesn't know it exists yet.

## Do our header's really need to include each other?

Consider -

```cpp
// student.h

class Course; // this fixes it
class Student {
    const int ID;
    std::string name;
    Course **courses;
    public:

}
```

Size of Student reles on the size of an `int`, a `string`, and a `ptr`. We don't need to know the size of `Course` here. There is no **compilation dependency** between `student.h` and `course.h`. We only need to know that a type named `Course` exists.

***Solution-*** Forward declare the class, don't include the header.

A **compilation dependecy** exists if you must know the size of types in that header, or details of existing functions. Dont introduce compilation dependencies with `#includes` where none exists. 

## Compilation Dependencies Examples

Forward Declare
```cpp
// a.h
class B;
class A {
    int x;
    public:
        void Foo(B b)
}
```

Need to include
```cpp
// b.h
#include "B.h"
class B {
    C c; // need to know size of C to know size of B
    public:
}
```

Forward Declare
```cpp
// c.h
class A;
class C {
    A *ap;
    public:
}
```

Include 
```cpp
// d.h
#include "e.h"
class D {
    public:
        void Bar(E *e) {
            e->baz();
        }
}
```

## Polymorphism
There are two types of students. There are co-op and non co-op students. BUT, they are both students. i.e. We don't want 2 arrays in our course (1 for co-op students, 1 for regular). How do we achieve this?

We want to use polymorphism (many forms) through one generalized type (an interface), we get many specalized behaviours.

```cpp
class Student {
    // as before
    public:
        int Fees();
}
```

```cpp
// public student means everything you can do with a regular student, you do with a coop student

// if private student, then that means all student functions are private in coop student

class CoopStudent: public Student { // inheritance

}
```




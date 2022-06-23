# Week 4 Lecture 1

```cpp
// vec.h
#include<iostream>
// headers NEVER USE NAMESPACE

class Vec {
    int x,y;
    public:
        // defaulted params - only need to be in the header
        vcc(int x=0, int y=0); 
        friend ostream&operator<<(ostream &, const vcc &);
}
```

```cpp
// vec.cc
#include "vec.h"
#include <iostream>
using namespace std;

// if params are defaulted, function gets called with default values
Vec::Vec(int x, int y) : x{x}, y{y} {}
ostream &operator<<(ostream &out, const Vec &v) {
    return out << v.x << ", " << v.y;
}
```

## Defaulted Parameters

Node on defaulted function paramaters - they must be the last n params.

For eg.

```cpp
// This is invalid since default params are not in the end
int Foo(int x=0, int y, int z=1) {
    return (x+y)/z;
}

foo(3,5);
// Is this x=3, y=5, and z=1 OR is it x=0, y=3, and z=5 ???
```

### New class to store two Vectors
```cpp
// Basis.h
#include "vec.h"

class Basis {
    Vec v1, v2;
    public:
        Basis(const Vec &, const Vec &)

}
```

```cpp
// Basis.cc
#include "Basis.h"
#include "Vec.h"

Basis::Basis(const Vec & v1, const Vec & v2) : v1{v1}, v2{v2} {}
```

```cpp
// main.cc
#include "Basis.h"
#include "Vec.h"
#include <iostream>
using namespace std;

int main() {
    Vec v1{1,0};
    Vec v2{0,1};

    cout << v1 << " " << v2 << endl;
    
    Basis b{v1, v2};
}
```

To run, we use 
- `g++ -c vec.cc` (This works)
- `g++ -c basis.cc` (Breaks!)
- `g++ -c main.cc` (Breaks)
 
Error is due to not using `ifndef`. We are importing `vec.h` in `vec.cc`, but we are also importing `vec.h` in `basis.cc` and `main.cc`. There is a dual import going on here, and that is the problem. 

The class is being defined twice due to the import. This is since the **preprocessor** copies and pastes the includes header file right there.

In order to stop this from happening, we have to stop at the **preprocessor** directive. 

**Preprocessor** transforms your program before the compiler even see's it.

Two useful preprocessor directives are 

1. `#define FLAGNAME`
   - defines the preprocessor variable FLAGNAME to exist.
2. `ifndef FLAGNAME`
   - ```ifndef FLAGNAME \n ... \n endif```
   - If FLAGNAME is not defined, everything within the if stays in your program.
   - If it is defined, everything contained is removed before the compiler see's it.


Solution:
```cpp
// vec.h
#ifndef VEC_H_
#define VEC_H_

// vec file as before

#endif
```

This is called **Header Guard**. We should use on all headers to avoid double inclusion.

Now, we compile and link using 
- `g++ -std=c++14 -c vec.cc` (This works)
- `g++ -std=c++14 -c basis.cc` (This works)
- `g++ -std=c++14 -c main.cc` (This works)
- `g++ main.o vec.o basis.o -o main`

If I change `vec.cc`, what must we do? Must recompile `vec.cc` and relink
- `g++ -std=c++14 -c vec.cc` 
- `g++ main.o vec.o basis.o -o main`


If I change `basis.cc`, what must we do? Must recompile `basis.cc` AND `main.cc`

If I change `vec.h`, must recompile every file and relink

## This is not feasible since it takes more time to figure out *what* imports *what*
Since we do not want to manually keep track of all of these files and when they change, and what files must be recompiled becaue of this.

**Sol'n**: Unix makefiles

Example - makefile
```makefile
main : main.o vec.o basis.o
\t g++ main.o vec.o basis.o -o main

main.o : main.cc basis.h vec.h
\t git -std=c++14 -c main.cc

basis.o : basis.cc basis.h vec.h
\t git -std=c++14 -c basis.cc

vec.o : vec.cc vec.h
\t git -std=c++14 -c vec.cc
```

**Notes**  -
1. **\t** is a tab character (not spaces. editors auto replace tabs with spaces)

We need to type `make` in terminal to run makefile

## Automating makefile
```makefile
CXX = g++
CXXFLAGS = -std=c++14 -Wall -MMD
EXEC = myprogram
OBJECTS = main.o vec.o basis.o
DEPENDS = ${OBJECTS:.o-.d}
${EXEC}: ${OBJECTS}
\t ${CXX} ${CXXFLAGS} ${OBJECTS} -o ${EXEC}

-include ${DEPENDS}

.PHONY: clean

clean:
\t rm ${OBJECTS} ${EXEC} ${DEPENDS}
```

**Notes**  -
1. `make clean` removes the .o and the .d files
2. `DEPENDS` - every file in the objects variable, but change it from .o to .d

### Another Example

```cpp
// student.h
#include "course.h"

class Student {
    const int ID;
    std::string name;
    Course *courses; // courses we're in, capped at 5
};
```

```cpp
// course.h
#include "student.h"
class Course {
    std::string name;
    Student * enrolled;
    const int capacity;
    int currentlyEnrolled;

    Public:
        Course(std::string name, int capacity);
        void enroll(Student *);
}

```


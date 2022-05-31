# Week 5 Lecture 1

There are two types of students - regular & co-op.

```cpp
class Student {
    const int id;
    std::string name;
    int enrolledCourses;
    Courses** courses;
    
    public:
        int fees();
}
```

```cpp
class CoopStudent : public Student {
    // all the fields and methods from student also exist in coop student
    // note that we do not have Student's initialization function
    public:
        int fees();
}
```


```cpp
// student.cc
int Student::fees() {
    return 2500 + 500 * enrolledCourses;
}
```

```cpp
// coopstudent.cc
int CoopStudent::fees() {
    return 4500 + 500 * enrolledCourses;
}
```

## Error
We try to compile this, and it doesn't work. `enrolledCourses` is private in `Student`! So only `Student`'s methods and friends can access it. We must give `CoopStudent` access somehow.

**Note**: `Student` class is a **base class/superclass/parent class**.
**Note**: `CoopStudent` class is a **derived class/child class/subclass**.

## Protected class
Protected means accessible in current class and the derived classes. This lets child classes access it's parent's variables. The downside to this is that child classes can now break the parent classes. To circumvent this, we make a protected *getter method*. This prevents us from accidentally setting to something wrong.

```cpp
class Student {
    const int id;
    std::string name;
    int enrolledCourses;
    Courses** courses;
    
    protected:
        int getNumCourses() { return enrolledCourses; }

    public:
        int fees();
}
```

```cpp
// coopstudent.cc
using <string>

int CoopStudent::fees() {
    return 4500 + 500 * getNumCourses();
}
```

```cpp
// main.cc
int main() {
    CoopStudent s{0, "Abdur"};
}
// Doesnt compile since constructor for CoopStudent doesnt exist. We need Students constructor
```

## Fix for child constructor

```cpp
class CoopStudent : public Student {
    // all the fields and methods from student also exist in coop student
    // note that we do not have Student's initialization function
    public:
        int fees();
        CoopStudent(int id, std::string name);
}
```

```cpp
// coopstudent.cc
using <string>

int CoopStudent::fees() {
    return 4500 + 500 * getNumCourses();
}

// constructor for coopstudent
CoopStudent::CoopStudent(int id, std::string name) : Student {id, name} {}
```

We can't initialize `id` and `name` directly in `CoopStudent`'s constructor for 2 reasons -
1. The ctor is private to `Student`
2. By the time we initialize the fields, `id` and `name` have already been initialized


## Phases of Object Initialization
1. Space is allocated
2. Superclass components are initialized
3. Fields are initialized
4. Ctor body runs

## Phases of Object Destruction
Happens in the opposite direction of object initialization
1. Dtor body runs
2. Fields that are objects are destroyed
3. Superclass component is destroyed
4. Space is deallocated


### In our example
Since `student` has no default ctor, we **must** specify how to initialize it in the MIL as the compiler doesn't know how to initialize it otherwise. 

```cpp
// courses.h
class Course {
    // ...
    public: 
        int enrolledFees() {
            int x = 0;
            for (int i=0; i<curEnrolled; ++i) {
                x += enrolled[i]->fees();
            }
            return x;
        }
}
```

```cpp
// main.cc
int main() {
    CoopStudent a{0, "a"};
    Student b{1, "b"};

    CS247.enrolled(&a);
    CS247.enrolled(&b);


    a.enroll(&CS247); b.enroll(&CS247);
    cout << CS247.enrolledFees() << endl;
    // prints 6000, not 8000
}
```

## What went wrong?
Courses `enrolled` array was an array of `Student` *'s. But we stuck a ptr to a `CoopStudent` in there. How did we do this?

That is the beauty of public inheritance. A `CoopStudent` **is a** `Student`, so a `Student` ptr can point to a `CoopStudent`.

So, we can work polymorphically on different kinds of students through `Student` *'s, except that's not what we got. Instead, we got the behaviour of a regular `Student`. We got the fee for a regular student rather than a coop student. 

The compiler chooses which `fn fees` to run based on the static type of the data in the array. 

```cpp
cout << a.fees() << endl;
```

gives 5000, because the compiler sees `a`'s type is CoopStudent, and so it calls `CoopStudent::fees()`. 

But in `course::enrolledFees()`, we are operating on `Student` *'s. So it calls `Student::fees()` regardless of what type the pointer actually points at. 

## How do we solve this?

We need to solve this so the compiler chooses the method based on the actual runtime type of the object and **not** the static type. 

***Solution***: Declare the method virtual

```cpp
class Student {
    // ... as before
    public:
        virtual int fees();
}
```

All of a sudden we get the behaviour we want. The fees called on `Student` *'s calls the appropriate method based on the run-time type of the object, but **how is this possible?**

## Compiler leaves secrets

```cpp
// Before fees was virtual
cout << sizeof(Student) << " " << sizeof(CoopStudent) << endl;
    // printed 48 48

// After fees was virtual
cout << sizeof(Student) << " " << sizeof(CoopStudent) << endl;
    // printed 56 56
```

Adding the virtual function made our object larger by **1 pointer (8 bytes)**.

**Note:** C++ standard only specifies that the compiler must implement polymorphic behaviour of virtual functions through base class pointers and references. *It does not specify how the compiler must do this.*

However, virtually all compilers implement it this way.

## Why are our objects a pointer larger?
The compiler knows it may be asked to call this virtual function through a base class pointer or ref, and it must figure out which one to call. 

So, every time the compiler creates an object of a class with a virtual method, it sticks an extra pointer in there. 


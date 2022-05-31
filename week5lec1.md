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



# Entity vs. Value ADT

- Entity ADT represents a real world entity
- Value ADT represents some kind of abstract value

## Examples of entity ADT's
- Book's
- Student's
- Pizza's

## Example of value ADT's
- List's
- Force's
- Point's
- Rational number's
- Expression's

## Typically, Entity ADT's
- Disallow copying, or think twice before you write a copy (what should the behaviour be?)
  - What is the point of copying a student?
- Think before you overload operators, most don't make sense
- Methods on entities typically represent real world events
- Two entities are **not** equal if they have the same values. Typically only equal if they are stored in the **same location** in memory (same object)

## Typical, Value ADT's
- Allow copying
- Are equal if they have the same values
- Often have many overloaded operators

## Entity or Value?
These decisions usually come down to design and what makes sense for the system. 

Sometimes it's not so clear if it is an entity or value. For example, license plate ADT. 

Why it should be entity
- they represent real world objects
- cannot have duplicates

Why it should be value
- if two license plates are the same, then they are the same value

Can create an argument for both. We will create a value

### Our simplified requirements
A license plate will be 
- 3 letters followed by 3 digits
- Vanity plate, string of upto 7 characters. Minimum of 3
- Default ctor that builds next license plate
- Parameterized ctor that builds a vanity plate. Disallow hyphens in vanity plates

```cpp
class License {
    static string digits;
    static string letters; 
    string license;
    static void nextLicense();
   public:
    License();
    License(string s);
}
```

What are the `static string digits` and `static string letters`?
- A field that belongs to the **class**, not the individual object. It's shared across all objects of that type. (It is basically a global variable)

Where do we initialize these `static string`'s?
- Can't initialize it within the class. C++ standard says so 
- Definition and initialization must go in the implementation file (cpp)

**Note**: Static methods do not rely on a specific instance of an object. That is, they do not have an implicit `this` param. They can only refer to other static vars/methods. 

```cpp
// license.cc
#include <sstream>
#include <iomanip>

int intChar(char &) {
    // Pre 'a' <= c <= 'c'
    c++;
    if (c > 'z') {
        c = 'a';
        // carry int
        return 1;
    }
    return 0;
}

string License::digits = "000";
string License::letters = "aaa";
void License::nextLicense() {
    istringstream iss {digits};

    int num;
    iss >> num;
    num++;

    if (num == 1000) {
        num = 0;
        if (intChar(letters[2])) {
            if (intChar(letters[1])) {
                intChar(letters[0]);
                // assume no overflow in first letter
            }
        }
    }

    ostringstream oss {};
    oss << setfill('0') << setw(3) << num;
    letters = oss.str();

    // Problem - won't pad with leading zeroes
    // Solution - use iomanip's setfill
}

// default ctor
License::License() : license{letters + "-" + digits} {
    nextLicense();
}

// parameterized ctor
License::License(string s) {
    if (s.len < 3 || s.len > 7) {
        throw BadLicense{};
    }

    auto it = find(s.begin(), s.end(), '-');
    if (it != s.end()) {
        throw BadLicense{};
    }
}
```

## Shared Ownership through Unique Ptrs
We've talked about `unique_ptr` and using `unique_ptr`'s `get` to set up non-owning ptrs to the same data. But what about the case where we have **shared ownership**?

In the `memory` header we also have `std::shared_ptr<T>`. 

```cpp
{
    shared_ptr<int> sp = Make_Shared<int>(5);
    if (...) {
        // copy construction with sp
        shared_ptr<int>t {sp};
    } // t goes out of scope here. Does it delete the int it points at? NO because then we would have dangling sp. sp still points at it.
}
// sp goes out of scope, does free that memory now
```




# Week 9 Lecture 2

## Templates and Functions
```cpp
template<typename Itr, typename Ret>
Ret Sum(Iter start, Iter end) {
    Ret sum {}; // primitize types zero init with empty initializer call

    while (start != end) {
        sum += *start;
        ++start;
    }
    return sum;
}

int main() {
    vector<int> a {1,2,3,4};
    cout << sum<vector<int>::Iterator, int>(a.begin(), a.end());
    int arr[5] {1,2,3,4,5};
    cout << sum<int*, int> (arr, arr[5]);
}
```

Template classes must always parameterize instantiations. If the compiler knows types, template functions do not have to.

```cpp
template<typename Iter, typename Func>
void forEach(Iter start, Iter end, Func &f) {
    while (start != end) {
        f(*start);
        ++start;
    }
}

void print(string s) { cout << s << endl; }

int main() {
    vector<string> v{"Hello", "there"};
    for_each(v.begin(), v.end(), print);
}
```

Note that we didn't parameterize template, so the compiler infers paramter types of function `func` from static types. 

`for_each` and functions like it are defined in the STL iterator `<algorithm>` header (most of them operate on iterators). This header has a lot of useful functins so get used to it.

## What types `T` can be substitued for functions? 
Any types which are callable as a function (also param type must match type of dereferencing iterator)

## Overloading function calls via `operator()`
What other functions are called as functions? anything which defines `operator()` <- operator function call.

### Example 1 (Adder)
Say you want
- a function that adds 7 to its `int` param
- a function that adds -10 to its `int` param
- more of these specific add functions at any time

Do we make separate functions for each of them? No, we use **OOP**

```cpp
class Adder {
    int x;
    
    public:
        Adder(int x) : x{x} {}

        // second parentheses contains the parameter list 
        int operator()(int n) {
            return x + n;
        }
};

int main() {
    Adder add10 {10};
    Adder sub5 {-5};
    
    // we can treat the object as a function since it defines the operator() overload
    cout << add10(7) << " " << sub5(10) << endl; // prints: 17 5

    vector<int> v (5,6);
    // we can send our function in as a parameter that expects a function since we overload the operator() operator
    for_each(v.begin(), v.end, Adder{15}); // adds 15 to each element in v
};
```

### Example 2 (Increment Adder)
Want a function that adds 1 to its paramter the first time it's called. The second time, was it to add 2, and so on.

**We want to build arbitrary functions with state**

```cpp
class IncreasingPlus {
    int x = 1;

    public:
        void operator()(int &n) {
            n += x++;
        }
};

int main() {
    IncreasingPlus a;
    vector<int> v {0, 0, 0, 0};
    for_each(v.begin(), v.end(), a); // v is now {1,2,3,4}
    for_each(v.begin(), v.end(), a); // v is now {6,8,10,12}. a maintains its state
}
```
Objects that are callable as functions are called **function objects**

### Function Objects - objects that are callable as functions
Classes like this allow us to define families of functions (eg. `Adder`) and/or functions with state

## Lambda Functions
Another option for short functions. This is an anonymous function that doesn't have a name. 

```cpp
// lambdas in c++
[]->int(int n) { return n + 7; }
```
1. start with `[]` - lambda specifier/capture clause
2. `->T` - Return type (optional)
3. `(T)` - Param list
4. body

Instead of writing `Adder` in Example 1, we could have done 
```cpp
vector<int> v{1,2,3,4};
for_each(v.begin(), v.end(), []->int(int n) { return n + 7; });
```
## What if we have an integer `x` in main we want to add to each element?
```cpp
int x;
cin >> x;

vector<int> v{1,2,3,4};

for_each(v.begin(), v.end(). [x]->int(int n) {return n + x});
```
By default, lambda cannot access variables in caller's scope. We use capture clause to say what local variables we want to capture. Captures are copied by value, not reference by default. If we use `&x`, we capture by reference

## How does C++ implement Lambdas?
The compiler makes an object for each lambda we create

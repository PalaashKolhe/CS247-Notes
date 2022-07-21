# Week 12 Lecture 2

## Template Parameter Packs for a

- Variable amount of arguments

**We need ... below**

```cpp
template <typename T, typename... Args>

// base case
template <typename T>
void print() {}

void print(T first, Args... rest) {
    cout << first << endl;
    print(rest...);
}

int main() {
    print(5, 'c', "Hello there", 25);
}

/* prints out 
5
c
Hello there
25
*/
```

## Namespace
If we want to write our own vector class, we don't want it to clash with the `std::vector`. To do this, we place it in a namespace

```cpp
namespace CS247 {
    template <typename T>
    class Vector {
        size_t size, capacity;
        T* arr;

       public:
        void emplace_back(const T& x);
        ...
    }

    template <typename T>
    void Vector<T>::emplace_back(const T& x) {
        if (size == capacity) {
            // function that doubles our memory
            doubleMem(*this);
        }
        arr[size] = x;
    }
};
```







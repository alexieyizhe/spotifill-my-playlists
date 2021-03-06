**CS 246 |** Oct 18, 2018

#  More Operators

```cpp
struct Vec {
    int x, y;
    ...
    Vec operator+(const Vec &ither) {
        return { x + other.x, y + other.y }; 
    }
    Vec operator*(const int k) {
        return { x * k, y * k }; // this implements v * k
    }
};
```

`k * v`  must be non-member operator:

```cpp
Vec operator*(const int k, const Vec &v) { return v * k; }
```

__I/O Operators:__

```cpp
struct Vec {
    ...
    OStream &operator<<(ostream &out) {
        return out << x << ' ' << y;
    }
};
```

However, this makes `Vec` the first operand, not the second. You would have to use it like so: `v << cout`, which is confusing. So, we define operator `<<`, `>>` as _non-members_.

> Certain operators must be members:
>
> - operator=
> - operator[]
> - operator->
> - operator()
> - operator `T` where `T` is a type



# Arrays of Objects

```cpp
struct Vec {
    int x, y;
    Vec(int x, int y): x{x}, y{y}{}
};



Vec *vp = new Vec[10];  // WRONG, default constructor will be called but no default
Vec moreVecs[10];	    // constructor exists (so the items can't be initialized)
```

Our options to fix the above are:

1. Provide a default constructor: 
   _only do so if a default constructor makes sense (e.g. if you have a `Student` object, you don't want each initialized `Student` object to have the same name when initialized, so a default constructor __doesn't__ make sense here._

2. Provide values for initialization
   For _stack_ arrays:
   `Vec MoreVecs[3] = { {0, 0}, {1, 1}, {2, 4} }; `
   For _heap_ arrays of __known__ size:
   `Vec *vp = new Vec[3] { {0, 0}, {1, 1}, {2, 4} };`

3. For _heap_ arrays of __dynamic__ size, create an array of pointers:

   ```cpp
   Vec **vp = new Vec *[5];
   vp[0] = new Vec{0, 0};
   vp[1] = new Vec{1, 1};
   ... // do whatchu gotta do
   // make sure to DELETE ONCE YOU'RE DONE WITH THE ARRAY
   for(int i = 0; i < 5; i++) delete vp[i];
   delete [] vp;
   ```



# Const Objects

A const object is an object which has fields that cannot be modified. Const objects come up a lot, often as a result of parameters defining the object as `const`.

You _can_ call methods on const objects, but only methods that do not modify fields.

```cpp
struct Student {
    int assns, mt, final;
    float grade() const; // promises not to modify fields
};
```

The compiler verifies that const methods don't modify fields, and only const methods can be called on const objs.

### Use Case: Collecting Usage Stats on `Student` Objects

```cpp
struct Student {
    ...
    int numCalls = 0;
};
```

We want to be able to mutate `numCalls`, even on `const Student` objects. This field is not part of the "logic" of students (like `assns`, `mt`, or `final`). We can do so by declaring the field `mutable`.

```cpp
struct Student {
    ...
    mutable int numCalls = 0; // can be changed, even if object is a constant
    float grade() const {
        numCalls++;
        ...
        return ...;
    }
};
```



### Static Fields & Methods

For the above example, if we want to track `numCalls` on all `Student` objects instead of each individual one, we can use __static members__. These are associated with the _class itself_, not a particular instance of the `Student` object.

```cpp
struct Student {
    ...
    static int numStudents;
    Student(...): ... {
        numStudents++;
    }
};

int Student::numStudents = 0; // in .cc file
// static fields MUST be defined external to the class, since it should only be defined once and not for every creation of a Student object
```

A __static member function__ doesn't depend on an instance as well (so it has no `this` param). This means you can only access _static fields_ and other _static member functions_.

```cpp
struct Student {
    ...
    static int numStudents;
    ...
    static void howMany() {
        cout << numStudents;
    }
};

// in .cc
Student billy{ 60, 70, 80 }, jane{ 70, 80, 90 };
Student::howMany(); // prints 2
```



# Invariants & Encapsulation

Consider:

```cpp
struct Node {
    int data;
    Node *next;
    ~Node() { delete next; }
};


Node n1{1, new Node{2, nullptr}};
Node n2{3, nullptr};
Node n3{4, &n2};
```

This will cause _undefined behaviour_ when we try to delete `n3`, since the destructor tries to delete `&n2`, but `n2` is on the stack - we can only delete stuff that's on the heap!

`Node` relies on an assumption for its proper operation - that `next` is either `nullptr`, or was allocated by `new`. This is an __invariant__ - a statement that must hold true - that `Node` relies on.

However, we can't guarantee this invarient; we can't trust the user to use `Node` properly. To _enforce invarients_, we use __encapsulation__. 

- Treat objects as _black boxes_ - capsules, in a sense
  - Seal away the implementation
  - Only allow interaction through provided methods
  - Both the above are examples of __abstraction__

```cpp
struct Vec {
    Vec(int x, int y); // by default, visibility is public
private: // can't be accessed outside of the Vec struct
    int x, y;
public:
    Vec operator+(const Vec &other);
};
```

> Always try to define fields as `private` - only methods should really be `public`



# Classes

It's always better to have default visibility be `private`. So, we switch from `struct` to `class`:

```cpp
class Vec {
    int x, y;
public:
    Vec(int x, int y);
    Vec operator+(const Vec &other);
}
```

The __default visibility__ of `class` is __`private`__, compared to __`public`__ for `struct`.

> This is literally the only difference apart from one word being shorter than the other lol

```cpp
// list.h
class List {
    struct Node; // private nested class is only accessible inside class List
    Node *theList = nullptr;
public:
    ...
}
```



### Example: The `LinkedList` Class

```cpp
// list.h
class List {
    struct Node; //private nested class - only accessible within the List class
    Node *theList = nullptr;
    
public:
    void addToFront(int n);
    int &ith(int i); // reference allows us to mutate items in list directly
    ~List();
}


// list.cc
#include "list.h"

struct List::Node {
    int data;
    Node *next;
    Node(...): ... {...} // constructor
    ~Node() { delete next; }
}

void List::addToFront(int n) {
    theList = new Node{n, theList};
}

int &List::ith(int i) {
    Node *cur = theList;
    for(int n = 0; n < i; n++, cur=cur->next);
    return cur->data;
}
```

Now, only `List` can create/manipulate `Node` objects, so we can guarantee the invariant that `next` is always either `nullptr` or allocated by `new`.

However, now we can't traverse the list from `Node` to `Node` as we would a normal `LinkedList` without calling `ith()`, which is already $O(n)$ time itself.


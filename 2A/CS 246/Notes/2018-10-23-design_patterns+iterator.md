__CS 246 | __Oct 23, 2018

# Working with the `LinkedList` class

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



# Design Patterns

Certain programming problems arise often and we use __design patterns__ to keep track of solutions to these problems for reuse and mass adoption.

### Iterator ==Pattern==

**Problem:** we can't trust the user to access `Nodes`.
**Solution:** create a class that manages access to nodes

> Essentially, we create an abstraction of a pointer and walk the list without exposing the actual pointers.

For the above `LinkedList`, we implement the following improvements:

```cpp
class List {
    struct Node;
    Node *theList = nullptr;
public:
    class Iterator {
        Node *p;
    public:
        explicit Iterator(Node *p): p{p} {}
        int &operator*() { return p->data; }
        Iterator &operator++() { p = p->next; return *this; }
        bool operator!=(const Iterator &other) { return p != other.p; }
    };
    
    Iterator begin() { return Iterator{theList}; }
    Iterator end() { return Iterator{nullptr}; }
    
    ...
}

// Client code utilizing Iterator
int main() {
    List l;
    l.addToFront(1);
    l.addToFront(2);
    l.addToFront(3);
    
    for(List::Iterator it=l.begin(); it != l.end(); ++it) {
        cout << *it << endl; // this loop is linear O(n) time!
    }
}
```

### Automatic Type Deduction

```cpp
auto x = y;
```

The above code declares `x` to be the same type as its initializer (`y` in this case).

For the `List` and `Iterator` example:

```cpp
// long version
for(auto it = l.begin(); it != l.end(); ++it) {
    cout << *it << endl;
}

// shortcut
for(auto n:l) {
    cout << n << endl;
}
```

This shortcut is available for any class with:

1. Methods `begin()` and `end()` that produce iterators
2. The iterator must support `!=`, prefix `++`, and unary `*`

Note that the shortcut produces a copy - to iterate with references instead:

```cpp
for(auto &n:l) { // just declare it as references!
    ++n
}
```

### Back to Encapsulation

In the `List` example, the client cannot create iterators _except for one_:

```cpp
auto it = List::Iterator{nullptr};
```

which violates encapsulation (since it's the same as `end()`, if we change `end()` then this might break stuff). 

One option is to make `Iterator`'s constructor `private`, but now `List` cannot call it either. So, we give `List` privileged access to `Iterator` by making it a `friend`. 

```cpp
class List {
    ...
public:
    class Iterator {
        Node *p;
        explicit Iterator(Node *p);
    public:
        ...
        friend class List;
    }
}
```

Now, `List` can still create `Iterator`s, but the client can only create `Iterator`s by calling `begin()` or `end()`.

> Classes should be introverts: give them as few friends as possible since it weakens encapsulation.

For a better way to provide access to `private` fields, you want to write __accessor__ and __mutator__ methods.

```cpp
class Vec {
    int x, y;
public:
    ...
    int getX() const { return x; } // accessor
    void setY(int z) { y = z; } // mutator 
}
```

For `operator<<`, it can't be a member (would put ostream on right side, weird syntax) but still needs `x, y`. Just declare it as a `friend` function!

```cpp
class Vec {
    ...
    friend std::ostream &operator<<(std::ostream &out, const Vec &v);
}
```



# System Modelling

When working collaboratively, it's helpful to visualize the structure of the system - like abstractions and the relationships between them - to aid design and implementation.

### Unified Modelling Language (UML)

| Ver                                    | _Name_ |
| - | - |
|`-x: Integer`<br />`-y: Integer`| _Fields (optional)_ |
|`+getX: Integer`<br />`+getY: Integer`| _Methods (optional)_ |

The `-` and `+` represent private and public visibility, respectively.
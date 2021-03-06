__CS 246 |__ Nov 15, 2018

# Dependencies

```cpp
class Book {
public:
    virtual void accept(BookVisitor &v) { v.visit(*this); }
};

class Text:public Book {
public:
    void accept(BookVisitor &v) { v.visit(*this); }
};


class BookVisitor {
public:
    virtual void visit(Book &b) = 0;
    virtual void visit(Text &t) = 0;
    virtual void visit(Comic &c) = 0;
};
```

Suppose we want to develop an application that tracks how many of a certain type of `Book` there are. We want to track:

- `Book`s by __Author__
- `Text`s by __Topic__
- `Comic`s by __Hero__

We can add `virtual void updateCatalogue(...)` to each class, but we could just write a visitor:

```cpp
struct Catalogue:public BookVisitor {
    map<string, int> theCatalogue;
    
    void visit(Book &b) override { theCatalogue[b.getAuthor()]++; }
    void visit(Text &t) override { theCatalogue[t.getTopic()]++; }
    void visit(Comic &c) override { theCatalogue[c.getHero()]++; }
}
```

However, this code will not compile. Since `book.h` and `BookVisitor.h` include each other, this is a __circular include dependency__ and `book.h` will not be included a second time because of the _include guard_  in `book.h`. As a result, `Text` does not have the definition of a `Book` and doesn't know what it is.

### Compilation Dependencies

There are two options when requiring a dependency - you can either:

1. `#include` the dependency (what we normally do)
2. Forward declare the dependency (declare a skeleton version of the class with `class XXX;` just to show that there is such a class)

Consider the following classes defined in separate header `.h` files:

```cpp
class A { ... }; // A.h

class B: public A { ... }; // B.h


class C { // C.h
    A;
};

class D { // D.h
    A *a_ptr;
};

class E { // E.h
    A f(A x); 
};

class F { // F.h
    A f(A x) { ... x.someMethod(); ... }
};
```

When does a compilation dependency need to exist (_i.e. when do we need to `#include` the dependency_? For each class:

| Class | Requires `#include`? | Reasoning                                                    |
| ----- | -------------------- | ------------------------------------------------------------ |
| `B`   | ==Yes==              | Much like `C`, `B` being a subclass of  `A` is a lot like having an object field; it has all the fields that `A` does, so the compiler will require knowledge of the contents and size of `A`. |
| `C`   | ==Yes==              | Since `C` has an object field of type `A`, constructing an instance of `C` will require space allotment for `A`, which requires the compiler to know exactly how big and what is in `A`. |
| `D`   | No                   | Since there is just a pointer to `A` and all pointers are the same size, a forward declaration is sufficient to compile `D`. |
| `E`   | No                   | Since the function is just declared and not defined in the `E.h` file, a forward declaration is sufficient to compile `E`. In an `E.cc` file, we will have to `#include` the dependency of `A`. |
| `F`   | ==Yes==              | Since the function is defined and implemented, and an object of type `A` is used and a method of the object is called, the compiler needs to know everything about the object and thus everything about `A`. |

If the code doesn't need an `#include`, ==don't create a needless compilation dependency== by using `#include`unnecessarily. 

In the __implementation__ (`.cc`) of `D` and `C`, we _will_ need to know about class `A` since we are most likely manipulating objects of type `A` and whatnot. Then we can `#include "A.h"` and not worry about circular dependencies since a `.h` file will _never_ include a `.cc` file.

> This has the added benefit of reducing recompilation: in the example above, only `A, B, C, F` will need recompiling when `A` changes.



Now, consider the `XWindow` class.

```cpp
class XWindow {
    Display *d;                  // do we know what
    Window w;                   // all these 
    int s;                     // fields
    GC gc;                    // do?
    unsigned long colours[10];
public:
    ...
}
```

This class has a lot of clients (everyone in  $CS \ 246$). If you add or change one of these `private` members, _all clients must recompile._ The solution to this is the __pointer to implementation (PIMPL) idiom__.

###PIMPL ==Idiom==

We create a second `XWindowImpl` class that holds the "meat" of the class: all those fields that we had no idea what they did in the previous code block.

```cpp
// XWindowImpl.h
#include <X11/Xlib.h>
struct XWindowImpl {
    Display *d;
    Window w;
    int s;
    GC gc;
    unsigned long colours[10];
};


// Window.h
class XWindowImpl; // forward declaration

class XWindow {
    XWindowImpl *pImpl;
public:
    ... // no change from previous XWindow class
};


// Window.cc
#include "Window.h"
#include "XWindowImpl.h"

XWindow::XWindow(...): pImpl{ new XWindowImpl } {...}
// other methods stay the same except you replace fields:
// d becomes pImpl->d
// s becomes pImpl->s
```

Now, there's no need to include `Xlib.h`, and there is no compilation dependency on `XWindowImpl.h`. This means that clients also don't depend on `XWindowImpl.h`. 

If you keep all private fields in this `XWindowImpl.h`, then only `Window.cc` needs to be recompiled if you change `XWindow`'s implementation.


Furthermore, if you have several possible window implementations (_e.g. `XWindow`s and also `YWindow`s_), you can make the `Impl` struct a superclass:

> Insert UML diagram here

The __pImpl__ idiom with a class heirarchy of implementations is called the __Bridge__ ==Design Pattern==.           


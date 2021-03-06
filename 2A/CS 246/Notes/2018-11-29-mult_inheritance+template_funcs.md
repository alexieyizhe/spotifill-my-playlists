__CS 246 |__ Nov 29, 2018 (last​ lecture :cry:)

# Multiple Inheritance

A class can inherit from more than 1 class:

```cpp
class A{ int a; };
class B{ int b; };

class C: public A, public B {...} // class C will have fields a and b
```

However, you may have fields that are named the same on two superclasses that a subclass is inheriting from:

```cpp
class A{ int a; };
class B{ int a; };

class C: public A, public B {...} // class C will have two fields named a

C c;
cout << c.a << endl; // this is ambiguous

cout << c.A::a << endl; // this is how you need to say it
cout << c.B::a << endl;
```



Would there be two `a` fields if we have both class `A` and `B` being subclasses of the same superclass `X`, which has an `a` field that the two subclasses inherit from?

We make `X` a **virtual base class**:

```cpp
class X { int a; };

class A: virtual public X{...};
class B: virtual public X{...};

class C: public A, public B{...};
```



# Template Functions

```:cry:
template<typename T> T min(T x, T y) {
    return x < y ? x : y;
}

int x = 1, y = 2;
int z = min(x, y); // T = int - compiler figures this out based on the type of x, y
int a = min<int>(x, y); // this also works, but is more typing
auto f = min(1.0, 3.0); // T = double
```

`min` will work for any type `T` that has operators.

### C++ STL Library `<algorithm>`

The `<algorithm>` library contains a suite of template functions, many of which work for iterators.

There's the `find` method, which returns an iterator to a value the client wants to find, or an iterator to `last()` if not found:

```cpp
template<typename Iter, typename T>
Iter find(Iter first, Iter last, const T &val) {
    while(first != last) {
        if(*first == val) return first;
        ++first;
    }
    
    return last;
}
```

There's `count`, which is like `find`, but returns the number of occurrences of `val`.

There's also `copy`, which copies one container range from `first` to `last` to another, starting at `result`:

```cpp
template<typename InIter, typename OutIter>
OutIter copy(InIter first, InIter last, OutIter result) {
	// does the copying 
}

// client code
vector<int> v{1, 2, 3, 4, 5, 6, 7};
vector<int> w(4); // 0, 0, 0, 0
copy(v.begin() + 1, v.begin() + 5, w.begin()); // w = 2, 3, 4, 5
```

Note that __`copy` does not allocate new memory__ - it is the client's responsibility to do this and make sure there's enough space for the copying. 

> The above nuisance is because `copy` has no idea what `w` is; it can't, since it just has an iterator to `w` and thus `w` could be a `map`, `vector`, or anything else that has an iterator.

There's another called `transform`, which behaves similarly to `copy`, but accepts a function that will "transform" the values while copying them over:

```cpp
template<typename InIter, typename OutIter, typename Func>
OutIter copy(InIter first, InIter last, OutIter result, Func f) {
    while(first != last) {
		*result = f(*first);
		++first; 
		++result;
    }
    
    return result;
}


int add1(int n) { return n + 1; }
...
vector v{2, 3, 5, 7, 11};
vector w(v.size()); // all 0s
transform(v.begin, v.end(), w.begin(), add1); // 3, 4, 6, 8, 12
```

However, functions are not the only things that can be called as a function:

```cpp
vector v{2, 3, 5, 7, 11};
vector w(v.size()); // all 0s

// objects also have operator()
class Plus1 {
public:
    int operator()(int n) { return n + 1; }
};

transform(v.begin, v.end(), w.begin(), Plus1{}); // 3, 4, 6, 8, 12 (adds 1)


// might as well generalize while we're at it
class Plus {
    int increment;
public:
    Plus(int i): increment{i} {}
    int operator()(int n) { return n + increment; }
};

transform(v.begin, v.end(), w.begin(), Plus{2}); // 4, 5, 7, 9, 13 (adds 2)
```

Classes that are used like this are known as __function objects__. Apart from being able to generalize like above, another advantage to function objects are their ability to __maintain state:__

```cpp
class IncreasingPlus {
    int incrementBy = 0;
public:
    int operator() { return n + (incrementBy++); }
    void reset() { incrementBy = 0; }
};

vector<int> v(5, 0);   // both of these are
vector<int> w = v;     // 0, 0, 0, 0
transform(v.begin(), v.end(), w.begin(), IncreasingPlus{}); // w = 0, 1, 2, 3, 4
```

Other methods include `count_if` (returns the number of objects that match the function bool given), `accumulate` (similar to `foldr` in Racket), and more.

### Anonymous Functions 

When using template functions, it's sometimes annoying and seems like a waste to have to write "one-liner" functions for single use cases:

1. You have to explicitly create the function, adding one more thing you have to do to get your code working.
2. Functions cannot be defined inside other functions, so often your definition of a "one-liner" function will _not_ be right before the template function call (_i.e. if the template function call happens inside a bunch of nested functions_).

You can use__ anonymous lambda functions__ much like we did lambda functions in Racket. Consider the situation where you want to count how many `int`s in a `vector` are even:

```cpp
vector<int> v...;
bool even(int n) { return n % 2 == 0; }
int x = count_if(v.begin(), v.end(), even);

int x = count_if(v.begin(), v.end(), [](int n) { return n % 2 == 0; });
```

### Other Use Cases for Iterators

You can also apply the notion of iteration to other sources of data.

For example, you can create an iterator to an `ostream`:

```cpp
#include <iterator>

vector<int> v{1, 2, 3, 4, 5};
ostream::iterator<int> out { cout, ", "};
copy(v.begin, v.end(), out); // will print "1, 2, 3, 4, 5, "
```

As noted above, `copy` does not allocate new memory since it just has an iterator to the return value. So, the only way to have an automatically copying method is to use **an iterator whose assignment operator inserts a new item**.

```cpp
vector<int> v{1, 2, 3, 4, 5};
vector<int> w;
copy(v.begin(), v.end(), w.begin()); // undefined behaviour - segfaults
copy(v.begin(), v.end(), back_inserter(w)); // copies v to the end of w, adding new entries
```

This `back_inserter` iterator is available for any container with a `push_back` method.



> __Lushman says :speech_balloon: (last one):__ I hope C++ is your favourite programming language...for now.


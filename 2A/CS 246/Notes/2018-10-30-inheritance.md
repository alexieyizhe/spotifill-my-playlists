__CS 246 |__ Oct 30 2018

# Inheritance (cont.)

From the previous lecture, we wanted to write a method `isHeavy()` for our `Book` class and all of its subclasses. However, there are different defintions of "heavy" for different types of books:

- Regular old books are heavy if they contain > 200 pages
- `Text`books are heavy if they contain > 500 pages
- `Comic`books are heavy if they contain > 30 pages

So, we write the following code:

```cpp
class Book {
    ...
public:
    bool isHeavy() const { return length > 200; }
    ...
};

class Text:public Book {
    ...
public: 
    bool isHeavy() const { return length > 500; }
    ...
};
// and something similar for the Comic subclass

// create instances of the classes
Book b{...,..., 50};
Comic c{...,..., 40, ...};
cout << b.isHeavy(); // FALSE
cout << b.isHeavy(); // TRUE
```

Since we're in a "__is a__" relationship, we should be able to do the following:

```cpp
Book b = Comic{...,..., 40,...}; // this works!
```

Interestingly, `b.isHeavy()` will return `false` since it will invoke the `Book::isHeavy` method. Why?

A `Comic` object takes up more space in memory than a `Book` object, so when assigning a `Comic` object to a `Book` lvalue, the `Comic` object is __sliced__ (it has its `hero` field removed) and it is __coerced__ into a `Book` type. So, `Book b = Comic{...};` creates a `Book` and that's why the `Book::isHeavy` method is invoked.

When accessing objects through pointers or references, slicing is unnecessary and does not happen.

```cpp
Comic c = {...,..., 40,...};
Book *pb = &c;
Comic *pc = &c;
pc->isHeavy(); // TRUE
pb->isHeavy(); // FALSE - why?
```

The `Book::isHeavy` method is still invoked for the `pb` pointer. The compiler uses the type of the pointer/reference to decide which `isHeavy()` method to run - it _doesn't consider the actual type of the object_. The same object behaves differently depending on what is being used to access it.

We can make a `Comic` object act like a `Comic`, even when pointed to by the `Book` pointer, by using the `virtual` and `override` keywords:

```cpp
class Book {
    ...
protected:
    int length;
public:
    virtual bool isHeavy() const { return length > 200; }
    ...
};

class Comic:public Book {
    ...
public:
    bool isHeavy() const override { return length > 30; }
    ...
};
```

These `virtual` methods choose which class' method to invoke based on the actual type of the object at _runtime_ and not based on the accessor type.

Now, we can finish the solution for the question we set out to solve way way back - creating a collection of different books.

```cpp
Book *myBooks[20];
...
for (int i = 0; i < 20; i++) {
	cout << myBooks[i]->isHeavy() << endl;
}
// this will use the correct isHeavy method 
// Book::isHeavy for Book, Text::isHeavy for Text, etc
```

Accommodating multiple types under one abstraction is known as __polymorphism__.

> Fun fact: polymorphism is fancy Greek for "many types."

Being able to do this is _dangerous!_ Consider:

```cpp
class A {
    int x, y;
    ...
};

class B:public A {
    int z;
    ...
}

void f(A *a) {
    a[0] = A{6, 7};
    a[1] = A{8, 9};
}

B myArray[2] = {{1, 2, 3}, {4, 5, 6}};
f(myArray); // this will run, but what happens?
```

The above example will stick `A{8, 9}` halfway into `B[0]` and cause the data to become _misaligned_. Since they are different sizes and `a[1]` will increment the pointer arithmetic based on the size of `A`, not taking into account the extra field on `B`.

So, __never use arrays of objects polymorphically.__ Instead, use __arrays of pointers.__



### Inheritance & The Big 5

Consider the following example:

```cpp
class X {
    int *a;
	X(int n): a{ new int [n] } {}
    ~X() { delete [a]; }
};

class Y:public X {
    int *b;
public:
    Y(int n, int m): X(n), b{ new int[m] } {}
    ~Y() { delete [] b; }
};

X *myX = new Y{10, 20};
delete myX; // this will leak memory, since we call the destructor for X for an object of class Y
```

To ensure that deletion through superclass pointer will call the subclass destructor, _make the destructor virtual:_

```cpp
class X {
    ...
public:
    virtual ~X() { delete [] a; } 
    ...
};
```

> We do not need to use the `override` keyword for subclasses, since a destructor has only one signature (no possible mismatch in subclass methods).

__Always__ make the destructor virtual in classes that are meant to have subclasses - even if the destructor doesn't do anything (since subclasses might have destructors that do stuff that need to be called instead).

If a class is _not_ meant to have subclasses, declare it so using the `final` keyword:

```cpp
class Y final:public X {
    ...
}; // Y cannot have subclasses
```


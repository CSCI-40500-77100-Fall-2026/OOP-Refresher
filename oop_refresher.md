---
title: "Object-Oriented Programming Refresher"
author: Raffi Khatchadourian (outline based on a Java overview by Atanas Rountev, The Ohio State University)
semester: Fall 2026
lang: en
footer: Outline based on a Java overview by Atanas Rountev, The Ohio State University
license: Creative Commons Attribution-ShareAlike 4.0 International (CC BY-SA 4.0)
---

# Object-Oriented Programming Refresher

## Agenda

* Classes and objects.
* References and copies.
* Instance and class (static) members.
* Inheritance and constructors.
* Overloading vs. overriding.
* Polymorphism and dynamic dispatch.
* Abstract classes and interfaces.
* Where it all shows up: the Observer pattern.

::: notes
About 20 minutes. This covers exactly what the Observer Pattern assignment needs: abstract and concrete classes, inheritance, overriding, and polymorphism. Examples are in Python and C++, side by side, because those are the languages nearly all students choose for the assignment. Java notes appear where Java differs, for students who choose Java.
:::

## Classes and Objects

* A *class* is a blueprint; an *object* (or *instance*) is made from it.
* A class's *members* are its *fields* (data) and *methods* (behavior).

:::::::::::::: {.columns}
::: {.column width="50%"}

### Python

```python
class Rectangle:
    def __init__(self, height, width):
        self.height = height
        self.width = width

    def area(self):
        return self.height * self.width

r = Rectangle(1.0, 3.6)
print(r.area())  # 3.6
```

:::
::: {.column width="50%"}

### C++

```cpp
class Rectangle {
public:
    Rectangle(double height, double width)
        : height(height), width(width) {}
    double area() const {
        return height * width;
    }
    double height, width;
};

Rectangle r(1.0, 3.6);
std::cout << r.area();  // 3.6
```

:::
::::::::::::::

::: notes
*Instantiating* a class means creating an object from it: `Rectangle(1.0, 3.6)`.

A method runs on a particular object, the *receiver*. Python passes the receiver explicitly as the first parameter, named `self` by convention. C++ passes it implicitly as the pointer `this`, so `height` inside `area()` means `this->height`.

`__init__` (Python) and the member with the class's name (C++) are *constructors*: they set up a new object's initial state. The C++ `: height(height), width(width)` part is a *member initializer list*.

`const` after a C++ method promises the method does not change the object.

Java is like C++ here, except that objects are always created with `new`: `Rectangle r = new Rectangle(1.0, 3.6);`.
:::

## References and Copies

:::::::::::::: {.columns}
::: {.column width="50%"}

### Python

```python
a = Rectangle(1.0, 3.6)
c = a           # same object
c.width = 5.0
print(a.width)  # 5.0
```

* Variables *refer* to objects.
* Assignment shares the object.

:::
::: {.column width="50%"}

### C++

```cpp
Rectangle a(1.0, 3.6);
Rectangle c = a;    // a copy
c.width = 5.0;      // a.width is 3.6
Rectangle& r = a;   // another name for a
Rectangle* p = &a;  // the address of a
r.width = 5.0;      // a.width is 5.0
```

* Variables *hold* objects.
* Assignment copies; use references or pointers to share.

:::
::::::::::::::

> In C++, polymorphism works only through references and pointers (more in a few slides).

::: notes
Two names for one object is called *aliasing*. In Python (and in Java, where variables of class type are references), aliasing is the default. In C++, a variable of class type is the object itself, so `Rectangle c = a;` copies every field.

A C++ *reference* (`Rectangle&`) is another name for an existing object; it cannot be reseated. A *pointer* (`Rectangle*`) holds an object's address and can be changed or be `nullptr`; use `->` to reach members through it (`p->width`).

This matters for the Observer assignment: a subject keeps a list of *its* observers. In C++, that list must hold pointers (or references), not copies; otherwise, updating the list's copies does not update the real observers.
:::

## Instance vs. Class (Static) Members

:::::::::::::: {.columns}
::: {.column width="50%"}

### Python

```python
class Rectangle:
    count = 0  # one copy, shared

    def __init__(self, height, width):
        self.height = height  # one per object
        self.width = width
        Rectangle.count += 1
```

:::
::: {.column width="50%"}

### C++

```cpp
class Rectangle {
public:
    static inline int count = 0;  // one copy
    Rectangle(double height, double width)
        : height(height), width(width) {
        ++count;
    }
    double height, width;  // one per object
};
```

:::
::::::::::::::

* *Instance* members: each object has its own copy.
* *Class* (*static*) members: one copy for the whole class, much like a global variable.
* A static method has no `self` or `this`: it does not run on an object.
* Use static members sparingly; with too many, the code becomes procedural rather than object-oriented.

::: notes
Python calls these *class attributes*; C++ and Java call them *static members*. Python also has `@staticmethod` (no `self`) and `@classmethod` (receives the class instead of an object).

`static inline` lets a C++17 class define its static data member in the class body; older C++ needs a separate definition outside the class.

Java: `static int count = 0;`, and the same advice applies.
:::

## Inheritance

* A *subclass* inherits every member of its *superclass* and may add new ones.
* A superclass variable may refer to a subclass object.

:::::::::::::: {.columns}
::: {.column width="50%"}

### Python

```python
class Square(Rectangle):
    def __init__(self, side):
        super().__init__(side, side)

s = Square(2.0)
print(s.area())  # 4.0, inherited
```

:::
::: {.column width="50%"}

### C++

```cpp
class Square : public Rectangle {
public:
    explicit Square(double side)
        : Rectangle(side, side) {}
};

Square s(2.0);
std::cout << s.area();  // 4, inherited
```

:::
::::::::::::::

* Constructors are not inherited: a subclass constructor calls the superclass's (`super().__init__(...)`, or `Rectangle(...)` in the initializer list).

::: notes
A superclass is also called a *base class* (C++) or *parent class*; a subclass is a *derived class* or *child class*.

If a Python subclass's `__init__` forgets `super().__init__(...)`, the superclass's fields are never set, and the error surfaces later as an `AttributeError`. If a C++ subclass constructor does not name a superclass constructor, C++ calls the superclass's default constructor, and compilation fails if there is none.

`explicit` stops C++ from silently converting a `double` into a `Square`.

Python and C++ allow *multiple inheritance* (several superclasses). Java allows one superclass (`class Square extends Rectangle`), and its constructor calls `super(side, side);` as the first statement.
:::

## Overloading vs. Overriding

* *Overloading*: same name, different parameter types, in the same class.
	* C++ (and Java): `void scale(double f);` and `void scale(double fh, double fw);`.
	* Python has no overloading: a second `def scale` replaces the first. Use default arguments instead.
* *Overriding*: a subclass redefines an inherited method with the same signature.

:::::::::::::: {.columns}
::: {.column width="50%"}

### Python

```python
class RectangleWithHole(Rectangle):
    def __init__(self, height, width, hole):
        super().__init__(height, width)
        self.hole = hole

    def area(self):  # overrides
        return super().area() - self.hole
```

:::
::: {.column width="50%"}

### C++

```cpp
class RectangleWithHole : public Rectangle {
public:
    RectangleWithHole(double h, double w,
                      double hole)
        : Rectangle(h, w), hole(hole) {}
    double area() const override {
        return Rectangle::area() - hole;
    }
    double hole;
};
```

:::
::::::::::::::

* `super().area()` and `Rectangle::area()` call the overridden superclass version.

::: notes
A method's *signature* is its name plus its parameter types.

In C++, overriding needs the superclass method to be `virtual` (next slide): `Rectangle` now declares `virtual double area() const`. The `override` keyword asks the compiler to check that the method really overrides one; without it, a typo (e.g., forgetting `const`) silently declares a new, unrelated method.

Java overrides by default; the optional `@Override` annotation gives the same check as C++'s `override`. Java calls the superclass version with `super.area()`.
:::

## Polymorphism and Dynamic Dispatch

* Which `area()` runs is decided at *run time*, by the object's actual class, not by the variable's declared type.

:::::::::::::: {.columns}
::: {.column width="50%"}

### Python

```python
shapes = [Rectangle(2, 3), Square(2),
          RectangleWithHole(2, 3, 1)]
for s in shapes:
    print(s.area())  # 6, 4, 5
```

:::
::: {.column width="50%"}

### C++

```cpp
class Rectangle {
public:
    virtual ~Rectangle() = default;
    virtual double area() const { ... }
    ...
};

Rectangle r(2, 3);
Square q(2);
RectangleWithHole h(2, 3, 1);
std::vector<Rectangle*> shapes = {&r, &q, &h};
for (Rectangle* s : shapes)
    std::cout << s->area() << '\n';  // 6, 4, 5
```

:::
::::::::::::::

* In C++, only `virtual` methods are dispatched this way, and only through pointers or references.

> Q: In the C++ version, what is printed if `area()` is *not* `virtual`?

::: notes
Answer: 6, 4, 6. Without `virtual`, the call is bound at compile time to the declared type, `Rectangle`, so every object runs `Rectangle::area()`: 2×3 = 6, 2×2 = 4, and 2×3 = 6 for the rectangle with the hole, whose hole is ignored.

*Polymorphism* ("many forms"): one call, `s.area()`, runs different methods for different objects. *Dynamic dispatch* (also *dynamic binding* or *virtual dispatch*) is how: at run time, the method is looked up starting from the receiver object's class and moving up the inheritance chain until a match is found.

A class meant to be a superclass in C++ needs a `virtual` destructor (`virtual ~Rectangle() = default;`), so that deleting a subclass object through a superclass pointer runs the subclass's destructor too.

Python and Java dispatch every instance method dynamically (in Java, except `static`, `private`, and `final` methods), so they need no `virtual` keyword.
:::

## Abstract Classes

* An *abstract method* has no body. A class with one is *abstract* and cannot be instantiated.
* A subclass that implements every abstract method is *concrete*.

:::::::::::::: {.columns}
::: {.column width="50%"}

### Python

```python
from abc import ABC, abstractmethod

class Shape(ABC):
    @abstractmethod
    def area(self): ...

    def describe(self):
        return f"area {self.area()}"

Shape()  # TypeError: Can't instantiate
         # abstract class Shape ...
```

:::
::: {.column width="50%"}

### C++

```cpp
class Shape {
public:
    virtual ~Shape() = default;
    virtual double area() const = 0;
    std::string describe() const {
        return "area " + std::to_string(area());
    }
};

Shape s;  // error: cannot declare variable
          // 's' to be of abstract type 'Shape'
```

:::
::::::::::::::

* `Rectangle` would now extend `Shape` and implement `area()`.
* `describe()` is concrete but calls the abstract `area()`: the superclass fixes *when* `area()` is called, and each subclass decides *how* it works.

::: notes
In C++, `= 0` makes a method *pure virtual*, i.e., abstract. In Java, both the method and the class are marked `abstract`: `abstract double area();`.

A common Python mistake: `@abstractmethod` is only enforced when the class inherits from `ABC`. Without `ABC`, `Shape()` succeeds and the error appears only when the empty method is called. Raising `NotImplementedError` in the method body has the same weakness.

An abstract method can be the *declared* target of a call (`s.describe()` calls `area()` through a `Shape`), but never the method that actually runs, since no object of an abstract class can exist.

A concrete method that calls abstract ones, as `describe()` does here, is the *Template Method* design pattern.
:::

## Interfaces

* An *interface* is a contract: abstract methods only, no fields.
* Java has an `interface` keyword; in Python and C++, write an abstract class whose methods are all abstract.
* A class has one superclass in Java but can implement many interfaces; Python and C++ use multiple inheritance.

:::::::::::::: {.columns}
::: {.column width="50%"}

### Python

```python
class Drawable(ABC):
    @abstractmethod
    def draw(self): ...

class Square(Rectangle, Drawable):
    # __init__ as before
    def draw(self):
        print("[]")
```

:::
::: {.column width="50%"}

### C++

```cpp
class Drawable {
public:
    virtual ~Drawable() = default;
    virtual void draw() const = 0;
};

class Square : public Rectangle,
               public Drawable {
public:
    // constructor as before
    void draw() const override {
        std::cout << "[]";
    }
};
```

:::
::::::::::::::

* A variable of the interface's type can refer to any object whose class implements it: `Drawable* d = &q; d->draw();`.

::: notes
Java: `interface Drawable { void draw(); }` and `class Square extends Rectangle implements Drawable { ... }`.

Interfaces separate *what* an object can do from *how* it does it. Code written against `Drawable` works with any class that implements it, including classes written later. That is exactly how the Observer pattern lets a subject notify observers it knows nothing about.
:::

## Where It Shows Up: the Observer Pattern

``` {.mermaid format=svg}
classDiagram
    direction LR
    class Subject {
        <<abstract>>
        +Attach(Observer)*
        +Detach(Observer)*
        +Notify()*
    }
    class Observer {
        <<abstract>>
        +Update()*
    }
    class ConcreteSubject {
        -subjectState
        +GetState()
    }
    class ConcreteObserver {
        -observerState
        +Update()
    }
    Subject o-- "*" Observer : observers
    Subject <|-- ConcreteSubject
    Observer <|-- ConcreteObserver
    ConcreteObserver --> ConcreteSubject : subject
```

| Part of the pattern | Feature it uses |
|-----|-------|
| `Subject`, `Observer` | Abstract classes (`Observer` is effectively an interface). |
| `ConcreteSubject`, `ConcreteObserver` | Inheritance; implementing abstract methods. |
| The subject's list of observers | References (Python) or pointers (C++), not copies. |
| `Notify()` calls `Update()` on each observer | Dynamic dispatch. |

::: notes
This is the UML model from the Design Patterns lecture, redrawn: `Notify()` runs "for all o in observers: o->Update()", and `ConcreteObserver::Update()` sets `observerState = subject->GetState()`. The asterisks mark abstract methods.

The point to make: nothing in this diagram is new. Each part is one of the features from today. `Notify()` is written once, in `Subject`, against the abstract `Observer`; dynamic dispatch makes each call run the right `ConcreteObserver`'s `Update()`.

Whether `Attach`, `Detach`, and `Notify` are abstract or implemented once in `Subject` is a design choice students make in the assignment; the slide deliberately stops short of code.
:::

## Common Pitfalls

* C++: forgetting `virtual`. The superclass version runs, silently. Mark overrides with `override`.
* C++: storing objects by value in a `std::vector<Shape>`. Store pointers (e.g., `std::vector<Shape*>`) instead.
* C++: no `virtual` destructor in a class meant to be a superclass.
* Python: forgetting `super().__init__(...)` in a subclass.
* Python: using `@abstractmethod` without inheriting from `ABC`.

::: notes
Storing a subclass object in a variable or container of the superclass type by value *slices* it: only the superclass part is copied, and the subclass's fields and overrides are lost. A `std::vector<Shape>` will not even compile when `Shape` is abstract; with a concrete superclass, it compiles and quietly slices.

Smart pointers (`std::unique_ptr<Shape>`, `std::shared_ptr<Shape>`) avoid manual `delete`. For the Observer assignment, a subject usually does *not* own its observers, so plain pointers (or `std::weak_ptr`) fit its list.
:::

## Summary

* Classes are blueprints; objects are instances with their own fields.
* Python shares objects through references; C++ copies unless you use references or pointers.
* Subclasses inherit members, add new ones, and override methods.
* Dynamic dispatch picks the method from the object's actual class (in C++, only for `virtual` methods through pointers or references).
* Abstract classes and interfaces declare *what*; concrete subclasses define *how*.
* The Observer pattern is built entirely from these features.

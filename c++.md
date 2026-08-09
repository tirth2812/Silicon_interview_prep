# C++ Basic Coding Interview Cheat Sheet

## Table of Contents

**Basics**
1. [Program Structure + Output + Comments](#1-program-structure--output--comments)
2. [Variables + Data Types + Constants](#2-variables--data-types--constants)
3. [User Input](#3-user-input)
4. [Operators](#4-operators)
5. [Strings](#5-strings)
6. [Math](#6-math)
7. [Booleans](#7-booleans)

**Control Flow**
8. [If / Else / Ternary](#8-if--else--ternary)
9. [Switch](#9-switch)
10. [While Loop](#10-while-loop)
11. [For Loop](#11-for-loop)
12. [Break and Continue](#12-break-and-continue)

**Data**
13. [Arrays](#13-arrays)
14. [Structures](#14-structures)
15. [Enums](#15-enums)

**Memory**
16. [References](#16-references)
17. [Pointers](#17-pointers)
18. [Memory Management: new / delete / Stack vs Heap](#18-memory-management-new--delete--stack-vs-heap)

**Functions**
19. [Functions + Parameters: Value vs Reference](#19-functions--parameters-value-vs-reference)
20. [Function Overloading](#20-function-overloading)
21. [Scope](#21-scope)
22. [Recursion](#22-recursion)
23. [Lambda](#23-lambda)

**OOP / Classes**
24. [OOP Concepts](#24-oop-concepts)
25. [Classes and Objects](#25-classes-and-objects)
26. [Class Methods](#26-class-methods)
27. [Constructors](#27-constructors)
28. [Access Specifiers + Encapsulation](#28-access-specifiers--encapsulation)
29. [Friend Functions](#29-friend-functions)
30. [Inheritance](#30-inheritance)
31. [Polymorphism](#31-polymorphism)

**Quick Reference**
32. [. vs ->](#32---vs--)
33. [Common Interview Traps](#33-common-interview-traps)
34. [5-Minute Final Revision](#34-5-minute-final-revision)

---

# 1. Program Structure + Output + Comments

```cpp
#include <iostream>   // for cout, cin
using namespace std;  // allows cout instead of std::cout

int main() {
    cout << "Hello World" << endl;   // endl = newline + flush
    cout << "Hello\n";               // \n = newline only (faster)
    return 0;                        // 0 = successful termination
}
```

```cpp
// Single-line comment
/* Multi-line
   comment */
```

---

# 2. Variables + Data Types + Constants

```cpp
int    age    = 25;
double salary = 5000.50;   // more precise than float
float  rate   = 3.14f;     // f suffix for float
char   grade  = 'A';       // single quotes
bool   passed = true;
string name   = "Tirth";   // double quotes

int x = 10, y = 20, z = 30;  // multiple on one line

const int MAX = 100;          // cannot change after declaration
// MAX = 200;                 // ERROR
```

**Type casting:**
```cpp
double x = 10.8;
int y = static_cast<int>(x);  // y = 10 (truncates, not rounds)
```

---

# 3. User Input

```cpp
int age;
cin >> age;

int x, y;
cin >> x >> y;          // multiple inputs

string word;
cin >> word;            // stops at space

string sentence;
getline(cin, sentence); // reads full line including spaces
```

**Common trap — mixing cin and getline:**
```cpp
int age;
cin >> age;
cin.ignore();           // clears leftover newline from buffer
getline(cin, name);     // now works correctly
```

---

# 4. Operators

```cpp
// Arithmetic
a + b    // addition
a - b    // subtraction
a * b    // multiplication
a / b    // division
a % b    // remainder/modulo

// Integer division trap
5 / 2    // = 2 (both int, decimal dropped)
5.0 / 2  // = 2.5 (one double, works correctly)

// Comparison (returns bool)
== != > < >= <=

// Logical
&& (AND)   || (OR)   ! (NOT)

// Increment/Decrement
int x = 5;
int a = x++;   // a=5, x=6 (post: use THEN increment)
int b = ++x;   // x=7, b=7 (pre: increment THEN use)
```

---

# 5. Strings

```cpp
#include <string>
string s = "Hello";

s.length()          // 5
s.size()            // 5 (same as length)
s[0]                // 'H'  (access char)
s[0] = 'Y'         // "Yello" (modify char)

// Concatenation
string result = "Hello" + " " + "World";

// Loop through
for (int i = 0; i < s.length(); i++)
    cout << s[i];

for (char c : s)    // range-based (cleaner)
    cout << c;
```

---

# 6. Math

```cpp
#include <cmath>

max(10, 20)    // 20
min(10, 20)    // 10
sqrt(25)       // 5.0
pow(2, 3)      // 8.0
round(4.6)     // 5
ceil(4.2)      // 5  (always rounds up)
floor(4.8)     // 4  (always rounds down)
abs(-5)        // 5  (absolute value)
```

---

# 7. Booleans

```cpp
bool isValid  = true;
bool finished = false;
// true = 1, false = 0 internally

int x = 10;
bool result = (x > 5);   // result = true
```

---

# 8. If / Else / Ternary

```cpp
if (x > 0) {
    cout << "Positive";
}
else if (x < 0) {
    cout << "Negative";
}
else {
    cout << "Zero";
}

// Ternary (one-liner if/else)
int maxVal = (a > b) ? a : b;
// reads as: if a > b then a, else b
```

---

# 9. Switch

Use when comparing one variable against multiple fixed values.

```cpp
int day = 2;
switch (day) {
    case 1:
        cout << "Monday";
        break;       // MUST have break, else falls through
    case 2:
        cout << "Tuesday";
        break;
    default:
        cout << "Invalid";
}
```

**Without break — fall-through trap:**
```cpp
case 1:
    cout << "A";    // no break → falls into case 2
case 2:
    cout << "B";    // also runs if case 1 matched
```

---

# 10. While Loop

Use when number of iterations is not known beforehand.

```cpp
int i = 0;
while (i < 5) {
    cout << i << " ";
    i++;
}
// Output: 0 1 2 3 4

// do-while: runs at least once
do {
    cout << i;
    i++;
} while (i < 5);
```

---

# 11. For Loop

Use when number of iterations is known.

```cpp
for (int i = 0; i < 5; i++)
    cout << i;        // 0 1 2 3 4

// Reverse
for (int i = 5; i >= 0; i--)
    cout << i;

// Nested
for (int i = 0; i < 3; i++)
    for (int j = 0; j < 3; j++)
        cout << i << "," << j << endl;

// Range-based (for arrays/strings)
for (int val : arr)
    cout << val;
```

---

# 12. Break and Continue

```cpp
// break: exits the entire loop
for (int i = 0; i < 10; i++) {
    if (i == 5) break;
    cout << i;          // 0 1 2 3 4
}

// continue: skips current iteration only
for (int i = 0; i < 5; i++) {
    if (i == 2) continue;
    cout << i;          // 0 1 3 4
}
```

---

# 13. Arrays

Fixed size, same type, index starts at 0.

```cpp
int arr[5] = {10, 20, 30, 40, 50};
arr[0]  // 10
arr[4]  // 50
arr[2] = 100;   // modify

// Size
int size = sizeof(arr) / sizeof(arr[0]);

// Loop
for (int i = 0; i < 5; i++)
    cout << arr[i];

for (int val : arr)     // range-based
    cout << val;
```

**Trap:** `arr[5]` is out of bounds — valid indexes are 0 through n-1.

---

# 14. Structures

Groups different types together.

```cpp
struct Student {
    string name;
    int    age;
    double GPA;
};

Student s1;
s1.name = "Tirth";
s1.age  = 25;
s1.GPA  = 3.8;

// Or initialize directly
Student s2 = {"Tirth", 25, 3.8};

cout << s1.name;
```

---

# 15. Enums

Named integer constants — makes code readable.

```cpp
enum Day {
    MONDAY,     // = 0 by default
    TUESDAY,    // = 1
    WEDNESDAY   // = 2
};

Day today = MONDAY;

// Custom values
enum Status {
    SUCCESS =  0,
    ERROR   = -1,
    BUSY    =  1
};
```

---

# 16. References

An alias — another name for the same variable.

```cpp
int x   = 10;
int &ref = x;     // ref IS x

ref = 20;         // x is now 20 too

// Most useful in function parameters (avoids copying, allows modification)
void change(int &x) {
    x = 100;
}
int a = 10;
change(a);        // a is now 100

// Read-only reference (avoids copy, prevents modification)
void print(const string &text) {
    cout << text;
}
```

---

# 17. Pointers

Stores a memory address.

```cpp
int  x   = 10;
int *ptr = &x;    // ptr holds the ADDRESS of x

x      // 10          (value)
&x     // 0x100       (address of x)
ptr    // 0x100       (same address — ptr stores it)
*ptr   // 10          (value AT that address — dereference)

*ptr = 50;            // x is now 50

// Null pointer — always initialize if not pointing to anything
int *p = nullptr;
// Never dereference nullptr: *p would crash
```

**Memory diagram:**
```
x         ptr
+------+   +--------+
|  10  |   | 0x100  |
+------+   +--------+
0x100

ptr  → stores 0x100 (address of x)
*ptr → value at 0x100 = 10
```

---

# 18. Memory Management: new / delete / Stack vs Heap

```cpp
// Stack — automatic, freed when scope ends
int x = 10;     // lives on stack

// Heap — manual, you must free it
int *ptr = new int;       // allocate single int on heap
*ptr = 10;
delete ptr;               // free
ptr = nullptr;            // good habit

// Dynamic array on heap
int *arr = new int[5];
arr[0] = 10;
delete[] arr;             // MUST use delete[] for arrays
arr = nullptr;
```

**Rules:**
```
new    → delete
new[]  → delete[]
Forget to delete → memory leak (memory never returned to OS)
```

---

# 19. Functions + Parameters: Value vs Reference

```cpp
// Basic function
int add(int a, int b) {
    return a + b;
}
int result = add(10, 20);   // result = 30

// void — no return value
void printHello() {
    cout << "Hello";
}
```

**Pass by Value — original unchanged:**
```cpp
void change(int x) {
    x = 100;         // only the copy changes
}
int a = 10;
change(a);
cout << a;           // still 10
```

**Pass by Reference — original changes:**
```cpp
void change(int &x) {
    x = 100;         // changes the actual variable
}
int a = 10;
change(a);
cout << a;           // 100
```

| | Value | Reference |
|---|---|---|
| Copies data? | Yes | No |
| Original modified? | No | Yes |
| Use for large read-only? | `const T&` | `const T&` |

---

# 20. Function Overloading

Same name, different parameters — compiler picks the right one.

```cpp
int    add(int a, int b)       { return a + b; }
double add(double a, double b) { return a + b; }
int    add(int a, int b, int c){ return a + b + c; }

add(10, 20);        // calls int version
add(5.5, 2.5);      // calls double version
add(1, 2, 3);       // calls 3-param version
```

**Cannot overload by return type alone:**
```cpp
int    add(int a, int b);   // ERROR — same params, only return type differs
double add(int a, int b);
```

---

# 21. Scope

```cpp
int x = 10;          // global scope — accessible everywhere below

void test() {
    int y = 20;      // local scope — only inside test()
    cout << x;       // can access global x
    cout << y;       // can access local y
}
// y unavailable here

if (true) {
    int z = 30;      // block scope — only inside this block
}
// z unavailable here
```

**Rule: prefer the smallest scope that works.**

---

# 22. Recursion

Function calls itself. Every recursion needs a **base case** to stop.

```cpp
int factorial(int n) {
    if (n <= 1) return 1;        // base case — stops recursion
    return n * factorial(n - 1); // recursive call
}

factorial(4)
= 4 * factorial(3)
= 4 * 3 * factorial(2)
= 4 * 3 * 2 * factorial(1)
= 4 * 3 * 2 * 1 = 24
```

Without a base case → **infinite recursion → stack overflow.**

---

# 23. Lambda

An anonymous (unnamed) function — can be stored and passed around.

```cpp
// Basic
auto greet = []() {
    cout << "Hello";
};
greet();

// With parameters
auto add = [](int a, int b) {
    return a + b;
};
cout << add(10, 20);   // 30

// Capture surrounding variable
int x = 10;
auto print = [x]() { cout << x; };   // capture x by value
auto inc   = [&x]() { x++; };        // capture x by reference
```

**Captures:**
```
[x]   → x by value (copy)
[&x]  → x by reference
[=]   → all visible variables by value
[&]   → all visible variables by reference
```

---

# 24. OOP Concepts

Four pillars:

| Concept | Meaning |
|---|---|
| **Encapsulation** | Bundle data + methods, hide internal details |
| **Inheritance** | Derived class reuses base class |
| **Polymorphism** | Same interface, different behavior |
| **Abstraction** | Expose only what's needed, hide complexity |

---

# 25. Classes and Objects

```cpp
class Student {
public:
    string name;
    int    age;

    void display() {
        cout << name << " " << age;
    }
};

Student s1;          // create object
s1.name = "Tirth";
s1.age  = 25;
s1.display();

// Class  = blueprint
// Object = actual instance created from blueprint
```

---

# 26. Class Methods

**Inside class:**
```cpp
class Car {
public:
    void start() { cout << "Starting"; }
};
```

**Outside class (definition separated):**
```cpp
class Car {
public:
    void start();   // declaration only
};

void Car::start() { // :: is scope resolution operator
    cout << "Starting";
}
```

---

# 27. Constructors

Runs automatically when an object is created. Same name as class, no return type.

```cpp
class Student {
public:
    string name;
    int    age;

    Student() {                        // default constructor
        name = "Unknown";
        age  = 0;
    }

    Student(string n, int a) {         // parameterized constructor
        name = n;
        age  = a;
    }
};

Student s1;                 // calls default constructor
Student s2("Tirth", 25);   // calls parameterized constructor
```

---

# 28. Access Specifiers + Encapsulation

```cpp
class BankAccount {
private:
    double balance;          // hidden — only accessible inside class

public:
    void setBalance(double value) {    // controlled setter
        if (value >= 0)
            balance = value;
    }
    double getBalance() {             // controlled getter
        return balance;
    }
};
```

| Specifier | Same Class | Derived Class | Outside |
|---|---|---|---|
| `public` | Yes | Yes | Yes |
| `protected` | Yes | Yes | No |
| `private` | Yes | No | No |

**Encapsulation:** protect data with private, expose controlled access via public methods.

---

# 29. Friend Functions

Not a class member, but granted access to private/protected members.

```cpp
class Box {
private:
    int width = 10;

public:
    friend void display(Box b);   // declaration grants access
};

void display(Box b) {
    cout << b.width;   // allowed because of friend declaration
}
```

Use sparingly — breaks encapsulation when overused.

---

# 30. Inheritance

Derived class inherits members of the base class.

```cpp
class Animal {
public:
    void eat() { cout << "Eating"; }
};

class Dog : public Animal {    // Dog inherits from Animal
public:
    void bark() { cout << "Barking"; }
};

Dog d;
d.eat();    // inherited from Animal
d.bark();   // Dog's own method
```

**Levels:**
```
Animal → Dog → Puppy   (multilevel)
Animal → Dog            (single)
Animal → Dog, Cat       (hierarchical)
```

---

# 31. Polymorphism

Same interface, different behavior depending on the object.

**Compile-time (function overloading):** resolved at compile time.

**Runtime (virtual functions):** resolved at runtime based on actual object type.

```cpp
class Animal {
public:
    virtual void sound() {         // virtual = can be overridden
        cout << "Animal sound";
    }
};

class Dog : public Animal {
public:
    void sound() override {        // override the base version
        cout << "Bark";
    }
};

Animal *ptr = new Dog();   // base pointer → derived object
ptr->sound();              // calls Dog::sound() → "Bark"
delete ptr;
```

Without `virtual` → would call `Animal::sound()` regardless of actual object.

---

# 32. . vs ->

```cpp
Student s;
s.display();         // object → use  .

Student *ptr = &s;
ptr->display();      // pointer → use ->

// These are equivalent:
ptr->display();
(*ptr).display();
```

**Rule: dot for objects, arrow for pointers.**

---

# 33. Common Interview Traps

```cpp
// = vs ==
x = 5;      // assignment
x == 5;     // comparison (use in if conditions)

// Integer division
5 / 2       // = 2  (use 5.0/2 or (double)5/2 for 2.5)

// Array bounds
int arr[5];
arr[5];     // OUT OF BOUNDS — valid is arr[0] to arr[4]

// Pointer vs value
int *p = &x;
p           // address
*p          // value at address
&p          // address of the pointer itself

// new/delete pairing
new    → delete
new[]  → delete[]    // don't mix these

// Pass by value — original unchanged
void fun(int x) { x = 100; }   // caller's variable unchanged

// Pass by reference — original changes
void fun(int &x) { x = 100; }  // caller's variable changes

// Constructor rules
// Same name as class, no return type, called automatically
// Can be overloaded

// virtual keyword
// Required for runtime polymorphism
// Without it, base version is called even through derived pointer
```

---

# 34. 5-Minute Final Revision

```cpp
// VARIABLE & CONSTANT
int x = 10;
const int MAX = 100;

// INPUT / OUTPUT
cin >> x;
cout << x;
getline(cin, name);     // full line with spaces

// CONTROL FLOW
if (x > 0) { }
for (int i = 0; i < n; i++) { }
while (condition) { }
switch (x) { case 1: break; default: }

// ARRAY
int arr[5] = {1,2,3,4,5};
int size = sizeof(arr)/sizeof(arr[0]);

// STRING
string s = "hello";
s.length();   s[i];   s + "world";

// REFERENCE
int &ref = x;        // alias, modifies original

// POINTER
int *ptr = &x;
*ptr = 20;           // modifies x
ptr = nullptr;       // safe null

// DYNAMIC MEMORY
int *p = new int(10);
delete p;   p = nullptr;

int *arr = new int[5];
delete[] arr;

// FUNCTION
int add(int a, int b) { return a + b; }
void fun(int x)  { }    // pass by value
void fun(int &x) { }    // pass by reference

// RECURSION
int fact(int n) {
    if (n <= 1) return 1;
    return n * fact(n-1);
}

// LAMBDA
auto add = [](int a, int b) { return a + b; };

// CLASS
class Car {
private:
    int speed;
public:
    Car() { speed = 0; }          // constructor
    void setSpeed(int s) { speed = s; }
    int  getSpeed() { return speed; }
    virtual void drive() { cout << "Driving"; }
};

// OBJECT
Car c;
c.setSpeed(60);

// POINTER TO OBJECT
Car *ptr = new Car();
ptr->drive();
delete ptr;

// INHERITANCE
class ElectricCar : public Car {
public:
    void drive() override { cout << "Driving silently"; }
};
```

**Mental Map:**
```
C++
├── Basics:    variables, types, input/output, operators, strings
├── Control:   if/else, switch, for, while, break/continue
├── Data:      arrays, structs, enums
├── Memory:    references, pointers, new/delete, stack/heap
├── Functions: value vs reference, overloading, scope, recursion, lambda
└── OOP:       class, object, constructor, access specifiers,
               encapsulation, friend, inheritance, polymorphism
```

Notes on Constructors and member initializer lists

Constructor is a special non-static member function of a class that is used to initialize
objects of its class type. In the definition of a constructor of a class, _member initializer list_
specifies the initializers for direct and virtual bases and non-static data members. 
(Not to be confused with ```std::initializer_list```).

(_since C++ 20_) A constructor must not be a coroutine


Constructor Syntax

Constructors are declared using member function declarators of the following form:

  _class-name_ **(** _parameter-list_ (_optional_) **)** ```except-spec```(_optional_) attr(_optional_)

where _class-name_ must name the current class (or current instantiation of a class template), or, when
declared at namespace scope or in a friend declaration, it must be a qualified class name.

The only specifiers allowed in the _decl-specifier-seq_ of a constructor declaration are ```friend```,
```inline```, ```constexpr``` (_since C++ 11_), ```consteval``` (_since C++ 20_), and ```explicit```
(in particular, no return type is allowed). Note that _cv-_ and _ref-qualifiers_ are not allowed either;
```const``` and ```volatile``` semantics of an object do not kick in until the most-derived constructor
completes.

The body of function definition of any constructor, before the opening brace of the compound statement, 
may include the _member initializer list_, whose syntax is the colon character ```:```, followed by
the comma-separated list of one or more _member-initializers_ each of which has the following syntax


_class-or-identifier_ **(** _expression-list_ **)** 

initializes the base or member named by _class-or-identifier_ using direct initialization or, if
_expression-list_ is empty, value-initialization.      
# TODO: continue here!

Non-static member functions


A non-static member function is a function that is declared in a member specification of a class without
a static or friend specifier. 

```cpp
class S {
    int mf1(); // non-static member function declaration
    void mf2() volatile, mf3() &&; // can be cv-qualified and reference-qualified
         // the declaration above is equivalent to two separate declarations
         // void mf2() volatile;
         // void mf3() &&;

    int mf4() const { return data; } // can be defined inline
    virtual void mf5() final; // can be virtual, can use final/override
    S() : data(12) {} // consructors are member functions too
    int data;    
};
int S::mf1() { return 7; ] // if not defined inline, has to be defined at namespace
```

Any function declarations are allowed, with additional syntax elements that are only available for
non-static member functions: ```final``` and ```override``` specifiers, pure specifiers, cv-qualifiers,
ref-qualifiers, and member initialization lists. 


A non-static member function of class X may be called:
1) for an object of type X using the class member access operator
2) for an object of a class derived from X
3) directly from within the body of a member function of X
4) directly from within the body of a member function of a class derived from X

Calling a member function of class X on an object of any other type invokes undefined behavior


Within the body of a non-static member function of X, any id-expression E (e.g. an idenitifier) that
resolves to a non-type non-static member of X or of a base class of X, is transformed to a member access
expression ```(*this).E``` (unless it is already a part of a member access expression). This does not 
occur in template definition context, so a name may have to be prefixed with ```this->``` explicitly
to become dependent.

```cpp
struct S {
   int n;
   void f();
};
void S::f() {
   n = 1; // transformed to (*this).n = 1
}
int main() {
   S s1, s2;
   s1.f(); // changes s1.n
}
```


```const```- and ```volatile```-qualified member functions


A non-static member function can be declared with a ```const```, ```volatile```, or ```const volatile```
qualifier (this qualifier appears after the parameter list in the function declaration). Differently
```cv```-qualfied functions have different types and so may overload each other.


In the body of ```cv```-qualified function, ```*this``` is ```cv```-qualified, e.g. in a ```const``` 
member function, only other ```const``` member functions may be called normally. (A non-```const```
member function may still be called if ```const_cast``` is applied or through an access path that
does not involve ```this```).

```cpp
#include <vector>
struct Array {
    std::vector<int> data;
    Array(int sz) : data(sz) {}
    // const member function
    int operator[](int idx) const {
                            // this has type const Array*
         return data[idx];  // transformed to (*this).data[idx];
    }
    // non-const member function
    int& operator[](int idx) {
                            // this has type Array*
         return data[idx];  // transformed to (*this).data[idx];
    }
};

int main()
{
   Array a(10);
   a[1] = 1;  // OK: the type of a[1] is int&
   const Array ca(10);
   ca[1] = 2; // Error: the type of ca[1] is int
}
```


```ref```-qualified member functions

A non-```static``` member function can be declared with no ref-qualifier, with an lvalue
ref-qualifier (the token & after the parameter list) or the rvalue ref-qualifier (the token &&
after the parameter list). During overload resolution, non-static cv-qualified member function of 
class X is treated as follows:

* no ```ref```-qualifier: the implicit object parameter has type lvalue reference to cv-qualified X
and is additionally allowed to bind rvalue implied object argument
* lvalue ```ref```-qualifier: the implicit object parameter has type lvalue reference to 
  ```cv```-qualified X
* rvalue ref-qualifier: the implicit object parameter has type lvalue reference to cv-qualified X


```cpp
#include <iostream>

struct S {
    void f() &   { std::cout << "lvalue\n"; }
    void f() &&  { std::cout << "rvalue\n"; }
};

int main() {
    S s;
    s.f();              // prints "lvalue"
    std::move(s).f();   // prints "rvalue"
    S().f();            // prints "rvalue"
``` 

_Note_: unlike ```cv```-qualification, ```ref```-qualification does not change the properties of
the ```this``` pointer: within a rvalue ref-qualified function, ```*this``` remains an lvalue 
expression. 


Virtual and pure virtual functions

A non-```static``` member function may be declared _virtual_ or _pure virtual_.


Special member functions


constructors and destructors are non-static member functions that use a special syntax for their
declarations. Some member functions are _special_: under certain circumstances they are defined 
by the compiler even if not defined by the user. They are:
* default constructor
* copy constructor
* move constructor
* copy assignment operator
* destructor

Special member functions along with comparison operators (_since C++ 20_) are the only functions
that can be defaulted, that is, defined using ```= default``` instead of the function body.

Example

```cpp
#include <iostream>
#include <string>
#include <utility>
#include <exception>

struct S {
    int data;

    // simple converting constructor (declaration)
    S(int val);

    // simple explicit constructor (declaration)
    explicit S(std::string str);

    // const member function (definition)
    virtual int getData() const { return data; }
}; 

// definition of the constructor
S::S(int val) : data(val) {
    std::cout << "ctor1 called, data = " << data << '\n';
}

// this constructor has a catch clause
S::S(std::string str) try : data(std::stoi(str)) {
    std::cout << "ctor2 called, data = " << data << '\n';
} catch(const std::exception&) {
    std::cout << "ctor2 failed, string was '" << str << "'\n";
    throw;  // ctor's catch clause should always rethrow
}

struct D : S {
    int data2;
    // constructor with a default argument
    D(int v1, int v2 = 11) : S(v2), data2(v2) {}

    // virtual member function
    int getData() const override { return data*data2; }

    // lvalue-only assignment operator
    D& operator=(D other) & {
        std::swap(other.data, data);
        std::swap(other.data2, data2);
        return *this;
    }
};

int main() 
{
    D d1 = 1;
    S s2("2");
    try {

    } catch (const std::exception&) {}
    std::cout << s2.getData() << '\n';

    D d2 (3,4);
    d2 = d1;
    // D(5) = d1; // ERROR: no suitable overload for operator=
}
```

Output:

```
ctor1 called, data = 1
ctor2 called, data = 2
ctor2 failed, string was 'not a number'
2
ctor1 called, data = 3
```


Direct initialization
initializes an object from explicit set of constructor arguments


Direct initialization Syntax:


initialization with nonempty parenthesized list of expressions or braced-init-lists (_since C++11_):

_T object_ **(** _arg_ **);**

_T object_ **(** _arg1_, _arg2_, ... **);**


initialization of an object of non-class type with a single brace-enclosed initializer (note: for class
types and other uses of braced-init-list see the list-initialization section below):

_T object_ **{** _arg_ **};**


initialization of a _prvalue_ temporary (_until C++17_), the result object of a _prvalue_ (_since C++17_)
by functional cast or with a parenthesized expression list

_T_ **(** _other_ **)**

_T_ **(** _arg1_, _arg2_, ... **)**


initialization of a _prvalue_ temporary (_until C++17_), the result object of a prvalue (_since C++17_)
by a ```static_cast``` expression

**static_cast<** _T_ **>(** _other_ **)**





List initialization
# TODO: finish this

Value initialization
# TODO: finish this

